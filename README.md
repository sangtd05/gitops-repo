# GitOps Repository

Welcome to the GitOps repository for maintaining the infrastructure and application state of our Kubernetes cluster. This repository leverages [ArgoCD](https://argoproj.github.io/argo-cd/) to ensure that the live state of the cluster always matches the desired state defined in this git repository.

## Project Structure

The repository is organized to separate ArgoCD configuration from the actual Kubernetes resources, promoting clarity and scalability.

```
.
├── apps/
│   └── infrastructure/   # ArgoCD Application definitions (The "What" and "Where")
│       ├── traefik.yaml
│       ├── headlamp.yaml
│       └── ...
├── resources/            # Kubernetes manifests and Helm values (The "Contents")
│   ├── argocd-network/
│   ├── headlamp/
│   └── ...
└── bootstrap/
    └── root.yaml         # The Root App (App-of-Apps) that kicks everything off
```

## Getting Started

### Prerequisites

- A Kubernetes Cluster
- CLI Tools: `kubectl`, `argocd` (optional but recommended)

### Bootstrapping

To start managing the cluster with this GitOps repo, simply apply the root application. This utilizes the **App-of-Apps** pattern, where one ArgoCD "Application" (`root.yaml`) is responsible for creating and syncing all other Applications defined in `apps/infrastructure/`.

1.  **Install ArgoCD** (if not already installed):
    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

2.  **Apply the Root Application**:
    ```bash
    kubectl apply -f bootstrap/root.yaml
    ```

    ArgoCD will now detect the `infrastructure-root` application, which points to the `apps/infrastructure` directory. It will then sync and deploy all applications found there, which in turn will deploy the resources found in `resources/`.

## Adding a New Application

1.  **Create Resources**: Add your Kubernetes manifests (Deployment, Service, Ingress, etc.) or Helm chart values to a new directory in `resources/<app-name>`.
2.  **Define Application**: Create a new ArgoCD Application YAML in `apps/infrastructure/<app-name>.yaml`.
    - Set the `source.path` to `resources/<app-name>`.
    - Set the `destination.namespace` to where you want the app to run.
3.  **Commit & Push**: Commit your changes to the `main` branch. ArgoCD will automatically detect the new Application definition and sync it.

## Wrapper Chart Pattern

For third-party Helm charts (like Traefik, Prometheus, etc.), we use the **Wrapper Chart Pattern**. instead of defining huge `values` blocks inside the ArgoCD Application manifest.

1.  **Create a local chart** in `resources/<app-name>`.
2.  **Define dependencies** in `Chart.yaml` pointing to the upstream chart.
3.  **Put values** in `values.yaml` in the local chart directory.
4.  **Point ArgoCD** to the local directory `resources/<app-name>`.

This keeps our Application definitions clean and allows for better version control of our configuration.