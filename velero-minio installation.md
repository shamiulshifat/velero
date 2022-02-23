# Velero: README.md
Backup and migrate Kubernetes resources
*******************
First install docker from here:
https://docs.docker.com/engine/install/ubuntu/
*****************
MinIO : Object Storage for AI, Opensource, S3 compatible.

Step 1: Run minio container
```
sudo docker pull minio/minio
```
***************
If docker is unable to pull image from server, run:
```
sudo chmod 666 /var /run /docker.sock
```
***************
Step 2: Download the minio library and change the permissiona and run the minio server
```
wget https://dl.minio.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo ./minio server /minio
```
Step 3: Download Velero 1.6.2 latest release
```
wget https://github.com/vmware-tanzu/velero/releases/download/v1.6.2/velero-v1.6.2-linux-amd64.tar.gz
tar zxf velero-v1.6.2-linux-amd64.tar.gz
sudo mv velero-v1.6.2-linux-amd64/velero /usr/local/bin/
rm -rf velero*
```
Step 4: Create credentials file (Needed for velero initialization and to connect to the object source)
```
cat <<EOF>> minio.credentials
[default]
aws_access_key_id=minioadmin
aws_secret_access_key=minioadmin
EOF
```
Step 5: Install Velero in the Kubernetes Cluster
-create a bucket in minio and enter the name in here as bucket
get your master node ip by
```
kubectl get nodes -o yaml
```
enter the ip address as s3 address here. you can also login to minio use this address with port
```
velero install \
--provider aws \
--use-restic \
--plugins velero/velero-plugin-for-aws:v1.2.1 \
--bucket testing \
--secret-file ./minio.credentials \
--backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://<ip-address>:9000
```
I through error of some config for micork8s:
```
sudo microk8s kubectl config view --raw > ~/.kube/config
```
Step 6: Check and verify velero is up and running in velero namespace
```
velero version
kubectl get ns
kubectl get all -n velero (pods should be up and running)
kubectl get crds -n velero
```
*************
Velero uses Restic for backing up Kubernetes volumes. To let Restic know of the kubelet directory in the MicroK8s context we need to patch its daemonset manifest:
```
sudo microk8s kubectl -n velero patch daemonset.apps/restic --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/volumes/0/hostPath/path", "value":"/var/snap/microk8s/common/var/lib/kubelet/pods"}]'

```
*************************
Notice Restic pods are not running. giving: crashbackloop. to run:
https://mannimal.blog/2019/10/04/backing-up-your-kubernetes-applications-with-velero-v1-1/
*******************************************
Lets deploy a app with namspace, e.g : nginx
--create namespace:
```
kubectl get crds -n velero
kubectl create ns nginx-test
kubectl get ns
```
Create a deployment:
```
kubectl create deploy nginx --image nginx --replicas=2 -n nginx-test
```
Check pods:
```
watch kubectl get pods -n nginx-test
```
escape terminala by ctrl+c

Now create a nginx service exposed on nodeport 80
```
kubectl create service nodeport nginx --tcp=80:80 -n nginx-test

```
see all service with cluster ip
```
 kubectl get all -n nginx-test
 

```
***************************************
Step 7: Steps to create a backup using velero
```
velero help backup
velero backup-location get
velero backup get

Enable tab completion for preferred shell "zsh" or "bash"
source <(velero completion zsh)
kubectl get backups -n velero

velero backup create firstbackup (For entire cluster)
velero backup create firstbackup --include-namespaces testing (For particular ns)
velero backup create firstbackup --include -namespaces testing --exclude-resources pods (excluding resources from particular ns)
```
e.g:
```
velero backup create fnginxbackup --include-namespaces nginx-test
```
check if backup is on progress:
```
velero backup logs nginxbackup
velero backup describe nginxbackup2

```
*************
Now delete the namespace "nginx-test"
```
kubectl delete ns nginx-test
kubectl get all -n nginx-test
```
Now perform restore of nginx-test
**************
Step 8: Restore velero backup to the cluster:
```
velero get restore
velero restore create firstbackup-restore1 --from-backup firstbackup
kubectl get restore -n velero
velero restore describe firstbackup-restore1
velero restore delete firstbackup-restore1
velero backup delete firstbackup
```
After restoration, check new restored namespaces are added or not: 
```
kubectl get ns
```
then  services can be accessed from the cluster ip and port:
``
kubectl get service --namespace <enter namespace>
```
```
Step 9: Backup schedule using velero:
```
velero schedule get
kubectl get schedule -n velero
velero schedule create firstschedule --schedule="@every 2m" --include-namespaces default
velero schedule describe firstschedule
velero schedule get
kubectl get schedule -n velero
velero backup get
velero schedule delete --all
```
Step 10: Create backup with TTL( Time to Live):
```
velero backup create secondbackup  --include-namespaces default --ttl 2h
velero backup describe secondbackup
velero backup get
```
***************
See for more example and how to create pvc
https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/
*****************
https://docs.pivotal.io/tkgi/1-13/velero-stateful-lb.html
***************
https://mannimal.blog/2019/10/04/backing-up-your-kubernetes-applications-with-velero-v1-1/
*************************
