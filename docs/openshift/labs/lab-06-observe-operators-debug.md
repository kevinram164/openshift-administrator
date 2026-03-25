# Lab 6 — Quan sát cluster, Operators, gỡ lỗi

**Mục tiêu:** Đọc trạng thái control plane, OperatorHub; dùng `oc debug`, events, logs.

**Thời gian ước lượng:** 40–60 phút.

**Quyền:** Một số lệnh cần **cluster-admin** (clusteroperators, namespace openshift-* ).

## 6.1 Cluster Operators (admin)

```bash
oc get clusteroperators
oc get clusteroperator openshift-apiserver -o yaml | less
```

Ghi nhận: `Available`, `Progressing`, `Degraded`.

## 6.2 Nodes

```bash
oc get nodes -o wide
oc describe node <tên-node>
```

## 6.3 Operators / OLM (đọc)

```bash
oc get operators -A 2>/dev/null || true
oc get csv -A | head -30
oc get subscriptions -A | head -20
```

Trên **Console**: OperatorHub → chọn một operator → xem tab **YAML** Subscription (không bắt buộc cài nếu policy không cho). **Lab cài đặt chi tiết từ Hub / OLM:** [lab-08-operatorhub-olm.md](lab-08-operatorhub-olm.md).

## 6.4 Events và mô tả object

```bash
oc new-project lab-06-$(whoami)
oc new-app --name=demo --image=quay.io/openshift/hello-openshift -l app=demo
oc get events --sort-by=.lastTimestamp -A | tail -30
oc describe pod -l app=demo
```

## 6.5 Debug pod

```bash
oc debug deployment/demo -- bash
# hoặc
oc debug -h
```

Thoát shell debug khi xong.

## 6.6 (Admin) must-gather — chỉ khi được phép

```bash
# oc adm must-gather --dest-dir=/tmp/mg-$(date +%Y%m%d)
```

Không chạy trên production giờ cao điểm nếu không cần; lab chỉ đọc tài liệu lệnh.

## 6.7 Dọn dẹp

```bash
oc delete project lab-06-$(whoami)
```

**Kiểm tra đạt:** Giải thích được ý nghĩa một dòng trong `oc get co`; biết xem CSV/subscription; dùng được `oc describe` + `oc debug` cho workload.
