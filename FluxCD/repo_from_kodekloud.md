
demo app that can be used for fluxCD deployment
Use branches to switch between demos
https://github.com/sid-demo/bb-app-source/

fluxCD setup

flux bootstrap git --url=https://443-port-fb82123156594aa7.labs.kodekloud.com/bob/block-buster --username=bob --password=Bob_pass123 --branch=master --token-auth=true --path=flux-clusters/dev-cluster--ca-file=/var/lib/gitea/custom/cert.pem

git clone https://github.com/sid-demo/bb-app-source/

For this step, Create a folder named 1-demo within the flux-clusters/dev-cluster directory of the repository "block-buster."

Also copy the manifests (deployment.yml, namespace.yml, and service.yml) from the "bb-app-source" repository into the newly created 1-demo folder. Ensure that the folder structure matches the specified format:


block-buster/
└── flux-clusters/
    └── dev-cluster/
        └── 1-demo/
            ├── deployment.yml
            ├── namespace.yml
            └── service.yml

after copy the files do push changes on master for FluxCD repo
it will trigger the update on K8s for a specific namespace


# second lab setup

flux bootstrap git --url=https://443-port-c93990506f9647f7.labs.kodekloud.com/bob/block-buster --username=bob --password=Bob_pass123 --branch=master --token-auth=true --path=flux-clusters/dev-cluster --ca-file=/var/lib/gitea/custom/cert.pem

git clone https://github.com/sid-demo/bb-app-source/  (2-demo branch)

To configure flux to pull from the external "bb-app-source" repository, follow these steps:


Generate a flux source with the specified specifications:
Name: 2-demo-source-git-bb-app
URL: https://github.com/sid-demo/bb-app-source
Branch: Use the 2-demo branch.
Timeout: Set it to 10 seconds.

Once you've created the flux source, export the configuration to the following path:
~/block-buster/flux-clusters/dev-cluster/2-demo-source-git-bb-app.yml

flux create source git 2-demo-source-git-bb-app \
  --url=https://github.com/sid-demo/bb-app-source \
  --branch=2-demo \
  --interval=10s \
  --export > ~/block-buster/flux-clusters/dev-cluster/2-demo-source-git-bb-app.yml


To apply the manifests from the external "bb-app-source" repository using Flux, you need to create a Flux Kustomization with the following specifications
Name: 2-demo-kustomize-git-bb-app
Source: Use the previously defined "2-demo-source-git-bb-app" to fetch the manifests.
Target Namespace: Set it to "2-demo" to specify the namespace where the resources will be applied.
Timeout: Set the timeout to 10 seconds
Path: Specify "manifests" as the path to the directory where the manifests are located within the "bb-app-source" repository.
Prune: Enable this option by setting it to "true".
Export Path: Export the Kustomization configuration to the following path:
~/block-buster/flux-clusters/dev-cluster/2-demo-kustomize-git-bb-app.yml


flux create kustomization 2-demo-kustomize-git-bb-app \
 --source GitRepository/2-demo-source-git-bb-app \
 --prune true \
 --interval 10s \
 --target-namespace 2-demo \
 --path manifests  \
 --export > ~/block-buster/flux-clusters/dev-cluster/2-demo-kustomize-git-bb-app.yml


 as a result
 controlplane block-buster on  master [?] ➜  git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        flux-clusters/dev-cluster/2-demo-kustomize-git-bb-app.yml
        flux-clusters/dev-cluster/2-demo-source-git-bb-app.yml

git commit, git push

-> resources are created under k8s namespace 2-demo
kubectl get -n 2-demo svc block-buster-service
