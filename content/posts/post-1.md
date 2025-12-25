---
title: "1. Tổng quan về Lập trình mạng trong Java"
date: 2024-12-20
draft: false
tags: ["Java", "Network"]
categories: ["Java", "Network"]
featured_image: "https://source.unsplash.com/random/800x600/?technology,network"
summary: "Khám phá nền tảng của lập trình mạng với Java: Hiểu về IP, Protocol, Port, Socket và cách xây dựng ứng dụng Client-Server đầu tiên."
---

Lập trình mạng (Network Programming) là nền tảng của các hệ thống phân tán và ứng dụng internet hiện đại. Trong bài viết này, chúng ta sẽ đi sâu vào cách Java hỗ trợ giao tiếp mạng thông qua gói `java.net`.

## 1. Các khái niệm cơ bản
Trước khi bắt đầu code, hãy nắm vững các thuật ngữ quan trọng:

### IP Address (IPv4/IPv6)
Địa chỉ IP giống như số nhà của bạn. Nó là định danh duy nhất của một thiết bị trên mạng để các thiết bị khác có thể tìm thấy.
- **IPv4**: Ví dụ `192.168.1.1` (32-bit).
- **IPv6**: Ví dụ `2001:0db8:85a3...` (128-bit) ra đời do sự cạn kiệt của IPv4.

### Port (Cổng)
Nếu IP là địa chỉ căn chung cư, thì Port chính là số phòng. Một máy tính có thể chạy nhiều ứng dụng mạng cùng lúc (Web, Mail, Chat). Port giúp Hệ điều hành chuyển dữ liệu đến đúng ứng dụng.
- **Port 80/443**: Web (HTTP/HTTPS).
- **Port 21**: FTP.
- **Port 0-1023**: Cổng dành riêng cho hệ thống (System Ports).

### Protocol (Giao thức)
Là bộ quy tắc giao tiếp. Hai giao thức phổ biến nhất ở tầng vận chuyển là:
- **TCP (Transmission Control Protocol)**: Tin cậy, đảm bảo dữ liệu đến nơi và đúng thứ tự (Dùng cho Web, Email).
- **UDP (User Datagram Protocol)**: Nhanh, không đảm bảo, chấp nhận mất gói tin (Dùng cho Game, Video Streaming).

### Socket
Socket là một điểm cuối (endpoint) trong liên kết truyền thông hai chiều giữa hai chương trình đang chạy trên mạng.
> **Socket = IP Address + Port Number**

---

## 2. Java Networking API
Java cung cấp một bộ thư viện phong phú trong gói `java.net`:

| Class | Mô tả |
| :--- | :--- |
| `InetAddress` | Đại diện cho địa chỉ IP (cả tên miền và số). |
| `URL`, `URLConnection` | Làm việc với các tài nguyên web chuẩn (HTTP). |
| `Socket`, `ServerSocket` | Dùng cho giao tiếp TCP. |
| `DatagramSocket` | Dùng cho giao tiếp UDP. |

---

## 3. Ví dụ thực hành: Lấy địa chỉ IP của một Domain
Dưới đây là đoạn code đơn giản để phân giải tên miền (DNS Lookup) thành địa chỉ IP, bước đầu tiên để hiểu cách các máy tính "tìm thấy nhau".

```java
import java.net.InetAddress;
import java.net.UnknownHostException;

public class IPFinder {
    public static void main(String[] args) {
        String domain = "www.google.com";
        try {
            // Lấy địa chỉ IP từ tên miền
            InetAddress address = InetAddress.getByName(domain);
            
            System.out.println("---------------------------------");
            System.out.println("Domain Name: " + domain);
            System.out.println("IP Address:  " + address.getHostAddress());
            System.out.println("Host Name:   " + address.getHostName());
            System.out.println("---------------------------------");
            
            // Kiểm tra xem có kết nối được không (ICMP Ping)
            if (address.isReachable(2000)) {
                System.out.println("Status: Online (Reachable)");
            } else {
                System.out.println("Status: Offline or Blocked");
            }
            
        } catch (UnknownHostException e) {
            System.err.println("Không thể tìm thấy IP cho domain: " + domain);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Kết quả chạy chương trình:
```text
---------------------------------
Domain Name: www.google.com
IP Address:  142.250.191.68
Host Name:   www.google.com
---------------------------------
Status: Online (Reachable)
```

## 4. Tại sao chọn Java cho Lập trình mạng?
1.  **Cross-platform**: Code một lần, chạy mọi nơi (Linux server, Windows client).
2.  **Mạnh mẽ (Robust)**: Tự động quản lý bộ nhớ, xử lý ngoại lệ tốt.
3.  **Multithreading**: Java hỗ trợ đa luồng cực tốt, điều kiện tiên quyết cho các Server hiệu năng cao xử lý hàng ngàn kết nối cùng lúc.

Trong các bài tiếp theo, chúng ta sẽ bắt tay vào xây dựng ứng dụng Chat Client-Server thực tế với TCP Socket.
