# Install

helm repo add diseq https://diseq.github.io/helm-charts
helm repo update
cat << EOF | helm install csi-rclone diseq/csi-rclone --create-namespace --namespace csi-rclone -f -
EOF

# Example

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
  labels:
    name: pv-demo
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 10Gi
  storageClassName: rclone
  csi:
    driver: csi-rclone
    volumeHandle: data-id
    volumeAttributes:
      remote: "mydrive"
      remotePath: "/bucket/"
      RCLONE_CONFIG_MYDRIVE_TYPE: "s3"
      RCLONE_CONFIG_MYDRIVE_PROVIDER: "other"
      RCLONE_CONFIG_MYDRIVE_ENV_AUTH: "false"
      RCLONE_CONFIG_MYDRIVE_ACCESS_KEY_ID: "<accesskey>"
      RCLONE_CONFIG_MYDRIVE_SECRET_ACCESS_KEY: "<secretkey>"
      RCLONE_CONFIG_MYDRIVE_ENDPOINT: "https://s3.fr-par.scw.cloud"
      RCLONE_CONFIG_MYDRIVE_LOCATION_CONSTRAINT: "fr-par"
      RCLONE_CONFIG_MYDRIVE_ACL: "private"
      RCLONE_CONFIG_MYDRIVE_REGION: "fr-par"
      RCLONE_CACHE_INFO_AGE: "72h"
      RCLONE_CACHE_CHUNK_CLEAN_INTERVAL: "15m"
      RCLONE_DIR_CACHE_TIME: "5s"
      RCLONE_VFS_CACHE_MODE: "writes"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim-demo
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: rclone
  resources:
    requests:
      storage: 10Gi
  volumeName: pv-demo
```


```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  containers:
  - image: ubuntu
    command:
      - "sleep"
      - "604800"
    imagePullPolicy: IfNotPresent
    name: ubuntu
    volumeMounts:
      - mountPath: "/mnt/data"
        name: data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: pv-claim-demo
  restartPolicy: Always
```
