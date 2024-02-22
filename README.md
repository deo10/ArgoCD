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