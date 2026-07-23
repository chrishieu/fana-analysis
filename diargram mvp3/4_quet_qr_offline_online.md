# Flow: Quét QR merch offline → online (My Space)

**Nguồn:** `mvp2_Feature_Breakdown.md` — mục 7. Quét QR merch offline → online

## Overview
"Studio" = My Space → Bộ Sưu Tập (đồng bộ cách hiểu với mục 6). Kế thừa toàn bộ cơ chế QR unique/redeem 1 lần đã có trong BRD F4.1 — không phải cơ chế mới.

## Flow Diagram

```mermaid
flowchart TD
    A["Fan mở My Space"] --> B["Bấm nút 'Quét vật phẩm' (icon QR)"]
    B --> C["Màn quét: Camera QR hoặc nhập mã tay"]
    C --> D["Submit mã"]
    D --> E["API redeem QR:<br/>validate unique + 1 lần/tài khoản"]
    E --> F{"Mã hợp lệ & chưa dùng?"}
    F -->|Không| G["Popup thất bại: 'Mã đã dùng' / 'Thất bại'"]
    G --> C
    F -->|Có| H["Popup trạng thái: đang xử lý"]
    H --> I["Mapping merch type → reward<br/>(Photocard/Album/Lightstick/Trang phục/Merch nhỏ/Limited edition/Vé concert)"]
    I --> J["Popup thành công: 'Bạn nhận được [tên vật phẩm]'"]
    J --> K{"Loại reward?"}
    K -->|E-card| L["Gán vào Sổ E-card"] --> O["Điều hướng: Sổ E-card"]
    K -->|Vật phẩm trang trí| M["Gán vào Inventory"]
    M --> P{"Fan bấm 'Place in studio'?"}
    P -->|Có| Q["Đặt vật phẩm vào My Space / Bộ sưu tập"]
    K -->|Trang phục (skin)| N["Gán vào Inventory"]
    N --> R{"Fan chọn skin?"}
    R -->|Có| S["Gán skin lên Nhân vật"]
```
