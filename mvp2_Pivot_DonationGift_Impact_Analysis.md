# Phân tích tác động — Bỏ Star, chuyển sang Donation Gift mua trực tiếp bằng IAP

**Ngày:** 2026-08-26
**Trạng thái:** Chờ chốt 3 điểm ở mục 4 trước khi cập nhật FSD chính thức.

---

## 0. Định nghĩa pivot (đã chốt)

- Gift **có thể không gắn sẵn Idol lúc mua**. Mỗi Idol có 1 mục "tặng quà" riêng (theo sự kiện hoặc mặc định). Fan mua Gift bằng IAP, sau đó tặng vào mục tặng quà của Idol mình chọn — tức là **mua trước, chọn Idol lúc tặng** (giống hành vi Star cũ). *(trả lời câu B)*
- Hệ quả: cần lưu trữ vào kho item và gift có thể mua tại cửa hàng

## 1. Quyết định đã chốt

| Vấn đề | Quyết định |
|---|---|
| Mốc tính FPP (điểm paid-spend theo Fan-Idol) | Tính **tại thời điểm tặng (donate)**, không phải lúc mua |
| Gift mua rồi không bao giờ tặng, nằm mãi trong kho | **Chấp nhận** — không cần cơ chế nhắc/hết hạn |
| FPP tính theo giá nào nếu giá gift đổi giữa lúc mua và lúc tặng | Tính theo **giá lúc mua** |
| Thời điểm ghi nhận Gross doanh thu (Revenue Split, chia theo idol_share_%) | ghi Gross ngay lúc mua (đúng thời điểm tiền vào), để trống `idol_id`; giao dịch chỉ xuất hiện trong báo cáo của Idol cụ thể khi Fan thực sự tặng cho idol đó. Trạng thái trung gian: "đã mua, chưa gắn Idol". |

## 2. Tác động theo từng tài liệu (cập nhật theo quyết định ở mục 1)

### 🔴 FPP & Leaderboard
FSD hiện mô tả FPP được tính khi Fan "Donate (Star)" — tức phải quy đổi Star sang VNĐ rồi mới tính điểm. Cần bỏ hẳn lớp quy đổi này: Donation Gift có giá tiền thật trực tiếp. Thay đổi quan trọng hơn là **trigger tính FPP đổi từ hành động mua sang hành động tặng** — nghĩa là phải tách rõ 2 bước riêng biệt: (1) Fan mua gift → gift vào kho, chưa tính FPP; (2) Fan tặng gift cho Idol → lúc này FPP mới cộng. Kho Donation Gift cần có trạng thái `chưa tặng` / `đã tặng` để theo dõi việc này. Gift tồn kho không tặng thì không cần cơ chế nhắc hay hết hạn.

### 🔴 IAP Revenue Split
Thay đổi lớn hơn là cách ghi nhận doanh thu: hệ thống ghi Gross doanh thu ngay tại thời điểm Fan mua gift (đúng lúc tiền vào), nhưng để trống `idol_id` vì lúc mua Fan chưa chọn tặng cho ai — giao dịch ở trạng thái trung gian "đã mua, chưa gắn Idol". Khi Fan thực sự tặng, hệ thống gắn `idol_id` vào giao dịch đó, và nó mới xuất hiện trong báo cáo doanh thu của Idol được tặng, áp dụng % chia doanh thu (idol_share_%) theo cấu hình tại **thời điểm tặng** — đề xuất dùng % lúc tặng vì đó là lúc giao dịch chia doanh thu thực sự phát sinh (nếu % này đổi giữa lúc mua và lúc tặng thì vẫn cần confirm thêm). Open Question cũ về việc có cần phân biệt "sự kiện tặng quà mặc định" hay không vẫn còn treo, chưa liên quan tới pivot này.

### 🟡 Ecard Collect/Burn, Sticker
Hai tài liệu này chỉ cần đổi thuật ngữ: câu "Star chỉ còn dùng để tặng quà Idol (Digital Gift)" đổi thành "Donation Gift". Logic mua/gộp/burn giữ nguyên, vì hai tính năng này đã pivot sang tiền thật hoàn toàn từ 17/08 và không còn liên quan gì đến Star nữa.

### 🟡 Ranking & Point System
Trong bảng liệt kê các loại tiền tệ có liên quan đến EXP, dòng "Star" đổi thành "Donation Gift", vẫn giữ nguyên kết luận là Donation Gift không cộng EXP. Cơ chế EXP nói chung không thay đổi.

### 🟢 Kho item
Gift mua mà chưa tặng có thể đưa vào kho item

### 🟡 Tab "Cửa hàng"
Cửa hàng hiện tại chỉ bán vật phẩm sở hữu vĩnh viễn (E-card/Space Item/Sticker/Avatar Frame). Donation Gift khác bản chất — mua để tiêu đi (tặng) chứ không phải trang trí lâu dài — nên luồng UX của Cửa hàng cần thêm một nhánh mới: sau khi mua, Fan chọn tặng ngay hoặc cất vào kho, rồi sau đó quay lại kho để chọn tặng cho Idol.

## 3. Việc KHÔNG đổi

- Kênh thanh toán (IAP Apple/Google + MoMo), % phí store, bảng phí MoMo — giữ nguyên.
- Cơ chế EXP/Level, cơ chế mua E-card/Sticker/Space Item/Avatar Frame bằng tiền thật tại Cửa hàng — giữ nguyên (đã pivot từ 17/08, không liên quan Star).
- FPP vẫn là hệ điểm nuôi bằng chi tiêu — chỉ đổi input, không đổi mục đích/công thức luỹ tiến.
- `FSD_mvp2_Digital_Merch_QR_Gift.md`, `FSD_mvp2_Comment.md` — chỉ có câu "không trừ Star" mang tính khẳng định free action, xoá Star không ảnh hưởng logic, có thể để nguyên hoặc chỉnh câu chữ cho gọn.