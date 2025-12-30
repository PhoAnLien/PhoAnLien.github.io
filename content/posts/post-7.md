---
title: "7. Tự xây dựng HTTP Server bằng Java Socket"
date: 2024-12-26
draft: false
tags: ["Java", "HTTP", "Web Server"]
categories: ["Java", "Network"]
featured_image: "https://source.unsplash.com/random/800x600/?web,http"
summary: "Hiểu sâu về giao thức HTTP bằng cách tự tay viết một Web Server đơn giản. Phân tích cấu trúc HTTP Request và Response."
---

Chúng ta thường dùng Apache, Nginx hay Tomcat để chạy web. Nhưng bạn có bao giờ thắc mắc bên dưới chúng hoạt động thế nào không? Thực chất, Web Server cũng chỉ là một ứng dụng Socket Server giao tiếp theo chuẩn **HTTP** (HyperText Transfer Protocol).

## 1. Cấu trúc giao thức HTTP
HTTP là giao thức dựa trên văn bản (text-based).

### HTTP Request (Client gửi lên)
```text
GET /index.html HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0...
(Dòng trống)
(Body nếu có)
```

### HTTP Response (Server trả về)
```text
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 100
(Dòng trống)
<html>...</html>
```

## 2. Xây dựng SimpleWebServer
Chúng ta sẽ viết một Server lắng nghe ở cổng 8080. Khi người dùng truy cập trình duyệt, Server sẽ trả về một trang HTML chào mừng.

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Date;

public class SimpleWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(8080);
        System.out.println("Web Server listening on port 8080...");

        while (true) {
            try (Socket socket = server.accept()) {
                // 1. Đọc Request (In ra console để xem)
                InputStreamReader isr = new InputStreamReader(socket.getInputStream());
                BufferedReader reader = new BufferedReader(isr);
                String line = reader.readLine(); // Chỉ đọc dòng đầu: GET / HTTP/1.1
                System.out.println("Request: " + line);

                // 2. Chuẩn bị nội dung trả về
                String html = "<html><body><h1>Hello from Java Web Server!</h1><p>Time: " + new Date() + "</p></body></html>";
                
                // 3. Gửi HTTP Response Header
                PrintWriter out = new PrintWriter(socket.getOutputStream());
                out.println("HTTP/1.1 200 OK");
                out.println("Content-Type: text/html; charset=UTF-8");
                out.println("Content-Length: " + html.length());
                out.println(); // Dòng trống bắt buộc phân cách Header và Body
                
                // 4. Gửi Body (Nội dung HTML)
                out.println(html);
                out.flush(); // Đẩy dữ liệu đi ngay lập tức
                
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 3. Chạy thử
1.  Chạy chương trình Java trên.
2.  Mở trình duyệt truy cập `http://localhost:8080`.
3.  Bạn sẽ thấy dòng chữ **"Hello from Java Web Server!"** cùng thời gian hiện tại.

## 4. Bài học rút ra
Mọi Web Server phức tạp đều bắt đầu từ nguyên lý này:
1.  Lắng nghe kết nối TCP (Socket).
2.  Đọc và phân tích chuỗi văn bản theo quy chuẩn HTTP.
3.  Xử lý logic (Route, Controller...).
4.  Gửi lại chuỗi văn bản phản hồi.

Hiểu điều này giúp bạn debug lỗi mạng tốt hơn khi làm việc với REST API sau này.
