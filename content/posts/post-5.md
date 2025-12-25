---
title: "5. Input/Output Stream và Serialization trong Java"
date: 2024-12-24
draft: false
tags: ["Java", "IO"]
categories: ["Java", "IO"]
featured_image: "https://source.unsplash.com/random/800x600/?data,stream"
summary: "Phân biệt Byte Stream và Character Stream. Hướng dẫn kỹ thuật Serialization để gửi cả một Object Java qua mạng."
---

Trong Lập trình mạng, việc "gửi tin nhắn" thực chất là ghi dữ liệu vào một đường ống (Output Stream) và đọc dữ liệu từ đường ống đó (Input Stream). Java chia I/O thành hai họ lớn.

## 1. Phân loại Streams

### Byte Streams (Luồng Byte)
- **Class gốc**: `InputStream`, `OutputStream`
- **Công dụng**: Đọc/Ghi dữ liệu nhị phân thô (Raw binary) như hình ảnh, video, file nhạc, file .exe.
- **Ví dụ**: `FileInputStream`, `FileOutputStream`, `Socket.getOutputStream()`.

### Character Streams (Luồng Ký tự)
- **Class gốc**: `Reader`, `Writer`
- **Công dụng**: Đọc/Ghi văn bản (Text). Nó tự động xử lý các bảng mã (Encoding) như UTF-8, giúp tránh lỗi font chữ.
- **Ví dụ**: `FileReader`, `FileWriter`.

> **Mẹo**: Nếu bạn làm việc với Socket TCP (văn bản), hãy bọc (wrap) Byte Stream trong Character Stream (InputStreamReader) để dễ xử lý.

## 2. Java Serialization (Tuần tự hóa)
Giả sử bạn có một đối tượng `User`:
```java
class User {
    String name = "Thanh";
    int age = 21;
}
```
Làm sao gửi cả cục Object này qua mạng cho Server? Bạn không thể gửi từng biến lẻ tẻ vì rất cực. **Serialization** là quá trình biến object thành một dòng byte để gửi đi.

### Cách thực hiện
1.  Class phải implement giao diện `Serializable` (như một con tem chứng nhận).
2.  Sử dụng `ObjectOutputStream` để ghi và `ObjectInputStream` để đọc.

**Gửi Object (Client):**
```java
// Yêu cầu: Class User implements Serializable
User user = new User("Nguyen Van A", 25);

OutputStream os = socket.getOutputStream();
ObjectOutputStream out = new ObjectOutputStream(os);

out.writeObject(user); // Gửi cả object đi
```

**Nhận Object (Server):**
```java
InputStream is = socket.getInputStream();
ObjectInputStream in = new ObjectInputStream(is);

try {
    User receivedUser = (User) in.readObject();
    System.out.println("Tên: " + receivedUser.name);
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

### Lưu ý bảo mật
Deserialization (giải nén object) là một lỗ hổng bảo mật tiềm tàng nếu bạn nhận dữ liệu từ nguồn không tin cậy, hacker có thể chèn mã độc vào object. Hãy cẩn thận!
