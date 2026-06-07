# Hướng dẫn tích hợp Watch CLI vào Hermes Agent

Hướng dẫn này giúp bạn cấu hình và sử dụng **Watch CLI** như một skill/tool trong hệ thống **Hermes Agent**.

## Tổng quan

Watch CLI là công cụ theo dõi thời gian thực cho nhiều nguồn dữ liệu (giá crypto, chứng khoán, thời tiết, system metrics). Khi tích hợp vào Hermes Agent, nó trở thành một skill mạnh mẽ để giám sát và cảnh báo tự động.

## Cách 1: Cài đặt như một External Tool

### Bước 1: Cài đặt Watch CLI

```bash
# Clone repository
git clone https://github.com/your-org/watch-cli.git
cd watch-cli

# Chạy script cài đặt
chmod +x install.sh
./install.sh
```

### Bước 2: Cấu hình trong Hermes Agent

Thêm cấu hình tool vào file config của Hermes Agent (thường là `config.yaml` hoặc `hermes.config.json`):

```yaml
# hermes.config.yaml
tools:
  watch-cli:
    enabled: true
    path: "/path/to/watch-cli/bin/watch"
    commands:
      - crypto
      - stock
      - weather
      - system
    default_interval: 5s
    max_retries: 3
```

Hoặc định dạng JSON:

```json
{
  "tools": {
    "watch-cli": {
      "enabled": true,
      "path": "/usr/local/bin/watch",
      "commands": ["crypto", "stock", "weather", "system"],
      "default_interval": "5s",
      "max_retries": 3
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
description: Real-time monitoring tool for crypto, stocks, weather, and system metrics
author: Watch CLI Team

entry_point: /usr/local/bin/watch

commands:
  - name: crypto
    description: Track cryptocurrency prices in real-time
    parameters:
      - name: symbol
        type: string
        required: true
        description: Cryptocurrency symbol (e.g., BTC, ETH)
      - name: interval
        type: duration
        required: false
        default: 5s
        description: Update interval
      - name: currency
        type: string
        required: false
        default: USD
        description: Fiat currency for price display

  - name: stock
    description: Monitor stock prices
    parameters:
      - name: symbol
        type: string
        required: true
        description: Stock ticker symbol (e.g., AAPL, GOOGL)
      - name: interval
        type: duration
        required: false
        default: 10s

  - name: weather
    description: Track weather conditions
    parameters:
      - name: city
        type: string
        required: true
        description: City name
      - name: interval
        type: duration
        required: false
        default: 60s

  - name: system
    description: Monitor system resources (CPU, RAM, Disk)
    parameters:
      - name: interval
        type: duration
        required: false
        default: 2s
      - name: metrics
        type: array
        required: false
        description: Specific metrics to monitor

capabilities:
  - real-time-monitoring
  - alert-thresholds
  - multi-source-support
  - offline-mode

rate_limits:
  requests_per_minute: 60
  concurrent_watches: 10
```

## Cách 2: Tích hợp như MCP Server (Model Context Protocol)

Nếu Hermes Agent hỗ trợ MCP, bạn có thể chạy Watch CLI như một MCP server:

### Bước 1: Cài đặt MCP Server

```bash
cd mcp-server
npm install
npm run build
```

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

### Ví dụ 1: Theo dõi giá Bitcoin

```
User: Theo dõi giá BTC cho tôi
Hermes: [Sử dụng watch-cli crypto] Đang theo dõi BTC/USD...

BTC/USD: $67,234.50 (+2.34%) 📈
Cập nhật: 2024-01-15 10:30:45
Biến động 24h: +$1,532.10
```

### Ví dụ 2: Giám sát cổ phiếu Apple

```
User: Giám sát AAPL mỗi 10 giây
Hermes: [Sử dụng watch-cli stock] Đã thiết lập giám sát AAPL...

AAPL: $185.92 (+0.87%) 📈
Cập nhật: 2024-01-15 10:31:00
Volume: 52.3M
Market Cap: $2.89T
```

### Ví dụ 3: Theo dõi thời tiết

```
User: Thời tiết Hà Nội thế nào?
Hermes: [Sử dụng watch-cli weather] Đang lấy dữ liệu thời tiết...

🌤️ Hà Nội, VN
Nhiệt độ: 28°C (82°F)
Độ ẩm: 75%
Gió: 12 km/h NE
Cập nhật: 2024-01-15 10:31:15
```

### Ví dụ 4: Giám sát hệ thống

```
User: Giám sát tài nguyên hệ thống
Hermes: [Sử dụng watch-cli system] Đang theo dõi system metrics...

💻 System Metrics
CPU: 45.2% | RAM: 8.4GB/16GB (52.5%) | Disk: 120GB/500GB (24%)
Cập nhật: 2024-01-15 10:31:30
```

## Thiết lập Alert và Ngưỡng cảnh báo

Hermes Agent có thể tự động cảnh báo khi đạt ngưỡng:

```yaml
# alerts.yaml
alerts:
  - name: btc_price_alert
    tool: watch-cli
    command: crypto
    condition: "price > 70000 OR price < 60000"
    notification:
      type: slack
      channel: "#crypto-alerts"

  - name: high_cpu_alert
    tool: watch-cli
    command: system
    condition: "cpu_usage > 90"
    notification:
      type: email
      recipients:
        - admin@example.com
```

## Các lệnh thường dùng

| Lệnh | Mô tả | Ví dụ |
|------|-------|-------|
| `watch crypto <symbol>` | Theo dõi crypto | `watch crypto BTC` |
| `watch stock <symbol>` | Theo dõi chứng khoán | `watch stock AAPL` |
| `watch weather <city>` | Theo dõi thời tiết | `watch weather Hanoi` |
| `watch system` | Giám sát hệ thống | `watch system` |
| `watch stop` | Dừng tất cả watch | `watch stop` |
| `watch list` | Liệt kê đang theo dõi | `watch list` |

## Xử lý lỗi thường gặp

### Lỗi: Tool not found
```
Solution: Kiểm tra path trong cấu hình Hermes Agent
hermes.config.yaml:
  tools:
    watch-cli:
      path: "/usr/local/bin/watch"  # Đảm bảo path đúng
```

### Lỗi: Rate limit exceeded
```
Solution: Giảm tần suất request hoặc tăng rate_limit
tools:
  watch-cli:
    rate_limits:
      requests_per_minute: 30
```

### Lỗi: API key invalid
```
Solution: Cập nhật API key trong biến môi trường
export WATCH_CLI_API_KEY=your_new_key
```

## Best Practices

1. **Giới hạn số lượng watch đồng thời**: Không nên theo dõi quá 10 nguồn cùng lúc
2. **Sử dụng interval hợp lý**: 
   - Crypto: 5-10s
   - Stock: 10-30s
   - Weather: 60s+
   - System: 2-5s
3. **Enable offline mode** khi không cần dữ liệu real-time
4. **Lưu logs** để debug và audit
5. **Thiết lập alert thresholds** phù hợp để tránh spam thông báo

## Tài liệu tham khảo

- [SKILL.md](../SKILL.md) - Hướng dẫn chung cho Claude Code và OpenClaw
- [docs/output-schema.md](../docs/output-schema.md) - Schema đầu ra
- [docs/exit-codes.md](../docs/exit-codes.md) - Mã lỗi và ý nghĩa
- [examples/](../examples/) - Ví dụ sử dụng

## Hỗ trợ

Nếu gặp vấn đề khi tích hợp:
1. Kiểm tra logs của Hermes Agent
2. Chạy `watch --version` để xác minh cài đặt
3. Test độc lập trước khi tích hợp: `watch crypto BTC`
4. Xem [GitHub Issues](https://github.com/your-org/watch-cli/issues)

---

**Version**: 1.0.0  
**Last Updated**: 2024-01-15  
**Compatible with**: Hermes Agent v2.0+
