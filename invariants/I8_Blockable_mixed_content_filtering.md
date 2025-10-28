**Invariant I.8 – “Blockable mixed content filtering”**
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.8: “Blockable mixed content filtering”**
> *Every request performed by the browser is either a toplevel request, its URL is potentially trustworthy, or the request context does not prohibit mixed content.*

### Mục tiêu:
Đảm bảo rằng **với mỗi yêu cầu mạng**, ít nhất một trong ba điều kiện phải đúng: ngữ cảnh cho phép mixed content, URL đích là đáng tin cậy (ví dụ HTTPS/WSS), hoặc yêu cầu là top-level (điều hướng chính). Nếu không thỏa, yêu cầu mixed content blockable phải bị cấm.

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `net-request(_, url, _, type, origin, doc-url, ancestors, _, _)@tr_t1` — một yêu cầu mạng được khởi tạo trong ngữ cảnh trình duyệt tại thời điểm `t1`; bao gồm `url` đích, loại `type`, `origin`, `doc-url` (tài liệu khởi tạo), và `ancestors` (danh sách khung cha).
- `doc-sets-settings-prohibits-mixed-security-contexts(origin, doc-url, ancestors)` — predicate kiểm tra ngữ cảnh tài liệu có cấm mixed content hay không (cấu hình bảo mật của document/ancestor).
- `is-url-potentially-trustworthy(url)` — kiểm tra xem URL đích có là nguồn đáng tin cậy (HTTPS/WSS/localhost) hay không.
- `type = main_frame` — chỉ ra yêu cầu là top-level navigation.
- `nil = ancestors` — danh sách ancestors rỗng (không có ancestor, tức là top-level).

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
BLOCKABLE-MIXED-CONTENT-FILTERED(tr) :=
net-request(_, url, _, type, origin, doc-url, ancestors, _, _, _)@tr_t1 →
doc-sets-settings-prohibits-mixed-security-contexts(origin, doc-url, ancestors) ∨
is-url-potentially-trustworthy(url) ∨
(type = main_frame ∧ nil = ancestors)
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `net-request(... )@tr_t1` | Một yêu cầu mạng được khởi tạo trong trình duyệt. |
| `doc-sets-settings-prohibits-mixed-security-contexts(origin, doc-url, ancestors)` | Ngữ cảnh tài liệu hoặc chính sách bảo mật có thể cấm mixed content (do CSP, settings, hoặc ancestor). |
| `is-url-potentially-trustworthy(url)` | URL đích là an toàn (HTTPS, WSS, localhost...). |
| `(type = main_frame ∧ nil = ancestors)` | Yêu cầu là top-level navigation (không có ancestor). |
| `→` | Nếu tiền đề xảy ra thì ít nhất một trong các hậu đề phải đúng; nếu không, invariant bị vi phạm. |

---

### Diễn giải dễ hiểu:

> Mỗi khi trình duyệt thực hiện một yêu cầu, nếu ngữ cảnh trang không cho phép mixed content, thì yêu cầu phải được gửi tới một URL an toàn hoặc là một điều hướng top-level. Nếu không có điều kiện nào đúng, yêu cầu mixed content sẽ bị chặn — đảm bảo trang HTTPS không vô tình tải nội dung blockable từ HTTP.

---

## 5. Ý nghĩa bảo mật thực tế

Invariant này bảo vệ:
- **Ngăn chặn tải nội dung blockable không an toàn** trong các trang an toàn.  
- **Duy trì chính sách mixed-content** của trình duyệt: chỉ cho phép mixed content khi ngữ cảnh cho phép, URL an toàn, hoặc khi là điều hướng chính.  
- **Giảm rủi ro injection và downgrade** bằng cách ngăn tải các script/iframe/CSS HTTP trong trang HTTPS.

Nếu invariant này bị vi phạm:
- Trang an toàn có thể tải nội dung nguy hiểm từ HTTP, dẫn tới injection mã độc, rò rỉ dữ liệu, hoặc các tấn công downgrade.

---
