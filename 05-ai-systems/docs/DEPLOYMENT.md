# 🧠 Deployment Guide

> **AI Systems — Intelligent Core Platform for Autonomous Decision-Making** — [← Back to Service](../README.md) | [← All Services](../../README.md)

---

Docker Local Development
docker compose up -d            # Start all services
docker compose logs -f api      # Watch API logs
docker compose exec api bash    # Shell into API container
docker compose down -v          # Tear down (removes volumes)
AWS ECS Fargate Production Deployment
# 1. Push image to ECRaws ecr get-login-password --region me-south-1 | \  docker login --username AWS --password-stdin \  <account>.dkr.ecr.me-south-1.amazonaws.comdocker build -t novus-service .docker tag novus-service:latest <account>.dkr.ecr.me-south-1.amazonaws.com/novus-service:latestdocker push <account>.dkr.ecr.me-south-1.amazonaws.com/novus-service:latest# 2. Deploy via Terraformcd terraform/terraform initterraform plan -var-file=prod.tfvarsterraform apply
GitHub Actions CI/CD Pipeline
# .github/workflows/deploy.ymlname: Deploy to Productionon:  push:    branches: [main]jobs:  test-and-deploy:    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4      - name: Run tests        run: docker compose -f docker-compose.test.yml run --rm test      - name: Build and push image        run: |          docker build -t $IMAGE_URI .          docker push $IMAGE_URI      - name: Deploy to ECS        run: aws ecs update-service --force-new-deployment \             --cluster novus-prod --service novus-api

---

*[← Back to AI Systems](../README.md) | [novusaidynamics.com](https://novusaidynamics.com)*
