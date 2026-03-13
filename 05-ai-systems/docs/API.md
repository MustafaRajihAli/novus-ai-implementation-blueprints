# 🧠 API Documentation

> **AI Systems — Intelligent Core Platform for Autonomous Decision-Making** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---


| POST | /api/v1/ai/reason |

| Description | Submit a reasoning task to the LLM engine with optional tool use and RAG context |
Request:
{  "task_type": "classify_and_draft",  "input": "Customer email: 'My shipment SHP-4421 hasn't arrived in 3 weeks...'",  "tools": ["lookup_shipment", "check_delivery_status"],  "knowledge_base_id": "kb_logistics_01",  "structured_output_schema": {    "category": "string",    "priority": "low|medium|high|critical",    "draft_response": "string",    "escalate": "boolean"  }}
Response:
{  "task_id": "task_7721",  "result": {    "category": "delayed_shipment",    "priority": "high",    "draft_response": "We apologize for the delay with SHP-4421...",    "escalate": false  },  "confidence": 0.92,  "tokens_used": 847,  "latency_ms": 1240,  "tool_calls": [    { "tool": "lookup_shipment", "result": "Last scan: Dubai customs, Apr 12" }  ]}

| POST | /api/v1/ai/vision/extract |

| Description | Extract structured data from a document image using OCR + LLM |
Request:
{  "document_url": "s3://novus-docs/forms/app_88421.jpg",  "extraction_schema": {    "applicant_name": "string",    "national_id": "string",    "date_of_birth": "date",    "address": "string",    "requested_service": "string"  },  "language": "ar"}
Response:
{  "document_id": "doc_88421",  "extracted": {    "applicant_name": "Ahmed Al-Rashidi",    "national_id": "1982-BGH-4421",    "date_of_birth": "1982-03-14",    "address": "Baghdad, Mansour District, Block 4",    "requested_service": "Building Permit"  },  "confidence_scores": {    "applicant_name": 0.98, "national_id": 0.94,    "date_of_birth": 0.97, "address": 0.89, "requested_service": 0.96  },  "needs_human_review": false}

| POST | /api/v1/ai/anomaly/detect |

| Description | Run anomaly detection on a time-series data array |
Request:
{  "series_id": "daily_shipment_volume_port_umm_qasr",  "data_points": [    { "ts": "2025-04-01", "value": 1240 },    { "ts": "2025-04-02", "value": 1190 }  ],  "sensitivity": "medium",  "forecast_horizon_days": 7}
Response:
{  "anomalies": [    { "ts": "2025-04-02", "value": 1190, "expected": 1235,      "score": 0.81, "type": "low_volume", "severity": "warning" }  ],  "forecast": [    { "ts": "2025-04-08", "predicted": 1220, "lower": 1140, "upper": 1310 }  ],  "recommendation": "Volume 3.6% below seasonal baseline. Check port access logs."}

| POST | /api/v1/kb/ingest |

| Description | Ingest a document into the RAG knowledge base |
Request:
{  "kb_id": "kb_logistics_01",  "document": {    "title": "SLA Policy v2.1",    "content": "...",    "metadata": { "department": "operations", "effective_date": "2025-01-01" }  }}
Response:
{  "document_id": "doc_kb_7821",  "chunks_created": 14,  "embedding_model": "text-embedding-3-small",  "indexed_at": "2025-04-15T09:21:00Z"}

---

*[← Back to AI Systems](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
