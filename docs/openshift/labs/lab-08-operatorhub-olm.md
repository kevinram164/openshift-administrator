# Lab 8 — OperatorHub và OLM (cài operator từ catalog)

**Mục tiêu:** Làm quen **OperatorHub** trên Console và luồng **OLM** qua CLI: `PackageManifest`, `OperatorGroup`, `Subscription`, `ClusterServiceVersion` (CSV), kiểm tra operator chạy; gỡ cài có kiểm soát.

**Thời gian ước lượng:** 45–75 phút.

**Quyền:** Cài operator vào **project của bạn** thường cần quyền tạo `OperatorGroup` / `Subscription` trong project đó. Một số operator chỉ hỗ trợ **All namespaces** → cần **cluster-admin** và namespace `openshift-operators`. Lab dưới đây ưu tiên operator cài được kiểu **một namespace** (SingleNamespace).

**Môi trường air-gapped:** Catalog có thể chỉ có mirror nội bộ; nếu không thấy gói như lab, dùng `oc get catalogsource -n openshift-marketplace` và chọn package có trong cluster.

---

## 8.1 Xem catalog có gì (CLI)

```bash
oc get catalogsource -n openshift-marketplace
oc get packagemanifests | head -40
```

Tìm một package phù hợp lab (nhẹ, cài **Single namespace**). Gợi ý tra cứu:

```bash
oc get packagemanifests -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | sort | less
```

Chọn **một** tên package (ví dụ từ community hoặc Red Hat — tùy cluster). Xem kênh mặc định và source:

```bash
PKG=<ten-package-vi-du>
oc get packagemanifest "$PKG" -o yaml | less
```

Ghi lại: `status.defaultChannel`, `status.channels[*].currentCSV`, và trong `status.channels[*].entries` hoặc spec — **installModes** (SingleNamespace vs AllNamespaces).

---

## 8.2 Cài qua Console (OperatorHub)

1. Đăng nhập Console → **Operators** → **OperatorHub**.
2. Tìm operator (hoặc lọc theo Provider / Capability).
3. Bấm **Install**:
   - **Installation mode:** *A specific namespace* → chọn project lab (tạo trước: `oc new-project lab-08-$(whoami)`).
   - **Update channel** / **Approval:** *Automatic* (lab) hoặc *Manual* để xem InstallPlan trước khi duyệt.
4. Sau khi cài: **Operators** → **Installed Operators** → chọn project → xem trạng thái **Succeeded** và tài nguyên do operator tạo (CRD, pod trong project hoặc `openshift-operators`).

**Bài tập:** Chụp/ghi lại: tên operator, channel, namespace cài, và có bao nhiêu pod CSV tạo ra.

---

## 8.3 Cài qua CLI (OperatorGroup + Subscription)

Tạo project lab:

```bash
oc new-project lab-08-$(whoami)
NS=$(oc project -q)
```

**OperatorGroup** (chỉ target namespace lab — SingleNamespace):

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: lab-08-og
  namespace: $NS
spec:
  targetNamespaces:
  - $NS
EOF
```

**Subscription** — thay `PKG`, `CHANNEL`, `SOURCE` theo output bước 8.1:

```bash
PKG=<package-name>
CHANNEL=<channel-vi-du-stable-hoac-fast>
SOURCE=redhat-operators
# Nếu package thuộc community: SOURCE=community-operators
# Nếu certified: SOURCE=certified-operators

cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ${PKG}-sub
  namespace: $NS
spec:
  channel: ${CHANNEL}
  name: ${PKG}
  source: ${SOURCE}
  sourceNamespace: openshift-marketplace
  installPlanApproval: Automatic
EOF
```

Theo dõi:

```bash
oc get subscription -n "$NS" -w
oc get installplan -n "$NS"
oc get csv -n "$NS"
oc get pods -n "$NS"
```

Kỳ vọng: CSV phase **Succeeded**; operator pod(s) **Running**.

---

## 8.4 Đọc CSV và CRD

```bash
oc get csv -n "$NS"
oc describe csv -n "$NS" | less
oc get crd | grep -i <tu-khoa-operator>
```

Trên Console: **Installed Operators** → operator → tab **All instances** / tạo CR mẫu (nếu operator yêu cầu CR để “bật” chức năng).

---

## 8.5 Approval Manual (tuỳ chọn)

Xóa Subscription cũ (nếu muốn làm lại), tạo lại với `installPlanApproval: Manual`, rồi:

```bash
oc get installplan -n "$NS"
oc describe installplan <ten> -n "$NS"
oc patch installplan <ten> -n "$NS" -p '{"spec":{"approved":true}}' --type=merge
```

---

## 8.6 Gỡ cài (dọn OLM)

Thứ tự tham khảo (tùy operator có finalizer hay không):

```bash
# Xóa mọi CR do user tạo (Custom Resource) trước — xem tài liệu operator
oc delete subscription <ten-sub> -n "$NS"
oc delete csv -n "$NS" --all
oc delete operatorgroup lab-08-og -n "$NS"
```

Nếu còn InstallPlan / job: `oc get installplan,ip -n "$NS"` và xóa. CRD cluster-scoped thường **không** xóa tay trừ khi biết rõ — lab production nên đọc doc uninstall của từng operator.

---

## 8.7 Dọn project lab

```bash
oc delete project lab-08-$(whoami)
```

---

## 8.8 Liên hệ với Lab 6

[Lab 6](lab-06-observe-operators-debug.md) tập trung **quan sát** `clusteroperators` và danh sách CSV/Subscription toàn cluster. **Lab 8** là bước **cài / gỡ** một operator cụ thể từ **OperatorHub** (catalog OLM).

**Kiểm tra đạt:** Tự chọn được một package từ `PackageManifest`; cài thành công (Console hoặc YAML); giải thích được vai trò **Subscription → InstallPlan → CSV**; gỡ sạch trong project lab không để lại subscription/csv chờ.
