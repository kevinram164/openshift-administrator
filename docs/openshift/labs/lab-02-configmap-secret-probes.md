# Lab 2 — ConfigMap, Secret, biến môi trường, Probe

**Mục tiêu:** Gắn cấu hình và bí mật vào Pod; hiểu liveness/readiness.

**Thời gian ước lượng:** 35–50 phút.

## 2.1 Project

```bash
oc new-project lab-02-$(whoami)
```

## 2.2 ConfigMap (literal + từ file)

```bash
oc create configmap demo-cfg --from-literal=MESSAGE="Hello from CM"
echo "app.properties=color=red" > /tmp/app.properties
oc create configmap demo-files --from-file=/tmp/app.properties
oc get cm
oc describe cm demo-cfg
```

## 2.3 Secret (opaque)

```bash
oc create secret generic demo-secret --from-literal=token=lab-secret-value
oc get secret demo-secret -o yaml
```

## 2.4 Deployment dùng env từ ConfigMap và Secret

Dùng image in số env (ví dụ `busybox` sleep — đơn giản hơn dùng một deployment nhỏ). Hoặc tạo deployment `hello-openshift` và thêm env:

```bash
oc new-app --name=demo --image=quay.io/openshift/hello-openshift -l app=demo
```

Chỉnh deployment thêm env (ví dụ `MESSAGE` nếu app hỗ trợ; với hello-openshift có thể chỉ mount file). **Bài tập:** viết patch hoặc `oc set env` / YAML:

```bash
oc set env deployment/demo --from=configmap/demo-cfg
oc set env deployment/demo --from=secret/demo-secret --prefix=SECRET_
oc rollout status deployment/demo
```

*(Nếu image không đọc các biến đó, mục tiêu lab vẫn đạt nếu bạn thấy env trong `oc describe pod` / `oc get pod -o yaml`.)*

## 2.5 Mount ConfigMap dạng file

Tạo deployment từ `nginxinc/nginx-unprivileged`, mount `demo-files` vào `/usr/share/nginx/html/config/` (hoặc path hợp lệ) bằng `oc set volume` hoặc YAML.

```bash
# Gợi ý: oc set volume deployment/demo --add --name=cfg --type=configmap --configmap-name=demo-files --mount-path=/tmp/cfg
```

Kiểm tra: `oc exec` vào pod, `ls /tmp/cfg`.

## 2.6 Probe (YAML)

Xuất deployment, thêm:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 5
```

`oc apply -f` hoặc `oc edit deployment demo`. Quan sát: `oc describe pod` → Events.

## 2.7 Dọn dẹp

```bash
oc delete project lab-02-$(whoami)
```

**Kiểm tra đạt:** Có ít nhất một ConfigMap + một Secret được reference trong Pod; có probe trong spec deployment.
