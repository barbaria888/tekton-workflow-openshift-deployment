# GitHub Actions + OpenShift Pipelines (Tekton)

> [!NOTE]
> **Hybrid CI/CD:** GitHub Actions handles CI. OpenShift Pipelines (Tekton) handles Kubernetes-native delivery.

## Architecture

```mermaid
flowchart LR
    DEV[Developer] --> GH[GitHub]

    GH --> CI[GitHub Actions]
    CI --> LINT[Flake8]
    LINT --> TEST[Nose]

    TEST --> TEK[Tekton]
    TEK --> CLONE[Git Clone]
    CLONE --> VALIDATE[Lint + Test]
    VALIDATE --> BUILD[Buildah]
    BUILD --> REG[Container Registry]
    REG --> OCP[OpenShift]

    OCP --> APP[Application]
```

## CI

```mermaid
flowchart LR
    A[Push / PR] --> B[Checkout]
    B --> C[Dependencies]
    C --> D[Flake8]
    D --> E[Nose + Coverage]
```

**Runtime:** `python:3.9-slim`

## CD

```mermaid
flowchart TD
    A[Tekton Pipeline]
    A --> B[Workspace Cleanup]
    B --> C[Source]
    C --> D[Validation]
    D --> E[Buildah]
    E --> F[Push Image]
    F --> G[OpenShift Deploy]
```

> [!NOTE]
> The committed `.tekton/tasks.yml` contains the reusable **cleanup** and **Nose** tasks. The complete Tekton flow is documented in the accompanying implementation walkthrough.

## Shared Workspace

```mermaid
flowchart LR
    PVC[(oc-lab-pvc)] --> T1[Cleanup]
    PVC --> T2[Clone]
    PVC --> T3[Test]
    PVC --> T4[Build]
```
<img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1761993257371/594b05b4-6858-42e1-9c97-407771d84219.png">

**Workspace:** `output`
**Storage:** `1Gi PVC`

## Container

```mermaid
flowchart TD
    A[python:3.9-slim]
    A --> B[Install Dependencies]
    B --> C[Copy Service]
    C --> D[Create service User]
    D --> E[Run as Non-Root]
    E --> F[Gunicorn :8000]
```

## Repository

```text
.github/workflows/
└── workflow.yml          # GitHub Actions CI

.tekton/
├── tasks.yml             # Tekton tasks
└── README.md

service/                  # Application
tests/                    # Tests
Dockerfile                # Runtime image
bin/setup.sh              # Local environment
```

## Delivery Model

```mermaid
flowchart LR
    SOURCE[Source] --> VALIDATE[Validate]
    VALIDATE --> PACKAGE[Package]
    PACKAGE --> DELIVER[Deliver]
    DELIVER --> RUN[OpenShift]
```

### References

* **Implementation:** [GitHub Repository](https://github.com/barbaria888/tekton-workflow-openshift-deployment?utm_source=chatgpt.com)
* **Walkthrough:** <a href="https://hardik0811arora.hashnode.dev/when-github-actions-sync-with-openshift-pipelines-cicd">*When GitHub Actions Sync with OpenShift Pipelines: CI/CD* — Hashnode, 22 Nov 2025 </a>
