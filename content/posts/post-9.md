---
title: "9. Bảo mật kết nối mạng: SSL/TLS Socket trong Java"
date: 2024-12-28
draft: false
tags: ["Java", "Security", "SSL"]
categories: ["Java", "Security"]
featured_image: "https://source.unsplash.com/random/800x600/?lock,security"
summary: "Tại sao HTTP lại không an toàn? Hướng dẫn nâng cấp ứng dụng Socket thường thành SSLSocket được mã hóa (Giống HTTPS)."
---

Khi bạn gửi mật khẩu qua Socket thường, nó được truyền dưới dạng văn bản (Clear Text). Hacker ngồi ở giữa (Man-in-the-Middle) có thể đọc được tất cả. Để bảo mật, ta cần mã hóa đường truyền bằng **SSL/TLS**.

## 1. SSL/TLS là gì?
- **SSL (Secure Sockets Layer)**: Công nghệ mã hóa cũ.
- **TLS (Transport Layer Security)**: Bản nâng cấp an toàn hơn của SSL.
- Java cung cấp lớp `SSLSocket` và `SSLServerSocket` để hỗ trợ việc này.

## 2. Quy trình mã hóa
Để chạy được SSL, Server cần một "Chứng minh thư" kỹ thuật số gọi là **Keystore** (chứa Private Key và Public Certificate).

### Tạo Keystore (Dùng lệnh keytool)
Mở terminal và chạy lệnh sau để tạo một file chứng chỉ tự ký (Self-signed):
```bash
keytool -genkey -keystore mykeystore.jks -keyalg RSA
```
*(Nhập mật khẩu là "password" cho dễ nhớ)*

## 3. Code SSL Server
Khác với Socket thường, ta cần nạp Keystore trước khi mở cổng.

```java
import javax.net.ssl.*;
import java.io.*;
import java.security.KeyStore;

public class SecureServer {
    public static void main(String[] args) throws Exception {
        // 1. Load Keystore
        char[] password = "password".toCharArray();
        KeyStore ks = KeyStore.getInstance("JKS");
        ks.load(new FileInputStream("mykeystore.jks"), password);

        // 2. Tạo KeyManagerFactory
        KeyManagerFactory kmf = KeyManagerFactory.getInstance("SunX509");
        kmf.init(ks, password);

        // 3. Khởi tạo SSLContext
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(kmf.getKeyManagers(), null, null);

        // 4. Tạo SSLServerSocket
        SSLServerSocketFactory ssf = sslContext.getServerSocketFactory();
        SSLServerSocket serverSocket = (SSLServerSocket) ssf.createServerSocket(8443);
        
        System.out.println("Secure Server listening on port 8443...");

        // 5. Chấp nhận kết nối như bình thường
        while(true) {
            SSLSocket socket = (SSLSocket) serverSocket.accept();
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            out.println("Hello Secure World!");
            socket.close();
        }
    }
}
```

## 4. Code SSL Client
Client cần tin tưởng chứng chỉ của Server (TrustStore). Trong môi trường Develop, ta thường dùng cờ buộc tin tưởng tất cả (Trust All) hoặc import chứng chỉ vào Java TrustStore.

```java
import javax.net.ssl.*;
import java.io.*;

public class SecureClient {
    public static void main(String[] args) throws Exception {
        // Để đơn giản demo, ta dùng socket factory mặc định
        // (Lưu ý trong thực tế cần cấu hình TrustStore nếu dùng Self-signed Cert)
        System.setProperty("javax.net.ssl.trustStore", "mykeystore.jks");
        System.setProperty("javax.net.ssl.trustStorePassword", "password");

        SSLSocketFactory ssf = (SSLSocketFactory) SSLSocketFactory.getDefault();
        SSLSocket socket = (SSLSocket) ssf.createSocket("localhost", 8443);

        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        System.out.println("Server nói: " + in.readLine());
        
        socket.close();
    }
}
```

## 5. Kết luận
Bảo mật không phải là tính năng "có cho vui", nó là yêu cầu bắt buộc. Với Java `SSLSocket`, bạn có thể nâng cấp bất kỳ ứng dụng Chat/Game nào của mình lên chuẩn bảo mật công nghiệp, đảm bảo dữ liệu người dùng luôn an toàn.
