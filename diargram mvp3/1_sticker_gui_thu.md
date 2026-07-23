# Flow: Sticker (Gửi thư)

**Nguồn:** `mvp2_Feature_Breakdown.md` — mục 2. Sticker (Gửi thư)

## Overview
Sticker là tính năng bổ sung cho Gửi thư hiện có (song song Fan Letter, không thay thế). Sticker bán theo **cả bộ** (không mua lẻ), chèn không giới hạn số lượng trong 1 thư.

## Flow Diagram

```mermaid
flowchart TD
    A["Fan mở màn Gửi thư"] --> B["Bottom sheet chọn Sticker<br/>(hiển thị các bộ đã sở hữu)"]
    B --> C{"Có bộ sticker mới phát hành?"}
    C -->|Có| D["Hiển thị tag 'new' / popup thông báo"]
    C -->|Không| E
    D --> E{"Fan chọn 1 bộ sticker"}
    E -->|Bộ đã sở hữu Free/Paid| F["Mở bộ, hiển thị danh sách sticker trong bộ"]
    E -->|Bộ Paid chưa sở hữu| G["Vào màn Bộ sưu tập Sticker"]
    G --> H{"Đủ Star để mua cả bộ?"}
    H -->|Đủ| I["API mua cả bộ bằng Star<br/>(trừ Star, cộng ownership user)"]
    H -->|Không đủ| J["Thông báo nạp thêm Star"] --> K["Màn nạp Star"]
    I --> F
    F --> L["Chèn sticker vào nội dung thư<br/>(không giới hạn số lượng/thư)"]
    L --> M["BE lưu reference sticker đã chèn trong nội dung thư"]
    M --> N["Tiếp tục soạn thư → Gửi"]
```
