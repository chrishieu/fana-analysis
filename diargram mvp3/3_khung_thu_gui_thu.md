# Flow: Khung thư (Gửi thư)

**Nguồn:** `mvp2_Feature_Breakdown.md` — mục 4. Khung thư (Gửi thư)

## Overview
Khung thư liên kết trực tiếp với Sổ E-card (mục 6) — "thẻ" dùng để đổi khung thư chính là **E-card**. Thư có khung có thể chọn Public/Private, tải về JPG (logo Fanation đã in sẵn trên khung nên không cần watermark riêng).

## Flow Diagram

```mermaid
flowchart TD
    subgraph OWN["Sở hữu Khung thư (Bộ sưu tập Khung thư)"]
        M1["Hoàn thành Mission"] --> Add["Cộng Khung thư vào Kho"]
        M2["Đổi bằng E-card (burn theo tỷ lệ)"] --> Add
        M3["Mua bằng Star"] --> Add
    end

    Add --> Compose["Màn soạn thư"]
    Compose --> ChooseFrame["Chọn Khung (Frame) trong danh sách sở hữu"]
    ChooseFrame --> Content["Nhập nội dung / chèn hình cá nhân"]
    Content --> Toggle{"Chọn Public hay Private?"}
    Toggle -->|Public| Send1["Gửi thư → hiển thị trong Idol Hub feed"]
    Toggle -->|Private| Send2["Gửi thư → chỉ riêng tư"]

    Send1 --> Download{"Fan tải thư về ảnh?"}
    Send2 --> Download
    Download -->|Có| Render["API render: ghép Khung + hình cá nhân<br/>+ logo Fanation (có sẵn trên asset khung)"]
    Render --> Export["Export file JPG"]
    Export --> Storage["Lưu Storage/CDN"]
```
