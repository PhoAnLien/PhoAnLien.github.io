---
title: "7. Làm chủ Bất đồng bộ: Promise và Async/Await"
date: 2024-12-26
draft: false
tags: ["JavaScript", "Async"]
categories: ["JavaScript", "Async"]
featured_image: "https://source.unsplash.com/random/800x600/?time,clock"
summary: "Từ Callback Hell đến cú pháp Async/Await hiện đại. Cách xử lý các tác vụ tốn thời gian như gọi API mà không làm treo giao diện."
---

JavaScript là ngôn ngữ đơn luồng (single-threaded), nghĩa là nó chỉ làm một việc tại một thời điểm. Vậy tại sao nó có thể gọi API, đọc file, hẹn giờ... cùng lúc mà giao diện không bị treo? Đó là nhờ cơ chế **Event Loop** và **Asynchronous** (Bất đồng bộ).

## 1. Thời kỳ đen tối: Callback Hell
Ngày xưa, để xử lý việc A xong rồi mới làm việc B, ta truyền hàm B vào hàm A.
```javascript
getData(function(a) {
    getMoreData(a, function(b) {
        getEvenMoreData(b, function(c) { 
            // Cấu trúc hình kim tự tháp (Pyramid of Doom)
            // Code thụt lề quá sâu, khó đọc, khó bắt lỗi
        });
    });
});
```

## 2. Promise (ES6) - Lời hứa
Promise đại diện cho giá trị chưa có ngay lập tức nhưng sẽ có trong tương lai. Nó có 3 trạng thái:
- **Pending**: Đang chờ.
- **Fulfilled**: Thành công (Resolve).
- **Rejected**: Thất bại (Reject).

```javascript
getData()
    .then(a => getMoreData(a)) // Trả về promise tiếp theo
    .then(b => console.log(b)) // Xử lý kết quả cuối
    .catch(err => console.error(err)); // Bắt lỗi chung cho cả chuỗi
```
Code đã phẳng hơn, dễ đọc hơn.

## 3. Async / Await (ES8) - Chân ái
Đây là cú pháp "syntactic sugar" bọc lấy Promise, giúp viết code bất đồng bộ trông giống hệt code đồng bộ thông thường.

```javascript
async function main() {
    try {
        const a = await getData(); // Dừng ở đây đợi a xong
        const b = await getMoreData(a); // Dừng đợi b xong
        console.log(b);
    } catch (err) {
        // Bắt lỗi như code bình thường
        console.error("Lỗi rồi:", err);
    }
}
```

**Quy tắc:**
- `await` chỉ được dùng trong hàm `async`.
- `await` sẽ làm hàm tạm dừng cho đến khi Promise trả về kết quả.

Đây là tiêu chuẩn hiện đại mà mọi Developer JS cần nắm vững để làm việc với API.
