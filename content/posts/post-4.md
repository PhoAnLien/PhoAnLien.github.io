---
title: "4. Multithreading: Xử lý đa luồng cho Server"
date: 2024-12-23
draft: false
tags: ["Java", "Multithreading"]
categories: ["Java", "Multithreading"]
featured_image: "https://source.unsplash.com/random/800x600/?cpu,threads"
summary: "Tại sao Server đơn luồng lại chậm? Giải pháp sử dụng Thread và Thread Pool để xây dựng Server phục vụ hàng ngàn Client cùng lúc."
---

Hãy tưởng tượng bạn là nhân viên duy nhất tại một quầy vé. Nếu bạn mất 5 phút để phục vụ khách hàng A, thì khách hàng B phí sau phải chờ 5 phút mới đến lượt. Đây chính là vấn đề của **Single-threaded Server** (Server đơn luồng).

Để giải quyết, Server cần khả năng **Multithreading** (Đa luồng) - giống như việc mở thêm nhiều quầy vé để phục vụ song song.

## 1. Mô hình Thread-per-Client
Mỗi khi server chấp nhận một kết nối mới (`accept()`), thay vì xử lý ngay tại luồng chính (Main Thread), nó sẽ tạo một `Thread` mới riêng biệt cho Client đó.

```java
// Main Thread: Chỉ lo mở cửa đón khách
while (true) {
    Socket clientSocket = serverSocket.accept();
    System.out.println("New Client Connected!");

    // Tạo luồng mới để phục vụ khách này
    Thread t = new Thread(new ClientHandler(clientSocket));
    t.start();
}
```

Code xử lý logic (Class `ClientHandler`):
```java
class ClientHandler implements Runnable {
    private Socket socket;

    public ClientHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            // Xử lý gửi nhận tin nhắn dài dòng ở đây
            // ...
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 2. Vấn đề của việc tạo quá nhiều Thread
Tạo một Thread trong Java khá tốn kém tài nguyên (RAM, CPU Context Switching). Nếu có 10.000 user kết nối cùng lúc, server có thể bị **Crash** do hết bộ nhớ (Out of Memory).

## 3. Giải pháp: Thread Pool (Hồ chứa luồng)
Thay vì tạo vô tội vạ, ta tạo sẵn một số lượng luồng cố định (ví dụ: 100 luồng).
- Client 1-100 được phục vụ ngay.
- Client 101 sẽ phải chờ trong hàng đợi (Queue) cho đến khi có một luồng rảnh rỗi.

Java hỗ trợ việc này qua `ExecutorService`.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MultiThreadedServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(5000);
        
        // Tạo Thread Pool với tối đa 10 nhân viên
        ExecutorService pool = Executors.newFixedThreadPool(10);
        
        while (true) {
            Socket client = serverSocket.accept();
            // Giao việc cho Pool, Pool sẽ tự phân phối luồng
            pool.execute(new ClientHandler(client));
        }
    }
}
```

Sử dụng Thread Pool giúp Server hoạt động ổn định, kiểm soát được tải và không bị sập khi traffic tăng đột biến. Đây là mô hình chuẩn cho các server Java truyền thống.
