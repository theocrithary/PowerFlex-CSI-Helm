# PowerFlex-CSI-Helm

## Install Helm
`
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3\
chmod 700 get_helm.sh\
./get_helm.sh\
`

# Install Git
yum install git

## Create a new project
- e.g. powerflex
oc new-project powerflex

## Install the external snapshot controller
git clone https://github.com/kubernetes-csi/external-snapshotter/
cd ./external-snapshotter
git checkout release-6.2
kubectl kustomize client/config/crd | kubectl create -f -
kubectl -n kube-system kustomize deploy/kubernetes/snapshot-controller | kubectl create -f -

## Clone the CSI driver repo
git clone -b v2.5.0 https://github.com/dell/csi-powerflex.git

## Create vxflexos-config secret
kubectl create secret generic vxflexos-config -n powerflex --from-file=config=config/config.yaml

## Install the CSI driver
dell-csi-helm-installer/csi-install.sh --namespace powerflex --values config/values.yaml

## After deployment of the instance, check the pods are running in the namespace
oc get pods -n powerflex

```
NAME                                   READY   STATUS    RESTARTS   AGE
vxflexos-controller-6dbf64f6fb-fnf86   5/5     Running   0          2m50s
vxflexos-controller-6dbf64f6fb-vw9zc   5/5     Running   0          2m50s
vxflexos-node-jl7pr                    2/2     Running   0          2m50s
vxflexos-node-k27vs                    2/2     Running   0          2m50s
vxflexos-node-k8kld                    2/2     Running   0          2m50s
```

## Create storage class
kubectl create -f config/storageclass.yaml

## Create a volume snapshot class
kubectl create -f config/volumesnapshotclass.yaml


#############################################
## Test the CSI driver
#############################################

oc new-project test
cd config
curl -LO https://github.com/theocrithary/PowerFlex-CSI-Helm/raw/main/pvc.yaml
curl -LO https://github.com/theocrithary/PowerFlex-CSI-Helm/raw/main/test_pod.yaml
curl -LO https://github.com/theocrithary/PowerFlex-CSI-Helm/raw/main/snapshot-of-pvol0.yaml

kubectl create -f config/test_pvc.yaml
kubectl create -f config/test_pod.yaml
kubectl create -f config/test_snapshot.yaml

## Check that all items are created successfully
kubectl get persistentvolumes
kubectl get volumesnapshot
kubectl describe storageclass eqx-powerflex-sc
kubectl describe pod theo-test-pod1

## Log into the pod and test the PV for read/write access
kubectl exec -it theo-test-pod1 -- bash
mount | grep data0
cd /data0
touch test

## Clean up and remove all test items
kubectl delete pvol0-snap1
kubectl delete pod theo-test-pod1
kubectl delete pvc pvol0
kubectl delete ns test
