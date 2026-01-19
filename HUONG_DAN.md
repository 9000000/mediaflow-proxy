# Hướng Dẫn Sử Dụng MediaFlow Proxy (Tiếng Việt)

## 1. Giới Thiệu
**MediaFlow Proxy** là một máy chủ proxy mạnh mẽ chuyên dụng để xử lý và tối ưu hóa các luồng media (video/audio).
Nó giúp bạn:
- Chuyển đổi luồng MPEG-DASH (bao gồm cả stream có DRM ClearKey) sang định dạng HLS phổ biến.
- Tăng tốc độ phát video nhờ tính năng tải trước thông minh (Smart Pre-buffering).
- Làm `Proxy` trung gian để ẩn danh hoặc vượt qua các hạn chế về địa lý.
- Hỗ trợ tốt cho các trình phát IPTV và Stremio.

## 2. Cài Đặt

### Cách 1: Sử Dụng Docker (Khuyên Dùng)
Đây là cách đơn giản và ổn định nhất.

1.  **Chuẩn bị:**
    - Cài đặt [Docker](https://www.docker.com/) và Docker Compose trên máy tính hoặc VPS của bạn.
    - Tải mã nguồn dự án về máy.

2.  **Cấu hình:**
    - Copy file `env.example` thành `.env`:
      ```bash
      cp env.example .env
      ```
    - Mở file `.env` và sửa dòng `API_PASSWORD=your_secure_password` thành mật khẩu của bạn.

3.  **Chạy Server:**
    Mở terminal tại thư mục dự án và chạy:
    ```bash
    docker compose up -d
    ```
    Server sẽ khởi động tại địa chỉ: `http://localhost:8888`

### Cách 2: Chạy Trực Tiếp (Python)
Yêu cầu máy đã cài Python 3.10 trở lên.

1.  **Cài đặt thư viện:**
    ```bash
    pip install -r requirements.txt
    # Hoặc nếu dùng uv
    uv sync
    ```

2.  **Chạy Server:**
    Thiết lập mật khẩu và chạy:
    ```bash
    # Trên Windows (PowerShell)
    $env:API_PASSWORD="mat_khau_cua_ban"; uvicorn mediaflow_proxy.main:app --host 0.0.0.0 --port 8888

    # Trên Linux/Mac
    API_PASSWORD="mat_khau_cua_ban" uvicorn mediaflow_proxy.main:app --host 0.0.0.0 --port 8888
    ```

## 3. Danh Sách Đầy Đủ Biến Môi Trường (Environment Variables)

Dưới đây là bảng chi tiết tất cả các biến môi trường bạn có thể cấu hình trong file `.env`.

### Cấu Hình Cốt Lõi (Core)
| Tên Biến | Mặc Định | Mô Tả |
| :--- | :--- | :--- |
| **`API_PASSWORD`** | *(Trống)* | **(Bắt buộc)** Mật khẩu để bảo vệ API chống truy cập trái phép. |
| `LOG_LEVEL` | `INFO` | Mức độ chi tiết của log (`DEBUG`, `INFO`, `WARNING`, `ERROR`). |

### Giao Diện & Tiện Ích (UI & Features)
| Tên Biến | Mặc Định | Mô Tả |
| :--- | :--- | :--- |
| `DISABLE_HOME_PAGE` | `False` | Tắt giao diện trang chủ (trả về lỗi 403). |
| `DISABLE_DOCS` | `False` | Tắt trang tài liệu API Swagger (`/docs`). |
| `DISABLE_SPEEDTEST` | `False` | Tắt trang kiểm tra tốc độ (`/speedtest`). |
| `ENABLE_STREAMING_PROGRESS` | `False` | Bật hiển thị tiến trình streaming trong log console. |
| `CLEAR_CACHE_ON_STARTUP` | `False` | Xóa sạch bộ nhớ đệm khi khởi động (dùng cho debug/dev). |

### Kết Nối & Proxy (Network)
| Tên Biến | Mặc Định | Mô Tả |
| :--- | :--- | :--- |
| `PROXY_URL` | *(Trống)* | URL Proxy mặc định (VD: `http://user:pass@IP:port` hoặc `socks5://...`). |
| `ALL_PROXY` | `False` | Áp dụng Proxy mặc định cho tất cả các request. |
| `DISABLE_SSL_VERIFICATION_GLOBALLY` | `False` | Tắt kiểm tra SSL/TLS (Không khuyến nghị, chỉ dùng khi cần thiết). |
| `TIMEOUT` | `60` | Thời gian chờ tối đa cho HTTP request (giây). |
| `FORWARDED_ALLOW_IPS` | `127.0.0.1` | Danh sách IP tin cậy để nhận header forwarded (khi chạy sau Nginx/Cloudflare). |
| `USER_AGENT` | *(Chrome)* | User-Agent giả lập được sử dụng khi gửi request. |
| `TRANSPORT_ROUTES` | *(Trống)* | Cấu hình định tuyến nâng cao (JSON) để chỉ định domain nào đi qua proxy nào. |

### Tối Ưu Hóa HLS (HLS Pre-buffering)
| Tên Biến | Mặc Định | Mô Tả |
| :--- | :--- | :--- |
| `ENABLE_HLS_PREBUFFER` | `False` | Bật tính năng tải trước cho luồng HLS. |
| `HLS_PREBUFFER_SEGMENTS` | `5` | Số lượng segment tải trước. Tăng lên giúp mượt hơn nhưng tốn RAM/Băng thông. |
| `HLS_PREBUFFER_CACHE_SIZE` | `50` | Số lượng segment tối đa lưu trong bộ nhớ đệm. |
| `HLS_PREBUFFER_MAX_MEMORY_PERCENT` | `80` | Giới hạn % RAM hệ thống tối đa cho việc cache HLS. |
| `HLS_PREBUFFER_EMERGENCY_THRESHOLD` | `90` | Ngưỡng % RAM khẩn cấp để bắt đầu xóa cache mạnh tay. |
| `HLS_PREBUFFER_INACTIVITY_TIMEOUT` | `60` | Thời gian chờ (giây) để dừng tải playlist khi người dùng không xem nữa. |
| `HLS_SEGMENT_CACHE_TTL` | `300` | Thời gian tồn tại (giây) của segment trong cache. |

### Tối Ưu Hóa DASH (DASH Pre-buffering)
| Tên Biến | Mặc Định | Mô Tả |
| :--- | :--- | :--- |
| `ENABLE_DASH_PREBUFFER` | `True` | Bật tính năng tải trước cho luồng MPEG-DASH. |
| `DASH_PREBUFFER_SEGMENTS` | `5` | Số lượng segment tải trước cho DASH. |
| `DASH_PREBUFFER_CACHE_SIZE` | `50` | Số lượng tối đa segment DASH lưu trong cache. |
| `DASH_PREBUFFER_MAX_MEMORY_PERCENT` | `80` | Giới hạn % RAM cho cache DASH. |
| `DASH_PREBUFFER_EMERGENCY_THRESHOLD` | `90` | Ngưỡng % RAM khẩn cấp cho DASH. |
| `DASH_PREBUFFER_INACTIVITY_TIMEOUT` | `60` | Thời gian chờ (giây) khi không hoạt động. |
| `DASH_SEGMENT_CACHE_TTL` | `60` | Thời gian sống (TTL) của cache segment DASH. |
| `MPD_LIVE_INIT_CACHE_TTL` | `60` | TTL cho file khởi tạo live stream. |
| `MPD_LIVE_PLAYLIST_DEPTH` | `8` | Số lượng segment hiển thị trong danh sách live stream. |

### Tích Hợp Stremio
| Tên Biến | Mặc Định | Mô Tả |
| :--- | :--- | :--- |
| `STREMIO_PROXY_URL` | *(Trống)* | URL Stremio server để proxy nội dung khác nếu cần. |
| `M3U8_CONTENT_ROUTING` | `mediaflow` | Cách xử lý link M3U8: `mediaflow` (qua proxy), `stremio`, hoặc `direct` (trực tiếp). |

## 4. Sử Dụng

Sau khi server chạy, bạn có các công cụ sau:

- **Trang chủ:** `http://localhost:8888/` (Kiểm tra server đang chạy).
- **Tài liệu API:** `http://localhost:8888/docs`
    - Đây là nơi quan trọng nhất. Nó liệt kê tất cả các đường dẫn (endpoint) bạn có thể dùng để get link, xem phim, v.v.
    - Bạn có thể nhập mật khẩu API vào nút "Authorize" để thử nghiệm trực tiếp các lệnh.
- **Kiểm tra tốc độ:** `http://localhost:8888/speedtest` (Test tốc độ kết nối đến RealDebrid/AllDebrid).

## 5. Mẹo Tối Ưu
- **Máy cấu hình thấp (RAM < 1GB):** Hãy tắt `ENABLE_HLS_PREBUFFER` và `ENABLE_DASH_PREBUFFER` hoặc giảm `PREBUFFER_SEGMENTS` xuống còn 2-3.
- **Mạng chậm:** Tăng `TIMEOUT` lên cao hơn (ví dụ 120s) để tránh lỗi ngắt kết nối giữa chừng.
