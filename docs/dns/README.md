# DNS — zone `mbfs.vn` và wildcard cho OpenShift Ingress

Tài liệu lưu cấu hình đã thống nhất khi nhiều application dùng tên `*.mbfs.vn` và traffic đi qua **VIP Ingress HAProxy** (OpenShift Router).

## Vấn đề

- Tên kiểu `web-test.mbfs.vn` **không** thuộc zone `apps.ocp01.mbfs.vn` hay `ocp01.mbfs.vn`.
- Chỉ có `db.apps.ocp01.mbfs.vn` và `db.ocp01.mbfs.vn` **không đủ** để phục vụ `*.mbfs.vn`.
- Cần zone authority cho **`mbfs.vn`** với wildcard `*` trỏ cùng VIP Ingress (ví dụ lab: **10.1.34.11**).

## File trong repo

| File | Mục đích |
|------|-----------|
| [`db.mbfs.vn`](db.mbfs.vn) | Zone mẫu — copy lên DNS server (bastion). |

Các zone hiện có trên bastion **giữ nguyên** (không thay thế bằng file này):

- `db.apps.ocp01.mbfs.vn` — `*.apps.ocp01.mbfs.vn`
- `db.ocp01.mbfs.vn` — `*.ocp01.mbfs.vn`
- `db.reverse.10.1.34` — PTR

## Triển khai trên bastion

1. Copy zone file:

   ```bash
   cp docs/dns/db.mbfs.vn /etc/named/zones/db.mbfs.vn
   chown root:named /etc/named/zones/db.mbfs.vn
   chmod 640 /etc/named/zones/db.mbfs.vn
   ```

2. Thêm vào **`/etc/named.conf`** (hoặc file được `include`):

   ```text
   zone "mbfs.vn" {
       type master;
       file "/etc/named/zones/db.mbfs.vn";
   };
   ```

3. Kiểm tra và nạp lại:

   ```bash
   named-checkzone mbfs.vn /etc/named/zones/db.mbfs.vn
   rndc reload
   ```

4. Kiểm tra resolver (thay `@` bằng IP NS nếu cần):

   ```bash
   dig +short web-test.mbfs.vn @10.1.34.2
   dig +short random-name.mbfs.vn @10.1.34.2
   ```

   Kỳ vọng: **10.1.34.11** (Ingress VIP trong lab).

## Phía OpenShift

- Wildcard DNS **chỉ** đưa traffic tới router; mỗi hostname vẫn cần **Route** (hoặc Ingress) với `host` khớp.
- VIP Ingress phải khớp **HAProxy** (`listen ingress-80` / `ingress-443` bind **10.1.34.11** trong lab).

## Lưu ý BIND / vận hành

- Wildcard `*.mbfs.vn` chỉ khớp **một nhãn**: `app.mbfs.vn` có; `a.b.mbfs.vn` cần zone con / wildcard khác nếu sau này có mô hình đó.
- Bản ghi **cụ thể** luôn **ưu tiên** hơn wildcard (MX, TXT, `@`, v.v.).
- **`@` (apex `mbfs.vn`)**: không được wildcard `*` cover — thêm `IN A` (hoặc record khác) nếu cần.
- **Serial** trong SOA: tăng mỗi lần chỉnh zone.
- **SOA RNAME**: thay `contact.ocp01.mbfs.vn.` bằng email chuẩn SOA (ký tự `@` trong email thành `.` trong zone).

## Xung đột DNS cũ

Nếu `web-test.mbfs.vn` (hoặc `*.mbfs.vn`) từng trỏ IP khác (ví dụ 10.1.23.10), gỡ hoặc sửa tại **mọi** nguồn authority / forwarder để chỉ còn một đáp án nhất quán với zone `mbfs.vn` mới.
