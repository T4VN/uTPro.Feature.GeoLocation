# Screenshots cần capture

Đặt ảnh PNG vào thư mục này (cùng level với file README này).

## Danh sách ảnh cần chụp:

### 1. `detection-flow.png`
- **Mô tả**: Diagram/screenshot thể hiện detection flow
- **Gợi ý**: Có thể dùng diagram tool (draw.io / Excalidraw) vẽ flow:
  Request → Cloudflare Header? → AWS Header? → Azure Header? → IP Resolve → Offline DB → Result
- **Dùng ở**: README.md (NuGet + Marketplace), Docs site main page

### 2. `integration-flow.png`
- **Mô tả**: Diagram thể hiện cách GeoLocation tích hợp với uTPro localization
- **Gợi ý**: Flow chart:
  1. GeoLocation middleware → detect country → store in HttpContext
  2. uTPro Localization middleware → check URL → check cookie → check GeoLocation → set culture
- **Dùng ở**: Docs getting-started page, Marketplace

### 3. `appsettings-example.png` (optional)
- **Mô tả**: Screenshot appsettings.json với GeoLocation config highlighted
- **Dùng ở**: Configuration docs page

## File paths sau khi copy sang public repo:

Marketplace đọc từ:
- `https://raw.githubusercontent.com/T4VN/uTPro.Feature.GeoLocation/main/Image/Screenshots/detection-flow.png`
- `https://raw.githubusercontent.com/T4VN/uTPro.Feature.GeoLocation/main/Image/Screenshots/integration-flow.png`

Docs site đọc từ:
- `/screenshots/uTPro.Feature.GeoLocation/detection-flow.png`
- `/screenshots/uTPro.Feature.GeoLocation/integration-flow.png`

## Sau khi chụp xong:
1. Đặt vào folder này: `Image/Screenshots/`
2. Copy vào docs site: `uTPro.Website.Docs/screenshots/uTPro.Feature.GeoLocation/`
3. Chạy `pwsh ./pack.ps1` → sẽ tự mirror sang public repo
