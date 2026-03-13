# 🌾 System Architecture

> **Smart Agriculture — AI Drone Precision Spraying System** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---

Architecture Flow:
User (Farm Manager) → React DashboardReact Dashboard → API Gateway (AWS API GW / nginx)API Gateway → FastAPI BackendFastAPI Backend → PostgreSQL + PostGIS (field/mission data)FastAPI Backend → Redis (real-time telemetry)FastAPI Backend → AI Inference Service (pest detection model)AI Inference Service → S3 (raw multispectral images)AI Inference Service → Treatment Map GeneratorTreatment Map Generator → Drone Mission Controller (DJI Cloud API)Drone Mission Controller → Physical Drone FleetPhysical Drone Fleet → Telemetry Stream → Redis → Dashboard

---

*[← Back to Smart Agriculture](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
