# Lab 4 — StorageClass và PVC

**Mục tiêu:** Xác định StorageClass mặc định, tạo PVC, gắn vào Pod, kiểm tra dữ liệu giữ khi xóa Pod.

**Thời gian ước lượng:** 30–45 phút.

**Lưu ý:** Cần cluster có **dynamic provisioner** hoặc PV tĩnh; nếu không có SC, lab có thể chỉ dừng ở bước quan sát.

## 4.1 Quan sát storage

```bash
oc get storageclass
oc get pv
oc describe storageclass
```

Ghi lại **default SC** (annotation `storageclass.kubernetes.io/is-default-class`).

## 4.2 Project

```bash
oc new-project lab-04-$(whoami)
```

## 4.3 PVC

```bash
cat <<'EOF' | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  # storageClassName: <ten-sc>   # bỏ comment nếu không có default
EOF

oc get pvc -w
```

## 4.4 Pod gắn PVC

```bash
oc new-app --name=demo --docker-image=nginxinc/nginx-unprivileged -l app=demo
oc set volume deployment/demo --add --name=data --type=pvc --claim-name=demo-pvc --mount-path=/usr/share/nginx/html
```

Hoặc viết Pod/Deployment YAML có `volumes` + `volumeMounts`.

## 4.5 Kiểm tra

```bash
oc exec deployment/demo -- sh -c 'echo lab > /usr/share/nginx/html/from-pvc.txt'
oc exec deployment/demo -- cat /usr/share/nginx/html/from-pvc.txt
oc delete pod -l app=demo
# Đợi pod mới Running
oc exec deployment/demo -- cat /usr/share/nginx/html/from-pvc.txt
```

## 4.6 Dọn dẹp

```bash
oc delete project lab-04-$(whoami)
```

**Kiểm tra đạt:** PVC `Bound`; file trên volume còn sau khi pod tái tạo.
