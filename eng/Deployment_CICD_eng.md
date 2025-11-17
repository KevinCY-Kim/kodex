# KevinCY-Kodex — Deployment & CI/CD Guideline

**Version:** 2025.11  
**Scope:** FastAPI · Docker · GitHub Actions

---

## 1. CI/CD Pipeline Overview

The CI/CD pipeline for `KevinCY-Kodex` aims to deliver code changes to users in a stable and automated manner.

**Overall Flow**:
`Git Push` (feature branch) → `Pull Request` → **[CI]** Run `Lint & Test` → `Code Review` & `Merge` (main branch) → **[CD]** `Docker Build & Push` → `Deploy` (staging/production)

---

## 2. Branching Strategy

Follows a simplified version of **Git-Flow**.

-   `main`: The main branch that is always in a deployable state.
-   `develop`: A branch where features under development for the next release are integrated (optional).
-   `feature/<feature-name>`: A branch for developing new features. Branched from `main` or `develop`.
-   `fix/<bug-name>`: A branch for fixing bugs.

All work starts from a `feature` or `fix` branch and is merged into the `main` branch via a `Pull Request`.

---

## 3. CI (Continuous Integration)

Builds the CI pipeline using **GitHub Actions**.

-   **Trigger**: On `push` or `pull_request` events to the `main` branch.
-   **Key Jobs**:
    1.  **Linting**: Checks code style using `ruff` or `flake8`.
    2.  **Testing**: Runs `pytest` to ensure all tests pass.
-   **Goal**: To guarantee that all code merged into the `main` branch meets minimum quality standards.

**Example Workflow (`.github/workflows/ci.yml`)**:
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v3
      with:
        python-version: '3.11'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Lint with ruff
      run: |
        pip install ruff
        ruff check .
    - name: Test with pytest
      run: |
        pip install pytest
        pytest
```

---

## 4. Dockerization Strategy

-   **`Dockerfile` Standard**:
    -   **Base Image**: Use the official `python:3.11-slim` image.
    -   **Multi-stage Builds**: Separate dependency installation from the actual execution environment to minimize the final image size.
    -   **Non-root User Execution**: Run the application as a non-root user, such as `appuser`, for security.
-   **`.dockerignore`**: Configure to prevent unnecessary files like `.git`, `__pycache__`, and `.env` from being included in the image.

---

## 5. CD (Continuous Deployment)

Builds the CD pipeline using **GitHub Actions**.

-   **Staging Environment**:
    -   **Trigger**: Deploys automatically whenever there is a `push` to the `main` branch.
    -   **Process**: Builds a Docker image, pushes it to a container registry (e.g., Docker Hub, GHCR), and then deploys the latest image to the `staging` server.
-   **Production Environment**:
    -   **Trigger**: Deploys on **manual execution** or when a **Git Tag** is created (e.g., `v1.0.0`).
    -   **Process**: Deploys a specific version of the Docker image, verified in `staging`, to the `production` server.

---

## 6. Secret Management

Never include sensitive information like API keys or DB connection info in the code.

-   **Local Environment**: Use `.env` files, which are added to `.gitignore` to be excluded from version control.
-   **CI/CD Environment**: Use **GitHub Secrets** to inject them as environment variables within the workflow.
-   **Deployment Environment (Staging/Production)**:
    -   Set directly as server environment variables.
    -   Recommended to use a cloud provider's Secret Manager service (e.g., AWS Secrets Manager, GCP Secret Manager).

---

## 7. Monitoring & Rollback

-   **Monitoring**: Continuously monitor the application's status (error rate, response time, etc.) after deployment. (Refer to the logging/monitoring rules in `Architecture_eng.md`)
-   **Rollback**: A procedure must be in place to quickly roll back to a previous stable Docker image version if a serious problem occurs after deployment.

---

End of KevinCY-Kodex Deployment & CI/CD Guideline