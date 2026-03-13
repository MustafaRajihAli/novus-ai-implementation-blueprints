# 🌾 API Documentation

> **Smart Agriculture — AI Drone Precision Spraying System** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---


| POST | /api/v1/missions/survey |

| Description | Schedule a new aerial survey mission |
Request:
{  "field_id": "fld_2847",  "drone_id": "drn_001",  "altitude_m": 45,  "overlap_pct": 80,  "scheduled_at": "2025-04-15T06:00:00Z"}
Response:
{  "mission_id": "msn_7291",  "status": "scheduled",  "estimated_duration_min": 42,  "waypoints_count": 312}

| GET | /api/v1/missions/{mission_id}/treatment-map |

| Description | Retrieve AI-generated pest treatment map for a completed survey |
Request:
GET /api/v1/missions/msn_7291/treatment-map
Response:
{  "mission_id": "msn_7291",  "field_id": "fld_2847",  "total_hectares": 4000,  "infected_hectares": 340,  "infection_pct": 8.5,  "zones": [    { "zone_id": "z_01", "type": "fungal_blight", "severity": "high",      "bbox": [43.12, 36.04, 43.18, 36.09], "recommended_dosage_ml_ha": 220 }  ],  "geojson_url": "https://s3.../maps/msn_7291.geojson"}

| POST | /api/v1/missions/spray |

| Description | Dispatch precision spray mission based on a treatment map |
Request:
{  "treatment_map_id": "map_7291",  "drone_id": "drn_002",  "chemical_id": "chem_tebuconazole",  "override_zones": []}
Response:
{  "spray_mission_id": "spr_4421",  "status": "dispatched",  "target_zones": 8,  "estimated_chemical_liters": 74.8,  "estimated_duration_min": 18}

| GET | /api/v1/fields/{field_id}/history |

| Description | Pull full mission and treatment history for a field |
Request:
GET /api/v1/fields/fld_2847/history?limit=10
Response:
{  "field_id": "fld_2847",  "missions": [    { "date": "2025-04-15", "type": "survey", "infection_pct": 8.5, "chemical_used_liters": 74.8 },    { "date": "2025-04-01", "type": "survey", "infection_pct": 2.1, "chemical_used_liters": 18.2 }  ],  "season_total_chemical_liters": 93.0,  "season_baseline_liters_if_blanket": 1100}

---

*[← Back to Smart Agriculture](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
