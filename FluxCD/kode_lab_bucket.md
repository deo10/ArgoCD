In this step, we will configure Flux server.
Setup the Flux server using the flux bootstrap command with the following specifications:

username: bob
password: Bob_pass123
Path: flux-clusters/dev-cluster
branch: master
url: https://git.example.com/bob/block-buster.git


The sample command looks like following:

flux bootstrap git --url=<url> --username=<> --password=<> --branch=master --token-auth=true --path=<path> --ca-file=/var/lib/gitea/custom/cert.pem

---
# step#1
flux bootstrap git --url=https://git.example.com/bob/block-buster.git --username=bob --password=Bob_pass123 --branch=master --token-auth=true --path=flux-clusters/dev-cluster --ca-file=/var/lib/gitea/custom/cert.pem

# step#2
encode secret to connect to S3 bucket
echo -n "minioadmin" | base64
bWluaW9hZG1pbg==

# step#3
minio-secret.yaml and add creds
apiVersion: v1
kind: Secret
metadata:
  name: minio-crds
  namespace: flux-system
type: generic
data:
  accesskey: bWluaW9hZG1pbg==
  secretkey: bWluaW9hZG1pbg==

# step#4
apply yaml
kubectl apply -f secret.yaml


# step#5
To create flux bucket source, you can run the following command.
flux create source bucket 4-demo-source-minio-s3-bucket-bb-app \
  --bucket-name bucket-bb-app \
  --secret-ref minio-crds \
  --endpoint minio.minio-dev.svc.cluster.local:9000  \
  --provider generic \
  --insecure \
  --export > ~/block-buster/flux-clusters/dev-cluster/4-demo-source-minio-s3-bucket-bb-app.yml

or use yaml file (not working):
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: Bucket
metadata:
  name: 4-demo-source-minio-s3-bucket-bb-app
  namespace: flux-system
spec:
  bucketName: bucket-bb-app
  endpoint: minio.minio-dev.svc.cluster.local:9000
  insecure: true
  interval: 1m0s
  provider: generic
  secretRef:
    name: minio-crds

# step#6
create a flux kustomization to apply the manifests.
To create flux kustomization which uses the previously created flux source, you can run the following command.
flux create kustomization 4-demo-kustomize-minio-s3-bucket-bb-app \
  --source Bucket/4-demo-source-minio-s3-bucket-bb-app \
  --target-namespace 4-demo \
  --path ./manifests \
  --prune true \
  --export > ~/block-buster/flux-clusters/dev-cluster/4-demo-kustomize-minio-s3-bucket-bb-app.yml


Or, you can directly create the manifest like following at the mentioned location.
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: 4-demo-kustomize-minio-s3-bucket-bb-app
  namespace: flux-system
spec:
  interval: 1m0s
  path: ./manifests
  prune: true
  sourceRef:
    kind: Bucket
    name: 4-demo-source-minio-s3-bucket-bb-app
  targetNamespace: 4-demo

# step7
Make changes on source files and push it to master
fluxcd is creating resources under 4-demo namespace

k get all -n 4-demo
NAME                                READY   STATUS    RESTARTS   AGE
pod/block-buster-84d59cf55c-76lsg   1/1     Running   0          19s

NAME                           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/block-buster-service   NodePort   10.109.246.76   <none>        80:30004/TCP   19s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/block-buster   1/1     1            1           19s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/block-buster-84d59cf55c   1         1         1       19s


