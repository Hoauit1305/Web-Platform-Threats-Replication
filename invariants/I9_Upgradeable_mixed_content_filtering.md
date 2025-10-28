**Invariant I.9 – “Upgradeable mixed content filtering”**
---

## 1. Mục tiêu bảo mật (Security Goal)

**Invariant I.9:**  
> For every non-toplevel network request performed by the browser whose URL is not potentially trustworthy, the request context does not prohibit mixed content or the request type is not upgradeable.

### Mục tiêu:
Đảm bảo rằng với **mọi yêu cầu mạng không phải top-level (ví dụ iframe, subresource, script)** mà URL của nó **không đáng tin cậy (HTTP, không mã hóa)** — thì:
- Hoặc là ngữ cảnh của trang không cấm mixed content, **hoặc**
- Loại yêu cầu đó **không phải là loại có thể nâng cấp** (non-upgradeable).

Invariant này nhằm đảm bảo chính sách mixed content được trình duyệt tuân thủ chặt chẽ khi xử lý các yêu cầu không an toàn ở cấp thấp hơn (sub-frame hoặc resource).

---

## 2. Các sự kiện (Events) và thuộc tính (Predicates) liên quan

- `net-request(_, url, _, type, origin, doc-url, ancestors, _, _)@tr_t1` — yêu cầu mạng được khởi tạo trong trình duyệt.  
- `¬is-url-potentially-trustworthy(url)` — URL đích **không đáng tin cậy** (HTTP, FTP, vv.).  
- `type ≠ main_frame` — yêu cầu **không phải top-level navigation** (ví dụ subresource).  
- `does-settings-prohibits-mixed-security-contexts(origin, doc-url, ancestors)` — ngữ cảnh tài liệu hiện tại cấm mixed content.  
- `¬is-mixed-content-upgradeable(type)` — loại yêu cầu không thể được nâng cấp từ HTTP → HTTPS (non-upgradeable).  

---

## 3. Công thức logic bậc nhất (FOL Formula)

```
UPGRADEABLE-MIXED-CONTENT-FILTERED(tr) :=
net-request(_, url, _, type, origin, doc-url, ancestors, _, _)@tr_t1 ∧
¬is-url-potentially-trustworthy(url) ∧
type ≠ main_frame →
(¬does-settings-prohibits-mixed-security-contexts(origin, doc-url, ancestors) ∨
 ¬is-mixed-content-upgradeable(type))
```

---

## 4. Giải thích logic từng phần

| Thành phần | Ý nghĩa |
|------------|---------|
| `net-request(... )@tr_t1` | Một yêu cầu mạng được khởi tạo trong trình duyệt. |
| `¬is-url-potentially-trustworthy(url)` | URL đích không đáng tin cậy (HTTP, không mã hóa). |
| `type ≠ main_frame` | Yêu cầu không phải điều hướng chính (ví dụ: ảnh, script, iframe). |
| `¬does-settings-prohibits-mixed-security-contexts(...)` | Ngữ cảnh hiện tại không cấm mixed content. |
| `¬is-mixed-content-upgradeable(type)` | Loại nội dung không thể nâng cấp từ HTTP → HTTPS. |
| `→` | Nếu tất cả các điều kiện trên đúng, ít nhất một trong các hậu đề phải đúng; nếu không, invariant bị vi phạm. |

---

## Diễn giải dễ hiểu:

> Mỗi khi trình duyệt tải một tài nguyên phụ (subresource) qua kết nối không an toàn (HTTP), nếu trang mẹ đang cấm mixed content và loại tài nguyên đó có thể nâng cấp (ví dụ script, image...), thì yêu cầu này **phải bị chặn**.  
> Tuy nhiên, nếu trang **không cấm mixed content**, hoặc loại tài nguyên **không thể nâng cấp**, thì trình duyệt có thể tiếp tục xử lý.

Nói cách khác:
> Trình duyệt chỉ được phép tải tài nguyên không an toàn khi điều đó **không vi phạm chính sách mixed content** hoặc **loại tài nguyên không thể nâng cấp an toàn**.

---

## 5. Ý nghĩa bảo mật thực tế

Invariant này bảo vệ:
- **Ngăn chặn tải nội dung không an toàn (HTTP)** trong các ngữ cảnh HTTPS nghiêm ngặt.  
- **Duy trì chính sách mixed content upgradeable**: chỉ cho phép nâng cấp tài nguyên an toàn khi có thể.  
- **Giảm nguy cơ tấn công downgrade hoặc injection** từ nội dung HTTP không mã hóa.

Nếu invariant này bị vi phạm:
- Trình duyệt có thể tải script, iframe hoặc tài nguyên HTTP trong trang HTTPS → gây rò rỉ dữ liệu hoặc tấn công chèn mã.

---
