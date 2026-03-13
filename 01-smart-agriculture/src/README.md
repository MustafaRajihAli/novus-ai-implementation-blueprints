# 🌾 Code Implementation

> **Smart Agriculture — AI Drone Precision Spraying System** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---

Pest Detection Service

```json
# src/services/pest_detection.py"""Runs YOLOv8 segmentation on a multispectral GeoTIFF and returnsa list of infected zones with bounding boxes and severity scores."""import torchimport numpy as npimport rasteriofrom ultralytics import YOLOfrom src.models.treatment_map import Zone, TreatmentMapMODEL_PATH = "models/pest_detector_v3.pt"CONFIDENCE_THRESHOLD = 0.72class PestDetectionService:    def __init__(self):        self.model = YOLO(MODEL_PATH)        self.model.to("cuda" if torch.cuda.is_available() else "cpu")    def analyze_field_image(self, geotiff_path: str, field_id: str) -> TreatmentMap:        """        Load multispectral GeoTIFF, compute NDVI/NDRE indices,        run inference, return structured treatment map.        """        with rasterio.open(geotiff_path) as src:            # Bands: 0=Blue, 1=Green, 2=Red, 3=RedEdge, 4=NIR            red = src.read(3).astype(float)            nir = src.read(5).astype(float)            red_edge = src.read(4).astype(float)            transform = src.transform            crs = src.crs        # NDVI: normalized difference vegetation index        ndvi = (nir - red) / (nir + red + 1e-8)        # NDRE: sensitive to early-stage chlorophyll loss        ndre = (nir - red_edge) / (nir + red_edge + 1e-8)        # Stack into 3-channel input expected by the model        rgb_composite = np.stack([            self._normalize(ndvi),            self._normalize(ndre),            self._normalize(red)        ], axis=-1).astype(np.uint8)        results = self.model(rgb_composite, conf=CONFIDENCE_THRESHOLD)        zones = []        for box in results[0].boxes:            cls_name = self.model.names[int(box.cls)]            bbox_geo = self._pixel_to_geo(box.xyxy[0].tolist(), transform)            zones.append(Zone(                pest_type=cls_name,                confidence=float(box.conf),                bbox_geo=bbox_geo,                recommended_dosage_ml_ha=self._dosage_lookup(cls_name, float(box.conf))            ))        return TreatmentMap(field_id=field_id, zones=zones, crs=str(crs))    def _normalize(self, arr: np.ndarray) -> np.ndarray:        arr_min, arr_max = arr.min(), arr.max()        return ((arr - arr_min) / (arr_max - arr_min + 1e-8) * 255).astype(np.uint8)    def _pixel_to_geo(self, xyxy: list, transform) -> list:        """Convert pixel bounding box to geographic coordinates."""        x1, y1, x2, y2 = xyxy        lon1, lat1 = transform * (x1, y1)        lon2, lat2 = transform * (x2, y2)        return [lon1, lat1, lon2, lat2]    def _dosage_lookup(self, pest_type: str, confidence: float) -> float:        base = {"fungal_blight": 220, "aphid_colony": 180, "water_stress": 0}        return base.get(pest_type, 200) * min(confidence + 0.2, 1.0)
```

Mission A P I

```json
# src/api/missions.pyfrom fastapi import APIRouter, HTTPException, BackgroundTasksfrom src.services.pest_detection import PestDetectionServicefrom src.services.mission_planner import MissionPlannerServicefrom src.models.mission import SurveyRequest, SprayRequestfrom src.db import get_dbimport uuidrouter = APIRouter(prefix="/api/v1/missions", tags=["missions"])detector = PestDetectionService()planner = MissionPlannerService()@router.post("/survey")async def schedule_survey(req: SurveyRequest, bg: BackgroundTasks):    """Schedule an aerial survey mission for a field."""    mission_id = f"msn_{uuid.uuid4().hex[:8]}"    waypoints = planner.generate_grid_waypoints(        field_id=req.field_id,        altitude_m=req.altitude_m,        overlap_pct=req.overlap_pct    )    # Dispatch to drone controller asynchronously    bg.add_task(planner.dispatch_survey, mission_id, req.drone_id, waypoints)    return {        "mission_id": mission_id,        "status": "scheduled",        "waypoints_count": len(waypoints),        "scheduled_at": req.scheduled_at    }@router.get("/{mission_id}/treatment-map")async def get_treatment_map(mission_id: str, db=get_db()):    """Return the AI-generated treatment map for a completed survey."""    mission = db.query_mission(mission_id)    if not mission or mission.status != "completed":        raise HTTPException(404, "Mission not found or not yet completed")    return mission.treatment_map@router.post("/spray")async def dispatch_spray(req: SprayRequest, bg: BackgroundTasks):    """Dispatch a precision spray mission from a treatment map."""    spray_id = f"spr_{uuid.uuid4().hex[:8]}"    bg.add_task(planner.dispatch_spray_mission, spray_id, req)    return {"spray_mission_id": spray_id, "status": "dispatched"}
```

Dockerfile

```dockerfile
FROM python:3.11-slimWORKDIR /app# Install GDAL/GEOS for rasterio geospatial processingRUN apt-get update && apt-get install -y \    gdal-bin libgdal-dev gcc g++ \    && rm -rf /var/lib/apt/lists/*COPY requirements.txt .RUN pip install --no-cache-dir -r requirements.txtCOPY . .EXPOSE 8000CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

docker-compose.yml

```yaml
version: "3.9"services:  api:    build: .    ports: ["8000:8000"]    env_file: .env    depends_on: [postgres, redis]    volumes:      - ./models:/app/models  postgres:    image: postgis/postgis:15-3.3    environment:      POSTGRES_DB: ${DB_NAME}      POSTGRES_USER: ${DB_USER}      POSTGRES_PASSWORD: ${DB_PASSWORD}    volumes:      - pg_data:/var/lib/postgresql/data  redis:    image: redis:7-alpine    command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru  worker:    build: .    command: celery -A src.worker worker --loglevel=info --concurrency=4    env_file: .env    depends_on: [redis, postgres]  grafana:    image: grafana/grafana:latest    ports: ["3000:3000"]    volumes:      - grafana_data:/var/lib/grafanavolumes:  pg_data:  grafana_data:
```

.env.example
# DatabaseDB_HOST=postgresDB_PORT=5432DB_NAME=agri_platformDB_USER=agri_adminDB_PASSWORD=change_in_production# RedisREDIS_URL=redis://redis:6379/0# AWSAWS_ACCESS_KEY_ID=AWS_SECRET_ACCESS_KEY=AWS_REGION=me-south-1S3_BUCKET_IMAGES=novus-agri-imagesS3_BUCKET_MAPS=novus-agri-treatment-maps# AI ModelMODEL_PATH=models/pest_detector_v3.ptINFERENCE_DEVICE=cuda# Drone IntegrationDJI_APP_ID=DJI_APP_KEY=DJI_CLOUD_API_URL=https://openplatform.dji.com/api/v1# WeatherOPENWEATHER_API_KEY=# AlertsSMTP_HOST=smtp.sendgrid.netSMTP_API_KEY=

---

*[← Back to Smart Agriculture](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
