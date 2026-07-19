# FSD — MVP2: Cơ chế Xếp hạng và Tính điểm (Fan Rank/Level & Point System)

> **Loại tài liệu:** Functional Specification Document (FSD)
> **CR liên quan:** MVP2
> **Nguồn tham chiếu:** `Fanation - Mechanic of Ranking - Mission - Point Star Card.docx` (mục 1, 3, 4 — brief kỹ thuật của Product Owner Võ Quân, 07/07/2026), `mvp2_Feature_Breakdown.md` (mục 6 — "Poin Star Card" là input còn thiếu cho tài liệu này), `ba.md`/`pd.md` (tech stack, lịch sử quyết định Star Economy/dual-currency)
> **Tech stack:** React Native (FE) · NestJS (BE) · PostgreSQL (DB)
> **Trạng thái:** Draft — bảng 6 Level (0-5), model Star↔Points (`k`), và catalog Tier T1-T5 đã **CHỐT** (2026-07-19); còn vài giá trị "xxx" chưa điền nhưng không chặn tiến độ, bổ sung sau (xem mục 8)

---

## 1. Tổng quan

Tính năng định nghĩa cách Fan **tích điểm (Points/Pts)** qua các hành động tương tác hàng ngày, và cách điểm tích luỹ đó **tự động nâng Rank/Level** của Fan (0 → 5). Level là thước đo tiến trình chính của Fan, hiển thị dưới dạng badge cạnh avatar/tên, và gate việc unlock Studio (Your Space) theo cấp độ cao hơn.

Cơ chế gồm 2 phần không tách rời:

1. **Bảng ngưỡng Rank** — Level 0-5, mỗi Level ứng với 1 ngưỡng điểm tích luỹ cố định (mục 6.2).
2. **Catalog Tier hành động (T1-T5)** — danh sách chuẩn hoá "hành động nào cho bao nhiêu điểm, giới hạn bao nhiêu lần" (mục 6.3). Đây chính là câu trả lời cho "tính điểm" — hiện tại các nhiệm vụ (mission) nên ánh xạ vào 1 trong 5 Tier này; việc có mở rộng thêm catalog hoặc phát sinh cách tính điểm mới trong tương lai hay không vẫn để ngỏ, chưa chốt.

**Phân chia trách nhiệm khi triển khai (đã chốt với PO):**
1. **CMS là nơi cấu hình catalog Tier** — hằng số `G`/`k`, ngưỡng Level, đơn giá + giới hạn từng dòng Tier T1-T4 đều sửa trên CMS, không hard-code trong Mobile app lẫn BE.
2. **Mobile app chỉ là nơi trigger hành động** — khi Fan thực hiện 1 hành động (check-in, comment, viết thư...), Mobile app gọi API tương ứng; **toàn bộ việc tính điểm, kiểm tra giới hạn, kiểm tra lên Level đều do BE xử lý** dựa trên config đọc từ CMS — Mobile app không tự tính điểm, chỉ hiển thị kết quả BE trả về (điểm mới, Level mới nếu có).

---

## 2. Phạm vi (Scope)

### Trong phạm vi
- 3 hằng số gốc: `1 Star = 1.000 VNĐ`, `G` (tốc độ tích điểm tối đa khi cày miễn phí), `k` (tỷ lệ điểm quy đổi khi chi Star)
- Bảng 6 cấp Level (0→5): tên gọi + ngưỡng điểm tích luỹ tương ứng
- Cơ chế **auto level-up**: hệ thống tự kiểm tra và nâng Level khi điểm tích luỹ chạm ngưỡng, không cần Fan thao tác, nhưng có gửi thông báo (noti) cho Fan khi họ lên Level
- Catalog Tier T1-T5 (danh sách hành động & phân nhóm Tier) — cấu trúc catalog cố định, nhưng Admin có quyền thay đổi các tham số của từng hành động (điểm/lượt, giới hạn tần suất, tổng tối đa/tháng) qua CMS
- Quy tắc Points được tích luỹ **riêng theo từng cặp (Fan, Idol)** — không gộp chung giữa các idol khác nhau (nhất quán với cách Your Space/Studio vận hành theo từng idol). **Ngoại lệ:** hành động "Viết thư cho idol" tính cap gộp chung across mọi idol Fan theo dõi, để tránh farm điểm bằng cách viết thư nhiều idol khác nhau (xem mục 6.3)
- Quan hệ **Star → Points** qua Tier T5 (chi Star trực tiếp: nạp Star, donate fan project, mua Idol E-card bằng Star → quy đổi thêm điểm theo tỷ lệ `k`, không giới hạn tần suất)

---

## 3. Actors

| Actor | Vai trò |
|---|---|
| **Fan** | Thực hiện hành động tương tác, tích điểm, tự động lên Level theo từng idol đang theo dõi |
| **Idol/Content** | Hưởng lợi gián tiếp (Fan tương tác nhiều hơn với nội dung/idol) — không thao tác trực tiếp lên cơ chế điểm |
| **Admin** | Cấu hình hằng số `G`/`k`, ngưỡng Level, đơn giá + giới hạn từng dòng Tier qua CMS |
| **BE System (Point/Rank Engine)** | Cộng dồn điểm theo giới hạn Tier, kiểm tra ngưỡng, trigger sự kiện lên Level, emit event cho các hệ thống phụ thuộc (Studio unlock, Badge, Mission progress) |

---

## 4. User Stories

**US-1 (Fan)**
> Là một Fan, tôi muốn tích điểm mỗi khi thực hiện hành động tương tác với idol tôi theo dõi (check-in, comment, viết thư, thêm lịch...), để tiến trình Rank của tôi với idol đó tăng dần.

**US-2 (Fan)**
> Là một Fan, khi điểm tích luỹ chạm ngưỡng Level tiếp theo, tôi muốn Level của tôi tự động tăng và badge cạnh avatar/tên cập nhật ngay, mà không cần thao tác xác nhận.

**US-3 (Fan)**
> Là một Fan, tôi muốn biết rõ giới hạn điểm mỗi hành động (vd tối đa 5 lượt comment/ngày) để không bất ngờ khi thấy hành động không cộng thêm điểm nữa.

**US-4 (Fan)**
> Là một Fan, khi tôi chi Star (nạp Star, donate fan project, hoặc mua Idol E-card bằng Star), tôi muốn được cộng thêm điểm tương ứng số Star đã chi, để việc trả phí cũng đóng góp vào tiến trình Rank, song song với cách cày miễn phí.

**US-5 (Admin)**
> Là Admin, tôi muốn chỉnh hằng số `G`, `k`, ngưỡng Level, hoặc đơn giá/giới hạn từng dòng Tier ngay trên CMS, để cân bằng lại tốc độ tích điểm mà không cần Tech deploy lại code.

---

## 5. Sơ đồ luồng (Diagrams)

### 5.1 Luồng Fan — thực hiện hành động, tích điểm, lên Level

```mermaid
flowchart TD
    subgraph MOBILE["📱 Mobile App (FE) — chỉ trigger + hiển thị"]
        A[Fan thực hiện 1 hành động: check-in / comment / viết thư / donate...] --> B["Báo hành động lên hệ thống — không tự tính hay gửi kèm điểm"]
    end

    subgraph BE["⚙️ Backend (BE) — toàn bộ logic tính điểm"]
        B --> C["Xác định hành động thuộc Tier nào (T1-T5), đọc đơn giá/giới hạn từ cấu hình (do CMS quản lý)"]
        C --> D{"Đã đạt giới hạn tần suất/tháng của hành động này chưa?"}
        D -- Đã đạt cap --> E["Hành động vẫn xử lý thành công (vd comment vẫn lưu), nhưng KHÔNG cộng thêm điểm"]
        D -- Chưa đạt cap --> F["Cộng điểm theo đơn giá Tier vào sổ Points của cặp (Fan, Idol)"]
        F --> G["Cập nhật bộ đếm tần suất hành động (ngày/tháng)"]
        G --> H{"Tổng điểm tích luỹ mới có vượt ngưỡng Level kế tiếp?"}
        H -- Có --> I["Auto tăng Level (không cần Fan xác nhận)"]
        I --> J["Emit event 'fan.level_up' (user_id, idol_id, new_level)"]
        J --> K[Unlock Studio/Your Space theo Level mới]
        J --> L["Trigger downstream: reward Milestone (nếu có), Mission progress (hệ thống riêng)"]
    end

    E --> M["Xác nhận lại cho Mobile app: điểm mới + (nếu có) level mới"]
    H -- Không --> M
    L --> M

    subgraph MOBILE2["📱 Mobile App (FE)"]
        M --> N["Cập nhật UI: thanh tiến trình điểm, badge Level cạnh avatar/tên (nếu đổi)"]
    end
```

> **Nguyên tắc bắt buộc (yêu cầu chức năng, không quy định cách triển khai):** Mobile app không được tự cộng điểm hay tự xác định Level — mọi con số hiển thị phải lấy từ xác nhận của hệ thống. Điểm/giới hạn/ngưỡng luôn dựa trên cấu hình do CMS quản lý (mục 6.3), không hard-code trong app.

### 5.2 Luồng chi Star quy đổi thêm điểm (Tier T5)

```mermaid
flowchart TD
    A["Fan chi Star: nạp Star / donate fan project / mua Idol E-card bằng Star"] --> B["BE xử lý giao dịch chi Star tương ứng (nạp/donate/mua E-card)"]
    B --> C["Song song: cộng điểm = số Star đã chi × k (hằng số quy đổi)"]
    C --> D["Cộng vào sổ Points của cặp (Fan, Idol) — không giới hạn tần suất, không có cap tháng (T5 = ∞)"]
    D --> E{"Tổng điểm tích luỹ mới có vượt ngưỡng Level kế tiếp?"}
    E -- Có --> F[Trigger auto level-up — giống luồng 5.1 bước I-M]
    E -- Không --> G[Kết thúc, chỉ cập nhật thanh tiến trình]
```

---

## 6. Yêu cầu chức năng chi tiết

### 6.1 Bối cảnh — 3 loại "tiền tệ" liên quan (tham chiếu, không thuộc phạm vi build của FSD này)

| Tiền tệ | Chức năng chính | Liên quan tới cơ chế điểm |
|---|---|---|
| **Pts (điểm)** | Tích luỹ để tự động nâng Rank/Level | Đối tượng chính của FSD này |
| **Star** | Donate cho idol; mua Idol E-card (C, R) | Chi Star → cộng thêm Points theo `k` (Tier T5) — xem mục 5.2 |
| **Idol E-card** | Đổi lấy vật phẩm trang trí Studio của đúng idol phát hành thẻ | Ngoài phạm vi — xem `mvp2_Feature_Breakdown.md` mục 6. Nêu ở đây chỉ để tham khảo bối cảnh (Star → E-card → vật phẩm) |

### 6.2 Bảng ngưỡng Level (Rank)

| Level | Tên gọi | Ngưỡng điểm tích luỹ | Thời gian cày chay thuần (ước tính) |
|---|---|---|---|
| 0 | Fan Nhí | 0 | Ngay khi vào app |
| 1 | Fan Sôi Nổi | 500 | ~2 tuần |
| 2 | Fan Ưu Tú | 2.500 | ~2 tháng |
| 3 | Fan Cứng | 5.500 | ~4,4 tháng |
| 4 | Fan Đắm Đuối | 15.000 | ~12 tháng |
| 5 | Fan-tasic | 30.000 | ~24 tháng |

- Ngưỡng = số tháng cày chay thuần × `G`. Level 4 (12 tháng) và Level 5 (24 tháng) là 2 mốc cứng khoá thang đo, các mốc còn lại nội suy theo cấp số nhân.
- `G` (tốc độ tích điểm tối đa khi cày chay) = **1.250 điểm/tháng** — xem đối chiếu số liệu thực tế ở mục 6.3 và cảnh báo ở mục 8 (OQ-3).
- `k` (tỷ lệ điểm/Star khi chi Star) = **1 điểm/Star** (số nguyên).
- Level, ngưỡng, `G`, `k` **phải lưu dạng config trên CMS**, không hard-code — đổi `G`/`k` chỉ cần chạy lại phép tính ngưỡng, không sửa từng dòng catalog Tier.

### 6.3 Catalog Tier hành động (T1-T5)

Nguyên tắc: hành động cùng độ phức tạp → cùng mức điểm, bất kể loại hành động. Mọi nhiệm vụ tương lai phải ánh xạ vào đúng 1 trong 5 dòng dưới đây.

| Tier | Độ phức tạp | Hành động | Điểm | Giới hạn | Tổng tối đa/tháng |
|---|---|---|---|---|---|
| **T1 — Micro** (<5s) | Rất thấp | Check-in ngày | 5đ | 1 lần/ngày | 150 |
| | | Tương tác nhẹ (thả tim / bấm mở stream nhạc) | 2đ/lượt | Tối đa 3 lượt/ngày | 180 |
| | | Thay item trong My Space (skin/vật dụng trang trí) | 1đ/lượt | 1 lượt/ngày | 30 |
| **T2 — Light** (~30s) | Thấp | Comment bài đăng của idol | 3đ/lượt | Tối đa 5 lượt/ngày | 450 |
| | | Thêm sự kiện vào My Calendar | 2đ/sự kiện | Tối đa 2 lượt/ngày | 120 |
| **T3 — Medium** (2-5 phút) | Trung bình | Viết thư cho idol | 20đ/thư | Tối đa 4 thư/tháng — **tính gộp chung across tất cả idol Fan đang theo dõi** (ngoại lệ so với nguyên tắc điểm theo từng cặp Fan-Idol ở mục 2), không tính riêng theo từng idol | 80 |
| | | Donation (tặng Gift) | 5đ/lượt | 1 lần/ngày | 150 |
| **T4 — Milestone** (cột mốc) | Trung bình-cao | Giữ streak 7/14/30 ngày | 20/25/30đ (+item mốc 14, 30) | 1 lần/mốc/tháng | 75 |
| | | Thêm đủ 10 sự kiện/tháng | 20đ | 1 lần/tháng | 20 |
| | | Tương tác đủ 20 lượt/tháng | 25đ | 1 lần/tháng | 25 |
| | | Nạp mốc 100/200/**xxx** Star | 10/20/**xxx** điểm/mốc | 3 lần/tháng | **chưa xác định** |
| | | Lên hạng | 0đ | — | Quà: Idol E-card (loại chưa chốt, xem mục 8) |
| | | Thu thập đủ **xxx** thẻ C / **xxx** thẻ R / ... | 10đ / 100đ / **xxx**đ | Không giới hạn | ∞ |
| **T5 — Monetary** (chi Star) | Quy đổi tiền | Nạp Star / donate fan project / mua Idol E-card bằng Star | 1đ/Star (= `k`) | Không giới hạn | ∞ |

> **Lưu ý:** T5 **không** thuộc hệ thống Mission (Phần 5 tài liệu nguồn) — là hành động tự do, luôn sẵn có, không có trạng thái hoàn thành/re-new, không giới hạn tần suất.

**Cấu hình qua CMS:** mỗi dòng hành động trong catalog trên đều được cấu hình qua CMS — Admin có quyền chỉnh sửa điểm/lượt, giới hạn tần suất, và tổng tối đa/tháng của từng hành động. Thay đổi chỉ áp dụng cho giao dịch điểm phát sinh **sau** thời điểm lưu, **không hồi tố** điểm đã cộng trước đó (xem Edge case #6).

**CMS — module quản lý User:** màn quản lý Fan trong CMS cần hiển thị được Star, Point, và Level hiện tại của từng Fan.

### 6.4 Mobile App (FE) — Trigger hành động & hiển thị kết quả cho Fan

Mobile app **không tự tính điểm hay tự xác định lên Level** — số liệu điểm/Level luôn do hệ thống xác nhận.

- Khi Fan thực hiện hành động (check-in, comment, viết thư, thêm lịch...) → hành động đó cần được báo lên hệ thống để tính điểm; cơ chế đồng bộ cụ thể (thời điểm gọi, cách cập nhật UI) do Dev quyết định, cân nhắc thêm bài toán performance của app
- Khi có xác nhận lên Level mới → nên có thông báo (notification) cho Fan, thống nhất với cách xử lý noti đã nêu ở mục 2 (Phạm vi)

---

## 7. Edge Cases

| # | Tình huống | Xử lý đề xuất |
|---|---|---|
| 1 | Nhiều hành động cộng điểm xảy ra gần như đồng thời cho cùng 1 Fan/idol (race condition), có thể khiến kiểm tra ngưỡng Level bị lệch | Cộng điểm + kiểm tra ngưỡng Level phải nằm trong 1 transaction atomic, lock theo `(user_id, idol_id)` |
| 2 | Fan thực hiện hành động đã đạt cap tần suất trong ngày/tháng (vd comment lần thứ 6/ngày) | Hành động vẫn thực hiện được bình thường (vẫn comment thành công), chỉ **không cộng thêm điểm** — không chặn hành động |
| 3 | Đầu tháng mới, bộ đếm giới hạn tần suất reset | Chỉ reset bộ đếm tần suất (ngày/tháng), **giữ nguyên** điểm tích luỹ tổng và Level hiện tại — Level không bao giờ tự giảm |
| 4 | Fan theo dõi nhiều idol, mỗi idol có điểm/Level riêng | Điểm/Level luôn gắn theo ngữ cảnh 1 idol cụ thể — không gộp chung |
| 5 | Fan chi Star mua Idol E-card — vừa nhận E-card vừa được cộng điểm T5 | 2 hiệu ứng phải xảy ra cùng lúc trong 1 transaction, không được chỉ thực hiện 1 trong 2 nếu giao dịch thất bại giữa chừng |
| 6 | Admin sửa ngưỡng Level hoặc đơn giá Tier khi Fan đang có điểm tích luỹ gần ngưỡng cũ | Không hồi tố — Fan giữ nguyên điểm đã tích, chỉ áp ngưỡng/đơn giá mới cho giao dịch phát sinh sau khi lưu cấu hình |
| 7 | Fan Letter bị Idol report vi phạm sau khi điểm "Viết thư cho idol" đã được cộng | Điểm của letter đó bị **huỷ bỏ** — trừ khỏi tổng điểm tích luỹ ngay khi report được xử lý. **Level giữ nguyên, không hạ cấp** kể cả khi tổng điểm sau khi trừ tụt dưới ngưỡng Level hiện tại — vì Level giảm ảnh hưởng lớn tới phạm vi mua Space Item đã unlock |

---

## 8. Open Questions (chưa có câu trả lời từ stakeholder)

| # | Câu hỏi | Ảnh hưởng | Ghi chú |
|---|---|---|---|
| ~~1~~ | ~~Mâu thuẫn bảng Level~~ | — | **✅ Đã chốt 2026-07-19:** bảng 6 Level (0-5) trong tài liệu này áp dụng **từ MVP2 trở đi**; bảng 5 Level (0-4) cũ chỉ còn giá trị lịch sử cho MVP1, không dùng cho thiết kế mới. Studio layout/milestone reward theo bảng 6-level mới **chưa có FSD riêng** — cần bổ sung trước khi FE thiết kế Your Space cho MVP2. |
| ~~2~~ | ~~Mâu thuẫn model Star↔Points~~ | — | **✅ Đã chốt 2026-07-19 — Ghi đè:** model `k` = 1 điểm/Star (Tier T5) trong tài liệu này **ghi đè** quyết định dual-currency tách biệt hoàn toàn Star/Points trước đó (2026-06-04). Từ nay: chi Star (nạp/donate/mua E-card) luôn cộng thêm Points theo `k`. |
| ~~3~~ | ~~Tổng điểm T1-T4 không khớp `G`~~ | — | **✅ Đã làm rõ 2026-07-19:** catalog Tier trong tài liệu chỉ mang tính ví dụ minh hoạ, không cần khớp tuyệt đối với `G` — không phải gap chặn code, không cần Content rà soát lại số liệu vì lý do này. |
| 4 | 3 giá trị còn để **"xxx"** chưa điền: (a) điểm thưởng mốc nạp 100/200/...Star, (b) số lượng thẻ C/R cần thu thập cho từng mốc, (c) điểm thưởng tương ứng | Không ảnh hưởng phần còn lại của catalog | **Không chặn tiến độ** — nếu sau này Content phát sinh số liệu cụ thể cho 2 dòng milestone này thì bổ sung sau |
| 5 | Level tự động tăng 100% khi chạm ngưỡng — quà đi kèm rank-up (Idol E-card, theo Phần 5.3 đề xuất — chưa lock) có cần Fan chủ động "claim" giống cơ chế Khung theo Rank đã chốt ở `mvp2_Feature_Breakdown.md` mục 3 không? | Ảnh hưởng UX khi lên Level — auto vs. cần thao tác nhận quà | **Tạm chưa cần xác nhận** — để sau khi phát triển xong tính năng cốt lõi (auto level-up + catalog Tier), có thể cân nhắc bổ sung cơ chế claim sau |
| ~~6~~ | ~~Khi điểm bị huỷ do Fan Letter bị report, Level có bị hạ theo không?~~ | — | **✅ Đã chốt:** giữ nguyên Level, không hạ cấp — vì Level giảm sẽ ảnh hưởng lớn tới phạm vi mua Space Item (item đã unlock theo Level cũ), gây trải nghiệm tiêu cực nếu bị thu hồi ngược |
