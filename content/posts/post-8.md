---
title: "8. Fetch API: Cách gọi API chuẩn trong JavaScript"
date: 2024-12-27
draft: false
tags: ["JavaScript", "API"]
categories: ["JavaScript", "API"]
featured_image: "https://source.unsplash.com/random/800x600/?internet,connection"
summary: "Thay thế XMLHttpRequest cũ kỹ bằng Fetch API hiện đại. Hướng dẫn gửi GET/POST Request và xử lý kết quả JSON."
---

Trong lập trình web hiện đại (SPA - Single Page Application), việc Client (Browser) gọi API lấy dữ liệu từ Server là việc làm hàng ngày. `Fetch API` là công cụ native mạnh mẽ được xây dựng sẵn trong mọi trình duyệt hiện đại.

## 1. Cơ bản: GET Request
Hàm `fetch()` nhận vào URL và trả về một Promise.

```javascript
// Lấy thông tin user từ GitHub API
fetch('https://api.github.com/users/PhoAnLien')
  .then(response => {
     // Fetch không reject lỗi HTTP (404, 500)
     if (!response.ok) {
         throw new Error('Lỗi mạng hoặc 404');
     }
     return response.json(); // Chuyển Stream body về JSON
  })
  .then(data => {
      console.log("User:", data.login);
      console.log("Avatar:", data.avatar_url);
  })
  .catch(error => console.error('Có lỗi xảy ra:', error));
```

## 2. Nâng cao: POST Request
Để gửi dữ liệu (ví dụ: form đăng nhập) lên server, ta cần thêm tham số `options`.

```javascript
async function login(username, password) {
    const url = 'https://example.com/api/login';
    const data = { username, password };

    try {
        const response = await fetch(url, {
            method: 'POST', // Phương thức gửi
            headers: {
                // Báo cho server biết ta gửi JSON
                'Content-Type': 'application/json', 
            },
            body: JSON.stringify(data), // Chuyển object JS thành chuỗi JSON
        });

        const result = await response.json();
        console.log('Token:', result.token);
        
    } catch (error) {
        console.error('Lỗi đăng nhập:', error);
    }
}
```

## 3. Lưu ý quan trọng
- **CORS**: Khi gọi API khác domain, bạn có thể gặp lỗi Cross-Origin Resource Sharing. Server cần cho phép domain của bạn truy cập.
- **Cookies**: Fetch mặc định không gửi cookies. Cần thêm `credentials: 'include'` nếu muốn gửi session.

Fetch API mạnh mẽ, linh hoạt và là nền tảng để các thư viện như `Axios` phát triển thêm các tính năng tiện ích.
