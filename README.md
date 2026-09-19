<div align="center">
    
# GitHub Actions + OpenShift Pipelines on Tekton CD

<img alt="github-actions" height="60" src="https://camo.githubusercontent.com/8e53fdc2a5df470f09d96aa7fce050cf1668249476538dcd4968f99b5826b03c/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f64657669636f6e732f64657669636f6e2f69636f6e732f676974687562616374696f6e732f676974687562616374696f6e732d6f726967696e616c2e737667">
<img height="70" alt="tekton-cd" src="https://github.com/user-attachments/assets/596b5c47-3883-49fb-b8da-b2879ba73be5" />
<img alt="openshift" src="https://camo.githubusercontent.com/dca99335c853eca1c45d0207eec8bedac56f329b45d05eb2b2dcf3ce213c4308/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f332f33612f4f70656e53686966742d4c6f676f547970652e737667" height="70">
</div>

> [!NOTE]
> **Hybrid CI/CD:** GitHub Actions handles CI. OpenShift Pipelines (Tekton) handles Kubernetes-native delivery.
<img src="https://github.com/barbaria888/tekton-workflow-openshift-deployment/blob/main/images/ChatGPT%20Image%20Nov%201%2C%202025%2C%2004_35_13%20PM.png">

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




**Workspace:** `output`
**Storage:** `1Gi PVC`
<img src="https://github.com/barbaria888/tekton-workflow-openshift-deployment/blob/main/images/oc-pipelines-console-pvc-details.png">

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
<img src="https://github.com/barbaria888/tekton-workflow-openshift-deployment/blob/main/images/oc-pipelines-oc-final.png">
<img src="https://github.com/barbaria888/tekton-workflow-openshift-deployment/blob/main/images/oc-pipelines-oc-green.png">

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
