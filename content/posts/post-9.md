---
title: "9. DOM Manipulation: Thao tác với giao diện Web"
date: 2024-12-28
draft: false
tags: ["JavaScript", "DOM"]
categories: ["JavaScript", "DOM"]
featured_image: "https://source.unsplash.com/random/800x600/?browser,web"
summary: "JavaScript làm gì trên trình duyệt? Tìm hiểu về DOM Tree, cách truy xuất phần tử, thay đổi style và lắng nghe sự kiện người dùng."
---

DOM (Document Object Model) là cây cấu trúc của trang HTML. Trình duyệt biến mã HTML thành một cây các đối tượng (Object) để JavaScript có thể tương tác.

## 1. Truy xuất phần tử (Selectors)
Để thay đổi cái gì, bạn phải "chọn" được nó trước.

- `document.getElementById('header')`: Nhanh nhất, lấy theo ID.
- `document.querySelector('.btn-primary')`: Chọn phần tử **đầu tiên** khớp CSS Selector.
- `document.querySelectorAll('li')`: Chọn **tất cả**, trả về `NodeList` (giống mảng).

## 2. Thay đổi nội dung và Style
```javascript
const title = document.querySelector('h1');

// 1. Thay đổi nội dung
title.textContent = "Xin chào JavaScript!"; 

// 2. Thay đổi style (CSS inline)
title.style.color = "red";
title.style.fontSize = "2rem";

// 3. Thay đổi Class (Khuyên dùng)
title.classList.add("highlight");
title.classList.toggle("hidden");
```

## 3. Lắng nghe sự kiện (Event Listener)
Tương tác người dùng (click, gõ phím, scroll) là cốt lõi của web.

```javascript
const btn = document.querySelector('#submit-btn');

btn.addEventListener('click', function(event) {
    event.preventDefault(); // Ngăn form reload trang
    alert('Bạn đã click nút!');
    console.log(event.target); // Xem ai đã click
});
```

## 4. Cơ chế Event Bubbling
Khi bạn click vào một thẻ `<span>` nằm trong thẻ `<div>`, sự kiện click sẽ kích hoạt ở `span` trước, sau đó "nổi bọt" (bubble) lên `div`, rồi lên `body`...
Hiểu điều này giúp bạn áp dụng kỹ thuật **Event Delegation**: Thay vì gắn sự kiện cho 1000 dòng trong bảng, hãy gắn 1 sự kiện vào cái bảng (table) cha và kiểm tra `event.target`.

```javascript
// Gắn 1 lần cho cha, xử lý cho tất cả con
document.querySelector('#parent-list').addEventListener('click', (e) => {
    if (e.target.tagName === 'LI') {
        console.log("Bạn chọn mục:", e.target.textContent);
    }
});
```

Hiểu rõ DOM giúp bản làm chủ được giao diện người dùng mà không phụ thuộc quá nhiều vào các Framework như React hay Vue trong những tác vụ đơn giản.
