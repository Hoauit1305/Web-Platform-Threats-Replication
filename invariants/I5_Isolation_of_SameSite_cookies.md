**Invariant I.5 – “Isolation of SameSite cookies (Confidentiality)”**
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.5: “Confidentiality of SameSite cookies”**

> * If a `SameSite=Lax/Strict` cookie should not be attached to a request to load a page p, then it is not attached to that request, it is not accessible by scripts in p nor attached to requests initiated by p.*

### Mục tiêu:
Đảm bảo rằng **cookie có thuộc tính `SameSite=Lax` hoặc `SameSite=Strict`** chỉ được gửi hoặc truy cập trong **các bối cảnh same-site**, và không bao giờ bị rò rỉ sang bối cảnh **cross-site**.

Invariant này đảm bảo tính **bảo mật (confidentiality)** của cookie SameSite, ngăn cookie bị gửi hoặc đọc trong các tương tác giữa site khác nhau (cross-site request hoặc cross-origin script).

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `cookie-jar-set(name, value, {secure, same-site, path, domain, host-only})@tr_t1` — sự kiện lưu cookie vào cookie jar với các thuộc tính `SameSite`, `Secure`, `Path`, `Domain`, `HostOnly`.
- `(same-site = SS-Lax ∨ same-site = SS-Strict)` — cookie có chính sách SameSite Lax hoặc Strict.
- `net-request(_, url, method, type, origin, _, _, redirs, {cookies}, _)@tr_t2` — sự kiện gửi HTTP request có thể chứa cookies.
- `cookie-match(path, domain, secure, host-only, url)` — kiểm tra xem cookie có phù hợp với URL không.
- `¬cookie-match-samesite(same-site, type, origin, method, redirs, url)` — điều kiện cookie không phù hợp với quy tắc same-site (cross-site request).
- `js-get-cookie(ctx, cookies')@tr_t3` — sự kiện JavaScript cố gắng đọc cookie trong ngữ cảnh `ctx`.
- `url = ctx-location(ctx)` — xác định URL tương ứng với ngữ cảnh script.
- `net-request(_, url', method', type', origin', doc-url', _, redirs', {cookies'}, _)@tr_t3` — một request khác được gửi từ `doc-url'` có thể mang theo cookies.
- `cookie-should-be-sent(...)` — kiểm tra xem cookie có đủ điều kiện được gửi trong request không.
- `name ++ "=" ++ value ∉ split-cookie(cookies)` — cookie không nằm trong tập cookies gửi đi.
- `name ++ "=" ++ value ∉ split-cookie(cookies')` — cookie không nằm trong tập cookies đọc được bởi JavaScript hoặc request khác.

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
SAMESITE-COOKIES-CONFIDENTIALITY(tr) :=
t1 < t2 < t3 ∧
cookie-jar-set(name, value, {secure, same-site, path, domain, host-only})@tr_t1 ∧
(same-site = SS-Lax ∨ same-site = SS-Strict) ∧
net-request(_, url, method, type, origin, _, _, redirs, {cookies}, _)@tr_t2 ∧
cookie-match(path, domain, secure, host-only, url) ∧
¬cookie-match-samesite(same-site, type, origin, method, redirs, url) ∧
(
  (js-get-cookie(ctx, cookies')@tr_t3 ∧ url = ctx-location(ctx)) ∨
  (net-request(_, url', method', type', origin', doc-url', _, redirs', {cookies'}, _)@tr_t3 ∧
   doc-url' = some(url) ∧
   cookie-should-be-sent(path, domain, secure, same-site, host-only, type', origin', url', method', redirs'))
) →
(name ++ "=" ++ value ∉ split-cookie(cookies) ∧
 name ++ "=" ++ value ∉ split-cookie(cookies'))
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `t1 < t2 < t3` | Thứ tự thời gian: cookie được lưu → request gửi đi → script đọc hoặc request khác thực thi. |
| `cookie-jar-set(...)@tr_t1` | Cookie SameSite được tạo và lưu. |
| `(same-site = SS-Lax ∨ same-site = SS-Strict)` | Cookie sử dụng chính sách SameSite bảo mật. |
| `net-request(..., {cookies})@tr_t2` | Một yêu cầu HTTP được gửi đi và có thể mang cookie. |
| `cookie-match(...)` | Cookie tương ứng với URL request. |
| `¬cookie-match-samesite(...)` | Yêu cầu là cross-site, không cùng site với cookie. |
| `(js-get-cookie(...) ∨ net-request(..., {cookies'}))` | Một script hoặc request khác cố gắng truy cập hoặc gửi cookie. |
| `→ (name ++ "=" ++ value ∉ split-cookie(...))` | Cookie không được phép xuất hiện trong bất kỳ tập cookies nào của các ngữ cảnh cross-site. |

---

## Diễn giải dễ hiểu:

> Nếu một cookie có thuộc tính `SameSite=Lax` hoặc `SameSite=Strict`, thì cookie đó chỉ được gửi hoặc truy cập trong bối cảnh same-site.  
> Khi trình duyệt thực hiện request hoặc script cố gắng đọc cookie trong bối cảnh **không same-site**, cookie đó **phải bị loại bỏ** — không được gửi, không được đọc.

Nói cách khác:
> Cookie SameSite **chỉ được chia sẻ trong phạm vi cùng site**, ngăn mọi rò rỉ thông tin sang các site khác.

---

## 5. Ý nghĩa bảo mật thực tế

Invariant này bảo vệ:
- **Ngăn rò rỉ cookie xác thực hoặc dữ liệu người dùng** trong các yêu cầu cross-site.  
- **Chống tấn công CSRF và session hijacking**, đảm bảo cookie chỉ có hiệu lực trong phạm vi site gốc.  
- **Củng cố chính sách SameSite của trình duyệt** để bảo vệ quyền riêng tư người dùng.

Nếu invariant này bị vi phạm:
- Cookie có thể bị rò rỉ sang site khác, cho phép kẻ tấn công thực hiện hành vi như session theft hoặc CSRF.

---
