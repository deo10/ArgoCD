# bootstap flux-cd

flux bootstrap git --url=https://443-port-fb82123156594aa7.labs.kodekloud.com/bob/block-buster --username=bob --password=Bob_pass123 --branch=master --token-auth=true --path=flux-clusters/dev-cluster --ca-file=/var/lib/gitea/custom/cert.pem

# A repo named block-buster-helm-app is available in ArtifactHub.

URL: block-buster-helm-app
https://artifacthub.io/packages/helm/block-buster-app/block-buster-helm-app

block-buster-helm-app

This repo contains an helm artefact which is linked to a repo in GitHub.
This is used for FluxCD Demo.
Chart Name - block-buster-helm-app
URL - https://sidd-harth.github.io/block-buster-helm-app
Container Image - siddharth67/block-buster-dev:7.6.0

Sample values.yaml:

replicaCount: 1

service:
  type: NodePort
  nodePort: 30006

namespace:
  name: 6-demo
  
labels:
  app:
    name: block-buster
    version: 7.6.0
    env: dev

# Create a Flux source that pulls Helm chart manifests from the specified HELM Repository and save the configuration to a YAML file.

Here are the specifications:
HELM Repository URL: https://sidd-harth.github.io/block-buster-helm-app
Timeout: 10 seconds
Export Path:
~/block-buster/flux-clusters/dev-cluster/6-demo-source-helm-bb-app.yml

flux create source helm 2-demo-source-git-bb-app \
  --url=https://sidd-harth.github.io/block-buster-helm-app \
  --timeout=10s \
  --export > ~/block-buster/flux-clusters/dev-cluster/6-demo-source-helm-bb-app.yml


# Create a Flux HelmRelease that applies the manifests from the specified HelmRepository to a target namespace. 

Here are the specifications:
Name: 6-demo-helm-release-bb-app
Source: HelmRepository that created in previous step "6-demo-source-helm-bb-app"
Target Namespace: 6-demo
Timeout:10 seconds
Chart: block-buster-helm-app
Values File: ~/flux-training/helm/values.yml
Export Path:
 ~/block-buster/flux-clusters/dev-cluster/6-demo-helm-release-bb-app.yml

flux create helmrelease 6-demo-helm-release-bb-app \
--chart block-buster-helm-app \
--interval 10s \
--target-namespace 6-demo \
--source HelmRepository/6-demo-source-helm-bb-app \
--values ~/flux-training/helm/values.yml \
--export > ~/block-buster/flux-clusters/dev-cluster/6-demo-helm-release-bb-app.yml

as a result
➜  cat block-buster/flux-clusters/dev-cluster/6-demo-helm-release-bb-app.yml 
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: 6-demo-helm-release-bb-app
  namespace: flux-system
spec:
  chart:
    spec:
      chart: block-buster-helm-app
      reconcileStrategy: ChartVersion
      sourceRef:
        kind: HelmRepository
        name: 6-demo-source-helm-bb-app
  interval: 10s
  targetNamespace: 6-demo
  values:
    labels:
      app:
        env: dev
        name: block-buster
        version: 7.6.0
    namespace:
      name: 6-demo
    replicaCount: 1
    service:
      nodePort: 30006
      type: NodePort


# after the changes we made on block-buster/flux-clusters/dev-cluster/..yml

as it's git repo
we need to add changes, commit and push to origin master

then FluxCd will automatically create resources under 6-demo namespace

 ➜  flux get helmreleases 
NAME                            REVISION        SUSPENDED       READY   MESSAGE                                                                                                               
6-demo-helm-release-bb-app      7.6.0           False           True    Helm install succeeded for release 6-demo/6-demo-6-demo-helm-release-bb-app.v1 with chart block-buster-helm-app@7.6.0