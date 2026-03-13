# 🦿 API Documentation

> **Smart Prosthetics — AI-Adaptive Myoelectric Limb Platform** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---


| POST | /api/v1/patients/{patient_id}/calibration |

| Description | Start a new EMG calibration session — patient performs gesture sequence |
Request:
{  "session_type": "initial",  "gesture_sequence": ["grip", "release", "flex", "extend", "pinch"],  "repetitions_per_gesture": 8,  "device_id": "dev_PAT_0042"}
Response:
{  "session_id": "cal_8821",  "status": "in_progress",  "estimated_duration_min": 12,  "streaming_endpoint": "wss://api.novus.iq/ws/calibration/cal_8821"}

| POST | /api/v1/calibration/{session_id}/complete |

| Description | Finalize calibration — triggers model training on session data |
Request:
{  "session_id": "cal_8821",  "quality_check_passed": true}
Response:
{  "model_id": "mdl_PAT0042_v1",  "gesture_accuracy": {    "grip": 0.97, "release": 0.96, "flex": 0.93,    "extend": 0.95, "pinch": 0.89  },  "ota_push_scheduled": "2025-04-15T23:00:00Z"}

| GET | /api/v1/patients/{patient_id}/telemetry |

| Description | Retrieve daily usage and performance telemetry |
Request:
GET /api/v1/patients/PAT_0042/telemetry?date=2025-04-15
Response:
{  "patient_id": "PAT_0042",  "date": "2025-04-15",  "wear_time_hours": 9.4,  "total_gestures": 1847,  "error_rate_pct": 4.2,  "avg_response_ms": 118,  "grip_force_avg_n": 18.4,  "haptic_feedback_events": 342,  "battery_cycles_today": 1}

| POST | /api/v1/devices/{device_id}/ota |

| Description | Push an OTA firmware or model update to a device |
Request:
{  "update_type": "ai_model",  "model_version": "mdl_PAT0042_v2",  "schedule": "tonight"}
Response:
{  "ota_job_id": "ota_9921",  "device_id": "dev_PAT_0042",  "scheduled_at": "2025-04-15T23:00:00Z",  "package_size_kb": 284}

---

*[← Back to Smart Prosthetics](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
