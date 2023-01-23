# PowerFlex-CSI-OpenShift-Console

## Create a new project
- e.g. powerflex

## Create vxflexos-creds secret
- ensure the new project 'powerflex' is selected when creating the new secrets
- create a new key/value secret with the secret name = vxflexos-creds and key = config
- use the vxflexos-creds.yaml file included in this repo as reference and paste into the text field
- edit with your environment details (e.g. endpoint IP address, username, password)

## Create vxflexos-config secret
- ensure the new project 'powerflex' is selected when creating the new secrets
- create a new key/value secret with the secret name = vxflexos-config and key = config
- use the vxflexos-config.yaml file included in this repo as reference and paste into the text field
- edit with your environment details (e.g. endpoint IP address, username, password)

## Create vxflexos-certs secret
- create a new secret from yaml
- use the vxflexos-certs.yaml file included in this repo and replace the default yaml template
- no further changes are required

## Create vxflexos-config-params ConfigMap
- create a new ConfigMap
- use the vxflexos-config-params.yaml file included in this repo and replace the default yaml template
- no further changes are required

## Install Dell CSI Operator using Operator Hub
- use default options
- stable, all namespaces, namespace = openshift-operators, automatic approval
- view operator once complete and confirm status 'succeeded'

## Create CSI PowerFlex instance
- ensure 'powerflex' project is selected in the drop down
- go to CSI PowerFlex and create CSIVXFlexOS instance
- use the CSIVXFlexOS.yaml file included in this repo and replace the default yaml template
- edit with your MDM details

## After deployment of the instance, check the pods are running in the namespace
```
NAME                                   READY   STATUS    RESTARTS   AGE
vxflexos-controller-5947559678-r4k6p   5/5     Running   0          65m
vxflexos-node-tknjc                    2/2     Running   0          65m
vxflexos-node-vhxgl                    2/2     Running   0          65m
vxflexos-node-vr6l4                    2/2     Running   0          65m
```

## Create storage class
- Name: csi-powerflex-sc
- Reclaim policy: Delete
- Volume binding mode: Immediate
- Provisioner: csi-vxflexos.dellemc.com
- add the following parameters
```
storagepool:XXXXX
protectiondomain:default
systemID:XXXXX
```
- Allow PersistentVolumeClaims to be expanded

#############################################
# Test the CSI driver
#############################################

# Create a new PersistentVolumeClaim
- StorageClass: csi-powerflex-sc
- PersistentVolumeClaim name: pvol0-powerflex
- Access mode: Single user (RWO)
- Size: 16GiB
- Volume mode: Filesystem

# Create a test pod
- use the test_pod.yaml included in this repo

# Log into the pod and test the volume
- check the mount
```
mount | grep data0
```
- check the path it was mounted to
```
cd data0
df -h .
ls -al
```
- check for read / write access
```
touch test
vi test
```
- check the file size
```
ls -al
```

# Create a volumesnapshot class
- use the powerflex-volumesnapshotclass.yaml included in this repo

# Create a VolumeSnapshot
- select the pvol0-powerflex pvc created earlier
- give it a name
- select the vxflexos-snapclass created earlier

# Check the volume and snapshot were created successfully

- Login to the PowerFlex and confirm the volume / snapshot
