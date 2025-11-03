# WPT Check

`./wpt-check` là một script quản lý và chạy các bài kiểm tra **Web Platform Tests (WPT)** sử dụng Docker.

---

## Yêu cầu

- Docker đã được cài đặt và user hiện tại có quyền chạy Docker:

```bash
docker info
```

- Nếu chưa có quyền, thêm user vào nhóm docker:

```bash
sudo usermod -aG docker $USER
```

- Tải các docker images

```bash
./wpt-check pull
```

script sử dụng các Docker images từ GitHub Container Registry của `oobbooz`
| Image | Mục đích | Nguồn tải |
| -------------------- | ----------------------------------------- | ------------------------------------------- |
| `wpt-runner` | Chạy các bài test trên trình duyệt | `ghcr.io/oobbooz/wpt-runner:latest` |
| `wpt-trace-matching` | Phân tích trace `.json` từ quá trình test | `ghcr.io/oobbooz/wpt-trace-matching:latest` |
| `wpt-safari-runner` | (Tùy chọn) Chạy test trên Safari | `ghcr.io/oobbooz/wpt-safari-runner:latest` |

- Trace .json
  File trace `.json` được lưu trên host, thư mục mặc định `traces/ `
