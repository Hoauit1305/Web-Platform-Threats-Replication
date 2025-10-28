**Invariant I.2 – “Confidentiality of HttpOnly cookies”** 
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.2: “Confidentiality of HttpOnly cookies”**

> *Scripts can only access cookies without the `HttpOnly` attribute.*

### Mục tiêu:
Đảm bảo rằng **các script phía client (JavaScript)** **không thể truy cập được các cookie có thuộc tính `HttpOnly`**.  
- Nếu một cookie có cờ `HttpOnly=true`, nó chỉ được gửi qua kênh HTTP(S) trong yêu cầu/đáp ứng, và **không bao giờ được phép đọc thông qua JavaScript** (bằng `document.cookie`).

Invariant này bảo vệ tính **bảo mật (confidentiality)** của cookie khỏi việc bị rò rỉ qua script, đặc biệt trong các cuộc tấn công **Cross-Site Scripting (XSS)**.

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `cookie-jar-set(name, value, {http-only, secure, domain, path})@tr_t1` — sự kiện khi cookie được ghi vào cookie jar, gồm các thuộc tính `http-only`, `secure`, `domain`, và `path`.
- `js-get-cookie(ctx, cookies)@tr_t2` — sự kiện khi một script JavaScript trong ngữ cảnh `ctx` cố gắng đọc cookies thông qua API như `document.cookie`.
- `name ++ "=" ++ value ∈ split-cookie(cookies)` — kiểm tra xem cookie có tên và giá trị tương ứng xuất hiện trong tập cookies mà script đang đọc.
- `cookie-match(path, domain, secure, ctx-location(ctx))` — kiểm tra rằng cookie có phạm vi áp dụng hợp lệ với vị trí ngữ cảnh `ctx`.
- `http-only = false` — ràng buộc yêu cầu cookie **không có** cờ `HttpOnly` để có thể được truy cập bởi script.

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
HTTP-ONLY-INVARIANT(tr) :=
t2 > t1 ∧
cookie-jar-set(name, value, {http-only, secure, domain, path})@tr_t1 ∧
js-get-cookie(ctx, cookies)@tr_t2 ∧
name ++ "=" ++ value ∈ split-cookie(cookies) ∧
cookie-match(path, domain, secure, ctx-location(ctx)) ⇒
http-only = false
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `t2 > t1` | Việc script đọc cookie xảy ra sau khi cookie đã được tạo. |
| `cookie-jar-set(...)@tr_t1` | Cookie được lưu trong cookie jar với các thuộc tính `http-only`, `secure`, `domain`, `path`. |
| `js-get-cookie(ctx, cookies)@tr_t2` | Một script trong ngữ cảnh `ctx` gọi API để đọc cookie. |
| `name ++ "=" ++ value ∈ split-cookie(cookies)` | Cookie đang được đọc nằm trong tập cookies mà script có thể truy cập. |
| `cookie-match(path, domain, secure, ctx-location(ctx))` | Kiểm tra rằng cookie có phạm vi hợp lệ với vị trí script đang chạy. |
| `⇒ http-only = false` | Chỉ khi `HttpOnly=false` thì cookie mới được phép truy cập bởi script. Nếu `HttpOnly=true`, việc đọc cookie này sẽ vi phạm invariant. |

---

### Diễn giải dễ hiểu:

> Khi một script trên trang web cố gắng đọc cookies thông qua `document.cookie`, chỉ những cookie **không có cờ `HttpOnly`** mới được phép xuất hiện trong kết quả.  
> Nếu một cookie có `HttpOnly=true` mà vẫn có thể được đọc bởi script, thì trình duyệt đã vi phạm invariant này — điều đó đồng nghĩa với việc cookie nhạy cảm (ví dụ session token) có thể bị rò rỉ qua XSS.

---

## 5. Ý nghĩa bảo mật thực tế

IInvariant này đảm bảo rằng:
- **Cookie nhạy cảm (thường có cờ `HttpOnly`)** không thể bị đọc thông qua JavaScript.  
- **Ngăn ngừa rò rỉ token đăng nhập hoặc session ID** trong trường hợp xảy ra lỗ hổng XSS.  
- **Duy trì tính bảo mật của phiên người dùng (session confidentiality)**.  

Nếu invariant này bị vi phạm, attacker có thể sử dụng mã JavaScript để đánh cắp cookie `HttpOnly`, từ đó chiếm quyền điều khiển tài khoản người dùng.

---
