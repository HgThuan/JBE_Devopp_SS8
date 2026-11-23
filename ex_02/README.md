# Bài 2 (SS08): Hoàn thiện Dockerfile Multi-stage Build (`cart-service`)

## 1. File `Dockerfile` hoàn chỉnh

Xem file cấu hình tại: [`Dockerfile`](./Dockerfile)

```dockerfile
# Stage 1: Build mã nguồn
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./gradlew bootJar --no-daemon

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 2. Giải thích các phần đã điền

1. **`FROM eclipse-temurin:17-jre-alpine`:**
   - Ở Stage 2 (Runtime), ứng dụng Java đã được biên dịch thành file `.jar` nên không còn cần bộ công cụ biên dịch (JDK, `javac`, Gradle cache, mã nguồn thô).
   - Ta sử dụng image **JRE (Java Runtime Environment)** trên nền **Alpine Linux** gọn nhẹ thay vì JDK để giảm mạnh kích thước image và thu hẹp bề mặt tấn công (Attack Surface).
2. **`COPY --from=builder /app/build/libs/*.jar app.jar`:**
   - Sử dụng từ khóa `--from=builder` để chỉ sao chép duy nhất thành phẩm `.jar` từ Stage 1 (`builder`) sang thư mục `/app/app.jar` ở Stage 2.
   - Toàn bộ mã nguồn thô (`src/`), Gradle wrapper, thư mục `.gradle` cache và công cụ build đều bị loại bỏ khỏi image cuối cùng.

---

## 3. So sánh hiệu quả dung lượng thực tế (`docker images`)

*(Đang cập nhật kết quả đo thực tế từ GitHub Actions Linux runner...)*
