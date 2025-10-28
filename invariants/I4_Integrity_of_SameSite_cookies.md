**Invariant I.4 – “Integrity of SameSite cookies”**
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.4: “Integrity of SameSite cookies”**

> *A `SameSite=Lax/Strict` cookie can only be set for domain d through HTTP responses*

### Mục tiêu:
Đảm bảo rằng **cookie có cờ `SameSite=Lax` hoặc `SameSite=Strict`** chỉ được tạo trong những điều kiện an toàn:
- Yêu cầu (request) tạo cookie phải đến từ **cùng site (same-site)** với domain nhận cookie.
- Hoặc yêu cầu đó phải là **top-level navigation** (người dùng trực tiếp điều hướng).

Invariant này đảm bảo rằng **cookie có thuộc tính SameSite** không thể được đặt bởi các trang web bên thứ ba (cross-site), từ đó bảo vệ chống lại tấn công **Cross-Site Request Forgery (CSRF)**.

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `net-request(id, url, _, type, origin-url, _, _, _, _)@tr_t1` — sự kiện gửi HTTP request, có chứa `origin-url` và loại request (`type`).
- `net-response(id, url, {set-cookie-headers}, _)@tr_t2` — sự kiện phản hồi HTTP chứa header `Set-Cookie`.
- `set-cookie ∈ set-cookie-headers` — một header Set-Cookie cụ thể.
- `split-cookie(set-cookie)` — phép tách header thành các phần tử như `name=value`, `SameSite=Lax`, v.v.
- `"SameSite" ++ SS ∈ split-cookie(set-cookie)` — giá trị thuộc tính `SameSite` của cookie (`Lax` hoặc `Strict`).
- `same-site(_, _)` — kiểm tra hai URL có thuộc cùng một site không.
- `cookie-jar-set(name, value, {samesite, path, domain})@tr_t3` — sự kiện lưu cookie vào cookie jar.
- `cookie-match(path, domain, _, url)` — kiểm tra phạm vi cookie hợp lệ với URL phản hồi.
- `url-site(url, site)` — xác định site tương ứng với URL.
- `type = main_frame` — chỉ ra rằng request là top-level navigation.

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
SAMESITE-COOKIES-INTEGRITY(tr) :=
t1 < t2 < t3 ∧
net-request(id, url, _, type, origin-url, _, _, _, _)@tr_t1 ∧
net-response(id, url, {set-cookie-headers}, _)@tr_t2 ∧
set-cookie ∈ set-cookie-headers ∧
name ++ "=" ++ value ∈ split-cookie(set-cookie) ∧
"SameSite" ++ SS ∈ split-cookie(set-cookie) ∧
(SS = "Lax" ∧ same-site = SS-Lax ∨
 SS = "Strict" ∧ same-site = SS-Strict) ∧
cookie-jar-set(name, value, {samesite, path, domain})@tr_t3 ∧
cookie-match(path, domain, _, url) ∧
url-site(url, site) →
(type = main_frame ∨ url-site(origin-url, site))
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `t1 < t2 < t3` | Request gửi đi, phản hồi nhận về, và cookie được lưu vào cookie jar theo đúng thứ tự thời gian. |
| `net-request(..., origin-url, type)` | Yêu cầu HTTP được gửi đi từ `origin-url`, có loại request `type`. |
| `net-response(..., {set-cookie-headers})` | Phản hồi chứa các header `Set-Cookie`. |
| `set-cookie ∈ set-cookie-headers` | Một header `Set-Cookie` cụ thể được phân tích. |
| `"SameSite" ++ SS ∈ split-cookie(set-cookie)` | Header có thuộc tính `SameSite` và giá trị là `Lax` hoặc `Strict`. |
| `(SS = "Lax" ∧ same-site = SS-Lax ∨ SS = "Strict" ∧ same-site = SS-Strict)` | Kiểm tra tính hợp lệ của SameSite theo mức độ `Lax` hoặc `Strict`. |
| `cookie-jar-set(...)@tr_t3` | Cookie được lưu trong cookie jar tại thời điểm t3. |
| `cookie-match(path, domain, _, url)` | Cookie có phạm vi hợp lệ so với URL. |
| `url-site(url, site)` | Xác định site chứa cookie. |
| `→ (type = main_frame ∨ url-site(origin-url, site))` | Kết luận: chỉ chấp nhận nếu yêu cầu là top-level navigation hoặc cùng site. |

---

## Diễn giải dễ hiểu:

> Khi một trang web cố gắng đặt cookie có `SameSite=Lax` hoặc `SameSite=Strict`, cookie đó chỉ được chấp nhận nếu yêu cầu đến từ **cùng site** (same-site) hoặc là **điều hướng trực tiếp của người dùng (top-level navigation)**.  
> Nếu cookie được đặt bởi trang web thuộc site khác, invariant này sẽ bị vi phạm.

Nói cách khác:
> Cookie SameSite được thiết kế để **chặn việc đặt hoặc gửi cookie trong bối cảnh cross-site**, nhằm ngăn tấn công CSRF.

---

## 5. Ý nghĩa bảo mật thực tế

Invariant này bảo vệ:
- **Ngăn trang web thứ ba (cross-site)** chèn hoặc ghi đè cookie `SameSite`.  
- **Giảm nguy cơ CSRF**, vì cookie chỉ được gửi/đặt trong các ngữ cảnh đáng tin cậy (same-site hoặc top-level).  
- **Bảo toàn tính toàn vẹn** của các cookie xác thực (authentication cookies).

Nếu invariant này bị vi phạm:
- Trang web khác có thể đặt cookie `SameSite` sai quy tắc, mở ra nguy cơ tấn công CSRF hoặc session fixation.

---
