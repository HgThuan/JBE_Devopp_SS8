# Bài 1 (SS08): Sửa lỗi cấu hình chia sẻ Docker Socket (DooD)

## 1. Phân tích nguyên nhân gây lỗi

Thông báo lỗi:
```text
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

### Bản chất vấn đề:
1. **Kiến trúc Client-Server của Docker:** Lệnh `docker build`, `docker info`, `docker ps` chỉ là **Docker CLI (Client)**. Để thực hiện các lệnh này, Client phải gửi yêu cầu tới **Docker Daemon (`dockerd`)** đang chạy ngầm qua tệp Unix socket `/var/run/docker.sock`.
2. **Cô lập container Runner:** Khi chạy GitHub Self-hosted Runner dưới dạng container trên máy chủ Linux, container Runner là một không gian cô lập và **không tự chạy một Docker daemon bên trong** (vì chạy Docker-in-Docker đòi hỏi chế độ `privileged` kém an toàn và không tận dụng được cache của máy host).
3. Do chưa cấu hình bind mount tệp socket `/var/run/docker.sock` từ máy Host vào trong container Runner, khi CI chạy `docker build`, Docker CLI tìm kiếm `/var/run/docker.sock` bên trong container không thấy $\rightarrow$ phát sinh lỗi `Cannot connect to the Docker daemon`.

---

## 2. Giải pháp theo mô hình Docker-outside-of-Docker (DooD)

Chia sẻ (bind mount) tệp `/var/run/docker.sock` của máy Host vào đúng đường dẫn `/var/run/docker.sock` bên trong container Runner:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

Khi đó, mọi lệnh `docker` chạy trong pipeline CI sẽ gọi trực tiếp ra Docker Daemon trên máy Host để build image và tạo các container anh em (sibling containers).

*(Lưu ý bảo mật & phân quyền trên Linux: Nếu container Runner chạy với user không phải `root`, cần thêm `group_add` chứa GID của nhóm `docker` trên máy Host — lấy bằng lệnh `stat -c '%g' /var/run/docker.sock` — để user trong container có quyền đọc/ghi vào socket).*

---

## 3. File `docker-compose.yml` đã chỉnh sửa hoàn chỉnh

Xem chi tiết tại: [`docker-compose.yml`](./docker-compose.yml) và [`.env.example`](./.env.example).

```yaml
version: '3.8'

services:
  github-runner:
    image: myoung34/github-runner:latest
    container_name: quickbite-github-runner
    restart: unless-stopped
    environment:
      REPO_URL: https://github.com/HgThuan/JBE_Devopp_SS8
      RUNNER_NAME: quickbite-linux-runner
      RUNNER_WORKDIR: /tmp/runner/work
      RUNNER_SCOPE: repo
      ACCESS_TOKEN: ${GITHUB_RUNNER_PAT}
      LABELS: self-hosted,linux,x64,quickbite
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - runner_work:/tmp/runner/work
    group_add:
      - "${DOCKER_GID:-999}"

volumes:
  runner_work:
```

---

## 4. Kiểm chứng thực tế

Sau khi áp dụng cấu hình trên và khởi động lại Runner (`docker compose up -d --force-recreate`), kiểm tra lệnh `docker info` và `docker build` bên trong container đã kết nối thành công tới Docker Daemon của máy Host.
