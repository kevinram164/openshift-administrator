# Lab 3 — ServiceAccount và RBAC trong project

**Mục tiêu:** Tạo ServiceAccount, gán quyền tối thiểu trong namespace, chạy pod với SA đó.

**Thời gian ước lượng:** 30–45 phút.

## 3.1 Project

```bash
oc new-project lab-03-$(whoami)
```

## 3.2 ServiceAccount

```bash
oc create sa demo-sa
oc get sa
```

## 3.3 Role và RoleBinding (trong project)

Ví dụ: chỉ cho phép đọc Pod và Service trong project:

```bash
oc create role demo-reader --verb=get,list --resource=pods,services
oc create rolebinding demo-rb --role=demo-reader --serviceaccount="lab-03-$(whoami):demo-sa"
```

Kiểm tra token (OCP 4 thường dùng projected token):

```bash
oc run curl-test --image=curlimages/curl --restart=Never --serviceaccount=demo-sa -- \
  sleep 3600
# Hoặc dùng oc debug / job — mục tiêu: xác nhận SA có/không gọi API được
```

**Bài tập:** Với user thường, thử `oc auth can-i list pods --as=system:serviceaccount:lab-03-USER:demo-sa -n lab-03-USER`.

## 3.4 Pod chạy với SA tùy chỉnh

```bash
oc new-app --name=demo --image=quay.io/openshift/hello-openshift \
  --service-account=demo-sa -l app=demo
oc get pod -o jsonpath='{.items[0].spec.serviceAccountName}{"\n"}'
```

## 3.5 (Tuỳ chọn) Pull secret

Nếu image từ registry private: tạo `dockercfg` secret, `oc secrets link demo-sa <secret>` hoặc mount vào SA.

## 3.6 Dọn dẹp

```bash
oc delete project lab-03-$(whoami)
```

**Kiểm tra đạt:** Pod chạy với `demo-sa`; RoleBinding gắn role tùy chỉnh; `oc auth can-i` phản ánh đúng kỳ vọng.
