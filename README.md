# multi-cluster-orchestrator

Followed the instructions from: <https://github.com/GoogleCloudPlatform/gke-fleet-management/tree/main/multi-cluster-orchestrator>

In this demo, I followed the Gemma 3 vLLM sample: <https://github.com/GoogleCloudPlatform/gke-fleet-management/blob/main/multi-cluster-orchestrator/samples/gemma_vllm/README.md>

Notes: please review additional changes made to the **main.tf** and **network.yaml** files.

## Configure environment variables

```bash
PROJECT_ID=$(gcloud config get project)
PROJECT_NUMBER=$(gcloud projects describe $(gcloud config get-value project) --format="value(projectNumber)")

export GOOGLE_CLOUD_PROJECT=$PROJECT_ID
```

## Grant additional permissions if needed

1. Check which service account or user account is used by terraform. In my case (i.e. running a Code server & a terminal on a VM), terraform uses the default compute service account.

1. Check if the service account, which is used by terraform, has the needed permissions / roles

```bash
gcloud projects get-iam-policy addo-argolis-demo --flatten="bindings[].members" --format="table(bindings.role)" --filter="bindings.members:827859227929-compute@developer.gserviceaccount.com"
```

1. e.g. sample output

```bash
ROLE
roles/editor
roles/resourcemanager.projectIamAdmin
```

1. Grant additional permissions if needed

```bash
# If using a service account
gcloud projects add-iam-policy-binding addo-argolis-demo \
    --member="serviceAccount:827859227929-compute@developer.gserviceaccount.com" \
    --role="roles/owner"

## If needed
# --role="roles/resourcemanager.projectIamAdmin"
# --roles/iam.serviceAccountTokenCreator

# If using a user account
gcloud projects add-iam-policy-binding addo-argolis-demo \
    --member="user:andrew@ducdo.altostrat.com" \
    --role="roles/resourcemanager.projectIamAdmin"
```

1. Install additional packages in Rocky OS 

```bash
sudo dnf update -y
sudo dnf install epel-release -y
sudo dnf install snapd -y
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
# sudo snap install snapd
sudo snap install k6
```

1. Enable ArgoCD GUI access

```bash
helm ls -A
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade argocd argo/argo-cd -n argocd --set server.ingress.enabled=true
helm upgrade argocd argo/argo-cd -n argocd --set configs.params."server\.insecure"=true
# check
helm get all argocd -n argocd | more
# get the GUI admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

1. View ArgoCD GUI: Open Cloud Shell

```bash
gcloud container clusters get-credentials mco-hub --region us-central1
kubectl port-forward service/argocd-server -n argocd 8080:443
# open web viewer
```

### Notes

1. There is an issue with the **gke-l7-cross-regional-internal-managed-mc** and the issue was being fixed. For a workaround, the demo used **gke-l7-global-external-managed-mc** as included in the **network.yaml** file.
