# Flow: Khung avatar

**Nguồn:** `mvp2_Feature_Breakdown.md` — mục 3. Khung avatar

## Overview
Khung avatar có 2 nguồn earn độc lập: theo **Rank** (phải chủ động claim) và theo **Event** (earn qua mission hoặc mua bằng Star — không phải tiền thật, để tránh cảm giác P2W). Khung sự kiện giữ vĩnh viễn kể cả sau khi hết event.

## Flow Diagram

```mermaid
flowchart TD
    Start(["Fan đạt điều kiện Rank mới"]) --> Rank1["Khung Rank hiển thị trạng thái<br/>'chưa claim' trong Bộ sưu tập Khung"]
    Rank1 --> Claim{"Fan bấm 'Claim'?"}
    Claim -->|Có| CheckRank["API kiểm tra điều kiện Rank đã đạt"]
    CheckRank --> AddInv1["Cộng khung vào Kho<br/>(sở hữu vĩnh viễn)"]
    Claim -->|Chưa bấm| Rank1

    EventM["Fan hoàn thành Nhiệm vụ sự kiện<br/>(vd: thêm lịch ra mắt album vào Calendar)"] --> Earn["Nhận khung sự kiện tự động"]
    Earn --> AddInv2["Cộng khung vào Kho<br/>(giữ vĩnh viễn dù hết event)"]

    Collection["Màn Bộ sưu tập Khung"] --> BuyEvent{"Fan mua khung sự kiện bằng Star?"}
    BuyEvent -->|Có| PayStar["API mua khung bằng Star<br/>(không phải tiền thật trực tiếp)"]
    PayStar --> AddInv3["Cộng khung vào Kho"]

    AddInv1 --> Inventory["Kho Khung avatar"]
    AddInv2 --> Inventory
    AddInv3 --> Inventory

    Inventory --> SetActive["Fan chọn khung 'active'<br/>(chỉ 1 khung active/thời điểm)"]
    SetActive --> Display["Khung hiển thị quanh avatar:<br/>Comment / Profile / Leaderboard"]
    Display -.->|"Đổi active không giới hạn số lần"| SetActive
```
