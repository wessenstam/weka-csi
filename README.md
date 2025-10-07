# WEKA CSI Driver workshop

This repo holds some WEKA CSI examples and a fio load k8s module so loads can be simulated.

## Install one node k3s
curl -sfL https://get.k3s.io | sh -
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

## CSI DRIVER

### Install helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
### Install the Snapshotdriver
git clone https://github.com/kubernetes-csi/external-snapshotter  
cd external-snapshotter 
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f -
kubectl kustomize client/config/crd | kubectl create -f -
sleep 5

### Install CSI Driver
helm repo add csi-wekafs https://weka.github.io/csi-wekafs
sleep 5
helm install -n csi-wekafs --create-namespace csi-wekafs csi-wekafs/csi-wekafsplugin \
--set logLevel=6 \
--set pluginConfig.mountProtocol.allowNfsFailback=true \
--set pluginConfig.allowInsecureHttps=true 

### Set to one replica for the storage controller as we only have one node (with only one node available)
kubectl scale --replicas=1 deployment/csi-wekafs-controller -n csi-wekafs
