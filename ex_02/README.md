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

Kết quả đo thực tế khi build `cart-service` trên Ubuntu x86_64 runner ([Actions Run #45094410900](https://github.com/HgThuan/JBE_Devopp_SS8/actions/runs/45094410900)):

```text
REPOSITORY                                   TAG            SIZE
cart-service                                 multistage     303MB
cart-service                                 single-stage   962MB
```
*(Trong đó image `multistage` có kích thước giải nén trên đĩa là `303MB`, và kích thước nội dung nén `CONTENT SIZE` chỉ **`88.7MB`** khi lưu trữ/kéo qua registry).*

| Phương pháp | Base Image | Dung lượng Image (`docker images`) | Ghi chú |
| :--- | :--- | :---: | :--- |
| **Single-stage Build** | `eclipse-temurin:17-jdk-alpine` | **962 MB** | Chứa toàn bộ JDK, Gradle cache, mã nguồn thô (`src/`). |
| **Multi-stage Build** | `eclipse-temurin:17-jre-alpine` | **303 MB** (nén **88.7 MB**) | Chỉ chứa JRE + file `app.jar` đã build (**giảm ~68.5% dung lượng**). |

