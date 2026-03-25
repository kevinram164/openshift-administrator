# Lab thực hành OpenShift (OCP 4.x)

Chuỗi bài tập ngắn, làm trên cluster thật. Giả định bạn đã quen Kubernetes và có `oc` + quyền tạo project (và một số bài cần quyền rộng hơn — ghi rõ trong từng lab).

## Chuẩn bị

- Đăng nhập: `oc login ...`
- Namespace lab (đổi tên nếu muốn): `oc new-project lab-ocp-<tên-bạn>` hoặc dùng một project cố định và **dọn tài nguyên** sau mỗi bài.
- Ghi chú giai đoạn 1 cơ bản: [phase-1-oc-project-deploy-route-console.md](../phase-1-oc-project-deploy-route-console.md)
- DNS / hostname tùy biến: [dns/README.md](../../dns/README.md)

## Danh sách lab

| Lab | Nội dung | File |
|-----|----------|------|
| 1 | `oc`, project, deploy, scale, route | [lab-01-oc-fundamentals.md](lab-01-oc-fundamentals.md) |
| 2 | ConfigMap, Secret, probe | [lab-02-configmap-secret-probes.md](lab-02-configmap-secret-probes.md) |
| 3 | ServiceAccount, RBAC trong project | [lab-03-rbac-serviceaccount.md](lab-03-rbac-serviceaccount.md) |
| 4 | PVC, StorageClass | [lab-04-storage-pvc.md](lab-04-storage-pvc.md) |
| 5 | Route TLS, Ingress | [lab-05-route-tls-ingress.md](lab-05-route-tls-ingress.md) |
| 6 | Quan sát cluster, Operators, troubleshooting | [lab-06-observe-operators-debug.md](lab-06-observe-operators-debug.md) |
| 7 | NetworkPolicy (tuỳ CNI) | [lab-07-networkpolicy.md](lab-07-networkpolicy.md) |

Làm **theo thứ tự** từ 1 → 7 hoặc chọn bài theo nhu cầu; lab sau có thể dùng kỹ năng từ lab trước.

## Sau mỗi bài

- Xóa object thử nghiệm: `oc delete all -l app=...` hoặc `oc delete project ...` nếu project chỉ dùng cho lab.
