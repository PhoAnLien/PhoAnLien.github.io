---
title: "3. Ứng dụng Chat thời gian thực với UDP"
date: 2024-12-22
draft: false
tags: ["Java", "UDP"]
categories: ["Java", "UDP"]
featured_image: "https://source.unsplash.com/random/800x600/?speed,internet"
summary: "Tìm hiểu sự khác biệt giữa TCP và UDP. Hướng dẫn sử dụng DatagramSocket để xây dựng ứng dụng gửi tin nhắn tốc độ cao."
---

UDP (User Datagram Protocol) thường được ví như dịch vụ bưu phẩm thường: bạn gửi thư đi nhưng không chắc chắn 100% nó sẽ đến nơi hoặc đến đúng thứ tự. Tuy nhiên, UDP cực kỳ nhanh vì không cần thiết lập kết nối rườm rà (connectionless).

## 1. Khi nào dùng UDP?
Khác với TCP đảm bảo tin cậy, UDP được chọn cho tốc độ:
- **Video Streaming / VoIP (Zoom, Skype)**: Mất một vài frame hình ảnh không sao, nhưng độ trễ (latency) phải thấp để cuộc gọi mượt mà.
- **Game Online (FPS/MOBA)**: Cập nhật vị trí nhân vật cần tốc độ cao nhất (Real-time).
- **Broadcast/Multicast**: Gửi tin cho nhiều máy trong mạng LAN cùng lúc.

## 2. Các lớp chính trong Java UDP
Trong Java, chúng ta không dùng Stream (`InputStream`/`OutputStream`) mà dùng khái niệm gói tin (`Packet`).

| Lớp | Chức năng |
| :--- | :--- |
| `DatagramSocket` | "Bưu điện" - Nơi gửi và nhận các gói thư. |
| `DatagramPacket` | "Gói bưu kiện" - Chứa dữ liệu (byte array) và địa chỉ người nhận. |

## 3. Ví dụ: Ứng dụng gửi tin nhắn đơn giản

### Sender (Người gửi)
```java
import java.net.*;

public class UDPSender {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket();
        String message = "Hello UDP World!";
        
        // Chuyển chuỗi thành mảng byte
        byte[] buffer = message.getBytes();
        
        // Xác định địa chỉ người nhận (localhost)
        InetAddress receiverAddress = InetAddress.getByName("localhost");
        int receiverPort = 9876;
        
        // Đóng gói
        DatagramPacket packet = new DatagramPacket(
            buffer, 
            buffer.length, 
            receiverAddress, 
            receiverPort
        );
        
        // Gửi đi
        socket.send(packet);
        System.out.println("Đã gửi: " + message);
        socket.close();
    }
}
```

### Receiver (Người nhận)
Người nhận phải mở cổng `9876` để lắng nghe.

```java
import java.net.*;

public class UDPReceiver {
    public static void main(String[] args) throws Exception {
        // Mở cổng 9876
        DatagramSocket serverSocket = new DatagramSocket(9876);
        System.out.println("Đang chờ tin nhắn UDP...");
        
        byte[] receiveBuffer = new byte[1024]; // Bộ đệm 1KB
        
        while(true) {
            // Tạo gói rỗng để hứng dữ liệu
            DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);
            
            // Hàm này sẽ BLOCKED (chờ) cho đến khi có tin nhắn đến
            serverSocket.receive(receivePacket); 
            
            String receivedData = new String(
                receivePacket.getData(), 
                0, 
                receivePacket.getLength()
            );
            
            System.out.println("Nhận được: " + receivedData);
        }
    }
}
```

## 4. Tóm tắt TCP vs UDP
| Tiêu chí | TCP | UDP |
| :--- | :--- | :--- |
| **Kết nối** | Có kết nối (Connection-oriented) | Không kết nối (Connectionless) |
| **Độ tin cậy** | Cao (Có ACK, gửi lại nếu mất) | Thấp (Gửi và quên - Fire & Forget) |
| **Tốc độ** | Chậm hơn (do overhead nhiều) | Nhanh hơn |
| **Dữ liệu** | Dạng dòng (Stream) | Dạng gói (Datagram) |

Tùy vào mục đích sử dụng (cần tốc độ hay cần chính xác) mà ta chọn giao thức phù hợp.
