# Gitea Kubernetes Deployment with MySQL and ngrok

This project deploys Gitea (a Git hosting service) on a Kubernetes cluster using Helm, with MySQL as the external database. It includes instructions for exposing Gitea publicly using ngrok.

---

## Prerequisites

- Kubernetes cluster (e.g., Minikube, Codespaces Kubernetes)
- `kubectl` installed and configured
- `helm` installed
- `ngrok` installed (for public exposure)
- MySQL Helm chart installed via Bitnami
- Access to your Kubernetes cluster (e.g., `kubectl` configured to use the correct context)

---

## Step 1: Install MySQL on Kubernetes

1. Create a `mysql-values.yaml` file with the following content:

```yaml
auth:
  rootPassword: admin123
  username: gitea
  password: gitea123
  database: gitea_db

primary:
  service:
    type: ClusterIP


## ✅ Dev Environment (Codespaces)

```bash
pip install ansible kubernetes
ansible-playbook gitea/up.yml

2. Add Bitnami Helm repo and install MySQL:
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install mysql bitnami/mysql -f mysql-values.yaml

If you want to run the Kubernetes cluster locally, you can use Minikube:

Start Minikube:
minikube start

Make sure your kubectl context is set to minikube:
kubectl config use-context minikube
kubectl cluster-info

Step 2: Update Gitea Helm values.yaml
Ensure your gitea/values.yaml file includes:
redis-cluster:
  enabled: false
redis:
  enabled: false
postgresql:
  enabled: false
postgresql-ha:
  enabled: false

persistence:
  enabled: true
  size: 5Gi
  storageClass: standard

gitea:
  admin:
    username: gitea_admin
    password: admin123
    email: admin@example.com

  config:
    database:
      DB_TYPE: mysql
      HOST: mysql.default.svc.cluster.local:3306
      NAME: gitea_db
      USER: gitea
      PASSWD: gitea123

    session:
      PROVIDER: db
    cache:
      ADAPTER: db
    queue:
      TYPE: persistable-channel

Step 3: Deploy Gitea Helm Chart Using Kubernetes Job
Apply your RBAC and service account YAML files:
kubectl apply -f k8s/gitea-rbac.yaml

Deploy Gitea using your Helm job manifest:
kubectl apply -f k8s/prod-up.yaml

Monitor the job:
kubectl get jobs
kubectl logs -f job/gitea-prod-install

Step 4: Access Gitea Locally
Port forward Gitea HTTP service:
kubectl port-forward svc/gitea-http 3000:3000

Step 5: Expose Gitea Publicly Using ngrok
1. Install ngrok (if not already installed):
# For GitHub Codespaces:
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar -xvzf ngrok-v3-stable-linux-amd64.tgz
sudo mv ngrok /usr/local/bin/
chmod +x /usr/local/bin/ngrok

2. Add your ngrok auth token:
ngrok config add-authtoken <your_ngrok_auth_token>

3. Start ngrok tunnel to your local port 3000 (run this in a new terminal window):
ngrok http 3000

4. Copy the public URL from the ngrok output (e.g., https://abc123.ngrok-free.app) and open it in any browser to access Gitea publicly.

Troubleshooting
1. If the Gitea Helm job fails with permission errors, ensure your service account has proper RBAC permissions for resources like deployments, statefulsets, services, persistentvolumeclaims, networkpolicies, etc.

2. Use kubectl describe job gitea-prod-install and kubectl logs -f job/gitea-prod-install to diagnose issues.

3. Ensure your Kubernetes context is set to the right cluster (e.g., minikube).


Cleanup
To remove all deployed resources:
kubectl delete job gitea-prod-install
helm uninstall gitea
helm uninstall mysql
kubectl delete -f k8s/gitea-rbac.yaml

References
Gitea Helm Charts

Bitnami MySQL Helm Chart

ngrok Documentation