# 🧠 Repository Structure

> **AI Systems — Intelligent Core Platform for Autonomous Decision-Making** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---

Repository Structure:
/ai-systems-service/  README.md  architecture.md  deployment_guide.md  api_documentation.md  implementation_plan.md  requirements.txt  Dockerfile  docker-compose.yml  .env.example  /src/    /api/            # FastAPI route handlers    /services/       # Business logic and AI orchestration    /models/         # SQLAlchemy + Pydantic data models    /utils/          # Shared helpers (logging, auth, cache)  /config/           # Environment-specific config files  /tests/            # pytest unit + integration tests    /unit/    /integration/    /e2e/  /scripts/          # Database seeds, migration helpers  /docker/           # Multi-stage Dockerfiles, compose variants  /docs/             # OpenAPI export, architecture diagrams

---

*[← Back to AI Systems](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
