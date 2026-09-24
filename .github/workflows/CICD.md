# CI/CD Pipeline — Construction Technect

## Pipeline Flow
Developer → GitHub → Code Validation → Testing → Build → Docker Image → ECR → Deploy to ECS → Production → Monitoring

## Tool Selected: GitHub Actions
**Why:** The code already lives on GitHub, so there's zero extra setup — no separate server to host (unlike Jenkins), free for public/small private repos, and has ready-made actions for AWS (ECR login, ECS deploy) so the pipeline stays simple to write and maintain.

## Step-by-Step Explanation

### 1. Developer pushes code
A push to `main` (or a pull request) automatically triggers the GitHub Actions workflow — no manual step needed.

### 2. Code Validation
The pipeline runs a linter (ESLint) first, before anything else, because it's the fastest check — catching style/syntax errors here saves time instead of waiting for a full build to fail later.

### 3. Automated Testing
Unit and integration tests run via `npm run test`. If any test fails, the pipeline stops immediately — the code never proceeds to build or deploy.

### 4. Build
The application is compiled/built (`npm run build`) to confirm it produces a valid production build before it's even containerized.

### 5. Docker Image
A `Dockerfile` in the repo defines how to package the Nest.js backend into a container image. The pipeline builds this image and tags it with the Git commit SHA, so every image is traceable to the exact code that produced it.

### 6. Docker Image Storage
The image is pushed to **Amazon ECR** (Elastic Container Registry) — AWS's private, secure registry that integrates directly with ECS.

### 7. Deployment
The pipeline tells **ECS** to pull the new image and roll out a new deployment (`--force-new-deployment`), replacing old containers gradually while keeping the app available.

### 8. Production
Once ECS finishes rolling out, the new version is live and serving real user traffic.

### 9. Monitoring
**CloudWatch** watches the new deployment's logs, CPU/memory, and error rates immediately after rollout, so problems are caught within minutes.

## Handling Deployment Failures
- If **any earlier stage fails** (lint, test, build) — the pipeline stops there. The broken code never even reaches the Docker/deploy stage, so production is never touched.
- If **the ECS deployment itself fails** (e.g. new containers crash on startup) — ECS's built-in deployment circuit breaker detects unhealthy tasks and automatically stops the rollout, keeping the last healthy version running.

## Rollback Approach
1. Every Docker image is tagged with its Git commit SHA, so past versions are never overwritten — they stay in ECR.
2. To roll back, redeploy the previous known-good image tag:
```bash
aws ecs update-service \
  --cluster construction-technect-cluster \
  --service backend-service \
  --task-definition backend-task:<previous-revision-number> \
  --force-new-deployment
