# Lab 1 — Nền tảng `oc`, Project, Deploy, Scale, Route

**Mục tiêu:** Quen workflow OpenShift: project, label, xem tài nguyên, scale deployment, Route.

**Thời gian ước lượng:** 25–40 phút.

## 1.1 Project và context

```bash
oc new-project lab-01-$(whoami) --display-name="Lab 01"
oc project
oc get projects
```

## 1.2 Deploy từ image (non-root)

```bash
oc new-app --name=demo --image=quay.io/openshift/hello-openshift -l app=demo
oc get pods -w
# Ctrl+C khi Running
oc get all -l app=demo
```

Nếu không pull được image, dùng: `docker.io/nginxinc/nginx-unprivileged:latest` (cổng 8080).

## 1.3 Label và bộ lọc

```bash
oc label deployment/demo tier=lab --overwrite
oc get deploy -l tier=lab
```

## 1.4 Xem chi tiết và log

```bash
oc describe deployment demo
POD=$(oc get pod -l app=demo -o jsonpath='{.items[0].metadata.name}')
oc logs "$POD"
oc logs -f deployment/demo --tail=20
```

## 1.5 Scale

```bash
oc scale deployment/demo --replicas=2
oc get pods -l app=demo
oc scale deployment/demo --replicas=1
```

## 1.6 Route

```bash
oc expose svc/demo
oc get route
curl -skI "https://$(oc get route demo -o jsonpath='{.spec.host}')/"
```

## 1.7 (Tuỳ chọn) Xuất YAML, chỉnh và apply

```bash
oc get deployment demo -o yaml > /tmp/demo-deploy.yaml
# Sửa replicas hoặc annotation nhỏ, rồi:
oc apply -f /tmp/demo-deploy.yaml
```

## 1.8 Dọn dẹp

```bash
oc delete project lab-01-$(whoami)
```

**Kiểm tra đạt:** Tự làm lại từ đầu không nhìn ghi chú: tạo project → deploy → 2 replicas → 1 route → truy cập được URL.
