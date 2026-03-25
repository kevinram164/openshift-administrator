# OpenShift — Giai đoạn 1: `oc`, Project, Deploy, Route, Console

Tài liệu lab thực hành (OCP 4.x). Giả định đã cài **OpenShift CLI (`oc`)** tương thích cluster và có quyền tạo project.

---

## Mục tiêu

- Dùng `oc login`, Project, context.
- Deploy app từ image có sẵn.
- Tạo Service + **Route**, truy cập qua HTTPS.
- Đối chiếu với **Developer Console** (Topology, Routes).

---

## Bước 1 — Đăng nhập và kiểm tra

```powershell
oc version
oc login --server=https://api.<cluster-cua-ban>:6443 -u <user>
```

```powershell
oc whoami
oc config get-contexts
```

**Ý nghĩa:** Xác nhận đúng cluster và user; context trong kubeconfig tương tự `kubectl`.

---

## Bước 2 — Project (namespace làm việc)

```powershell
oc new-project devops-phase1 --display-name="Phase 1 lab"
oc project
oc get projects
```

**Ý nghĩa:** `new-project` tạo namespace (thường kèm metadata OpenShift). `oc project` là project/namespace hiện tại.

---

## Bước 3 — Deploy app từ image có sẵn

```powershell
oc new-app --name=hello --image=quay.io/openshift/hello-openshift -l app=hello
```

Nếu không pull được `hello-openshift`, thử image **nginx chạy non-root** (phù hợp SCC mặc định của OCP):

```bash
oc delete all -l app=hello
oc new-app --name=hello --image=docker.io/nginxinc/nginx-unprivileged:latest -l app=hello
```

**Không nên** dùng `nginxdemos/hello` hoặc image nginx “full” mặc định: chúng thường cần root và ghi `/var/cache/nginx` → trên OpenShift hay gặp `Permission denied` / CrashLoop (xem mục Troubleshooting).

Đợi pod Running:

```powershell
oc get pods -w
```

(Nhấn `Ctrl+C` khi pod đã `Running`.)

---

## Bước 4 — Service và Route

```powershell
oc get svc
oc expose svc/hello
oc get route
```

Xem host và thử truy cập:

```powershell
oc describe route hello
```

Từ máy client (DNS trỏ đúng router OCP):

```powershell
curl -k https://<host-tu-route>
```

**Ý nghĩa:** **Route** là cách expose HTTP(S) đặc trưng OpenShift (Router); TLS edge là pattern hay gặp.

---

## Bước 5 — Tổng quan object trong project

```powershell
oc get all
```

---

## Bước 6 — Dọn lab (tùy chọn)

```powershell
oc delete project devops-phase1
```

---

## Bước 7 — Console (web UI)

1. Mở URL console cluster (thường dạng `https://console-openshift-console.apps.<cluster>`).
2. Chọn **Developer** perspective.
3. Chọn project `devops-phase1` (tạo lại nếu đã xóa).
4. Xem:
   - **Topology** — workload, route.
   - **Workloads** — Pods, Deployments.
   - **Networking → Routes** — hostname, TLS.

---

## Checklist hoàn thành giai đoạn 1

| Việc | Hoàn thành |
|------|-------------|
| `oc login` / `oc whoami` | ☐ |
| `new-project`, `oc project`, `get projects` | ☐ |
| Deploy từ image, pod Running | ☐ |
| `expose svc`, Route truy cập được | ☐ |
| `oc get all` | ☐ |
| Topology / Routes trong Console | ☐ |

---

## Troubleshooting — nginx: `mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)`

**Nguyên nhân:** OpenShift chạy container với **UID không phải root** và chính sách bảo mật (SCC) hạn chế ghi filesystem. Image nginx chuẩn ghi cache dưới `/var/cache/nginx` → lỗi quyền, pod restart liên tục.

**Cách xử lý (lab):**

1. Xóa workload cũ và deploy lại bằng image thiết kế cho non-root, ví dụ:

   ```bash
   oc delete all -l app=hello
   oc new-app --name=hello --image=quay.io/openshift/hello-openshift -l app=hello
   ```

   hoặc `docker.io/nginxinc/nginx-unprivileged:latest` (lắng nghe cổng **8080**; `oc new-app` / Service thường map đúng theo image).

2. Kiểm tra: `oc get pods`, `oc logs <pod>`, `oc describe pod <pod>`.

**Giai đoạn 2** sẽ đi sâu SCC; tạm thời ưu tiên image “OpenShift-friendly” thay vì nới lỏng SCC (`anyuid`, v.v.) trên môi trường thật.

---

## Giai đoạn tiếp theo (tham chiếu)

- Giai đoạn 2: RBAC, ServiceAccount, **SCC** (khi pod bị từ chối hoặc không chạy đúng user/volume).

---

*Tài liệu ghi chú cá nhân — có thể chỉnh host, user, tên project cho đúng môi trường.*
