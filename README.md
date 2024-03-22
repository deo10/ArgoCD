# official docs
https://argo-cd.readthedocs.io/en/stable/operator-manual/architecture/

# ArgoCD installation options

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/VERSION/manifests/ha/install.yaml #change version number v2.4.11

helm repo add argo url/argo-helm
helm install my-argo-cd argo/argo-cd --version 4.8.0

curl -sSl -o /usr/local/bin/argocd url/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

# Kodekloud UI open lab steps
change svc argocd-server to NodePort
open url svc_ip:NodePort to see UI
use admin + password from secret below

get -n argocd secret initial-secret -o json
echo "pass" | base64 -d #to decrypt pass

# Create, list, sync application
argocd app create nginx \
--repo https://charts.bitnami.com/bitnami \
--helm-chart nginx --revision 13.2.10 \
--dest-server https://kubernetes.default.svc \
--dest-namespace nginx

argocd app create solar-system-app \
--repo [repo-url] \
--path ./solar-system \
--dest-namespace solar-system
--dest-server https://kubernetes.default.svc \

argocd app list

argocd app sync [app-name]

# Create, list, sync project

argocd proj create

argocd proj list

# Reconciliation loop - how often argo will sync with repo (default -3 min)
kubectl edit configmaps argocd-cm -n argocd

apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
data:
  timeout.reconciliation: 60s

# Application health

custom health check 
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations.health.ConfigMap: |
    hs = {}
    hs.status = "Healthy"
     if obj.data.TRIANGLE_COLOR == "white" then
        hs.status = "Degraded"
        hs.message = "Use any color other than White "
     end
    return hs

it's looking on configmap of the app
health-check/configmap.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: moving-shapes-colors
data:
  CIRCLE_COLOR: "pink"
  OVAL_COLOR: "lightgreen"
  SQUARE_COLOR: "orange"
  TRIANGLE_COLOR: "white"   # use white to get a Degraded message in ArgoCD
  RECTANGLE_COLOR: "blue" 
