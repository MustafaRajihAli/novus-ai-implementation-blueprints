# 🛡️ API Documentation

> **Defense AI — Autonomous Surveillance & Predictive Threat Analytics Platform** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---


| POST | /api/v1/threats/ingest |

| Description | Ingest a multi-source threat event from the fusion layer |
Request:
{  "source": "radar_array_07",  "event_type": "vehicle_movement",  "location": { "lat": 34.221, "lon": 42.887 },  "timestamp": "2025-04-15T02:14:33Z",  "confidence": 0.91,  "raw_data_ref": "s3://internal/radar/evt_20250415_021433.bin"}
Response:
{  "threat_id": "thr_8821",  "classification": "elevated",  "fused_sources": ["radar_array_07", "seismic_zone_3"],  "predicted_trajectory": [    { "lat": 34.229, "lon": 42.881, "eta_min": 18 },    { "lat": 34.241, "lon": 42.871, "eta_min": 31 }  ],  "recommended_actions": [    { "action": "redirect_drone", "asset": "uav_charlie", "waypoint": [34.229, 42.881] },    { "action": "alert_patrol", "unit": "patrol_07", "priority": "high" }  ],  "requires_authorization": true}

| POST | /api/v1/threats/{threat_id}/authorize |

| Description | Commander authorizes a recommended response action |
Request:
{  "commander_id": "cmd_0012",  "action_authorized": "redirect_drone",  "asset": "uav_charlie",  "notes": "Confirmed via radar correlation"}
Response:
{  "threat_id": "thr_8821",  "action": "redirect_drone",  "status": "executing",  "dispatch_timestamp": "2025-04-15T02:15:01Z",  "estimated_on_target_min": 8}

| GET | /api/v1/dashboard/live |

| Description | Stream live threat map state for commander dashboard |
Request:
GET /api/v1/dashboard/live  (SSE stream)
Response:
{  "active_threats": 3,  "monitoring": 12,  "elevated": 2,  "critical": 1,  "drone_status": {    "uav_alpha": "patrolling", "uav_bravo": "returning", "uav_charlie": "redirected"  },  "sector_coverage_pct": 94,  "last_update": "2025-04-15T02:15:44Z"}

| POST | /api/v1/cyber/anomaly |

| Description | Report a detected network anomaly for automated countermeasure |
Request:
{  "network_segment": "comms_sector_04",  "anomaly_type": "rf_jamming",  "frequency_mhz": 433.5,  "signal_strength_dbm": -42,  "detected_at": "2025-04-15T02:18:00Z"}
Response:
{  "countermeasure": "frequency_hop",  "new_frequency_mhz": 869.4,  "affected_units": ["patrol_07", "patrol_08", "checkpoint_14"],  "status": "applied",  "elapsed_ms": 340}

---

*[← Back to Defense AI](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
