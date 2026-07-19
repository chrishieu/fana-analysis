# FSD — MVP2: Nhiệm vụ Chuỗi (Quest Chain — điểm tăng dần theo độ khó, không vượt cap tháng)

> **Loại tài liệu:** Functional Specification Document (FSD)
> **CR liên quan:** MVP2
> **Nguồn tham chiếu:** `Fanation - Mechanic of Ranking - Mission - Point Star Card.docx` (mục 5.2), `FSD_mvp2_Ranking_Point_System.md` (catalog Tier, engine tính điểm, Actors — **dùng chung, không lặp lại ở đây**)
> **Tech stack:** React Native (FE) · NestJS (BE) · PostgreSQL (DB)
> **Trạng thái:** Draft — quy tắc chain ở mục 2 là **đề xuất mới của PD/BA** (tài liệu nguồn chưa có công thức rõ ràng), cần Content + PO xác nhận trước khi lock

---

## 1. Tổng quan

Quest Chain là **1 dạng nhiệm vụ chuỗi**: mỗi chain gắn với 1 hành động đã có sẵn trong catalog Tier (`FSD_mvp2_Ranking_Point_System.md` mục 6.3), chia thành **3 bậc tăng dần độ khó**. Bậc sau đòi hỏi nhiều lượt hành động hơn (khó hơn) → **điểm luỹ kế của bậc đó cũng cao hơn bậc trước**, nhưng bậc cuối cùng (khó nhất) luôn dừng đúng ở cap tháng đã có sẵn của hành động — **không vượt cap, không tạo điểm mới ngoài catalog Tier**.

Hệ thống cho phép tối đa **5 chain hoạt động cùng lúc**. Cơ chế CMS/Mobile app giống các FSD Mission khác: **CMS chọn hành động nào được làm chain + đặt mốc 3 bậc**; **Mobile app chỉ hiển thị tiến độ**, Fan hoạt động bình thường ở nơi gốc (comment, viết thư...) để lên bậc — không có thao tác riêng.

---

## 2. Quy tắc chain (đề xuất — giữ đơn giản)

Để dev dễ code và Fan dễ hiểu, đề xuất áp dụng **1 công thức duy nhất cho mọi chain**, không cần tính riêng từng trường hợp:

1. **Điều kiện 1 hành động được làm chain:** thuộc Tier T1-T3, có cap tháng ≥ 3, và **chưa có cơ chế tăng bậc riêng nào khác** (vd Check-in không làm chain vì đã có Streak 7/14/30 ngày ở Milestone).
2. **Mỗi chain luôn có đúng 3 bậc, tăng dần, bậc cuối = đúng cap tháng đã có** của hành động đó (không tạo cap mới). 2 bậc đầu là mốc trung gian nhỏ hơn — Content làm tròn số cho dễ nhớ, miễn tăng dần đều.
3. **Điểm mỗi bậc = luỹ kế số lượt đã đạt × đơn giá điểm gốc của hành động** (đơn giá không đổi) — vì bậc sau yêu cầu nhiều lượt hơn, **điểm của bậc sau tự nhiên cao hơn bậc trước**, và điểm ở bậc cuối luôn bằng đúng cap tháng hiện có, không phát sinh điểm ngoài catalog Tier.
4. **Tối đa 5 chain hoạt động cùng lúc** — nếu có đủ 5 hành động hợp lệ theo điều kiện 1, dùng hết cả 5; nếu chưa đủ, Admin để trống hoặc luân phiên hành động nổi bật, không tạo hành động giả cho đủ số.

## 3. Ví dụ minh hoạ

Áp dụng quy tắc trên vào catalog Tier hiện có, ra đúng 5 chain hợp lệ (đủ dùng cả 5, không cần luân phiên):

| Chain | Tier gốc | Bậc 1 | Bậc 2 | Bậc 3 (= cap tháng hiện có) |
|---|---|---|---|---|
| Viết thư cho idol | T3 (20đ/thư) | 1 thư (20đ) | 3 thư (60đ) | 4 thư (80đ) |
| Thêm sự kiện Calendar | T2 (2đ/sự kiện) | 20 sự kiện (40đ) | 40 sự kiện (80đ) | 60 sự kiện (120đ) |
| Comment bài đăng | T2 (3đ/lượt) | 50 lượt (150đ) | 100 lượt (300đ) | 150 lượt (450đ) |
| Thả tim / tương tác nhẹ | T1 (2đ/lượt) | 30 lượt (60đ) | 60 lượt (120đ) | 90 lượt (180đ) |
| Donation (tặng Gift) | T3 (5đ/lượt) | 10 lượt (50đ) | 20 lượt (100đ) | 30 lượt (150đ) |

> Điểm trong ngoặc là điểm luỹ kế tại bậc đó — tăng dần qua từng bậc, và bậc 3 luôn khớp đúng cột "Tổng tối đa/tháng" đã chốt ở catalog Tier. Số liệu bậc 1/2 chỉ là ví dụ minh hoạ, Content có thể làm tròn lại.

---

## 4. Yêu cầu chức năng

- **CMS:** chọn hành động nào (trong catalog Tier T1-T3 hợp lệ) được làm chain và đặt mốc 3 bậc — không tạo điểm/cap mới ngoài quy tắc mục 2
- **Mobile app:** hiển thị các chain đang hoạt động + tiến độ từng bậc (màn hình/vị trí hiển thị cụ thể do Design quyết định), Fan hoạt động ở đúng màn hình gốc của hành động — khi lên bậc mới có thông báo (notification), nhất quán với Daily Mission
- **Reset hàng tháng:** mọi chain reset về Bậc 1 mỗi đầu tháng — vì bậc 3 chính là cap tháng của hành động gốc, mà cap tháng đó vốn đã reset hàng tháng theo cơ chế ở FSD Ranking/Point (mục 7, Edge case #3). Áp dụng bất kể Fan đã hoàn thành Bậc 3 hay chưa — không phải chỉ riêng trường hợp hoàn thành sớm

---

## 5. Edge Cases

| # | Tình huống | Xử lý đề xuất |
|---|---|---|
| 1 | Admin đổi hành động của 1 chain giữa lúc Fan đang có tiến độ dở dang | Tiến độ chain cũ biến mất, điểm đã kiếm được vẫn giữ nguyên (không hồi tố) — nhất quán với FSD Ranking/Point |
| 2 | Fan đạt Bậc 3 (chạm cap tháng) trước khi hết tháng | Chain hiển thị "Hoàn thành", không tạo thêm bậc mới; reset về Bậc 1 khi sang tháng mới |
| 3 | Chưa đủ hành động hợp lệ để đạt 5 chain hoạt động cùng lúc | Admin để trống hoặc luân phiên hành động nổi bật (xem quy tắc 4, mục 2) |
| 4 | Fan theo dõi từ 2 idol trở lên | Riêng chain "Viết thư cho idol" tính **gộp chung** across tất cả idol Fan đang theo dõi (không tính riêng theo từng idol) — để tránh Fan spam viết thư nhiều idol khác nhau farm điểm, vì hành động này không cần nội dung thật từ idol (khác Comment/Calendar/Donation vốn phụ thuộc idol có bài đăng/sự kiện) |

---

## 6. Open Questions

| # | Câu hỏi | Ghi chú |
|---|---|---|
| 1 | Lên Bậc mới có phần thưởng riêng ngoài điểm không (giống Daily Mission đã chốt là không)? | Đề xuất: giữ nhất quán — không có thưởng riêng, chỉ tính điểm theo hành động |
