---
name: mainhi-veo3-integration
description: Hướng dẫn tích hợp, tự động hóa và điều khiển API cục bộ (cổng 8001) của Mai Nhi Veo3 Tool để tạo ảnh và video từ prompt bằng Python hoặc n8n.
---

# Hướng dẫn Kỹ năng: Tích Hợp & Điều Khiển API Mai Nhi Veo3 Tool

Kỹ năng này giúp Agent nhận diện, kiểm tra trạng thái và tương tác tự động với API của **Mai Nhi Veo3 Tool** đang chạy cục bộ trên máy của người dùng.

## 1. Nhận Diện Môi Trường Hoạt Động
- Ứng dụng chính: `Mai Nhi Veo3 Tool 3.0.8.exe`
- Cổng dịch vụ OpenAI Wrapper: `http://127.0.0.1:8001`
- Trạng thái hoạt động: Server HTTP Fast API chạy ngầm và kết nối với Chrome Extension (Google Flow) qua WebSocket (cổng `9227`).

## 2. Các Endpoint Tích Hợp (Chuẩn OpenAI API)
Tất cả các phần mềm tự động hóa (n8n, Make) hoặc Client API chỉ cần cấu hình Base URL trỏ về `http://127.0.0.1:8001/v1`.

### 2.1 Tạo Ảnh (Image Generation)
- **POST** `/v1/images/generations`
- **Payload cấu hình**:
  - `prompt`: Chuỗi mô tả ảnh.
  - `size`: `"1280x720"` (Tương đương tỷ lệ ngang 16:9) hoặc `"1024x1024"` (Vuông).
  - `model`: `"narwhal"` hoặc `"gem_pix_2"`.

### 2.2 Tạo Video (Video Generation)
- **POST** `/v1/videos/generations`
- **Payload cấu hình**:
  - `prompt`: Chuỗi mô tả chuyển động video.
  - `aspect`: `"portrait"` (dọc) hoặc `"landscape"` (ngang).
  - `duration`: `4`, `6`, `8` hoặc `10` (số giây, mặc định khuyên dùng là 8 giây).
  - `image_base64` (Tùy chọn cho luồng Image-to-Video): Chuỗi mã hóa Base64 của ảnh gốc kèm tiền tố `data:image/png;base64,`. Tiện ích sẽ lấy ảnh này làm khung hình xuất phát để tạo video.

## 3. Quy Trình Tự Động Tải & Lưu Kết Quả
Mỗi lần gửi lệnh tạo thành công, API sẽ trả về JSON chứa URL tải file cục bộ (ví dụ: `http://127.0.0.1:8001/download/flowagent_img_xxx.png`).

**Quy trình tải tự động bằng Python:**
```python
import os
import requests

response = requests.post("http://127.0.0.1:8001/v1/images/generations", json=payload)
if response.status_code == 200:
    url = response.json()['data'][0]['url']
    file_bytes = requests.get(url).content
    with open(os.path.join(save_directory, "output.png"), "wb") as f:
        f.write(file_bytes)
```

## 4. Troubleshooting
Nếu API trả về lỗi `503 - Google Flow extension is not connected`:
- Kiểm tra xem Google Chrome đã được mở chưa.
- Kiểm tra xem Chrome đã truy cập vào Google Flow/VideoFX và Extension Mai Nhi đã được bật chưa.

## 5. Lưu Ý Quan Trọng (Pitfalls) — bắt buộc đọc trước khi gen

- **reCAPTCHA `UNUSUAL_ACTIVITY` (HTTP 400 `reCAPTCHA evaluation failed`):** Google Flow khóa TẠM THỜI nếu gen liên tục/nhanh (batch, loop tight). KHÔNG phải ban vĩnh viễn — thường tự hết sau vài chục phút đến vài tiếng. Khắc phục: (1) **mô phỏng hành vi người dùng NGAY CẢ KHI TEST** — gen 1–2 ảnh/rần, delay random 30–90s giữa 2 lần, tuyệt đối không spam; (2) thao tác tay nhẹ trên tab Flow (scroll/click tool) hoặc reload extension (chrome://extensions) để lấy session key mới; (3) nếu hiện captcha thì giải tay. Giảm tần suất là tự hết.
- **WSL không gọi được loopback Windows:** Từ WSL, `curl http://127.0.0.1:8001` trả `Connection refused` (WSL2 có network namespace riêng, không thấy loopback của Windows host) dù `netstat` Windows hiện port LISTENING. PHẢI gọi script gen từ phía Windows: `cmd.exe /c "C:\venv\Scripts\python.exe C:\path\gen.py"`. Serve chạy trên Windows → client cũng phải chạy trên Windows. Xem `references/gen_protocol.md`.
- **Prompt bắt buộc tiếng Anh:** API reject non-ASCII (422/400). Luôn viết prompt tiếng Anh, lưu file `.txt` rồi truyền `--prompt-file` (hoặc payload JSON) để tránh lỗi quote tiếng Việt trong cmd.exe.
- **`narwhal` = Nano Banana 2, ảnh KHÔNG tốn credit.** Chỉ video (`/v1/videos/generations`) mới tốn credit.
- **Thumbnail AI Music = CHỈ dùng Nano Banana (narwhal) qua skill này.** KHÔNG dùng Flux-4B (302.ai) trừ khi anh Hoài yêu cầu cụ thể — đã bị loại khỏi kế hoạch dự án (yêu cầu 2026-07-20).

> Chi tiết quy trình gen + script pattern + health check: xem `references/gen_protocol.md`.
