https://fluxcd.io/flux/get-started/

curl -s https://fluxcd.io/install.sh | sudo bash

. <(flux completion bash)

export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-infra \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal


