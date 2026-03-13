# 🦾 Code Implementation

> **Advanced Robotics — Autonomous Quality Inspection & Cobot Assembly System** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---

Defect Classifier

```json
# src/services/defect_classifier.py"""Runs YOLOv8 on a unit image and optionally fuses with 3D pointcloud data for depth-aware defect localization."""import cv2import numpy as npimport open3d as o3dfrom ultralytics import YOLOfrom src.models.inspection import DefectResultMODEL_PATH = "models/pcb_defect_v4.pt"CONFIDENCE_THRESHOLD = 0.80HUMAN_REVIEW_THRESHOLD = 0.92  # Below this → flag for human checkclass DefectClassifier:    def __init__(self):        self.model = YOLO(MODEL_PATH)    def classify(self, image_path: str, pcd_path: str | None = None) -> DefectResult:        """        Classify defects in a PCB image.        If a point cloud is provided, adds 3D bounding box coordinates.        """        img = cv2.imread(image_path)        results = self.model(img, conf=CONFIDENCE_THRESHOLD)[0]        defects = []        for box in results.boxes:            conf = float(box.conf)            defect = {                "type": self.model.names[int(box.cls)],                "confidence": conf,                "bbox_2d": box.xyxy[0].tolist(),                "needs_human_review": conf < HUMAN_REVIEW_THRESHOLD,            }            if pcd_path:                defect["bbox_3d"] = self._localize_3d(pcd_path, defect["bbox_2d"])            defects.append(defect)        # Routing decision        if not defects:            routing = "pass"        elif all(d["type"] in ("solder_bridge", "missing_component") for d in defects):            routing = "rework"        else:            routing = "reject"        return DefectResult(defects=defects, routing=routing)    def _localize_3d(self, pcd_path: str, bbox_2d: list) -> list:        """        Extract 3D bounding box from point cloud using 2D ROI as seed region.        Simplified — production version uses ICP registration.        """        pcd = o3d.io.read_point_cloud(pcd_path)        pts = np.asarray(pcd.points)        x1, y1, x2, y2 = [int(v) for v in bbox_2d]        # Filter points within 2D projected region (simplified projection)        mask = (pts[:, 0] >= x1) & (pts[:, 0] <= x2) & (pts[:, 1] >= y1) & (pts[:, 1] <= y2)        roi_pts = pts[mask]        if len(roi_pts) == 0:            return []        return [            float(roi_pts[:, 0].min()), float(roi_pts[:, 1].min()), float(roi_pts[:, 2].min()),            float(roi_pts[:, 0].max()), float(roi_pts[:, 1].max()), float(roi_pts[:, 2].max()),        ]
```

Ros2 Node

```cpp
// src/ros2/inspection_node.cpp// ROS2 node that triggers image capture, calls the Python// classifier via gRPC, and publishes routing decisions to// the conveyor gate actuator topic.#include "rclcpp/rclcpp.hpp"#include "std_msgs/msg/string.hpp"#include "sensor_msgs/msg/image.hpp"#include "novus_msgs/msg/defect_result.hpp"#include <grpcpp/grpcpp.h>#include "classifier.grpc.pb.h"class InspectionNode : public rclcpp::Node {public:  InspectionNode() : Node("inspection_node") {    image_sub_ = this->create_subscription<sensor_msgs::msg::Image>(      "/camera/image_raw", 10,      std::bind(&InspectionNode::on_image, this, std::placeholders::_1));    gate_pub_ = this->create_publisher<std_msgs::msg::String>(      "/conveyor/gate_command", 10);    // gRPC channel to Python AI service    auto channel = grpc::CreateChannel(      "localhost:50051", grpc::InsecureChannelCredentials());    stub_ = Classifier::NewStub(channel);    RCLCPP_INFO(this->get_logger(), "Inspection node ready");  }private:  void on_image(const sensor_msgs::msg::Image::SharedPtr msg) {    // Call Python classifier via gRPC    ClassifyRequest req;    req.set_image_data(msg->data.data(), msg->data.size());    ClassifyResponse resp;    grpc::ClientContext ctx;    stub_->Classify(&ctx, req, &resp);    // Publish gate command    std_msgs::msg::String gate_cmd;    gate_cmd.data = resp.routing();  // "pass" | "rework" | "reject"    gate_pub_->publish(gate_cmd);    RCLCPP_INFO(this->get_logger(),      "Unit classified: %s (conf: %.3f)", resp.routing().c_str(), resp.confidence());  }  rclcpp::Subscription<sensor_msgs::msg::Image>::SharedPtr image_sub_;  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr gate_pub_;  std::unique_ptr<Classifier::Stub> stub_;};int main(int argc, char** argv) {  rclcpp::init(argc, argv);  rclcpp::spin(std::make_shared<InspectionNode>());  rclcpp::shutdown();}
```

Dockerfile

```dockerfile
FROM python:3.11-slim AS apiWORKDIR /appRUN apt-get update && apt-get install -y \    libgl1 libglib2.0-0 libgomp1 \    && rm -rf /var/lib/apt/lists/*COPY requirements.txt .RUN pip install --no-cache-dir -r requirements.txtCOPY . .EXPOSE 8000 50051CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]# --- Edge inference image (runs on Jetson) ---FROM nvcr.io/nvidia/l4t-pytorch:r35.2.1-pth2.0-py3 AS edgeWORKDIR /appCOPY requirements.edge.txt .RUN pip install --no-cache-dir -r requirements.edge.txtCOPY src/inference ./inferenceCMD ["python", "inference/edge_classifier.py"]
```

docker-compose.yml

```yaml
version: "3.9"services:  api:    build:      context: .      target: api    ports: ["8000:8000"]    env_file: .env    depends_on: [postgres, redis, influxdb]  postgres:    image: postgres:15-alpine    environment:      POSTGRES_DB: ${DB_NAME}      POSTGRES_USER: ${DB_USER}      POSTGRES_PASSWORD: ${DB_PASSWORD}    volumes: [pg_data:/var/lib/postgresql/data]  influxdb:    image: influxdb:2.7    ports: ["8086:8086"]    environment:      DOCKER_INFLUXDB_INIT_MODE: setup      DOCKER_INFLUXDB_INIT_ORG: novus      DOCKER_INFLUXDB_INIT_BUCKET: factory_metrics      DOCKER_INFLUXDB_INIT_ADMIN_TOKEN: ${INFLUX_TOKEN}    volumes: [influx_data:/var/lib/influxdb2]  grafana:    image: grafana/grafana:latest    ports: ["3000:3000"]    volumes:      - grafana_data:/var/lib/grafana      - ./config/grafana/dashboards:/etc/grafana/provisioning/dashboards  redis:    image: redis:7-alpinevolumes:  pg_data:  influx_data:  grafana_data:
```

.env.example
DB_NAME=robotics_platformDB_USER=novus_adminDB_PASSWORD=change_in_productionREDIS_URL=redis://redis:6379/0INFLUX_TOKEN=your_influxdb_token_hereINFLUX_URL=http://influxdb:8086GRPC_PORT=50051MODEL_PATH=models/pcb_defect_v4.ptINFERENCE_DEVICE=cudaMES_API_URL=https://mes.factory.internal/apiMES_API_KEY=SLACK_WEBHOOK_URL=

---

*[← Back to Advanced Robotics](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
