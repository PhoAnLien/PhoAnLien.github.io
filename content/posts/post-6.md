---
title: "6. Java NIO: Lập trình mạng hiệu năng cao với Non-blocking I/O"
date: 2024-12-25
draft: false
tags: ["Java", "NIO", "Network"]
categories: ["Java", "Network"]
featured_image: "https://source.unsplash.com/random/800x600/?server,performance"
summary: "Vượt qua giới hạn của IO truyền thống. Tìm hiểu về Buffer, Channel và Selector để xây dựng Server có khả năng xử lý hàng chục ngàn kết nối đồng thời (C10k problem)."
---

Trong các bài trước, chúng ta đã dùng `java.io` (Blocking I/O). Nhược điểm lớn nhất của nó là mỗi Client kết nối cần một Thread riêng biệt. Khi số lượng Client lên tới hàng ngàn, Server sẽ bị quá tải bộ nhớ. `Java NIO` (New I/O) ra đời để giải quyết vấn đề này.

## 1. Ba thành phần cốt lõi của NIO

### Channel (Kênh)
Giống như "đường ống" trong IO truyền thống, nhưng Channel có thể đọc và ghi hai chiều (bi-directional) và hỗ trợ thao tác bất đồng bộ.
- `SocketChannel`: Dùng cho TCP Client.
- `ServerSocketChannel`: Dùng cho TCP Server.
- `DatagramChannel`: Dùng cho UDP.

### Buffer (Bộ đệm)
Trong NIO, dữ liệu luôn được đọc vào hoặc ghi từ một `Buffer`. Nó giống như một khay chứa dữ liệu tạm thời.
```java
// Tạo buffer với dung lượng 1024 byte
ByteBuffer buffer = ByteBuffer.allocate(1024);
```

### Selector (Bộ chọn)
Đây là "vũ khí bí mật" của NIO. Một Selector có thể giám sát nhiều Channel cùng lúc. Nó cho phép **một Thread duy nhất** xử lý hàng ngàn kết nối.

## 2. Mô hình hoạt động của Selector
Thay vì chặn (block) chờ dữ liệu, Selector sẽ hỏi: *"Có Channel nào đang sẵn sàng để đọc/ghi không?"*.

1.  Đăng ký các Channel vào Selector với sự kiện quan tâm (`OP_ACCEPT`, `OP_READ`, ...).
2.  Selector chạy vòng lặp vô tận, chờ sự kiện xảy ra.
3.  Khi có sự kiện, Selector trả về danh sách các `SelectionKey`.
4.  Xử lý dữ liệu và quay lại chờ đợi.

## 3. Ví dụ: NIO Server đơn giản

```java
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class NioServer {
    public static void main(String[] args) throws Exception {
        // 1. Mở Selector
        Selector selector = Selector.open();

        // 2. Mở ServerSocketChannel và lắng nghe cổng 5000
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.bind(new InetSocketAddress(5000));
        serverSocket.configureBlocking(false); // Quan trọng: Chế độ Non-blocking

        // 3. Đăng ký nhận sự kiện kết nối mới (Accept)
        serverSocket.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("NIO Server started on port 5000...");

        while (true) {
            // Chờ sự kiện (Block cho đến khi có ít nhất 1 channel sẵn sàng)
            selector.select();

            // Lấy danh sách các key đã sẵn sàng
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> iter = selectedKeys.iterator();

            while (iter.hasNext()) {
                SelectionKey key = iter.next();

                if (key.isAcceptable()) {
                    // Xử lý kết nối mới
                    SocketChannel client = serverSocket.accept();
                    client.configureBlocking(false);
                    client.register(selector, SelectionKey.OP_READ);
                    System.out.println("New connection: " + client.getRemoteAddress());
                }

                if (key.isReadable()) {
                    // Xử lý đọc dữ liệu
                    SocketChannel client = (SocketChannel) key.channel();
                    ByteBuffer buffer = ByteBuffer.allocate(256);
                    int bytesRead = client.read(buffer);

                    if (bytesRead == -1) {
                        client.close();
                    } else {
                        String msg = new String(buffer.array()).trim();
                        System.out.println("Received: " + msg);
                    }
                }
                iter.remove(); // Xóa key đã xử lý
            }
        }
    }
}
```

## 4. Tại sao NIO nhanh hơn?
NIO cho phép xử lý I/O không chặn dòng thực thi (Non-blocking). Nó giúp tiết kiệm tài nguyên hệ thống (ít Thread hơn) và giảm chi phí chuyển ngữ cảnh (Context Switching), đặc biệt hiệu quả cho các ứng dụng chat, server game hoặc các hệ thống thời gian thực tải cao.
