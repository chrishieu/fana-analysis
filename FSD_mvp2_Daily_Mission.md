# FSD — MVP2: Nhiệm vụ Hàng ngày (Daily Mission)

> **Loại tài liệu:** Functional Specification Document (FSD)
> **CR liên quan:** MVP2
> **Nguồn tham chiếu:** `Fanation - Mechanic of Ranking - Mission - Point Star Card.docx` (mục 5.1), `FSD_mvp2_Ranking_Point_System.md` (catalog Tier, engine tính điểm, Actors — **dùng chung, không lặp lại ở đây**)
> **Tech stack:** React Native (FE) · NestJS (BE) · PostgreSQL (DB)
> **Trạng thái:** Draft — đề xuất mới, Content + Tech cần align trước khi lock danh sách nhiệm vụ

---

## 1. Tổng quan

Daily Mission chỉ là **1 màn hình tổng hợp**, không phải cơ chế tính điểm mới: mỗi nhiệm vụ Daily = 1 hành động đã có sẵn trong catalog Tier ở `FSD_mvp2_Ranking_Point_System.md` mục 6.3, được chọn ra hiển thị dưới dạng checklist reset mỗi ngày. Điểm, giới hạn, engine tính toán, Actors — tất cả dùng chung với FSD đó.

Cùng cơ chế CMS/Mobile app như FSD Ranking/Point: **CMS chọn hành động nào lên danh sách Daily** (không tạo điểm mới); **Mobile app chỉ hiển thị danh sách + tiến độ**, Fan hoạt động ở đúng màn hình gốc (comment, check-in...) để đạt mission, không có thao tác "hoàn thành" riêng.

**Ngoài phạm vi:** nhiệm vụ "Mời đồng fan — referral link" — cần cơ chế referral riêng, làm sau.

---

## 2. Danh sách nhiệm vụ Daily (đề xuất ban đầu)

Lấy nguyên từ catalog Tier đã chốt (không có điểm/giới hạn mới):

| Nhiệm vụ | Tier | Điểm/lượt | Giới hạn/ngày |
|---|---|---|---|
| Check-in ngày | T1 | 5đ | 1 lần |
| Thả tim / nghe nhạc / tương tác nhẹ | T1 | 2đ/lượt | 3 lượt |
| Comment bài đăng của idol | T2 | 3đ/lượt | 5 lượt |
| Thêm sự kiện vào My Calendar | T2 | 2đ/sự kiện | 2 lượt |

> "Viết thư cho idol" không đưa vào vì giới hạn theo tháng, không phải theo ngày.

---

## 3. Yêu cầu chức năng

- **CMS:** chọn hành động nào (trong catalog Tier hiện có) đưa vào danh sách Daily Mission, đặt tên/copy hiển thị — không tạo điểm/giới hạn mới
- **Mobile app:** hiển thị danh sách nhiệm vụ hôm nay + tiến độ, đọc từ hệ thống (không tự tính); khi hoàn thành hết nhiệm vụ trong ngày → có thông báo (notification) cho Fan
- **Reset mỗi ngày:** tiến độ nhiệm vụ về 0, không ảnh hưởng điểm/Level đã tích luỹ

---

## 4. Edge Cases (riêng của Daily Mission)

| # | Tình huống | Xử lý đề xuất |
|---|---|---|
| 1 | Fan hành động ngay lúc giao thời giữa 2 ngày | Tính theo mốc reset **00:00 giờ Việt Nam (UTC+7)** — đã chốt (xem OQ-3) |
| 2 | Admin gỡ 1 hành động khỏi danh sách giữa lúc Fan đang có tiến độ dở dang | Nhiệm vụ biến mất khỏi danh sách ngay, điểm đã kiếm vẫn giữ nguyên (không hồi tố) |

> Các edge case khác (cap hành động, race condition...) đã xử lý chung ở `FSD_mvp2_Ranking_Point_System.md` mục 7, không lặp lại.

---

## 5. Open Questions

| # | Câu hỏi | Ghi chú |
|---|---|---|
| 1 | "Mời đồng fan — referral link" cần cơ chế referral riêng | Làm sau, không thuộc scope FSD này |
| ~~2~~ | ~~Hoàn thành hết Daily Mission có phần thưởng riêng ngoài điểm từng hành động không?~~ | **✅ Đã chốt: Không** — không có phần thưởng riêng, chỉ tính điểm theo từng hành động như bình thường |
| ~~3~~ | ~~Múi giờ chuẩn để reset~~ | **✅ Đã chốt: 00:00 giờ Việt Nam (UTC+7)** |
