# FSD — MVP2: Nhiệm vụ Cột mốc (Milestone)

## 1. Tổng quan

Milestone Mission **chính là Tier T4** trong catalog Tier ở `FSD_mvp2_Ranking_Point_System.md` mục 6.3 — không tạo hành động mới, không tạo điểm mới. Khác biệt duy nhất so với catalog gốc: hiển thị dưới dạng **"thành tựu" (achievement)** thay vì nhiệm vụ ngày/tuần, nhất quán với cách Daily Mission (mục 5.1) và Quest Chain (mục 5.2) đều chỉ là 1 "view" khác nhau trên cùng catalog Tier, không phải cơ chế tính điểm riêng.

**Khác biệt UX so với 2 loại nhiệm vụ kia:**
- **Daily Mission** — reset mỗi ngày, hiển thị dạng checklist.
- **Quest Chain** — 3 bậc tăng dần trong tháng, hiển thị dạng progress bar nhiều nấc.
- **Milestone** — cột mốc rời rạc, đạt 1 lần theo đúng chu kỳ của hành động gốc (7/14/30 ngày cho streak; hàng tháng cho các mốc còn lại), hiển thị dạng **badge/thành tựu đã mở khoá**, không phải thanh tiến trình liên tục.

---

## 2. Danh sách nhiệm vụ Milestone (theo nguồn mục 5.3)

| Nhiệm vụ | Điểm | Giới hạn | Quà kèm theo |
|---|---|---|---|
| Giữ streak check-in 7 ngày | 20đ | 1 lần/mốc/tháng | — |
| Giữ streak check-in 14 ngày | 25đ | 1 lần/mốc/tháng | + 1 item trang trí (đã có ở Tier T4) |
| Giữ streak check-in 30 ngày | 30đ | 1 lần/mốc/tháng | + 1 item trang trí (đã có ở Tier T4) |
| Thêm đủ 10 sự kiện vào Calendar/tháng | 20đ | 1 lần/tháng | + 1 thẻ C, **random idol Fan đang follow** |
| Tương tác đủ 20 lượt/tháng | 25đ | 1 lần/tháng | + 1 thẻ C, **random idol Fan đang follow** |
| Lên hạng (Level up) | 0đ | — | **1 thẻ R** (theo nguồn mục 5.3, khớp đúng idol có Level vừa tăng — Level tính riêng theo từng cặp Fan-Idol) |

Riêng 2 mốc Calendar và Tương tác không gắn với 1 idol cụ thể (hành động có thể đến từ nhiều idol khác nhau Fan đang theo dõi), nên thẻ C thưởng được chọn **ngẫu nhiên trong số các idol Fan đang follow** — khác với "Lên hạng" vốn đã có sẵn ngữ cảnh 1 idol cụ thể.

Ngoài 6 dòng trên, Tier T4 còn 2 nhóm mốc khác (Nạp mốc Star 100/200/xxx, Thu thập đủ thẻ C/R/xxx) cũng thuộc dạng Milestone/achievement, nhưng số liệu "xxx" trong catalog gốc chưa điền.

---

## 3. Yêu cầu chức năng

- **Mobile app:** màn "Thành tựu" (vị trí/thiết kế cụ thể do Design quyết định) hiển thị danh sách Milestone kèm trạng thái đã đạt/chưa đạt — badge dạng mở khoá, không phải progress bar liên tục như Daily Mission
- **CMS:** không cấu hình logic tính điểm mới — dùng chung catalog Tier T4 (`FSD_mvp2_Ranking_Point_System.md` mục 6.3). Chỉ bổ sung field "quà kèm theo" (item trang trí hoặc thẻ E-card) cho từng dòng Milestone, để trống nếu Admin chưa gán quà
- Engine tính điểm/giới hạn/reset dùng chung 100% với `FSD_mvp2_Ranking_Point_System.md` — Milestone không có logic tính điểm riêng
- Quà kèm theo (item trang trí, thẻ E-card): cấp cùng lúc với điểm ngay khi Fan đạt mốc — **khác với quà rank-up ở `FSD_mvp2_Ecard_Collect_Burn.md` (đã chốt Fan phải chủ động claim)**
