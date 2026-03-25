# Lab 7 — NetworkPolicy

**Mục tiêu:** Hạn chế traffic giữa Pod trong namespace (default allow-all → deny + allow có chủ đích).

**Thời gian ước lượng:** 45–60 phút.

**Lưu ý:** Cần CNI hỗ trợ NetworkPolicy (OVN-Kubernetes trên OCP 4 thường có). Nếu cluster tắt NP, chỉ làm phần lý thuyết + `oc explain networkpolicy`.

## 7.1 Kiểm tra

```bash
oc explain networkpolicy.spec
oc get networkpolicy -A
```

## 7.2 Hai app: client và server

```bash
oc new-project lab-07-$(whoami)

oc new-app --name=server --image=quay.io/openshift/hello-openshift -l app=server
oc expose svc/server

oc new-app --name=client --image=curlimages/curl --command -- sleep 3600 -l app=client
```

Lấy URL route server, từ pod client:

```bash
SERVER=$(oc get route server -o jsonpath='{.spec.host}')
oc exec deployment/client -- curl -sk "https://$SERVER/" -o /dev/null -w "%{http_code}\n"
```

(Kỳ vọng: 200 hoặc mã thành công.)

## 7.3 Default deny ingress (chỉ trong namespace)

```bash
cat <<'EOF' | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
    - Ingress
EOF
```

Thử lại curl từ client → có thể fail (tùy implementation và traffic same-namespace).

## 7.4 Cho phép ingress từ pod có label `app=client`

```bash
cat <<'EOF' | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-client
spec:
  podSelector:
    matchLabels:
      app: server
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: client
      ports:
        - protocol: TCP
          port: 8080
EOF
```

Điều chỉnh `port` theo Service target (8080 cho hello-openshift).

Kiểm tra lại curl từ `client` → `server`.

## 7.5 Ghi chép

Ghi lại: trước/sau policy, hành vi thay đổi thế nào; vì sao traffic từ Router (ingress cluster) có thể cần rule riêng nếu test từ ngoài cluster.

## 7.6 Dọn dẹp

```bash
oc delete project lab-07-$(whoami)
```

**Kiểm tra đạt:** Hiểu `podSelector` / `ingress.from`; chứng minh được một luồng bị chặn và một luồng được phép.
