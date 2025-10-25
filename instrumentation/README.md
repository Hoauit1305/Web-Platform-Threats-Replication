# Phân tích Công cụ Đo lường (Instrumentation)

Thư mục này chứa logic để "đo lường" (instrument) các trình duyệt, dựa trên **Mục 4.3** của bài báo khoa học.

Mục tiêu chính của các công cụ này là **thu thập Dấu vết Thực thi (Execution Traces)**—một bản ghi chi tiết về mọi hành động liên quan đến bảo mật mà trình duyệt thực hiện khi chạy các kiểm thử WPT.

## 1. Kiến trúc Tổng thể

Phương pháp đo lường này bao gồm hai thành phần chính hoạt động song song để thu được bức tranh đầy đủ nhất:

1.  **Tiện ích mở rộng Trình duyệt (Browser Extension):** Giám sát các sự kiện và API *bên trong* trình duyệt.
2.  **Proxy bên ngoài (External Proxy):** Giám sát tất cả lưu lượng mạng *bên ngoài* trình duyệt.

---

## 2. Thành phần 1: Tiện ích mở rộng Trình duyệt

Tiện ích mở rộng được thiết kế để chạy ngầm và được chia thành hai phần với các nhiệm vụ riêng biệt:

### A. Background Script (Tập lệnh nền)

* **Vai trò:** Đây là "trung tâm giám sát" chính, chạy liên tục ở chế độ nền.
* **Nhiệm vụ:** Sử dụng các API cấp cao của trình duyệt để theo dõi trạng thái nội bộ.
    * **Giám sát Mạng:** Lắng nghe các sự kiện mạng (ví dụ: khi một yêu cầu sắp được gửi hoặc khi nhận được phản hồi).
    * **Giám sát Cookie:** Theo dõi kho cookie (CookieJar) của trình duyệt để phát hiện các sự kiện thiết lập, xóa, hoặc sửa đổi cookie.

### B. Content Script (Tập lệnh nội dung)

* **Vai trò:** Một đoạn mã được "tiêm" (inject) vào mọi trang web (và mọi `<iframe>`) ngay khi chúng bắt đầu tải.
* **Nhiệm vụ:** Theo dõi các lệnh gọi API JavaScript mà trang web thực thi.
    * **Bọc API (API Wrapping):** Nó sử dụng các kỹ thuật như `Proxy` để "bọc" các API nhạy cảm, đặc biệt là `document.cookie`.
    * **Gửi dữ liệu:** Khi phát hiện một API được gọi (ví dụ: `document.cookie = '...'`), nó sẽ ghi lại chi tiết và gửi thông tin này về cho Background Script để tổng hợp.

---

## 3. Thành phần 2: Proxy bên ngoài

Đây là một máy chủ proxy mà tất cả lưu lượng truy cập của trình duyệt đều phải đi qua.

* **Vai trò:** Hoạt động như một "người trung gian" (man-in-the-middle) để ghi lại tất cả các gói tin HTTP/HTTPS.
* **Nhiệm vụ:**
    1.  **Khắc phục Hạn chế:** Ghi lại các thông tin mà API của tiện ích mở rộng có thể bỏ lỡ hoặc bị ẩn (ví dụ: một số header nhạy cảm, nội dung request/response body).
    2.  **Cung cấp Dấu thời gian (Timestamp):** Cung cấp dấu thời gian chính xác hơn cho các sự kiện mạng, giúp sắp xếp thứ tự các sự kiện trong dấu vết cuối cùng một cách đáng tin cậy.

---

## 4. Luồng hoạt động (Data Flow)

1.  Trình duyệt được cấu hình để gửi tất cả lưu lượng truy cập qua **Proxy**.
2.  Khi một kiểm thử WPT chạy:
    * **Proxy** ghi lại yêu cầu mạng.
    * **Background Script** (Extension) cũng ghi lại sự kiện mạng đó (từ API trình duyệt).
    * **Content Script** (Extension) ghi lại bất kỳ lệnh gọi API JavaScript nào (như `document.cookie`).
3.  Tất cả các mẩu thông tin (từ Proxy, Background Script, và Content Script) được thu thập và kết hợp lại thành một tệp **JSON Dấu vết** (Trace File) duy nhất, chứa một chuỗi các sự kiện đã được sắp xếp theo thời gian.