# Hướng dẫn tích hợp Watch CLI vào Hermes Agent

Hướng dẫn này giúp bạn cấu hình và sử dụng **Watch CLI** như một skill/tool trong hệ thống **Hermes Agent**.

## Tổng quan

Watch CLI là công cụ phân tích video tự động dành cho AI agent. Nó kết hợp `yt-dlp` + `ffmpeg` + Whisper-class ASR để trích xuất khung hình (frames) và tạo phụ đề (transcript) từ video trên các nền tảng mạng xã hội như YouTube, X/Twitter, LinkedIn, TikTok, Vimeo, Reddit, Facebook. Khi tích hợp vào Hermes Agent, nó trở thành một skill mạnh mẽ để "xem" và phân tích video tự động.

## Cách 1: Cài đặt như một External Tool

### Bước 1: Cài đặt Watch CLI

#### Trên macOS/Linux

```bash
# Clone repository
git clone https://github.com/your-org/watch-cli.git
cd watch-cli

# Chạy script cài đặt
chmod +x install.sh
./install.sh
```

#### Trên Windows

**Phương pháp 1: Sử dụng Git Bash hoặc WSL (khuyến nghị)**

```bash
# Mở Git Bash hoặc WSL terminal
git clone https://github.com/your-org/watch-cli.git
cd watch-cli

# Chạy script cài đặt
chmod +x install.sh
./install.sh
```

**Phương pháp 2: Cài đặt thủ công trên Windows PowerShell**

1. **Cài đặt các dependencies:**

   ```powershell
   # Cài đặt Python (nếu chưa có)
   # Tải từ https://www.python.org/downloads/
   
   # Cài đặt yt-dlp
   pip install yt-dlp
   
   # Cài đặt ffmpeg
   # Tải từ https://ffmpeg.org/download.html
   # Hoặc sử dụng chocolatey: choco install ffmpeg
   ```

2. **Clone và cấu hình:**

   ```powershell
   # Clone repository
   git clone https://github.com/your-org/watch-cli.git
   cd watch-cli
   
   # Thêm thư mục vào PATH
   $env:Path += ";C:\path\to\watch-cli\bin"
   
   # Hoặc copy executable vào thư mục trong PATH
   Copy-Item bin\watch.exe C:\Users\<YourUser>\AppData\Local\Programs\Python\Python39\Scripts\
   ```

3. **Kiểm tra cài đặt:**

   ```powershell
   watch --version
   ```

### Bước 2: Cấu hình trong Hermes Agent

Thêm cấu hình tool vào file config của Hermes Agent (thường là `config.yaml` hoặc `hermes.config.json`):

```yaml
# hermes.config.yaml
tools:
  watch-cli:
    enabled: true
    path: "/path/to/watch-cli/bin/watch"
    default_frames: 8
    max_frames: 64
```

Hoặc định dạng JSON:

```json
{
  "tools": {
    "watch-cli": {
      "enabled": true,
      "path": "/usr/local/bin/watch",
      "default_frames": 8,
      "max_frames": 64
    }
  }
}
```

### Bước 3: Định nghĩa Skill Manifest

Tạo file manifest để Hermes Agent nhận diện skill:

```yaml
# skills/watch-cli/manifest.yaml
name: watch-cli
version: 1.0.0
description: Video analysis tool that extracts frames and generates transcripts from social media videos
author: Watch CLI Team

entry_point: /usr/local/bin/watch

commands:
  - name: analyze
    description: Analyze a video URL to extract frames and generate transcript
    parameters:
      - name: url
        type: string
        required: true
        format: uri
        description: A social video URL (YouTube, X/Twitter, LinkedIn, TikTok, Vimeo, Reddit, Facebook)
      - name: frames
        type: integer
        required: false
        default: 8
        minimum: 1
        maximum: 64
        description: Number of evenly-spaced frames to extract from the video

capabilities:
  - video-download
  - frame-extraction
  - speech-to-text
  - multi-platform-support
  - cookie-authentication
  - offline-mode

rate_limits:
  requests_per_minute: 30
  max_frames: 64
```

## Cách 2: Tích hợp như MCP Server (Model Context Protocol)

Nếu Hermes Agent hỗ trợ MCP, bạn có thể chạy Watch CLI như một MCP server:

### Bước 1: Cài đặt MCP Server

#### Trên macOS/Linux

```bash
cd mcp-server
npm install
npm run build
```

#### Trên Windows

**Sử dụng PowerShell hoặc Command Prompt:**

```powershell
# Đảm bảo đã cài Node.js (tải từ https://nodejs.org/)

# Di chuyển vào thư mục mcp-server
cd mcp-server

# Cài đặt dependencies
npm install

# Build project
npm run build

# Kiểm tra cài đặt
node dist/index.js --version
```

**Lưu ý cho Windows:**
- Đảm bảo `node` và `npm` đã được thêm vào PATH
- Nếu gặp lỗi permissions, chạy PowerShell với quyền Administrator
- Có thể cần cài đặt Visual C++ Redistributable nếu có lỗi khi build

### Bước 2: Cấu hình MCP trong Hermes

```yaml
# hermes.config.yaml
mcp_servers:
  watch-cli:
    command: node
    args:
      - /path/to/watch-cli/mcp-server/dist/index.js
    env:
      WATCH_CLI_API_KEY: your_api_key_here
```

### Bước 3: Khởi động Hermes Agent

```bash
hermes-agent --config hermes.config.yaml
```

## Sử dụng Watch CLI Skill trong Hermes Agent

### Ví dụ 1: Phân tích video YouTube về lập trình

```
User: Phân tích video này cho tôi: https://youtube.com/watch?v=abc123
Hermes: [Sử dụng watch-cli analyze] Đang tải video và trích xuất khung hình...

📹 Video Analysis Complete
- Frames extracted: 8
- Duration: 12:34
- Transcript: 1,523 words
- Language: English

Key topics detected: React hooks, component lifecycle, state management
```

### Ví dụ 2: Trích xuất kiến trúc hệ thống từ video tech talk

```
User: Trích xuất architecture diagram từ video: https://twitter.com/techconf/status/456789
Hermes: [Sử dụng watch-cli analyze --frames 16] Đang phân tích video...

🏗️ Architecture Diagram Extracted
- Components identified: API Gateway, Microservices, Database Cluster
- Data flow: Request → Gateway → Service A → DB
- Transcript available: Yes (2,341 words)
```

### Ví dụ 3: Tạo phụ đề cho video TikTok

```
User: Tạo transcript cho video TikTok: https://tiktok.com/@user/video/789012
Hermes: [Sử dụng watch-cli analyze] Đang xử lý video...

📝 Transcript Generated
- Duration: 0:58
- Words: 145
- Language: Vietnamese
- Confidence: 94.2%
```

### Ví dụ 4: Phân tích video hướng dẫn UI/UX

```
User: Phân tích UX pattern từ video: https://vimeo.com/123456789
Hermes: [Sử dụng watch-cli analyze --frames 12] Đang trích xuất frames...

🎨 UX Analysis Ready
- Frames: 12 key moments captured
- Interactions detected: swipe, tap, scroll animations
- Transcript: Full narration transcribed
- Suggested artifact: React component with Framer Motion
```

## Xử lý lỗi thường gặp

### Lỗi: Tool not found
```
Solution: Kiểm tra path trong cấu hình Hermes Agent
hermes.config.yaml:
  tools:
    watch-cli:
      path: "/usr/local/bin/watch"  # Đảm bảo path đúng
```

### Lỗi: Invalid video URL
```
Solution: Đảm bảo URL là từ nền tảng được hỗ trợ
Hỗ trợ: YouTube, X/Twitter, LinkedIn, TikTok, Vimeo, Reddit, Facebook
```

### Lỗi: Download failed (tag=download-auth)
```
Solution: Video yêu cầu đăng nhập. Cấu hình cookies:
- Watch CLI tự động lấy cookies từ browser đã đăng nhập
- Hoặc export cookies manually và đặt vào thư mục ~/.watch-cli/cookies/
```

### Lỗi: Transcript null (exit_code=4)
```
Solution: ASR model không tạo được transcript (video không có audio hoặc language không supported)
- Kiểm tra video có audio không
- Frames vẫn được trích xuất bình thường, chỉ transcript = null
```

### Lỗi: npm install thất bại (trên Windows)
```
Solution: 
1. Chạy PowerShell với quyền Administrator
2. Xóa thư mục node_modules và cài lại:
   Remove-Item -Recurse -Force node_modules
   npm install
3. Cài đặt Visual C++ Redistributable nếu cần
```

### Lỗi: ffmpeg không tìm thấy (trên Windows)
```
Solution:
1. Tải ffmpeg từ https://ffmpeg.org/download.html
2. Giải nén vào C:\ffmpeg
3. Thêm C:\ffmpeg\bin vào PATH:
   [Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\ffmpeg\bin", "User")
4. Khởi động lại terminal
```

## Xử lý lỗi thường gặp trên Windows

### Lỗi: Tool not found (Windows)
```powershell
# Kiểm tra xem watch có trong PATH không
where watch

# Nếu không tìm thấy, cập nhật path trong hermes.config.json:
{
  "tools": {
    "watch-cli": {
      "path": "C:\\Users\\<YourUser>\\AppData\\Local\\Programs\\Python\\Python39\\Scripts\\watch.exe"
    }
  }
}
```

### Lỗi: Cookie authentication (Windows)
```powershell
# Đường dẫn thư mục cookies trên Windows:
$env:USERPROFILE\.watch-cli\cookies\

# Export cookies từ trình duyệt Chrome/Edge và lưu vào file .txt
```

## Best Practices

1. **Chọn số frames hợp lý**: 
   - Video ngắn (< 2 phút): 4-8 frames
   - Video trung bình (2-10 phút): 8-16 frames
   - Video dài (> 10 phút): 16-32 frames
   - Tối đa: 64 frames

2. **Sử dụng cookie authentication** cho video yêu cầu đăng nhập (LinkedIn, private X, FB)

3. **Enable offline mode** khi cần phân tích lại video đã tải

4. **Lưu logs** để debug và audit

5. **Kết hợp với prompts** trong thư mục `prompts/` để tạo artifact cụ thể:
   - implement-from-video.md → code project
   - extract-architecture.md → architecture diagram
   - clone-ux.md → React component
   - paper-to-code.md → runnable notebook
   - tutorial-walkthrough.md → step-by-step guide

## Tài liệu tham khảo

- [SKILL.md](../SKILL.md) - Hướng dẫn chung cho Claude Code và OpenClaw
- [docs/output-schema.md](../docs/output-schema.md) - Schema đầu ra
- [docs/exit-codes.md](../docs/exit-codes.md) - Mã lỗi và ý nghĩa
- [examples/](../examples/) - Ví dụ sử dụng

## Hỗ trợ

Nếu gặp vấn đề khi tích hợp:
1. Kiểm tra logs của Hermes Agent
2. Chạy `watch --version` để xác minh cài đặt
3. Test độc lập trước khi tích hợp: `watch https://youtube.com/watch?v=dQw4w9WgXcQ`
4. Đọc [README.md](../README.md) để biết danh sách nền tảng được hỗ trợ
5. Xem [GitHub Issues](https://github.com/sonpiaz/watch-cli/issues)

### Hỗ trợ trên Windows

- Đảm bảo Git, Python, Node.js đã được thêm vào PATH
- Sử dụng PowerShell với quyền Administrator khi cần cài đặt system-level packages
- Đối với WSL (Windows Subsystem for Linux), làm theo hướng dẫn Linux/macOS
- Kiểm tra Windows Defender Firewall không chặn các kết nối network của watch-cli

---

**Version**: 1.0.0  
**Last Updated**: 2024-01-15  
**Compatible with**: Hermes Agent v2.0+  
**Windows Support**: PowerShell, Git Bash, WSL
