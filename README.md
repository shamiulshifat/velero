# Velero: README.md
Backup and migrate Kubernetes resources

MinIO : Object Storage for AI, Opensource, S3 compatible.

Step 1: Run minio container
```
sudo docker pull minio/minio
```
Step 2: Download the minio library and change the permissiona and run the minio server
```
wget https://dl.minio.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo ./minio server /minio>
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
```
velero install \
--provider aws \
--use-restic \
--plugins velero/velero-plugin-for-aws:v1.2.1 \
--bucket testing \
--secret-file ./minio.credentials \
--backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://<ip-address>:9000
```
Step 6: Check and verify velero is up and running in velero namespace
```
velero version
kubectl get ns
kubectl get all -n velero (pods should be up and running)
kubectl get crds -n velero
```
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
Step 8: Restore velero backup to the cluster:
```
velero get restore
velero restore create firstbackup-restore1 --from-backup firstbackup
velero get restore
kubectl get restore -n velero
velero restore describe firstbackup-restore1
velero restore delete firstbackup-restore -  not deleting from GUI
velero backup delete firstbackup
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
