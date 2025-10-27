# Chi tiết Triển khai Tiện ích mở rộng

## 1. Lý do Phân tách Thư mục (Chromium, Firefox, Safari)

Mặc dù kiến trúc tổng thể (Background Script + Content Script) là giống nhau, chúng ta bắt buộc phải chia mã nguồn thành 3 thư mục riêng biệt do **sự khác biệt cơ bản về API và tệp kê khai (manifest)** giữa các trình duyệt.

### A. Manifest V3 (Chromium) vs. Manifest V2 (Firefox)

Đây là khác biệt lớn nhất:
* **Chromium (`/chromium`)**: Sử dụng **Manifest V3**.
    * **Tập lệnh nền:** Bắt buộc phải là `service_worker` (`ext.js` trong trường hợp này). Service worker có vòng đời (lifecycle) khác, không chạy liên tục và bị tắt khi không dùng, điều này ảnh hưởng đến cách chúng ta duy trì trạng thái.
* **Firefox (`/firefox`)**: Tại thời điểm bài báo (hoặc code) được viết, Firefox vẫn chủ yếu sử dụng **Manifest V2**.
    * **Tập lệnh nền:** Sử dụng `background_page` (hoặc `background_script`), là một trang chạy liên tục, giữ trạng thái dễ dàng hơn.

### B. Khác biệt API và Kỹ thuật Cốt lõi

* **API Namespace:** Chromium dùng `chrome.*` (ví dụ: `chrome.webRequest`), trong khi Firefox dùng `browser.*` (ví dụ: `browser.webRequest`). Mặc dù có thể dùng polyfill, việc tách riêng code giúp quản lý các API đặc thù dễ dàng hơn.
* **API `"world": "MAIN"` (Content Script):** Kỹ thuật then chốt để "bọc" (wrap) `document.cookie` này có cách triển khai và các vấn đề bảo mật khác nhau giữa các trình duyệt, đòi hỏi mã nguồn riêng để xử lý.
* **Safari (`/safari`):** Safari WebExtensions có những hạn chế riêng. Như bài báo đã đề cập (Mục 4.3.2), việc giám sát CookieJar trên Safari khi chạy qua WebDriver là không thể (dẫn đến bỏ sót), đòi hỏi một phiên bản extension đặc thù, hoặc chấp nhận thiếu sót dữ liệu.

<!-- ## 2. Các Phát hiện Kỹ thuật (Từ Mã nguồn)

Trong quá trình triển khai, có một số lỗi kỹ thuật đã được lường trước và xử lý:

* **Lỗi `TypeError: Invalid URL`:** Lỗi này xảy ra trong tập lệnh nền (`ext.js`) khi cố gắng phân tích `details.documentUrl`. Nó được bọc trong `try...catch` vì nhiều yêu cầu (request) của trình duyệt (ví dụ: `about:blank`) không có URL tài liệu hợp lệ.
* **Lỗi `Duplicate script ID`:** Lỗi này xuất hiện khi tải extension là do code bị "thừa". Script `instr.js` được khai báo tải 2 lần:
    1.  Một lần trong `manifest.json`.
    2.  Một lần nữa bằng code JavaScript trong `ext.js`.
    
    Đây có thể là một đoạn code gỡ lỗi (debug) còn sót lại và có thể xóa bỏ an toàn. -->