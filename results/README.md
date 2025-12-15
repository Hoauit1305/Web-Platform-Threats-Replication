1. Cấu trúc Thư mục

- chromium/, firefox/, safari/: Chứa các kết quả đã chạy pipeline cho từng trình duyệt. Bên trong mỗi thư mục trình duyệt:
  + Các thư mục con (ví dụ: cookie-serialization-invariant/) chứa dấu vết thực thi (:json) và đầu ra Z3 (:z3) tương ứng đã được đánh dấu là SAT (Satisfiable).
- check-results.py: Script Python tự động hóa việc thực hiện các thử nghiệm.
- results.json: Tệp JSON tổng hợp kết quả (Mục 5.1), được phân loại theo [browser][invariant][test].

2. Script check-results.py

- Để xác minh toàn bộ kết quả trong Bảng 2, Mục 5.1 của bài báo, chạy lệnh sau:

```
./check-results.py
```

- Hoặc có thể chạy thử nghiệm cho từng trình duyệt một:

```
./check-results.py -b <browser>

```
