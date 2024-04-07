# addig a repo
argocd repo add https://3000-port-55cfc10cd89c4760.labs.kodekloud.com/bob/gitops-argocd

# create app with custom values

argocd app create helm-random-shapes \
--repo https://3000-port-55cfc10cd89c4760.labs.kodekloud.com/bob/gitops-argocd \
--path ./helm-chart \
--dest-server https://kubernetes.default.svc \
--sync-option CreateNamespace=true \
--helm-set color.circle=pink \
--helm-set color.square=red \
--helm-set service.type=NodePort \
--dest-namespace default 

# check and sync
argocd app sync argocd/helm-random-shapes
argocd app list

# check metadata of app
argocd app get helm-random-shapes

# cluster check and add new
argocd cluster list
argocd cluster add cluster_name

# new secret with configuration created
kubectl get secrets -A
argocd        cluster-cluster2-controlplane-2268674445   Opaque 

# creating app of apps using yaml from repo
kubectl apply -f https://3000-port-55cfc10cd89c4760.labs.kodekloud.com/bob/gitops-argocd/raw/branch/master/declarative/multi-app/app-of-apps.yml -n argocd