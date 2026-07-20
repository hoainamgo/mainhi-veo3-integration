# mainhi-veo3-integration

Hermes skill: tích hợp & điều khiển API cục bộ (cổng `8001`) của **Mai Nhi Veo3 Tool** để tạo ảnh và video từ prompt bằng Python hoặc n8n.

## Tóm tắt
- Ứng dụng: `Mai Nhi Veo3 Tool 3.0.8.exe`
- Cổng OpenAI Wrapper: `http://127.0.0.1:8001`
- Kết nối Chrome Extension (Google Flow) qua WebSocket (`9227`)

## Endpoints (chuẩn OpenAI)
- `POST /v1/images/generations` — tạo ảnh (`model`: `narwhal` | `gem_pix_2`, `size`: `1280x720` | `1024x1024`)
- `POST /v1/videos/generations` — tạo video (`aspect`: `portrait`|`landscape`, `duration`: 4/6/8/10)

Chi tiết xem trong [SKILL.md](SKILL.md).
