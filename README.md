# Dynamic NFS Provision
Deploy RBAC
```sh
$  kubectl create -f rbac.yaml
```
Check 
```sh
$  kubectl create -f rbac.yamlkubectl get clusterrole,clusterrolebinding,role,rolebinding | grep nfs
```
Edit class.yaml
```sh
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
name: managed-nfs-storage  # Change this
provisioner: example.com/nfs # Change this
parameters:
archiveOnDelete: “false”
```
Deploy class.yaml
```sh
$  kubectl create -f class.yaml
```
Check
```sh
$  kubectl get storageclass
```
Edit NFS provision
```sh
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
spec:
  selector:
    matchLabels:
      app: nfs-client-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes   # mount point
          env:
            - name: PROVISIONER_NAME
              value: example.com/nfs
            - name: NFS_SERVER
              value: 172.42.42.100  # Change this
            - name: NFS_PATH
              value: /srv/nfs/kubedata  # Change this
      volumes:
        - name: nfs-client-root
          nfs:
            server: 172.42.42.100   # Change this
            path: /srv/nfs/kubedata  # Change this
```  
Deploy NFS provisioner
```sh
$  kubectl create -f deployment.yaml
```
Edit 4-pvc-nfs.yaml
```sh
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  storageClassName: managed-nfs-storage   # this is the class name
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
```
Deploy PVC
```sh
$  kubectl create -f 4-pvc-nfs.yaml
```
Check
```sh
$  kubectl get pvc,pv
```
Deploy pod to check
```sh
$  kubectl create -f 4-busybox-pv-nfs.yaml
```
Check
```sh
$  kubectl describe pod busybox
```
Source:
- [blog.exxactcorp.com/deploying-dynamic-nfs-provisioning-in-kubernetes](https://blog.exxactcorp.com/deploying-dynamic-nfs-provisioning-in-kubernetes/)
