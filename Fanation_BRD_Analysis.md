# Fanation - Business Requirements Document (BRD) Analysis

> Phân tích từ PRD v1.0 | Cập nhật: 2026-05-21
> Tech stack: React Native (FE) | NestJS (BE) | PostgreSQL (DB) | GCP (Cloud Run + Cloud SQL + Firebase)
> Prototype: [Figma](https://www.figma.com/design/2CRmKrnI5cOLJosI834Jqs/)

---

## I. TỔNG QUAN DỰ ÁN

### 1.1. Vision & Mission
- **Vision**: Fandom playground chính thức đầu tiên tại Việt Nam
- **Mission**: Centralize mọi hoạt động fandom, tạo hệ sinh thái tương tác đa điểm chạm giữa fan và nghệ sĩ
- **Core Concept**: "Multi-interaction Fandom Playground" powered by "Proof of Work"

### 1.2. Hệ sinh thái chính

| Component | Mục đích | Features |
|-----------|----------|----------|
| **HOME** | News Feed tổng hợp | Aggregated feed từ các fandoms đã follow |
| **IDOL's HUB** | Multi-interaction fan ↔ idol | Timeline Feed, Fan Letter, Calendar, Digital Gifts, Events |
| **MUSIC GAME HUB** | Fandom playground | Rhythm Hive, Tiles Hop |
| **FAN PROJECT** | Proof of work + Community | Tạo/tham gia/theo dõi fan projects |
| **YOUR SPACE** | Proof of work + Gamification | Room decoration, level up, ranking |
| **DIGITAL MERCH** | Collectible system | QR scan, Digital Gallery, Exclusive Content |
| **CMS** | Platform management (Web) | Item/gift catalog, Star pricing, content moderation, user management |

---

## II. RBAC - ROLE-BASED ACCESS CONTROL

### 2.1. User Roles

```
┌─────────────────────────────────────────────────────────────────┐
│                        FANATION PLATFORM                        │
│              Unified Auth: Google Sign-in (Firebase)             │
│              Constraint: 1 email = 1 role duy nhất              │
├──────────────┬──────────────────────┬───────────────────────────┤
│     FAN      │   IDOL / ARTIST      │     ADMIN (CMS)           │
│  (End-user)  │  (2 sub-accounts)    │  (Platform Management)    │
│              │                      │                           │
│ - Mobile app │ ┌─ Management Acc    │ - CMS Web portal          │
│ - Google     │ │  (Quản lý nghệ sĩ) │ - Item/Gift catalog       │
│   Sign-in    │ │  - Google Sign-in  │ - Star pricing config     │
│ - Auto Fan   │ │  - Invite-only     │ - Content moderation      │
│   nếu email  │ │  - Post bài        │ - Fan project approval    │
│   mới        │ │  - Duyệt project   │ - User management         │
│              │ │  - Dashboard       │ - Mission/reward config   │
│              │ ├─ Artist Acc        │                           │
│              │ │  (Nghệ sĩ cá nhân) │                           │
│              │ │  - Google Sign-in  │                           │
│              │ │  - Invite-only     │                           │
│              │ │  - Thả tim letter  │                           │
│              │ │  - Xem digital gift│                           │
│              │ └────────────────────│                           │
└──────────────┴──────────────────────┴───────────────────────────┘
```

### 2.2. Permission Matrix

| Feature | Fan | Idol (Artist) | Idol (Management) | Admin (CMS) |
|---------|-----|---------------|-------------------|-------------|
| Login (Google Sign-in) | Yes (auto-create) | Yes (invite-only) | Yes (invite-only) | N/A |
| Follow/Unfollow Fandom | Yes | - | - | - |
| View Idol Hub Feed | Yes | - | Yes (own) | Yes |
| Thả tim bài đăng | Yes | Yes | - | - |
| Viết Fan Letter | Yes | - | - | - |
| Like Fan Letter | Yes | Yes | - | - |
| Collect Digital Gift (Stars) | Yes | - | - | - |
| Post bài (thường/sự kiện) | - | - | Yes | - |
| Quản lý bài đăng | - | - | Yes | Yes |
| Scan QR Merch | Yes | - | - | - |
| Xem Digital Merch Gallery | Yes | - | - | - |
| Tạo Fan Project | Yes (Level 3+) | - | - | - |
| Tham gia Fan Project | Yes | - | - | - |
| Duyệt Fan Project | - | - | Yes | Yes |
| Your Space (decorate) | Yes | - | - | - |
| Daily Mission | Yes | - | - | - |
| Chơi Game Hub | Yes | - | - | - |
| Dashboard (analytics) | - | - | Yes | Yes |
| Account Settings | Yes | - | Yes | Yes |
| CMS: Item/Gift Catalog | - | - | - | Yes |
| CMS: Star Pricing | - | - | - | Yes |
| CMS: Content Moderation | - | - | - | Yes |
| CMS: Mission Config | - | - | - | Yes |
| CMS: User Management | - | - | - | Yes |

### 2.3. Fan Level System & Gated Features

| Level | Tên | Points | Gated Features |
|-------|-----|--------|----------------|
| 0 | Fan Nhí | 0 | Basic access |
| 1 | Fan cứng | 500 | - |
| 2 | Fan "guộc" | 2,500 | - |
| 3 | Phú ông/Phú bà | 7,000 | Tạo Fan Project |
| 4 | Đại sứ fandom | 13,000 | - |

> **Lưu ý**: Points tích lũy **theo từng Idol riêng biệt**, không phải tổng chung.
> Points earned through engagement: check-in, daily missions, game play, fan project participation.

---

## III. EPIC & FEATURE BREAKDOWN

### Epic 1: Authentication & Onboarding

#### F1.1: Sign Up / Login (Unified)
| Field | Detail |
|-------|--------|
| **User Story** | As a user (Fan or Idol), I want to sign in with Google so that I can quickly access the app |
| **Method** | Google Sign-in (OAuth) — cổng chung cho Fan và Idol |
| **Platform** | Mobile (RN) |
| **Dependencies** | Google Sign-in API, Firebase Auth |
| **Constraint** | 1 email = 1 role duy nhất (Fan hoặc Idol, không cả hai) |

**Login Flow:**
```
User nhấn "Sign in with Google"
  → Firebase Auth → lấy email
  → Hệ thống check email trong DB:
      ├── Email chưa tồn tại       → Tạo Fan account → Onboarding (F1.2)
      ├── Email là Fan (đã onboard) → Trả token (role: FAN) → Home Feed
      └── Email là Idol             → Trả token (role: IDOL) → Idol Dashboard
```

**Acceptance Criteria:**
- AC1: Hiển thị Login screen với Google option khi mở app
- AC2: Gọi Google Sign-in API, lấy email làm identifier
- AC3: Email chưa tồn tại → tạo Fan mới → redirect Onboarding
- AC4: Email đã tồn tại → check role → redirect đúng interface (Fan app / Idol dashboard)
- AC5: Token chứa role (FAN | IDOL_MANAGEMENT | IDOL_ARTIST) để client route UI

#### F1.2: Onboarding Flow
| Field | Detail |
|-------|--------|
| **User Story** | As a new user, I want to set up my profile and choose fandoms so that I get personalized content |

**Requirements:**
- **Username rules:**
  - 5-15 ký tự
  - Cho phép: a-z, 0-9, "_" (không phân biệt hoa thường)
  - Phải có ít nhất 1 chữ cái
  - Không dấu, không khoảng trắng, không ký tự đặc biệt
  - Real-time validation hiển thị lỗi dưới input
- **Avatar:** Chọn từ gallery hoặc chụp mới
- **Choose Fandom:** Multi-select, có search, "Tiếp tục" disabled khi chưa chọn

**Acceptance Criteria:**
- AC1: Username không hợp lệ -> "Tiếp tục" bị darken
- AC2: Chưa chọn fandom -> "Tiếp tục" bị gray out
- AC3: Quit trước khi chọn fandom -> hiển thị lại trang chọn fandom

---

### Epic 2: Idol Hub (Fan View)

#### F2.1: Timeline Feed
| Field | Detail |
|-------|--------|
| **User Stories** | As a fan, I want to see my idol's official posts so that I can stay updated |
| **Prerequisite** | Đã login + đã follow fandom |

**Idol Hub Tab Structure:**
| Tab | Nội dung |
|-----|----------|
| Idol Feeds | Bài đăng thường + sự kiện từ Idol |
| Fan Letter | Thư/lời nhắn từ Fan gửi Idol |
| Events/Notice | Sự kiện và thông báo |

**Follow/Following:**
- Fan đã thuộc fandom: hiển thị "Following"
- Fan chưa thuộc fandom: hiển thị nút "Follow"

**Business Rules:**
- Chỉ hiển thị bài đã published từ fandom tương ứng
- Pagination: 10 bài/lần load, infinite scroll
- Cache feed 60 giây
- Pull-to-refresh bypass cache
- Bài bị admin ẩn/xóa -> không hiển thị

**Bài đăng có 2 loại:**

| | Bài đăng thường | Bài đăng sự kiện |
|---|---|---|
| Vị trí | Theo thời gian | Cố định đầu feed |
| Nội dung | Caption + media | Title + description + countdown |
| Tương tác | Thả tim | Thả tim + Xem chi tiết |
| Navigation | Kéo xuống xem thêm | Kéo ngang xem thêm sự kiện |

**Acceptance Criteria:**
- AC1: Feed load <= 2 giây
- AC2: Hiển thị 10 bài đầu tiên
- AC3: Scroll -> load thêm 10 bài
- AC4: Pull-to-refresh lấy data mới
- AC5: Tim cập nhật <= 5 giây
- AC6: Countdown đúng UTC+7, tự chuyển "Đang diễn ra" khi hết
- AC7: Empty state: "Chưa có bài đăng nào. Hãy đón chờ!"
- AC8: Offline: hiển thị cache + banner lỗi mạng

#### F2.2: Calendar (Lịch hoạt động)
| Field | Detail |
|-------|--------|
| **User Story** | As a fan, I want to tap a specific date on the calendar and see event details |
| **Range** | Quá khứ max 12 tháng, Tương lai max 12 tháng |

**Dot Indicator Colors:**
- Đã diễn ra: xám mờ
- Đang diễn ra: đỏ
- Sắp diễn ra: xanh lá

**Acceptance Criteria:**
- AC1: Mở calendar -> hiển thị tháng hiện tại, highlight hôm nay
- AC2: Swipe trái/phải chuyển tháng
- AC3: Nhấn ngày có sự kiện -> danh sách load <= 1 giây
- AC4: Nhấn ngày không có sự kiện -> "Không có sự kiện vào ngày này"
- AC5: Badge trạng thái cập nhật tự động
- AC6: Chuyển tháng rồi quay lại -> load từ cache

#### F2.3: Notification Banner / Noti Promotion
- Banner highlight hoạt động/ấn phẩm mới + button redirect Spotify/Youtube
- Chỉ 1 banner cùng lúc

#### F2.4: Fan Letter
| Field | Detail |
|-------|--------|
| **User Story** | As a fan, I want to write a letter to my idol so that I can express my feelings publicly |
| **Vai trò** | Thay thế Digital Gift trong việc thể hiện tình cảm Fan→Idol — thay vì tặng quà bằng tiền, Fan gửi lời nhắn bằng chữ |

**Flow cơ bản:**
```
Fan nhấn CTA Create → Chọn "Fan Letter" → Chọn Idol → Viết nội dung → Gửi
→ Letter hiển thị trong Idol Hub (tab Fan Letter), không qua duyệt
→ Idol + Fan khác có thể xem và tương tác (like/react)
```

**Đặc điểm chính:**
- **Ai viết:** Fan (tối đa 1 letter/ngày)
- **Gửi cho ai:** 1 Idol cụ thể
- **Nội dung:** Text thuần, tối đa 2,000 ký tự
- **Duyệt:** Không — hiển thị ngay sau khi gửi
- **Ai đọc được:** Idol + tất cả Fan (public trong Idol Hub)
- **Tương tác:** Like, React (Fan + Idol đều có thể)
- **Points:** Không tính points khi viết fan letter
- **Chi phí:** Miễn phí (không tốn Star hay tiền)

**CTA Create Button:**
- Floating button chung trên app
- Options khi nhấn: Fan Letter, Fan Project

**Acceptance Criteria:**
- AC1: Fan nhấn CTA Create -> hiển thị options "Fan Letter" / "Fan Project"
- AC2: Chọn Idol -> hiển thị text editor (max 2,000 ký tự)
- AC3: Gửi thành công -> letter xuất hiện ngay trong tab Fan Letter của Idol Hub
- AC4: Idol + Fan khác có thể like/react trên letter
- AC5: Fan chỉ gửi được 1 letter/ngày (reset 00:00 UTC+7)
- AC6: Vượt 2,000 ký tự -> disable nút gửi
- AC7: Empty state: "Chưa có fan letter nào. Hãy là người đầu tiên!"

---

### Epic 3: Your Space (Fan View)

#### F3.1: Studio Interface
| Field | Detail |
|-------|--------|
| **User Story** | As a fan, I want to view and decorate my Studio for each Idol I follow |
| **Concept** | Studio & Stage - mỗi Idol = 1 phòng riêng với theme riêng |

**Studio System:**
- Canvas room chiếm 60% màn hình, 40% còn lại là controls
- Theme theo idol (fandom color, poster, motif nội thất)
- Fan KHÔNG tự chọn theme -> hệ thống set theo Idol
- Fan thể hiện cá tính qua items trang trí + cách sắp xếp

**Furniture Panel (bottom panel):**
- Thanh scroll ngang hiển thị toàn bộ đồ nội thất fan đang sở hữu (Piano, Guitar, Plant, Sofa, Poster...)
- Đây là danh sách **SpaceItem** — khác với **Inventory** (cards/digital assets/game items)
- Drag item từ panel → drop vào canvas để đặt vào phòng
- Nút "+New" (cuối panel): flow mua nội thất mới — chi tiết TBD (xem Open Question #13b)

**Item Placement:**
- Drag & drop từ Furniture Panel vào phòng
- Pinch to zoom (phóng to/thu nhỏ)
- Xoay 45 độ mỗi lần
- Tap & hold → Di chuyển / Xoay / Cất lại về Furniture Panel (không phải Inventory)

**Snapshot Feature:**
- Full canvas render PNG (không crop)
- Bao gồm canvas + items, không bao gồm UI panels
- Watermark: Fanation + tên fan + tên Idol
- Lưu Camera Roll hoặc share social media

**Acceptance Criteria:**
- AC1: Studio default hiển thị đúng theme + items đã lưu
- AC2: Chuyển Studio <= 5 giây
- AC3: Lên level -> canvas thay đổi, items giữ nguyên vị trí
- AC4: Item đặt đúng vị trí, không xê dịch khi thoát/vào lại
- AC5: Snapshot render <= 3 giây, lưu Camera Roll, toast "Đã lưu"

#### F3.2: Daily Mission System

**Nhiệm vụ cố định hàng ngày:**
| Hành động | Thưởng | Giới hạn |
|-----------|--------|----------|
| Check-in (mở Your Space) | Theo streak | 1/ngày, cho default space |

> Danh sách mission và reward cấu hình qua CMS.

**Check-in Streak:**
| Ngày | Thưởng |
|------|--------|
| 1-6 | +3 pts/ngày |
| 7 | +10 pts |
| 8-13 | +3 pts/ngày |
| 14 | +10 pts + 1 wallpaper (1 lần) |
| 15-29 | +3 pts/ngày |
| 30 | +15 pts + 2 star (1 lần) |

> Streak reset nếu miss 1 ngày. Không reset sau 30 ngày. Khi reset: chỉ nhận points theo ngày, không nhận lại quà cột mốc.

**Nhiệm vụ cột mốc (one-time):**
| Hành động | Thưởng |
|-----------|--------|
| Thêm đủ 10 sự kiện vào calendar | +20 points |
| Lên level 2 | 1 tranh idol |
| Lên level 3 | 2 star + 1 rèm cửa |
| Lên level 4 | 2 star + 1 đàn piano |
| Lên level 5 | 5 star + 1 stage |

**Business Rules:**
- Missions reset 00:00 UTC+7 mỗi ngày
- Tự động check cột mốc khi hoàn thành -> tặng item
- Nút CHECK fallback nếu hệ thống miss
- Mission rewards cấu hình qua CMS

#### F3.3: My Calendar (Fan's Personal)
- Fan thêm sự kiện từ calendar idol -> centralized 1 calendar
- Ngày có sự kiện tô đỏ
- Thả tim sự kiện đang/sắp diễn ra

#### F3.5: Inventory (Kho vật phẩm)
| Field | Detail |
|-------|--------|
| **User Story** | As a fan, I want to view and manage my collectible items in an inventory |
| **Capacity** | Max 10 slots, global |
| **Phân biệt** | Inventory ≠ Furniture Panel. Đây là 2 hệ thống riêng biệt |

**Inventory chứa gì:**
| Loại item | Ví dụ | Nguồn |
|-----------|-------|-------|
| Cards | Idol card, collectible card | Mission / Streak / Mua bằng Star |
| Digital Assets | Digital poster, digital photo | QR scan merch / Mission |
| Game reward items | Items nhận được khi chơi game | Game Hub rewards |

> **KHÔNG chứa:** Đồ nội thất (SpaceItem) — nội thất được quản lý riêng qua Furniture Panel trong Studio.

**Rules:**
- Max 10 slots, global (dùng chung, không per-idol)
- Items không expire
- MVP1: không có trading/transfer giữa fans
- Kho đầy → không thể nhận item mới (mission/streak bị blocked, không mất reward)

**Nguồn nhận items:**
| Nguồn | Điều kiện |
|-------|-----------|
| Mission / Streak / Milestone | Nhận tự động khi hoàn thành (nếu còn slot) |
| QR Merch scan | Digital assets từ quét merch vật lý |
| Game Hub | Items reward từ gameplay |
| Mua bằng Star | Giá cấu hình qua CMS |

**Acceptance Criteria:**
- AC1: Inventory hiển thị slot counter "X/10"
- AC2: Kho đầy → mission/streak reward bị blocked, không mất → toast "Kho đầy, hãy giải phóng slot để nhận vật phẩm"
- AC3: Mua item bằng Star khi kho đầy → block transaction, toast "Hãy giải phóng slot trước khi mua"
- AC4: Mua item bằng Star → trừ Star atomic → item xuất hiện ngay
- AC5: Filter: Tất cả / Cards / Digital Assets / Game Items
- AC6: Usage của từng item type → TBD (xem Open Question #13a)

---

### Epic 4: Digital Merchandise

#### F4.1: QR Code Scanning
| Field | Detail |
|-------|--------|
| **User Story** | As a fan, I want to scan QR on physical merch to add digital version to my collection |
| **QR Rule** | Mỗi QR unique, chỉ kích hoạt 1 lần cho 1 tài khoản |

**Merchandise 2 loại:**
| Loại | Mô tả |
|------|--------|
| Vật lý | Redirect 3rd party (Shopee/Lazada), quét QR để xác nhận hàng thật |
| Digital | Digital assets, digital poster, có thể dùng trong game |

**Error Handling:**
| Trường hợp | Xử lý |
|-------------|--------|
| QR không hợp lệ | Toast lỗi, camera vẫn mở |
| QR đã claim bởi người khác | Bottom sheet: "Merch đã được thêm bởi fan khác" |
| QR đã claim bởi chính mình | Bottom sheet + 2 button: Scan lại / Đến gallery |

**Acceptance Criteria:**
- AC1: Mở camera <= 1 giây (khi có quyền)
- AC2: QR nhận diện <= 2 giây (QR rõ nét)
- AC3: Vật phẩm xuất hiện ngay trong gallery sau scan thành công

#### F4.2: Digital Gallery
- Grid 3 cột, responsive
- Lọc theo idol + danh mục đồng thời
- Danh mục: Tất cả, Album, Card, Photobook, Poster
- Sao gắn với merch có exclusive content
- Nút scan QR floating

#### F4.3: Exclusive Content
- **Có phí (Star):** Trả star -> mở khóa -> xem unlimited (giá cấu hình qua CMS)
- **Miễn phí:** Xem ngay -> unlimited
- **Bảo vệ:** Không share, không cap/quay màn hình
- Hỗ trợ pinch-to-zoom
- Loại content: Live photo, Voice/Video, Voucher

---

### Epic 5: Digital Gift

#### F5.1: Gift Universe Map
| Field | Detail |
|-------|--------|
| **Concept** | Map vũ trụ với 4 khu tương ứng 4 tầng quà |
| **Theme MVP1** | "Vũ trụ" - phòng trưng bày cá nhân |

**Gift Tiers (cấu hình qua CMS):**

| Khu | Range (Stars) | Số lượng quà |
|-----|--------------|-------------|
| Ánh Sao | 10 - 45 | 10 items |
| Tinh Tú | 50 - 200 | 10 items |
| Hào Quang | 700 - 1,500 | 5 items |
| Hẹn Ước (custom/idol) | 20 - 1,200 | 5 items (per idol) |

**Brightness mechanic:** Các khu sáng theo % tổng quà đã thắp sáng (100% = sáng rõ nhất)

> Star costs và gift catalog được quản lý hoàn toàn qua CMS. Admin có thể thêm/sửa/xóa gift items và điều chỉnh giá Star.

#### F5.2: Gift Card
- Search tên mình trên map -> hệ thống tạo card (tên user + danh sách quà đã thắp sáng)
- Lưu ảnh hoặc share social media

---

### Epic 6: Fan Project

#### F6.1: View Fan Projects
- Danh sách có filter: Billboard, Merchandise, Food-truck, Sự kiện, Từ thiện, Khác
- Highlight Level 5 projects ghim đầu 7 ngày
- Thẻ: Tên, mô tả (max 200 ký tự), "Verified by idol", deadline, số người tham gia, thời gian còn lại
- Chi tiết: Mô tả, kế hoạch, deadline
- Tìm kiếm theo tên project

#### F6.2: Create Fan Project
| Field | Detail |
|-------|--------|
| **Gate** | Level 3+ |
| **Limit** | 1 pending proposal tại 1 thời điểm |
| **Deadline** | 7-30 ngày từ start_date, max 60 ngày |

**3-Step Creation:**
1. Thông tin dự án (mục tiêu, kế hoạch, timeline)
2. Xác nhận quy định
3. Review & Submit

**Draft:** Lưu draft khi chuyển step, không mất data khi thoát

**Approval Pipeline:**
```
Fan Submit -> Auto-scan (<=24h) -> [Pass] -> PENDING (chờ Manager)
                                -> [Fail] -> REJECTED (rejected_by: system)

PENDING -> Manager Approve -> LIVE (go live ngay)
PENDING -> Manager Reject  -> REJECTED (kèm lý do)
PENDING -> start_date + 2 không duyệt -> REJECTED (auto, rejected_by: system)
```

**Timeline notifications:**
| Mốc | Action |
|------|--------|
| start_date - 2 | Push noti nhắc Manager lần 1 |
| start_date - 1 | Push noti nhắc Manager lần 2 |
| start_date + 2 | Auto-reject nếu chưa duyệt |

#### F6.3: Tham gia Fan Project
- Fan nhấn "Tham gia" để đăng ký tham gia project
- Hiển thị số người tham gia và thời gian còn lại
- Không có giao dịch tài chính — chỉ pledge tham gia
- Push noti khi dự án kết thúc
- Tham gia tính +points cho fan (configurable qua CMS)

---

### Epic 7: Game Hub (MVP1 - Open Beta)

#### F7.1: Song Choosing
- Dropdown filter theo fandom
- Default: hiển thị tất cả bài hát, sort order
- Preview music (play/pause/loop)
- MVP1: không giới hạn access, FE tự lưu bài hát (không có BE)

#### F7.2: Tiles Hop
| Field | Detail |
|-------|--------|
| **Type** | Rhythm game - bouncing character |
| **Controls** | Drag left/right |
| **Scoring** | +10/tile thành công, -10/tile miss, max 500, min 0 |
| **Tile types** | Short (nhảy ngay), Long (chạy rồi nhảy) |
| **Layout** | 3 cột chính + 2 cột trung gian = 5 cột, 1 tile/row |
| **MVP1** | Không giới hạn lượt, không thưởng |

#### F7.3: Rhythm Hive
| Field | Detail |
|-------|--------|
| **Type** | Rhythm game - tap notes |
| **Controls** | Tap + Hold/Slide |
| **Scoring** | +1/note hit, -1/note miss |
| **Note types** | Tap (single), Slide/Hold (trail) |
| **Layout** | 4 cột, max 2 notes/row (1 in col 1-2, 1 in col 3-4) |
| **Accuracy grades** | Hoàn hảo, Tuyệt vời, Xuất sắc, Tốt, Trượt |

**Shared Game UX:**
- Tap anywhere -> countdown 3s -> start
- Back button -> "Bạn chắc chắn muốn thoát?" -> Chơi tiếp (3s countdown) / Thoát
- Quit trước khi hết bài -> không tính điểm

---

### Epic 8: Account Settings (Fan)

#### F8.1: Profile Management
**Đổi avatar:**
- JPG/PNG, max 5MB, min 200x200px
- Crop 1:1, preview trước xác nhận
- Không giới hạn số lần

**Đổi username:**
- Max 3 lần trong đời tài khoản
- Cooldown 30 ngày giữa 2 lần
- Username cũ bị khóa 6 tháng
- Case-insensitive uniqueness check
- Không cho phép dấu

#### F8.2: Notification Settings
| Loại thông báo | Push | In-app | Tắt được |
|----------------|------|--------|----------|
| Bài đăng mới từ idol | Yes | Yes | Yes |
| Sự kiện sắp diễn ra | Yes | Yes | Yes |
| Nhắc chơi game, ranking | Yes | Yes | Yes |
| Fan letter có tương tác mới | Yes | Yes | Yes |
| Lên cấp | Yes | Yes | Yes |
| Fan project cập nhật | Yes | Yes | Yes |
| Hệ thống (bảo mật) | Yes | No | **No** |

#### F8.3: Social Media Linking
- OAuth flow -> đăng nhập provider -> cấp quyền -> redirect app

#### F8.4: Phone Number Verification
| Trường hợp | Xử lý |
|-------------|--------|
| Số đã liên kết account khác | "Số này đã được sử dụng" |
| OTP hết hạn | Resend sau 60s cooldown |
| Sai OTP 5 lần | Lock 15 phút |
| Đổi số | OTP vào số mới |
| Số format sai | Disable nút "Gửi mã OTP" ngay |

---

### Epic 9: Idol View - Management Account

#### F9.1: Account Setup (Invite-only, Google Sign-in)

**Invitation Flow:**
```
Admin (CMS) nhập Gmail của Idol
  → Hệ thống check email:
      ├── Email chưa tồn tại trong DB     → Gửi invitation email
      └── Email đã là Fan account          → Reject + thông báo Admin
                                              "Email này đang được sử dụng bởi Fan"

Idol nhận email → Nhấn accept → Verify OTP
  → Assign role IDOL cho email
  → Idol mở app → Login Google bằng đúng Gmail đó
  → Hệ thống nhận role IDOL → Onboarding Idol (profile setup)
  → Preview & Publish
```

**Invitation Rules:**
- Admin chỉ invite bằng Gmail (phải là @gmail.com hoặc Google Workspace)
- OTP gửi qua email, expiry 24h
- Sai OTP 5 lần → lock 15 phút
- Invitation chưa accept sau 7 ngày → expired, Admin có thể gửi lại
- Idol bắt buộc login bằng đúng Gmail đã được invite

**Profile Fields:**
| Field | Required | Validation |
|-------|----------|------------|
| Avatar | Yes | PNG/JPEG, max 5MB, crop 1:1 |
| Ảnh bìa | Yes | PNG/JPEG, max 6MB, 16:9 |
| Tên nghệ danh | Yes | VN có dấu, EN, số, khoảng trắng, `-`, `.`. No emoji. Unique check |
| Tên fandom | No | VN, EN, số, emoji OK. Not unique |
| Bio | No | Max 300 ký tự, emoji OK |
| Ngày debut | No | DD/MM/YYYY, 01/01/1970 -> today |
| Social links | No | Auto-detect domain: FB, YT, Spotify, TikTok |

#### F9.2: Dashboard
**6 Cards:**
| Card | Metric | Filter | Real-time |
|------|--------|--------|-----------|
| Fan mới follow | Count trong period | All filters | No |
| Fan huỷ follow | Count trong period | All filters | No |
| Bài đăng | Count trong period | All filters | No |
| Fan Letters nhận được | Count trong period | All filters | No |
| Project chờ duyệt | Current pending count | **Hôm nay only** | **Yes** |
| Sự kiện sắp diễn ra | Count 30 ngày tới | **Hôm nay only** | **Yes** |

**Filters:** Hôm nay | 1 tuần | 1 tháng | 1 năm | Chọn ngày (max 365 ngày trước)

**Action Items:** Fan project cần duyệt

#### F9.3: Post Management
**Bài đăng thường:**
- Max 10,000 ký tự
- Max 10 media files, mỗi file max 50MB
- Hỗ trợ PNG, JPEG, MP4
- Hẹn giờ đăng (không cho quá khứ)
- Auto-save draft khi thoát

**Bài đăng sự kiện:**
- Tên max 100 ký tự, Mô tả max 3,000 ký tự
- Media giống bài thường
- Có option "sự kiện nổi bật" (chỉ 1 cùng lúc)
- KHÔNG hỗ trợ hẹn giờ
- Không tạo sự kiện quá khứ
- Ngày-Giờ sự kiện chỉ được sửa **1 lần**

**Content Management:**
- Tabs: Bài đã đăng | Nháp | Đã lên lịch | Sự kiện nổi bật
- Bài đã đăng: filter All/Hôm nay/30 ngày/1 năm
- Cho phép: Sửa, Xoá (đã đăng + nổi bật); Đăng, Sửa, Xoá (nháp + lên lịch)

#### F9.4: Duyệt Fan Project (Manager)
**Statuses:**
| Status | Mô tả |
|--------|--------|
| PENDING | Chờ Manager duyệt (đã qua auto-scan) |
| LIVE | Đang nhận người tham gia |
| REJECTED | Bị từ chối (by manager hoặc system) |
| COMPLETED | Kết thúc sau end_date |

**3 Tabs:** Chờ duyệt | Đang live | Lịch sử

**Approve:** Xác nhận -> Push noti fan (async)
**Reject:** Chọn lý do chuẩn + ghi chú (max 500 ký tự) -> Push noti fan

#### F9.5: Digital Gift - Manager View
**Gallery:**
- Theme theo mùa (seasonal, quarterly)
- 3 chỉ số: tổng quà đã thắp sáng, số fan đã collect, top gifts
- Top 10 fan tháng (sort stars spent desc, tie-break: collect trước xếp trên)

---

### Epic 10: CMS - Content Management System

| Field | Detail |
|-------|--------|
| **Platform** | Web application |
| **Users** | Admin (Platform Management) |
| **Purpose** | Quản lý toàn bộ nội dung, vật phẩm, Star pricing và cấu hình platform |

#### F10.1: Item & Gift Catalog Management
- CRUD digital gift items (tên, hình ảnh, tier, sort order)
- Cấu hình Star cost cho từng item
- Gán gift items cho idol cụ thể (khu Hẹn Ước)
- Preview gift trên mobile app
- Quản lý seasonal themes

#### F10.2: Star Pricing Configuration
- Set/update Star costs cho: Digital Gifts, Exclusive Content, Space Items
- Bulk update pricing
- Price history tracking
- Preview pricing trước khi publish lên mobile app

#### F10.3: Content Moderation
- Review và moderate user-generated content (Fan Letters, Fan Projects)
- Auto-scan rules configuration
- Report management
- Ban/suspend user accounts

#### F10.4: Mission & Reward Configuration
- CRUD daily missions (hành động, thưởng, giới hạn)
- CRUD milestone missions
- Configure check-in streak rewards
- Configure points thresholds cho Fan Levels

#### F10.5: User Management
- View/search users (Fan, Idol, Management)
- Account status management (active, suspended, banned)
- Idol account invitations
- Role management

---

## IV. STAR ECONOMY (Virtual Currency)

> Stars là virtual currency nội bộ. Có 2 nguồn: kiếm qua engagement hoặc **mua qua IAP (In-App Purchase)**. Không có quy đổi ngược (rút tiền). Giá trị vật phẩm được cấu hình qua CMS.
> Chi tiết bundle pricing và balance model sẽ được bổ sung sau.

### 4.1. Cách kiếm Stars
| Source | Chi tiết | Loại |
|--------|----------|------|
| Check-in streak | Ngày 30: +2 stars | Free |
| Mission cột mốc | Level 3: +2 stars, Level 4: +2 stars, Level 5: +5 stars | Free |
| IAP Bundle | Mua Star bằng tiền thật qua App Store / Google Play | Paid |

> Nguồn kiếm Stars free có thể mở rộng qua CMS. IAP bundle tiers (giá + số star) sẽ được define riêng.

### 4.2. Cách tiêu Stars
| Action | Range | Cấu hình |
|--------|-------|----------|
| Collect Digital Gift | 10 - 1,500 stars/gift | CMS |
| Mở khóa Exclusive Content (Merch) | Tuỳ config | CMS |
| Mua Space Item (Inventory) | Tuỳ config, 0 = không bán | CMS |

### 4.3. Points System (Engagement-based)
| Action | Points | Giới hạn |
|--------|--------|----------|
| Check-in daily | 3-15 pts (theo streak) | 1/ngày |
| Tham gia Fan Project | Configurable | Per project |
| Mission cột mốc | Varies | One-time |

> Points tích lũy **per idol** -> lên level Fan (0->4). Point rewards configurable qua CMS.

---

## V. DIAGRAM FLOWS

### 5.1. Fan - Main User Journey

```
┌──────────┐    ┌───────────┐    ┌────────────┐    ┌─────────────┐
│  Install  │───>│  Google    │───>│ Onboarding │───>│  Home Feed  │
│   App     │    │  Sign-in   │    │ (profile + │    │ (Fandoms)   │
│           │    │            │    │  fandom)   │    │             │
└──────────┘    └───────────┘    └────────────┘    └──────┬──────┘
                                                          │
                    ┌─────────────────────────────────────┤
                    │              │              │        │
              ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼────┐ ┌─▼──────────┐
              │ Idol Hub  │ │Your Space │ │ Digital  │ │  Game Hub  │
              │           │ │           │ │  Gift    │ │            │
              │ -Timeline │ │ -Studio   │ │          │ │ -Tiles Hop │
              │ -Fan Letter│ │ -Mission  │ │ -Map     │ │ -Rhythm    │
              │ -Calendar │ │ -Calendar │ │ -Card    │ │  Hive      │
              │           │ │           │ │          │ │            │
              └─────┬─────┘ └─────┬─────┘ └────┬────┘ └────────────┘
                    │             │             │
              ┌─────▼─────┐ ┌────▼──────┐ ┌────▼─────┐
              │ Fan       │ │ Digital   │ │ Account  │
              │ Project   │ │ Merch     │ │ Settings │
              │           │ │           │ │          │
              │ -Browse   │ │ -Scan QR  │ │ -Profile │
              │ -Create   │ │ -Gallery  │ │ -Phone   │
              │ -Join     │ │ -Exclusive│ │ -Noti    │
              │           │ │           │ │ -Social  │
              └───────────┘ └───────────┘ └──────────┘
```

### 5.2. Fan Project Lifecycle

```
Fan (Level 3+)
      │
      ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Step 1:      │───>│ Step 2:      │───>│ Step 3:      │
│ Thông tin    │    │ Xác nhận     │    │ Review &     │
│ dự án       │    │ quy định     │    │ Submit       │
└──────────────┘    └──────────────┘    └──────┬───────┘
      ▲ (draft saved)                          │
      │                                        ▼
      │                              ┌──────────────────┐
      │                              │   Auto-scan      │
      │                              │   (<=24h)        │
      │                              └────┬─────────┬───┘
      │                                   │         │
      │                              Pass │    Fail │
      │                                   ▼         ▼
      │                          ┌──────────┐  ┌──────────┐
      │                          │ PENDING   │  │ REJECTED │
      │                          │ (chờ Mgr) │  │ (system) │
      │                          └──┬────┬──┘  └──────────┘
      │                    Approve  │    │  Reject
      │                             ▼    ▼
      │                     ┌────────┐  ┌──────────┐
      │                     │  LIVE  │  │ REJECTED │
      │                     │        │  │ (manager)│───> Fan nộp lại ─┐
      │                     └───┬────┘  └──────────┘                  │
      │                         │                                     │
      │◄──────────────────────────────────────────────────────────────┘
      │                    end_date
      │                         ▼
      │                  ┌────────────┐
      │                  │ COMPLETED  │
      │                  │ (hiển thị  │
      │                  │  kết quả)  │
      │                  └────────────┘
```

### 5.3. Fan Letter Flow

```
Fan
  │
  ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ CTA Create   │───>│ Chọn "Fan    │───>│ Chọn Idol    │
│ (floating    │    │  Letter"     │    │              │
│  button)     │    │              │    │              │
└──────────────┘    └──────────────┘    └──────┬───────┘
                                               │
                                               ▼
                                    ┌──────────────────┐
                                    │ Viết nội dung    │
                                    │ (text, max 2000  │
                                    │  ký tự)          │
                                    └────────┬─────────┘
                                             │
                                             ▼
                                    ┌──────────────────┐
                                    │ Gửi              │
                                    │ -> Hiển thị ngay │
                                    │   trong Idol Hub │
                                    │ -> Idol + Fan    │
                                    │   like/react     │
                                    └──────────────────┘
```

### 5.4. Star Economy Flow

```
                    ┌─────────────────────────┐
                    │      STAR SYSTEM         │
                    │  (Virtual Currency)      │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
         ┌────▼─────┐    ┌─────▼──────┐    ┌──────▼─────┐
         │  EARN    │    │   SPEND    │    │  POINTS    │
         │          │    │            │    │  (per Idol)│
         │-Check-in │    │-Collect    │    │            │
         │ streak   │    │ Gift      │    │-Check-in   │
         │-Mission  │    │ (10-1500) │    │-Join FP    │
         │ rewards  │    │-Unlock    │    │-Missions   │
         │          │    │ merch     │    │            │
         └──────────┘    └─────┬─────┘    └──────┬─────┘
                               │                  │
                     CMS configures              │
                     Star costs                   ▼
                                          ┌─────────────┐
                                          │  Level up   │
                                          │  -> Studio  │
                                          │  -> Badge   │
                                          │  -> Gates   │
                                          └─────────────┘
```

### 5.5. Idol Management - Post Flow

```
Manager Login
      │
      ▼
┌──────────────┐
│  Idol Feed   │
│  (preview)   │──────────────────────────┐
└──────┬───────┘                          │
       │                                  │
  ┌────▼─────┐                    ┌───────▼───────┐
  │ Tạo bài  │                    │ Quản lý       │
  │ đăng mới │                    │ nội dung      │
  └────┬─────┘                    └───────┬───────┘
       │                                  │
  ┌────▼──────────────┐          ┌────────▼────────┐
  │ Chọn loại:        │          │ Tabs:            │
  │ - Thông thường    │          │ - Đã đăng       │
  │ - Sự kiện         │          │ - Nháp          │
  └────┬──────────────┘          │ - Đã lên lịch   │
       │                          │ - Sự kiện nổi bật│
       ▼                          └─────────────────┘
  ┌────────────────┐
  │ Nhập nội dung  │
  │ + Media upload │
  └────┬───────────┘
       │
  ┌────▼────────┐   ┌──────────────┐   ┌──────────────┐
  │ Lưu nháp   │   │ Hẹn giờ      │   │ Xem trước    │
  │             │   │ (bài thường  │   │ & Đăng ngay  │
  └─────────────┘   │  only)       │   └──────────────┘
                    └──────────────┘
```

---

## VI. DATA ENTITIES (High-Level)

```
Users
├── Fan
│   ├── id, email, username, avatar, bio
│   ├── phone (verified)
│   ├── google_id, created_at
│   └── FanIdolRelation[] (per idol: level, points, default_space)
│
├── IdolManagement
│   ├── id, email, google_id
│   ├── idol_profile_id (FK)
│   └── invited_at, accepted_at
│
├── IdolArtist
│   ├── id, idol_profile_id (FK)
│   └── limited permissions
│
└── Admin
    ├── id, email, password_hash
    └── role, permissions

IdolProfile
├── id, stage_name, fandom_name, bio, debut_date
├── avatar, cover_image
├── social_links (FB, YT, Spotify, TikTok)
└── fandom_color, theme_config

Post
├── id, idol_profile_id (FK), type (NORMAL | EVENT)
├── content, media[], published_at, scheduled_at
├── event_name, event_description, event_date
├── is_featured_event, hearts_count
└── status (DRAFT | SCHEDULED | PUBLISHED | HIDDEN)

FanLetter
├── id, fan_id (FK), idol_profile_id (FK)
├── content (max 2000 chars)
├── likes_count, reactions_count
├── created_at
└── status (ACTIVE | HIDDEN)

FanLetterReaction
├── id, fan_letter_id (FK), user_id (FK), user_type
├── reaction_type (LIKE | ...)
└── created_at

Event (Calendar)
├── id, idol_profile_id (FK), post_id (FK)
├── name, description, start_time, end_time
├── status (UPCOMING | ONGOING | PAST)
├── links[], media[]
└── hearts_count

DigitalMerch
├── id, idol_profile_id (FK)
├── qr_code (unique), claimed_by (FK -> Fan)
├── category (ALBUM | CARD | PHOTOBOOK | POSTER)
├── name, description, image
├── exclusive_content[] (type, content_url, star_cost)
└── claimed_at

DigitalGift
├── id, name, image, star_cost
├── tier (ASIN_SAO | TINH_TU | HAO_QUANG | HEN_UOC)
├── idol_profile_id (nullable, for custom gifts)
├── is_active, sort_order
└── created_by (admin), updated_at

GiftCollection
├── id, fan_id (FK), idol_profile_id (FK)
├── gift_id (FK)
├── stars_spent, created_at
└── brightness_contribution

FanProject
├── id, creator_fan_id (FK), idol_profile_id (FK)
├── name, description, category
├── start_date, end_date
├── status (PENDING | LIVE | REJECTED | COMPLETED)
├── rejected_by (SYSTEM | MANAGER), reject_reason
├── participant_count
└── attachments[]

FanProjectParticipant
├── id, fan_project_id (FK), fan_id (FK)
├── joined_at
└── points_earned

YourSpace (per Fan per Idol)
├── id, fan_id (FK), idol_profile_id (FK)
├── level (0-4), points, is_default
├── items_placed[] (item_id, x, y, scale, rotation)
└── theme_config (auto from idol)

SpaceItem  ← Đồ nội thất (Furniture Panel trong Studio)
├── id, name, image
├── rarity (COMMON | RARE | EPIC | LEGEND)
├── idol_specific (boolean), idol_profile_id (nullable)
├── star_cost (int, 0 = không bán — chỉ unlock qua mission/level)
├── source_rule (PURCHASE_ONLY | MISSION_ONLY | BOTH)
└── is_active (CMS toggle)

FanSpaceItemOwnership  ← Fan sở hữu furniture nào
├── id, fan_id (FK), space_item_id (FK)
├── acquired_source (MISSION | STREAK | MILESTONE | PURCHASE)
└── acquired_at

InventoryItem  ← Cards / Digital Assets / Game reward items (Inventory F3.5)
├── id, name, image, description
├── type (CARD | DIGITAL_ASSET | GAME_ITEM)
├── idol_profile_id (nullable)
├── star_cost (int, 0 = chỉ earn)
├── source_rule (PURCHASE_ONLY | EARN_ONLY | BOTH)
└── is_active (CMS toggle)

FanInventory  ← Items fan đang có trong Inventory (max 10 slots)
├── id, fan_id (FK), inventory_item_id (FK)
├── acquired_source (MISSION | STREAK | MILESTONE | QR_SCAN | GAME | PURCHASE)
└── acquired_at

Mission
├── id, type (DAILY | MILESTONE | EVENT)
├── name, description, action_type
├── reward_points, reward_stars, reward_item_id
├── limit, reset_schedule
├── idol_profile_id (nullable)
└── is_active (CMS toggle)

FanMissionProgress
├── id, fan_id (FK), mission_id (FK)
├── completed, completed_at
├── streak_count (for check-in)
└── last_streak_date

StarTransaction
├── id, fan_id (FK)
├── type (COLLECT_GIFT | UNLOCK_MERCH | PURCHASE_ITEM | IAP_TOPUP | REWARD)
├── amount (dương = nhận, âm = tiêu), balance_after
├── reference_id, reference_type
├── iap_receipt (nullable, lưu receipt string cho IAP_TOPUP)
└── created_at

Notification
├── id, user_id (FK), user_type (FAN | MANAGEMENT | ARTIST | ADMIN)
├── type (POST | EVENT | LETTER | PROJECT | SYSTEM | MISSION)
├── title, body, deep_link
├── is_read, is_push_sent
└── created_at

IdolInvitation
├── id, email, invited_by (admin_id FK)
├── otp_code, otp_expires_at
├── status (PENDING | ACCEPTED | REJECTED | EXPIRED)
└── created_at, accepted_at

CMSAuditLog
├── id, admin_id (FK)
├── action, entity_type, entity_id
├── changes (JSON), created_at
└── ip_address
```

---

## VII. OPEN QUESTIONS & GAPS

### 7.1. Business Logic Gaps

| # | Câu hỏi | Ảnh hưởng | Độ ưu tiên |
|---|---------|-----------|------------|
| 1 | ~~**Star earning balance**~~ — **Đã xác nhận:** IAP (mua Star) được phép. Balance details sẽ bổ sung sau. | Economy | ~~CRITICAL~~ → RESOLVED |
| 1a | **IAP bundle tiers**: Bao nhiêu tier? (vd: $0.99/50⭐, $4.99/300⭐, $9.99/700⭐). Cần để design pricing + data model | Economy | **CRITICAL** |
| 1b | **Space Item price range**: Min/max star cost cho item trong kho? Cần để CMS config + balance | Economy | **HIGH** |
| 1c | **Apple/Google 30% cut**: Ai handle pricing để đảm bảo margin? | Finance/Legal | **HIGH** |
| 2 | **Points mechanism detail**: Exact points cho mỗi action? CMS configurable hay fixed? | Engagement | HIGH |
| 3 | **Fan Project participation**: "Tham gia" cụ thể là gì? Chỉ đăng ký/pledge hay có cam kết? | Feature | HIGH |
| 4 | **Fan Letter moderation**: Không duyệt trước -> cần auto-filter cho spam/abuse? | Compliance | HIGH |
| 5 | **CMS scope**: CMS cần những tính năng nào ngoài item/pricing management? | Architecture | HIGH |
| 6 | **Shop / Merchandise**: Physical merch redirect Shopee/Lazada — app hiển thị link/QR hay tích hợp sâu hơn? | Feature | MEDIUM |
| 7 | **Exclusive Content pricing**: Range Star cost bao nhiêu? Configurable qua CMS? | Pricing | MEDIUM |
| 8 | **Game Hub rewards post-MVP1**: MVP1 không thưởng, cần biết plan cho MVP2 để thiết kế data model | Architecture | MEDIUM |
| 9 | **Multi-management account**: Shared credentials hay sub-accounts? | Auth | MEDIUM |
| 10 | **Content moderation**: Auto-scan criteria cho fan project và fan letter | Compliance | MEDIUM |
| 11 | **Exclusive content DRM**: "Không thể cap/quay màn hình" - technical feasibility trên RN? | Technical | MEDIUM |
| 12 | **Your Space Level 5**: Bảng rank chỉ đến Level 4 (13000 pts) nhưng mission có "Lên level 5" | Data inconsistency | HIGH |
| 13a | **Inventory item usage**: Cards/Digital Assets/Game items trong Inventory dùng được gì? Chỉ xem/collect, hay có thể dùng trong game, đặt vào phòng, hay share? | Feature scope | **HIGH** |
| 13b | **"+New" button trong Furniture Panel**: Dẫn đến đâu? Shop mua bằng Star, hay catalog xem/unlock? Flow cụ thể? | Feature scope | **HIGH** |

### 7.2. Technical Considerations

| Concern | Detail | Suggestion |
|---------|--------|------------|
| Real-time updates | Tim count (<5s), Dashboard cards | WebSocket hoặc SSE cho high-freq; polling cho low-freq |
| QR unique constraint | 1 QR = 1 account, concurrent scan | DB unique constraint + optimistic locking |
| Star transaction atomicity | Trừ star + collect gift phải atomic | DB transaction + idempotency key |
| Canvas rendering (Your Space) | 60% screen, drag-drop, pinch-zoom | Consider react-native-canvas hoặc WebGL view |
| Music game performance | Rhythm sync, tile rendering | Consider native modules cho game engine |
| Streak timezone | Reset 00:00 UTC+7 | Server-side cron, store timezone per user |
| CMS architecture | Web portal, separate từ mobile API | Next.js / React admin panel, shared NestJS API |
| Fan Letter scalability | Public feed per idol, likes/replies | Pagination + caching strategy |

---

## VIII. FEATURE DEPENDENCY MAP

```
Sign Up/Login
    │
    ▼
Onboarding (Profile + Choose Fandom)
    │
    ├──> Home Feed (aggregated fandoms)
    │
    ├──> Idol Hub ──────────────────────> Calendar
    │        │                               │
    │        ├──> Timeline Feed              │
    │        │        │                      │
    │        │        ▼                      ▼
    │        │   Thả tim ──────────> Points System ◄── Daily Mission
    │        │                            │    ▲
    │        ├──> Fan Letter              │    │
    │        │   (like/react)             │    │
    │        │                            ▼    │
    │        └──> Digital Gift ──────> Your Space
    │             (collect with Stars)  (Studio)
    │                    ▲
    │                    │ CMS configures
    │                    │ Star costs
    │
    ├──> Digital Merch
    │        │
    │        └──> Scan QR ──> Exclusive Content
    │
    ├──> Fan Project
    │        │
    │        ├──> Create (requires Level 3)
    │        └──> Join ──> Points
    │
    ├──> Game Hub (standalone, MVP1 no rewards)
    │
    ├──> Account Settings
    │        ├──> Phone verification
    │        ├──> Social linking
    │        └──> Notification config
    │
    ├──> CMS (Web Portal - Admin)
    │        ├──> Item/Gift catalog
    │        ├──> Star pricing
    │        ├──> Mission config
    │        ├──> Content moderation
    │        └──> User management
    │
    └──> Notification System (cross-cutting)
```

**Critical Path cho MVP1:**
1. Auth (Google Sign-in) -> Onboarding
2. CMS setup (item catalog + Star pricing)
3. Idol Hub + Fan Letter (core content + interaction)
4. Digital Gift (engagement with Stars)
5. Your Space + Points System (engagement loop)
6. Fan Project (community feature)
7. Digital Merch (QR system)
8. Game Hub (engagement, no BE needed MVP1)

---

## IX. NON-FUNCTIONAL REQUIREMENTS (Inferred)

| Category | Requirement | Source |
|----------|------------|--------|
| Performance | Feed load <= 2s | PRD AC |
| Performance | Calendar event load <= 1s | PRD AC |
| Performance | QR scan <= 2s | PRD AC |
| Performance | Studio switch <= 5s | PRD AC |
| Performance | Snapshot render <= 3s | PRD AC |
| Consistency | Tim count update <= 5s | PRD AC |
| Consistency | Star balance real-time | CMS + mobile |
| Availability | Offline: cache + error banner | PRD AC |
| Security | OTP: rate limit + lock mechanism | PRD requirement |
| Security | Exclusive content: no screenshot/share | PRD requirement |
| Localization | Timezone: UTC+7 (Vietnam) | PRD requirement |
| CMS | Web responsive, role-based access | Platform requirement |
| CMS | Audit log for all config changes | Platform requirement |

---

*Document generated by BA Agent | Fanation Project | Updated: 2026-05-23*
