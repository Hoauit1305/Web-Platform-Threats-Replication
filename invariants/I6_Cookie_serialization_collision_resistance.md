**Invariant I.6 – “Cookie serialization collision resistance”**
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.6: “Cookie serialization collision resistance”**

> * A cookie with name n and value v is serialized to the string "n=v" when attached to requests or accessed via `document.cookie`.*

### Mục tiêu:
Đảm bảo rằng khi trình duyệt **gửi cookie trong request HTTP hoặc cung cấp cookie cho script**, cookie đó phải được **tuần tự hóa (serialize)** chính xác và duy nhất trong chuỗi cookie (`Cookie` header hoặc `document.cookie`).

Invariant này nhằm bảo vệ tính **toàn vẹn (integrity)** của quá trình tuần tự hóa cookie, đảm bảo không có lỗi chồng lặp hoặc ghi đè giá trị cookie sai khi gửi hoặc đọc.

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `cookie-jar-set(name, value, {secure, same-site, path, domain})@tr_t1` — sự kiện lưu cookie vào cookie jar với các thuộc tính liên quan.  
- `net-request(_, url, method, type, origin-url, _, _, redirs, {cookies}, _)@tr_t2` — sự kiện gửi HTTP request có chứa cookie.  
- `cookie-should-be-sent(path, domain, secure, same-site, type, origin-url, url, method, redirs)` — kiểm tra xem cookie có đủ điều kiện được gửi trong request không.  
- `js-get-cookie(ctx, cookies)@tr_t2` — sự kiện script đọc cookie thông qua `document.cookie`.  
- `url = ctx-location(ctx)` — xác định URL của ngữ cảnh script.  
- `cookie-match(path, domain, secure, url)` — kiểm tra rằng cookie phù hợp với URL hoặc ngữ cảnh.  
- `is-effective-cookie(t2, tr, name, value, domain, path, "")` — kiểm tra rằng cookie đang xét là “effective cookie” hợp lệ tại thời điểm t2.  
- `name ++ "=" ++ value ∈ split-cookie(cookies)` — cookie xuất hiện đúng trong chuỗi cookie.

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
COOKIE-SERIALIZATION-INVARIANT(tr) :=
t2 > t1 ∧
cookie-jar-set(name, value, {secure, same-site, path, domain})@tr_t1 ∧
(
  (net-request(_, url, method, type, origin-url, _, _, redirs, {cookies}, _)@tr_t2 ∧
   cookie-should-be-sent(path, domain, secure, same-site, type, origin-url, url, method, redirs)) ∨
  (js-get-cookie(ctx, cookies)@tr_t2 ∧ url = ctx-location(ctx) ∧
   cookie-match(path, domain, secure, url))
) ∧
is-effective-cookie(t2, tr, name, value, domain, path, "") →
name ++ "=" ++ value ∈ split-cookie(cookies)
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `t2 > t1` | Cookie được ghi vào cookie jar trước khi nó được gửi hoặc truy cập. |
| `cookie-jar-set(...)@tr_t1` | Cookie được lưu trữ với các thuộc tính cụ thể. |
| `net-request(..., {cookies})@tr_t2` | Một HTTP request chứa cookie được gửi đi. |
| `cookie-should-be-sent(...)` | Cookie hợp lệ và nên được gửi cùng với request đó. |
| `js-get-cookie(...)@tr_t2` | Một script đọc cookie thông qua `document.cookie`. |
| `cookie-match(...)` | Cookie tương ứng với URL hoặc ngữ cảnh script hiện tại. |
| `is-effective-cookie(...)` | Cookie hiện tại hợp lệ trong bộ cookie của trình duyệt. |
| `→ name ++ "=" ++ value ∈ split-cookie(cookies)` | Kết luận: cookie phải xuất hiện chính xác trong chuỗi cookie được gửi hoặc đọc. |

---

## Diễn giải dễ hiểu:

> Khi một cookie được gửi trong request hoặc được đọc bởi JavaScript, cookie đó **phải được tuần tự hóa chính xác** trong chuỗi cookie (`Cookie` header hoặc `document.cookie`).  
> Nếu cookie hợp lệ (effective cookie) nhưng không xuất hiện hoặc xuất hiện sai trong chuỗi cookie, thì invariant bị vi phạm.

Nói cách khác:
> Mỗi cookie hợp lệ phải được thể hiện đúng dạng `name=value` trong cookie string khi được gửi hoặc đọc.

---

## 5. Ý nghĩa bảo mật thực tế

Invariant này bảo vệ:
- **Tính toàn vẹn của dữ liệu cookie** khi truyền qua mạng hoặc truy cập nội bộ.  
- **Ngăn chặn lỗi trùng lặp hoặc sai giá trị cookie** khi serialize (ví dụ `Cookie: session=abc; session=xyz`).  
- **Đảm bảo hành vi thống nhất giữa cookie trong bộ nhớ và cookie trong header**.

Nếu invariant này bị vi phạm:
- Có thể dẫn tới **lỗi định danh phiên (session confusion)**, **ghi đè dữ liệu cookie**, hoặc **tấn công qua lỗi serialize**.

---
