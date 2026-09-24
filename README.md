## Assignment Documents
- [AWS Architecture Design](./aws%20architecture)
- [Database Troubleshooting Guide](./troubleshooting)
- [CI/CD Pipeline Design](./git-workflow)

# Construction Technect — CI/CD Pipeline

This repository contains a working CI/CD (Continuous Integration / Continuous Deployment) pipeline built for Construction Technect's SaaS application, using **GitHub Actions**.

## Pipeline Flow

Developer → GitHub → Code Validation → Testing → Build → Docker Image → Deployment → Production → Monitoring

Each stage runs automatically whenever code is pushed to the `devops-test` branch, and every stage only runs if the previous one succeeds — this ensures broken code never reaches production.

## Tool Selected: GitHub Actions

**Why GitHub Actions:**
- The codebase already lives on GitHub, so Actions requires no separate server or infrastructure (unlike Jenkins)
- Pipeline definitions live inside the repo (`.github/workflows/ci-cd.yml`), version-controlled alongside the code
- Free tier is generous, and it scales automatically using GitHub's own cloud runners
- Large marketplace of reusable actions reduces custom scripting
- Triggers natively on GitHub events (`push`, `pull_request`) with no manual webhook setup

## Pipeline Stages

| Stage | Purpose | Tool Used |
|---|---|---|
| **Code Validation** | Checks code style and catches syntax issues early | `flake8` |
| **Testing** | Runs automated unit tests to confirm functionality | `pytest` |
| **Build** | Confirms all required files are present before packaging | Shell script |
| **Docker Image** | Packages the app into a portable, standardized container | `Docker` |
| **Deployment** | Runs the containerized app (simulated production run) | `Docker run` |
| **Monitoring** | Health-checks the running container; fails the pipeline (and would alert a team) if the app isn't healthy | `docker inspect` |

## Project Files

| File | Purpose |
|---|---|
| `app.py` | Sample application code |
| `test_app.py` | Automated tests for the application |
| `Dockerfile` | Instructions to containerize the application |
| `.github/workflows/ci-cd.yml` | The CI/CD pipeline definition |

## How to View Pipeline Runs

Go to the **Actions** tab of this repository to see every pipeline run, including logs for each stage.

## Future Improvements (Real-World Next Steps)

- Deploy to a real cloud service (e.g., Render, AWS ECS) instead of simulating deployment
- Add real monitoring tools (Datadog, Sentry, Grafana) for 24/7 production observability
- Introduce Pull Requests + branch protection so validation/testing run before code is merged
- Add Slack/email notifications on pipeline failure
