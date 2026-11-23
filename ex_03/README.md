# Bài 3 (SS08): Thiết lập quy trình Build và Push Image lên GHCR (`payment-service`)

## 1. Phân tích lỗi `denied: requested access to the resource is denied`

Nguyên nhân thường gặp:
1. Đăng nhập Docker CLI bằng mật khẩu GitHub hoặc PAT chưa được cấp quyền `write:packages`.
2. Gắn tag image sai quy tắc namespace của GitHub Container Registry (`ghcr.io/<username_viết_thường>/<tên_image>:<tag>`), hoặc chứa ký tự **chữ hoa** (Docker/GHCR bắt buộc toàn bộ image reference phải là **chữ thường** — ví dụ `HgThuan` $\rightarrow$ `hgthuan`).

---

## 2. Các bước thực hiện & Câu lệnh CLI chuẩn

Xem file tóm tắt lệnh CLI tại: [`ghcr-commands.txt`](./ghcr-commands.txt).

### Bước 1: Tạo Personal Access Token (PAT) trên GitHub
- Truy cập: **GitHub Settings $\rightarrow$ Developer settings $\rightarrow$ Personal access tokens $\rightarrow$ Tokens (classic)**.
- Chọn **Generate new token (classic)** và cấp quyền tối thiểu cần thiết:
  - `write:packages` (tự động bao gồm `read:packages` và `repo` cần thiết để upload/download package).

### Bước 2: Đăng nhập Docker CLI vào GHCR bằng PAT
```bash
export CR_PAT="<YOUR_GITHUB_PAT>"
echo "$CR_PAT" | docker login ghcr.io -u HgThuan --password-stdin
```

### Bước 3: Build image và gắn tag chuẩn phiên bản `1.0.0` (viết thường toàn bộ namespace)
```bash
docker build -t ghcr.io/hgthuan/payment-service:1.0.0 ./ex_03/payment-service
```
*(Hoặc build trước rồi `docker tag`:)*
```bash
docker build -t payment-service:1.0.0 ./ex_03/payment-service
docker tag payment-service:1.0.0 ghcr.io/hgthuan/payment-service:1.0.0
```

### Bước 4: Push image lên GHCR
```bash
docker push ghcr.io/hgthuan/payment-service:1.0.0
```

---

## 3. Liên kết Package trên GitHub & Kết quả Push thực tế

- **Trang GitHub Packages cá nhân:** https://github.com/HgThuan?tab=packages
- **Package Repository:** https://github.com/HgThuan/JBE_Devopp_SS8/pkgs/container/payment-service
- **Image Reference:** `ghcr.io/hgthuan/payment-service:1.0.0`
- **Kết quả Push thực tế ([Actions Run #45094410900](https://github.com/HgThuan/JBE_Devopp_SS8/actions/runs/45094410900)):**
  ```text
  Login Succeeded
  1.0.0: digest: sha256:ae3f9ced0caf01f40d6f2d4e475d658331d2441f20d4259b1aa52f63a4fc8dd2 size: 856
  IMAGE                                   ID             DISK USAGE   CONTENT SIZE
  ghcr.io/hgthuan/payment-service:1.0.0   ae3f9ced0caf        303MB         88.7MB
  ```

