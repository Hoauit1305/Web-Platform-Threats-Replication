**Invariant I.1 – “Integrity of Secure cookies”**
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.1: “Integrity of Secure cookies”**

> *Cookies with the `Secure` attribute can only be set over secure channels.*

### Mục tiêu:
Đảm bảo rằng **cookie có gắn cờ `Secure` chỉ được tạo (Set-Cookie)** khi phản hồi chứa Set-Cookie được nhận từ một URL có giao thức an toàn (ví dụ `https` hoặc `wss`). Nếu cookie Secure được đặt từ một phản hồi không an toàn thì đó là vi phạm invariant.

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `net-response(_, url, {set-cookie-headers}, _)@tr_t1` — một sự kiện phản hồi mạng tại timestamp t1, gồm tập các headers `set-cookie-headers` và URL đích `url`.
- `set-cookie` — một phần tử của `set-cookie-headers` (một header Set-Cookie cụ thể).
- `split-cookie(set-cookie)` — phép tách header Set-Cookie thành các thành phần (ví dụ các cặp và thuộc tính); biểu kiến dùng để kiểm tra sự tồn tại của `name=value` và token `"Secure"` trong cùng header.
- `name ++ "=" ++ value ∈ split-cookie(set-cookie)` — chỉ ra rằng header Set-Cookie chứa cặp tên/giá trị của cookie đang xét.
- `cookie-jar-set(name, value, {Secure=true}, false)@tr_t2` — sự kiện khi cookie với thuộc tính `Secure=true` thực sự được ghi vào cookie-jar tại timestamp t2.
- `url-proto(url, "wss") ∨ url-proto(url, "https")` — predicate kiểm tra giao thức của URL là `wss` hoặc `https`.

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
SECURE-COOKIES-INVARIANT(tr) :=
t2 > t1 ∧
net-response(_, url, {set-cookie-headers}, _)@tr_t1 ∧
set-cookie ∈ set-cookie-headers ∧
name ++ "=" ++ value ∈ split-cookie(set-cookie) ∧
"Secure" ∈ split-cookie(set-cookie) ∧
cookie-jar-set(name, value, {Secure=true}, false)@tr_t2 →
(url-proto(url, "wss") ∨ url-proto(url, "https"))
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `t2 > t1` | Sự kiện ghi cookie vào cookie-jar xảy ra sau khi nhận phản hồi mạng chứa header Set-Cookie. |
| `net-response(... )@tr_t1` | Phản hồi mạng tại thời điểm t1 chứa header Set-Cookie. |
| `set-cookie ∈ set-cookie-headers` | Một header Set-Cookie cụ thể thuộc về tập các Set-Cookie của phản hồi. |
| `name ++ "=" ++ value ∈ split-cookie(set-cookie)` | Header Set-Cookie chứa cặp `name=value` của cookie đang xét. |
| `"Secure" ∈ split-cookie(set-cookie)` | Header Set-Cookie đó chỉ định thuộc tính `Secure`. |
| `cookie-jar-set(... )@tr_t2` | Trình duyệt thực sự lưu cookie có `Secure=true` vào cookie jar tại thời điểm t2. |
| `→ (url-proto(url, "wss") ∨ url-proto(url, "https"))` | Suy luận: nếu các tiền đề đúng thì URL nơi phản hồi đến **phải** dùng giao thức `wss` hoặc `https`. Nếu không thì invariant bị vi phạm. |

---

### Diễn giải dễ hiểu:

> Nếu một cookie có cờ `Secure` được đặt vào cookie jar từ một phản hồi HTTP (tức là trình duyệt nhận header `Set-Cookie` và sau đó lưu cookie với `Secure=true`), thì phản hồi đó **phải** được nhận qua một kết nối an toàn — cụ thể là URL của phản hồi phải dùng giao thức `https` hoặc `wss`. 
> Nếu cookie Secure được lưu nhưng phản hồi không dùng `https`/`wss`, thì hành vi đó vi phạm invariant và có thể là dấu hiệu của vấn đề bảo mật (ví dụ cookie injection hoặc session takeover).

---

## 5. Ý nghĩa bảo mật thực tế

Invariant này nói rằng: khi một cookie Secure được thực sự lưu vào cookie-jar do một header Set-Cookie vừa nhận, thì phản hồi đó **phải** đến từ một kết nối an toàn (HTTPS/WSS). Nếu cookie Secure được lưu mà phản hồi không dùng `https`/`wss` thì đó là hành vi không đúng theo invariant của tác giả và có thể dẫn tới các vấn đề bảo mật như cookie injection hay session takeover.

---
