velero install \
 --provider aws \
 --plugins velero/velero-plugin-for-aws:v1.0.0 \
 --bucket velero2  \
 --secret-file ./credentials-velero  \
 --use-volume-snapshots=true \
 --backup-location-config region=default,s3ForcePathStyle="true",s3Url=http://minio.velero.svc.cluster.local:9000,publicUrl=http://167.254.204.209:32000  \
 --image velero/velero:v1.4.0  \
 --snapshot-location-config region="default" \
 --use-restic
