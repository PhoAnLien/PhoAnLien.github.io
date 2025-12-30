---
title: "8. Java RMI: Gọi phương thức từ xa"
date: 2024-12-27
draft: false
tags: ["Java", "RMI", "RPC"]
categories: ["Java", "Network"]
featured_image: "https://source.unsplash.com/random/800x600/?connection,remote"
summary: "Tìm hiểu về công nghệ RMI, cách gọi một hàm nằm trên máy tính khác như thể nó đang ở máy mình. Nền tảng của các hệ thống phân tán."
---

Khi làm việc với mạng, đôi khi Socket là quá sơ cấp (low-level). Bạn muốn gọi hàm `tinhTong(a, b)` trên Server và nhận kết quả về mà không muốn lo lắng về việc đóng gói byte hay luồng dữ liệu. **Java RMI (Remote Method Invocation)** chính là giải pháp.

## 1. Cơ chế hoạt động
RMI cho phép một đối tượng trên máy A (Client) gọi phương thức của một đối tượng trên máy B (Server).
- **Stub (Client-side)**: Đại diện giả của Server ở phía Client.
- **Skeleton (Server-side)**: Nhận yêu cầu từ Stub, gọi hàm thực sự và trả kết quả.

## 2. Các bước triển khai RMI

### Bước 1: Định nghĩa Interface (Chung cho Client & Server)
Interface phải kế thừa `Remote` và các hàm phải ném ra `RemoteException`.

```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface Calculator extends Remote {
    int add(int a, int b) throws RemoteException;
}
```

### Bước 2: Cài đặt Server (Implementation)
Class thực thi logic phải kế thừa `UnicastRemoteObject`.

```java
import java.rmi.server.UnicastRemoteObject;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class CalculatorServer extends UnicastRemoteObject implements Calculator {
    
    protected CalculatorServer() throws RemoteException {
        super();
    }

    @Override
    public int add(int a, int b) throws RemoteException {
        System.out.println("Client yêu cầu tính tổng: " + a + " + " + b);
        return a + b;
    }

    public static void main(String[] args) {
        try {
            // Tạo Registry ở cổng 1099
            Registry registry = LocateRegistry.createRegistry(1099);
            
            // Đăng ký object với tên "CalcService"
            registry.rebind("CalcService", new CalculatorServer());
            
            System.out.println("RMI Server đã sẵn sàng!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Bước 3: Viết Client
Client sẽ tìm kiếm dịch vụ thông qua tên đã đăng ký.

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class CalculatorClient {
    public static void main(String[] args) {
        try {
            // Kết nối đến Registry lại localhost:1099
            Registry registry = LocateRegistry.getRegistry("localhost", 1099);
            
            // Tìm kiếm (Lookup) dịch vụ
            Calculator calc = (Calculator) registry.lookup("CalcService");
            
            // Gọi hàm từ xa như gọi hàm cục bộ
            int result = calc.add(10, 20);
            System.out.println("Kết quả từ Server: " + result);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 3. Ưu nhược điểm
- **Ưu điểm**: Code rất trong sáng, che giấu sự phức tạp của mạng.
- **Nhược điểm**: Chỉ hoạt động giữa các máy chạy Java (Tốc độ chậm hơn Socket thuần túy).

Ngày nay, RMI ít được dùng trực tiếp mà thay thế bằng Web Services (REST, gRPC) nhưng nó vẫn là nền tảng lịch sử quan trọng của các hệ thống phân tán Java (Enterprise Beans).
