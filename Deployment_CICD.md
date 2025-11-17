# KevinCY-Kodex — Deployment & CI/CD Guideline

**Version:** 2025.11  
**Scope:** FastAPI · Docker · GitHub Actions

---

## 1. CI/CD 파이프라인 개요 (CI/CD Pipeline Overview)

`KevinCY-Kodex`의 CI/CD 파이프라인은 코드 변경사항을 안정적이고 자동화된 방식으로 사용자에게 전달하는 것을 목표로 합니다.

**전체 흐름**:
`Git Push` (feature branch) → `Pull Request` → **[CI]** `Lint & Test` 실행 → `Code Review` & `Merge` (main branch) → **[CD]** `Docker Build & Push` → `Deploy` (staging/production)

---

## 2. 브랜치 전략 (Branching Strategy)

**Git-Flow**의 간소화된 버전을 따릅니다.

-   `main`: 항상 배포 가능한 상태를 유지하는 메인 브랜치.
-   `develop`: 다음 릴리즈를 위해 개발 중인 기능들이 통합되는 브랜치 (선택 사항).
-   `feature/<feature-name>`: 새로운 기능을 개발하는 브랜치. `main` 또는 `develop`에서 분기합니다.
-   `fix/<bug-name>`: 버그를 수정하는 브랜치.

모든 작업은 `feature` 또는 `fix` 브랜치에서 시작하며, `Pull Request`를 통해 `main` 브랜치로 병합됩니다.

---

## 3. CI (Continuous Integration)

**GitHub Actions**를 사용하여 CI 파이프라인을 구축합니다.

-   **트리거**: `main` 브랜치로의 `push` 또는 `pull_request` 이벤트 발생 시.
-   **주요 작업 (Jobs)**:
    1.  **Linting**: `ruff` 또는 `flake8`을 사용하여 코드 스타일을 검사합니다.
    2.  **Testing**: `pytest`를 실행하여 모든 테스트가 통과하는지 확인합니다.
-   **목표**: `main` 브랜치에 병합되는 모든 코드가 최소한의 품질 기준을 통과했음을 보증합니다.

**예시 워크플로우 (`.github/workflows/ci.yml`)**:
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

## 4. Dockerization 전략

-   **`Dockerfile` 표준**:
    -   **베이스 이미지**: 공식 `python:3.11-slim` 이미지를 사용합니다.
    -   **멀티-스테이지 빌드**: 의존성 설치와 실제 실행 환경을 분리하여 최종 이미지 크기를 최소화합니다.
    -   **비-루트 사용자 실행**: 보안을 위해 `appuser`와 같은 비-루트 사용자로 애플리케이션을 실행합니다.
-   **`.dockerignore`**: `.git`, `__pycache__`, `.env` 등 불필요한 파일이 이미지에 포함되지 않도록 설정합니다.

---

## 5. CD (Continuous Deployment)

**GitHub Actions**를 사용하여 CD 파이프라인을 구축합니다.

-   **Staging 환경**:
    -   **트리거**: `main` 브랜치에 `push`될 때마다 자동으로 배포됩니다.
    -   **프로세스**: Docker 이미지를 빌드하여 컨테이너 레지스트리(예: Docker Hub, GHCR)에 푸시한 후, `staging` 서버에 최신 이미지를 배포합니다.
-   **Production 환경**:
    -   **트리거**: **수동 실행** 또는 **Git 태그(Tag)** 생성 시 배포됩니다. (예: `v1.0.0`)
    -   **프로세스**: `staging`에서 검증된 특정 버전의 Docker 이미지를 `production` 서버에 배포합니다.

---

## 6. Secret 관리 (Secret Management)

API 키, DB 접속 정보 등 민감 정보는 코드에 절대 포함하지 않습니다.

-   **Local 환경**: `.env` 파일을 사용하며, 이 파일은 `.gitignore`에 추가하여 버전 관리에서 제외합니다.
-   **CI/CD 환경**: **GitHub Secrets**를 사용하여 워크플로우 내에서 환경 변수로 주입합니다.
-   **배포 환경 (Staging/Production)**:
    -   서버 환경 변수로 직접 설정.
    -   클라우드 제공업체의 Secret Manager 서비스(예: AWS Secrets Manager, GCP Secret Manager) 사용을 권장합니다.

---

## 7. 모니터링 및 롤백 (Monitoring & Rollback)

-   **모니터링**: 배포 후 애플리케이션의 상태(오류율, 응답 시간 등)를 지속적으로 모니터링합니다. (`Architecture.md`의 로깅/모니터링 규칙 참조)
-   **롤백**: 배포 후 심각한 문제가 발생할 경우, 이전 버전의 안정적인 Docker 이미지로 빠르게 롤백할 수 있는 절차를 마련해야 합니다.

---

End of KevinCY-Kodex Deployment & CI/CD Guideline