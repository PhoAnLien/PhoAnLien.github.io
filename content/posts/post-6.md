---
title: "6. Phân biệt var, let và const trong JavaScript hiện đại"
date: 2024-12-25
draft: false
tags: ["JavaScript", "ES6"]
categories: ["JavaScript", "ES6"]
featured_image: "https://source.unsplash.com/random/800x600/?javascript,code"
summary: "Tại sao nên từ bỏ var? Bảng so sánh chi tiết về Scope, Hoisting và khả năng Reassign của 3 từ khóa khai báo biến trong JS."
---

Từ phiên bản ES6 (2015), JavaScript đã thay đổi hoàn toàn cách khai báo biến. Việc hiểu rõ `var`, `let`, `const` không chỉ giúp code sạch hơn mà còn tránh được hàng tá lỗi logic khó tìm (bugs).

## Bảng so sánh nhanh

| Đặc điểm | `var` | `let` | `const` |
| :--- | :--- | :--- | :--- |
| **Phạm vi (Scope)** | Function Scope | Block Scope `{}` | Block Scope `{}` |
| **Hoisting** | Có (khởi tạo `undefined`) | Có (nhưng ở trong vùng chết - TDZ) | Có (nhưng ở trong vùng chết - TDZ) |
| **Gán lại (Reassign)** | Có | Có | Không |
| **Khai báo lại (Redeclare)** | Có | Không | Không |

---

## 1. Var - "Người cũ" (Nên tránh)
`var` có phạm vi là **Function Scope**, nghĩa là nếu bạn khai báo nó trong một khối `if` hay vòng lặp `for`, nó **VẪN** truy cập được ở bên ngoài khối đó (miễn là trong cùng hàm).

```javascript
if (true) {
    var x = 10;
}
console.log(x); // 10 (Vẫn chạy được! -> Dễ gây lỗi)
```

Ngoài ra, `var` bị **Hoisting** kỳ lạ: bạn có thể dùng biến trước khi khai báo nó (giá trị là `undefined`), điều này làm code rất khó debug.

## 2. Let - "Sự thay thế hoàn hảo"
`let` ra đời để sửa lỗi của `var`. Nó có phạm vi **Block Scope** (trong cặp ngoặc nhọn `{}`). Biến chỉ tồn tại và có ý nghĩa trong khối đó.

```javascript
if (true) {
    let y = 20;
}
console.log(y); // Error: y is not defined (Đúng logic!)
```

## 3. Const - "Bất biến" (Ưu tiên dùng)
`const` giống `let` về scope, nhưng bạn bắt buộc phải gán giá trị ngay khi khai báo và **không thể gán lại** giá trị mới cho nó.

### Hiểu lầm thường gặp về Const
Nhiều người nghĩ `const` tạo ra giá trị bất biến hoàn toàn. Sai! Nếu `const` là một Object hoặc Array, bạn **VẪN** có thể thay đổi nội dung bên trong nó.

```javascript
const user = { name: "Thanh" };

// Hợp lệ: Thay đổi thuộc tính bên trong
user.name = "John"; 

// Lỗi: Gán lại cả object
// user = { name: "Doe" }; // TypeError: Assignment to constant variable.
```

**Lời khuyên:** Hãy dùng `const` mặc định cho mọi biến. Chỉ khi nào bạn biết chắc chắn biến đó cần thay đổi giá trị (như biến đếm `i` trong vòng lặp), hãy đổi sang dùng `let`. Đừng bao giờ dùng `var` nữa.
