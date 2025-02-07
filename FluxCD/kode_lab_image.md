# setup fluxCD with ImageController

flux bootstrap git \
--url=https://git.example.com/bob/block-buster.git \
--username=bob --password=Bob_pass123 \
--branch=master --token-auth=true \
--path=flux-clusters/dev-cluster \
--ca-file=/var/lib/gitea/custom/cert.pem \
--components-extra="image-reflector-controller,image-automation-controller"

# work with docker hub

docker pull siddharth67/block-buster-dev:7.1.0

docker tag siddharth67/block-buster-dev:7.1.0 controlplane:32766/library/block-buster-dev-private:ver1

docker push controlplane:32766/library/block-buster-dev-private:ver1