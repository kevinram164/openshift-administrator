# Lab 5 — Route (TLS), Ingress

**Mục tiêu:** Tạo Route `edge`, redirect HTTP; thử Ingress `networking.k8s.io`; liên hệ DNS tùy chỉnh.

**Thời gian ước lượng:** 30–45 phút.

**Tham chiếu DNS:** [dns/README.md](../../dns/README.md)

## 5.1 Chuẩn bị app

```bash
oc new-project lab-05-$(whoami)
oc new-app --name=demo --image=quay.io/openshift/hello-openshift -l app=demo
oc expose svc/demo --name=demo-http
```

## 5.2 Route edge + redirect

```bash
oc delete route demo-http
oc create route edge demo --service=demo --port=8080-tcp
oc describe route demo
curl -skI "https://$(oc get route demo -o jsonpath='{.spec.host}')/"
```

Quan sát: `TLS Termination: edge`, `Insecure Edge Termination Policy` (có thể chỉnh `Redirect`).

## 5.3 Hostname tùy chỉnh (nếu DNS đã chuẩn)

```bash
oc patch route demo -p '{"spec":{"host":"demo-lab05.apps.<cluster-domain>"}}'
# hoặc host *.mbfs.vn nếu wildcard đã trỏ Ingress VIP
```

## 5.4 Ingress

```bash
cat <<'EOF' | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ing
  annotations:
    route.openshift.io/termination: edge
spec:
  rules:
    - host: demo-ing-lab05.apps.<cluster-domain>
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: demo
                port:
                  number: 8080
EOF
```

Thay `<cluster-domain>` bằng domain apps thật (ví dụ `ocp01.mbfs.vn`). Kiểm tra:

```bash
oc get ingress
oc describe ingress demo-ing
```

Nếu cần `ingressClassName`:

```bash
oc get ingressclass
```

Thêm `spec.ingressClassName: openshift-default` (hoặc tên class cluster bạn) vào YAML nếu controller yêu cầu.

## 5.5 Dọn dẹp

```bash
oc delete project lab-05-$(whoami)
```

**Kiểm tra đạt:** HTTPS qua Route edge; biết chỗ chỉnh host; tạo được Ingress và thấy status/route liên quan.
