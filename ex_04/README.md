# Bài 4 (SS08): Tích hợp CI/CD tự động Pull Image từ GHCR

## 1. File Workflow `.github/workflows/ci.yml`

Xem file workflow chuẩn ở thư mục gốc repository: [`/.github/workflows/ci.yml`](../.github/workflows/ci.yml)

---

## 2. Giải thích các cấu hình quan trọng

1. **Cấp quyền `permissions` cho `GITHUB_TOKEN`:**
   ```yaml
   permissions:
     contents: read
     packages: read
   ```
   - Mặc định, `GITHUB_TOKEN` có quyền bị giới hạn hoặc thiếu quyền đọc GitHub Packages khi pulling image private/internal. Khai báo `packages: read` cho phép workflow xác thực với GHCR bằng chính `secrets.GITHUB_TOKEN` và `${{ github.actor }}` mà không cần tạo PAT thủ công.

2. **Chuyển đổi tên Owner / Repository sang chữ thường (Lowercase):**
   - Tài khoản (`HgThuan`) và tên repo (`JBE_Devopp_SS8`) chứa chữ hoa, trong khi Docker & GHCR **bắt buộc image reference phải viết thường hoàn toàn**.
   - Khắc phục bằng lệnh `tr '[:upper:]' '[:lower:]'`:
     ```bash
     OWNER_LOWER=$(echo "${{ github.repository_owner }}" | tr '[:upper:]' '[:lower:]')
     REPO_LOWER=$(echo "${{ github.event.repository.name }}" | tr '[:upper:]' '[:lower:]')
     ```

3. **Kéo Image & Verify ở chế độ nền (`-d`):**
   - Workflow chỉ kéo image đã đóng gói sẵn từ GHCR (`docker pull`) và chạy `docker run -d` để kiểm chứng healthcheck, không biên dịch lại mã nguồn Java nên tổng thời gian thực thi rất nhanh (**chỉ 11 giây**, dưới 20 giây theo yêu cầu).

---

## 3. Kết quả chạy Workflow thực tế trên GitHub Actions

- **Link màn hình kết quả Workflow thành công (màu xanh):** https://github.com/HgThuan/JBE_Devopp_SS8/actions/runs/45094686253
- **Thời gian thực thi:** `11 giây` (`2026-11-23T01:27:33Z` $\rightarrow$ `2026-11-23T01:27:44Z`)
- **Log thực tế đã kiểm chứng:**
  ```text
  Status: Downloaded newer image for ghcr.io/hgthuan/payment-service:1.0.0
  Running container in detached mode (-d)...
  Waiting for /health endpoint...
  {"version":"1.0.0","service":"payment-service","status":"UP"}
  Verification PASSED: payment-service container is healthy!
  Started PaymentServiceApplication in 1.449 seconds
  ```

