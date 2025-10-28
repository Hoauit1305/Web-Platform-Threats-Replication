**Invariant I.3 – “Integrity of Host cookies”**
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.3: “Integrity of Host cookies”**

> *A `__Host-`cookie set for domain d can only be set by d or by scripts included in pages on d.*

### Mục tiêu:
Đảm bảo rằng **cookie có tiền tố `__Host-`** chỉ có thể được thiết lập khi:
- Không có thuộc tính `Domain` được chỉ định hoặc thuộc tính `Domain` trùng khớp hoàn toàn với host của URL gửi phản hồi hoặc ngữ cảnh script đặt cookie.
- Cookie được gắn cờ `Secure` và chỉ được gửi qua HTTPS (theo quy tắc tiêu chuẩn của cookie `__Host-`).

Invariant này bảo vệ tính **toàn vẹn của cookie `__Host-`**, ngăn chặn việc cookie bị chèn hoặc thay đổi từ các domain khác.

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `net-response(_, url, {set-cookie-headers}, _)@tr_t1` — phản hồi HTTP chứa các header `Set-Cookie` tại thời điểm t1.
- `set-cookie ∈ set-cookie-headers` — một header `Set-Cookie` cụ thể trong phản hồi.
- `"__Host-" ++ cname ++ "=" ++ cvalue ∈ split-cookie(set-cookie)` — header Set-Cookie định nghĩa cookie bắt đầu bằng tiền tố `__Host-`.
- `url-domain(url, host)` — trả về tên miền của URL đã gửi phản hồi.
- `js-set-cookie(ctx, set-cookie, _)@tr_t1` — script trong ngữ cảnh `ctx` gọi hàm để đặt cookie (ví dụ qua `document.cookie`).
- `url-domain(ctx-location(ctx), host)` — xác định host của ngữ cảnh script.
- `cookie-jar-set("__Host-" ++ cname, cvalue, {domain}, false)@tr_t2` — sự kiện ghi cookie `__Host-` vào cookie jar với thuộc tính `domain` được chỉ định tại thời điểm t2.
- `domain = host` — yêu cầu rằng domain trong cookie phải trùng với host của URL/ngữ cảnh.

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
HOST-INVARIANT(tr) :=
t2 > t1 ∧
( net-response(_, url, {set-cookie-headers}, _)@tr_t1 ∧
  set-cookie ∈ set-cookie-headers ∧
  "__Host-" ++ cname ++ "=" ++ cvalue ∈ split-cookie(set-cookie) ∧
  url-domain(url, host)
) ∨
( js-set-cookie(ctx, set-cookie, _)@tr_t1 ∧
  "__Host-" ++ cname ++ "=" ++ cvalue ∈ split-cookie(set-cookie) ∧
  url-domain(ctx-location(ctx), host)
) ∧
cookie-jar-set("__Host-" ++ cname, cvalue, {domain}, false)@tr_t2 ⇒
domain = host
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `t2 > t1` | Cookie được ghi vào cookie jar sau khi phản hồi hoặc hành động của script xảy ra. |
| `net-response(... )@tr_t1` | Phản hồi HTTP chứa header `Set-Cookie`. |
| `set-cookie ∈ set-cookie-headers` | Một header Set-Cookie cụ thể trong phản hồi. |
| `"__Host-" ++ cname ++ "=" ++ cvalue ∈ split-cookie(set-cookie)` | Header Set-Cookie định nghĩa cookie có tiền tố `__Host-`. |
| `url-domain(url, host)` | Lấy tên miền của URL phản hồi. |
| `js-set-cookie(ctx, set-cookie, _)@tr_t1` | Một script trong ngữ cảnh `ctx` cố gắng đặt cookie. |
| `url-domain(ctx-location(ctx), host)` | Lấy host của ngữ cảnh script đang đặt cookie. |
| `cookie-jar-set("__Host-" ++ cname, cvalue, {domain}, false)@tr_t2` | Cookie thực sự được lưu vào cookie jar. |
| `⇒ domain = host` | Domain của cookie phải trùng khớp chính xác với host của URL/ngữ cảnh đã đặt cookie. |

---

## Diễn giải dễ hiểu:

> Một cookie có tiền tố `__Host-` chỉ được phép tồn tại nếu **thuộc tính `Domain` của nó trùng khớp với host thực tế** của URL hoặc ngữ cảnh script đã đặt nó.  
> Nếu cookie `__Host-` được tạo từ một phản hồi hoặc script thuộc domain khác, thì đó là hành vi vi phạm invariant.

Nói cách khác:
> Trình duyệt chỉ được phép lưu cookie `__Host-` khi nó thuộc đúng domain của nguồn đã tạo ra nó — ngăn việc domain khác ghi đè cookie cùng tên.

---

## 5. Ý nghĩa bảo mật thực tế

Invariant này bảo vệ:
- **Chống cookie injection**: domain khác không thể ghi đè cookie `__Host-` của trang thật.  
- **Bảo vệ session và token** gắn với domain chính xác.  
- **Tăng tính cô lập cookie**, giúp cookie chỉ được truy cập và cập nhật trong cùng domain thực.

Nếu invariant này bị vi phạm:
- Kẻ tấn công có thể đặt cookie `__Host-` từ domain khác, dẫn đến chiếm đoạt phiên người dùng hoặc làm hỏng logic xác thực.

---
