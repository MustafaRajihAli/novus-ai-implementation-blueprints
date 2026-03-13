# 🦾 API Documentation

> **Advanced Robotics — Autonomous Quality Inspection & Cobot Assembly System** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---


| POST | /api/v1/inspection/classify |

| Description | Submit a captured board image for AI defect classification |
Request:
{  "unit_serial": "PCB-2025-88421",  "production_line": "line_2",  "image_base64": "...",  "point_cloud_url": "s3://novus-robotics/scans/PCB-2025-88421.pcd"}
Response:
{  "unit_serial": "PCB-2025-88421",  "classification": "rework",  "confidence": 0.94,  "defects": [    { "type": "solder_bridge", "location": "J14-J15", "severity": "medium",      "bbox_3d": [12.4, 8.2, 0.1, 13.1, 8.9, 0.3] }  ],  "routing": "rework_station_2",  "elapsed_ms": 42}

| POST | /api/v1/batches/{batch_id}/approve |

| Description | Human supervisor approves a completed batch for shipment |
Request:
{  "supervisor_id": "emp_0042",  "override_flagged_count": 0,  "notes": "All rework verified by line lead"}
Response:
{  "batch_id": "batch_20250415_L2",  "approved_units": 2840,  "rejected_units": 12,  "reworked_units": 38,  "approval_timestamp": "2025-04-15T14:22:10Z",  "certificate_url": "https://s3.../certs/batch_20250415_L2.pdf"}

| GET | /api/v1/metrics/line/{line_id} |

| Description | Pull live production metrics for a line |
Request:
GET /api/v1/metrics/line/line_2?window=1h
Response:
{  "line_id": "line_2",  "window": "1h",  "throughput_units_per_hour": 1087,  "defect_escape_rate_pct": 0.3,  "rework_rate_pct": 1.8,  "ai_confidence_avg": 0.971,  "human_overrides": 2}

---

*[← Back to Advanced Robotics](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
