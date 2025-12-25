---
title: "2. Tìm hiểu sâu về Socket TCP trong Java"
date: 2024-12-21
draft: false
tags: ["Java", "TCP"]
categories: ["Java", "TCP"]
featured_image: "https://source.unsplash.com/random/800x600/?server,code"
summary: "Hướng dẫn chi tiết về giao thức TCP: Cơ chế bắt tay 3 bước và xây dựng ứng dụng Echo Server hoàn chỉnh bằng Java Socket."
---

TCP (Transmission Control Protocol) là giao thức xương sống của Internet, đảm bảo dữ liệu được truyền đi tin cậy, đúng thứ tự và không bị mất mát. Trong bài này, chúng ta sẽ xây dựng một ứng dụng Client-Server thực tế.

## 1. Cơ chế bắt tay 3 bước (Three-way Handshake)
Trước khi dữ liệu được truyền đi, TCP phải thiết lập kết nối (connection-oriented). Quá trình này gọi là "Bắt tay 3 bước":

1.  **SYN**: Client gửi yêu cầu kết nối ("Tôi muốn nói chuyện!").
2.  **SYN-ACK**: Server đồng ý và gửi lại xác nhận ("OK, tôi nghe đây!").
3.  **ACK**: Client xác nhận lại một lần nữa ("Tốt, bắt đầu nhé!").

Kết nối lúc này được thiết lập (Established).

## 2. Xây dựng Echo Server với Java Socket
Chúng ta sẽ viết một ứng dụng đơn giản: Client gửi một câu chào, Server nhận và gửi trả lại nguyên văn dòng chữ đó (Echo).

### Server Code (`EchoServer.java`)
Server cần lắng nghe tại một cổng cố định (ví dụ: 5000) và chờ Client.

```java
import java.io.*;
import java.net.*;

public class EchoServer {
    public static void main(String[] args) {
        int port = 5000;
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Server đang chạy tại cổng " + port);

            // Chặn (block) cho đến khi có client kết nối
            Socket socket = serverSocket.accept();
            System.out.println("Client đã kết nối!");

            // Tạo luồng Input/Output
            InputStream input = socket.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(input));
            
            OutputStream output = socket.getOutputStream();
            PrintWriter writer = new PrintWriter(output, true);

            String text;
            while ((text = reader.readLine()) != null) {
                System.out.println("Nhận từ Client: " + text);
                writer.println("Server trả lời: " + text); // Gửi ngược lại
                
                if ("bye".equalsIgnoreCase(text)) break;
            }

        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
```

### Client Code (`EchoClient.java`)
Client sẽ kết nối đến `localhost` tại cổng `5000`.

```java
import java.net.*;
import java.io.*;

public class EchoClient {
    public static void main(String[] args) {
        String hostname = "localhost";
        int port = 5000;

        try (Socket socket = new Socket(hostname, port)) {
            // Gửi dữ liệu đi
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            out.println("Xin chào Server!");
            
            // Đọc phản hồi
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            System.out.println(in.readLine());

        } catch (UnknownHostException ex) {
            System.out.println("Không tìm thấy Server: " + ex.getMessage());
        } catch (IOException ex) {
            System.out.println("Lỗi I/O: " + ex.getMessage());
        }
    }
}
```

## 3. Lưu ý quan trọng
- **Đóng Socket**: Luôn đóng Socket (`socket.close()`) trong khối `finally` hoặc dùng `try-with-resources` để tránh rò rỉ tài nguyên (Memory Leak).
- **Luồng dữ liệu**: TCP xử lý dữ liệu dạng dòng (stream), nghĩa là ranh giới giữa các tin nhắn không được bảo toàn mặc định. Bạn cần quy định ký tự kết thúc (như xuống dòng `\n`) để phân biệt các tin nhắn, đó là lý do ta dùng `readLine()` và `println()`.

Ở bài sau, chúng ta sẽ xem xét giao thức "người anh em" của TCP là UDP.
