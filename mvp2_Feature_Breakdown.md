# MVP2 — Feature Breakdown cho Developer

> Nguồn: `change-request/mvp2_1.docx` (bản update của `mvp2.docx`). Update thật sự nằm ở **comment lề Word**, không phải nội dung thân — client (Quan Vo, Thuy Tien, My Ngo) đã trả lời phần lớn câu hỏi PD/BA đặt ra ở vòng review trước. File này tổng hợp lại theo hướng: đủ thông tin để dev bắt tay làm ngay, mục nào còn thiếu được đánh dấu rõ **⚠️ Cần xác nhận thêm**.
>
> Ngày tổng hợp: 2026-07-14

---

## 1. Bình luận (Comment)

**Đã chốt:** Giới hạn 1000 ký tự (sửa từ "không giới hạn"). Reaction gồm: Thả tim, Thả pháo hoa 🎉, + 2 loại khác (Thuy Tien trả lời bị lỗi chính tả — **⚠️ cần confirm lại chính xác tên 4 loại reaction với design**). User được xoá comment của mình, không được sửa. Idol/Admin được xoá/ẩn comment trên bài của mình. Comment Idol luôn được pin; Admin/Idol có quyền pin thêm; sort mặc định = mới nhất + tương tác cao lên trước. Comment bị auto-filter chặn → ẩn hẳn khỏi list công khai, vẫn lưu log để audit.

**⚠️ Còn treo:** Cho phép user tự report comment vi phạm hay không — câu hỏi gốc từ CR, chưa thấy trả lời trong đợt update này.

### FE
- Ô nhập bình luận dưới mỗi bài viết (Idol's Hub → Bài viết), đếm ký tự, chặn gửi khi vượt 1000
- Danh sách bình luận: avatar (kèm khung nếu có), tên, nội dung, thời gian, đếm số reaction theo từng loại
- Comment Idol hiển thị ghim (pin) đầu danh sách; nút Pin/Unpin chỉ hiện cho role Idol/Admin
- 4 nút/icon reaction (long-press hoặc picker) — dùng placeholder đến khi có xác nhận icon chính thức
- Menu xoá comment (chỉ hiện cho chủ comment hoặc Idol/Admin trên bài của họ) — không có nút sửa
- Toast khi comment bị chặn: "Bình luận chứa nội dung không phù hợp"
- Validate chặn dán link ngay trên client (kèm BE validate lại)
- Hiển thị tiến độ mission "Bình luận" trong daily mission tracker

### BE
- API tạo comment: validate max 1000 ký tự, chặn nội dung chứa link
- Auto-filter engine theo 5 nhóm từ khoá (spam link / cờ bạc / tục tĩu / kỳ thị / spam lặp — rate limit vd >6 comment/phút)
- CMS CRUD danh sách từ khoá blacklist theo nhóm
- Soft-hide comment bị chặn (ẩn khỏi public list, giữ log audit riêng)
- API xoá comment (owner hoặc Idol/Admin), API pin/unpin
- Bảng `CommentReaction` (comment_id, user_id, reaction_type) — multi-type
- Thuật toán sort: pin trước → mới nhất + engagement score (nếu cần trọng số phức tạp, handoff RnD định nghĩa công thức)
- Emit event hoàn thành mission "Bình luận" khi comment hợp lệ (không bị filter chặn) được tạo
- Rate limit theo user ở tầng BE (chống spam ngoài auto-filter)

---

## 2. Sticker (Gửi thư)

**Đã chốt:** Mua theo cả bộ (không mua lẻ). Không giới hạn số sticker chèn trong 1 lần soạn thư. Gửi thư có Sticker là bổ sung cho tính năng Gửi thư hiện có (không mâu thuẫn với Fan Letter).

**⚠️ Còn treo:** Giá Star cho bộ trả phí + nội dung "scheme" mission để sở hữu — Hiếu note "nhờ team Acc gửi sau", vẫn chưa có số liệu.

### FE
- Bottom sheet chọn sticker trong màn Gửi thư — hiển thị các bộ đã sở hữu
- Tag "new" hoặc popup khi có bộ sticker mới phát hành
- Màn bộ sưu tập sticker: bộ miễn phí (mặc định có) vs bộ trả phí (mua cả bộ bằng Star)
- Chèn sticker trực tiếp vào nội dung thư khi soạn (không giới hạn số lượng)

### BE
- API danh sách bộ sticker (free/paid) + trạng thái sở hữu theo user
- API mua cả bộ sticker bằng Star — **giá cụ thể chưa có, tạm để field cấu hình CMS rỗng chờ số liệu**
- CMS: CRUD bộ sticker, gán free/paid, gán mốc thời gian để tính "new"
- Lưu reference sticker đã chèn trong nội dung thư

---

## 3. Khung avatar

**Đã chốt:** Khung sự kiện hết event vẫn giữ vĩnh viễn trong kho (không thu hồi). Khung theo Rank cần user chủ động **claim** (không tự động cấp). Khung sự kiện: earn qua nhiệm vụ sự kiện (vd "thêm lịch ra mắt album vào calendar") HOẶC mua bằng **Star** (không phải tiền thật trực tiếp — giải toả lo ngại P2W status-symbol đã nêu ở review trước).

**⚠️ Còn treo:** Giới hạn số lượng khung sở hữu cùng lúc — Thuy Tien hỏi ngược "nếu không giới hạn thì có vấn đề gì không anh", chưa chốt. **Cần PD/Hiếu trả lời câu này trước khi lock DB schema** (unlimited stack vs capped inventory).

### FE
- Màn "Bộ sưu tập Khung" trong Cá nhân — lưới khung sở hữu + khung chưa mở khoá (mờ/khoá)
- Click khung Level ở trang cá nhân → mở màn bộ sưu tập, chọn khung active
- Nút "Claim" riêng cho khung Rank mới đạt (không tự động gắn)
- Khung hiển thị quanh avatar ở mọi nơi (comment, profile, leaderboard)
- Đổi khung active không giới hạn số lần

### BE
- API claim khung theo Rank (check điều kiện đã đạt rank)
- API mua khung sự kiện bằng Star
- Event Mission gắn khung sự kiện (phối hợp RnD định nghĩa mission cụ thể theo từng campaign)
- Lưu sở hữu khung vĩnh viễn, không thu hồi sau khi hết event
- API set khung active (1 khung active toàn hệ thống tại 1 thời điểm)
- **Chờ xác nhận:** có cap số lượng khung sở hữu không — ảnh hưởng thiết kế bảng lưu trữ + UI danh sách

---

## 4. Khung thư (Gửi thư)

**Đã chốt:** Thư có khung được chọn Public hoặc Private khi gửi. File tải về: định dạng **JPG**, không cần thêm watermark riêng vì đã có sẵn logo Fanation in trên khung. "Thẻ" dùng để mua khung thư = **E-card** (xác nhận, liên kết trực tiếp với mục 6).

### FE
- Màn soạn thư: mục chọn Khung Frame (để riêng, chờ chốt có gộp vào "Nền" hay không)
- Toggle Public/Private khi gửi
- Nút tải thư về dạng ảnh JPG (khung đã có sẵn logo Fanation)
- Màn bộ sưu tập Khung thư: sở hữu qua mission hoặc đổi bằng E-card/mua Star

### BE
- API render thư → ảnh JPG (ghép khung + hình cá nhân + logo Fanation có sẵn trên asset khung)
- API set public/private cho thư (public → hiển thị trong Idol Hub feed)
- API đổi Khung thư bằng E-card (burn E-card theo tỷ lệ) hoặc mua bằng Star — **tỷ lệ đổi E-card vẫn phụ thuộc file Poin Star Card (xem mục 6), chưa có**
- Storage/CDN cho ảnh thư export

---

## 5. Stream nhạc trong Profile Idol

**Đã chốt (thông tin vận hành quan trọng):** Đây **không phải** tính năng phát nhạc trong app — chỗ này chỉ **gắn link** (mỗi bài nhạc = 1 row), user tap vào row → mở **YouTube hoặc Spotify** lên nghe (2 nền tảng được xác nhận, không mở rộng thêm nền tảng khác ở bản này). Team Quản lý Nghệ sĩ (QLNS) tự thêm link qua luồng riêng dành cho nghệ sĩ — **giới hạn tối đa 7 link hiển thị**. Link chết do team QLNS chịu trách nhiệm phát hiện/cập nhật. Mission "bấm link nhạc": số lần bấm cần thiết để hoàn thành **linh hoạt theo từng đợt mission** (không cố định 1 lần/ngày).

**Mới xác nhận (chat bổ sung):** Luồng Idol (Idol Management Account / Idol Dashboard) **phải có 1 màn hình riêng** để team QLNS tự cấu hình (thêm/sửa/xoá) các link nhạc này — không phải chỉ quản lý qua Admin CMS chung, mà là 1 view thuộc chính app/dashboard phía Idol.

**Lưu ý PD:** rủi ro thưởng mission cho hành vi rời app (ngược DAU) — team dường như chấp nhận qua thiết kế "mission cấu hình linh hoạt". Không chặn dev, nhưng nên note lại khi RnD thiết kế mission cụ thể.

### FE
- Box "Nghe nhạc mới nhất" dưới "Mạng xã hội" trong Profile Idol (phía Fan)
- Danh sách tối đa 7 link (row), icon theo platform — chỉ YouTube và Spotify
- Tap vào row → mở app YouTube/Spotify tương ứng, fallback mở trình duyệt nếu chưa cài app
- Cập nhật tiến độ mission theo số link đã bấm (so với ngưỡng mission hiện tại)
- **Mới:** Màn cấu hình link nhạc trong luồng Idol Management Account/Idol Dashboard — cho phép QLNS tự thêm/sửa/xoá link YouTube/Spotify, giới hạn tối đa 7 link

### BE
- API cấu hình link nhạc theo idol (thêm/sửa/xoá), giới hạn cứng 7 link, chỉ nhận URL YouTube/Spotify — dùng bởi màn cấu hình phía Idol Dashboard (không phải Admin CMS chung)
- API lấy danh sách link theo idol (phía Fan xem)
- Track outbound click event, mission config cho phép đặt "số link cần bấm" linh hoạt (phối hợp Mission Config trong CMS)

---

## 6. Sổ E-card (Collect & Burn)

**Đã chốt — giải quyết được ẩn số lớn nhất của cả CR:** "**Studio**" = **My Space → Bộ Sưu Tập** (My Ngo xác nhận trực tiếp — áp dụng luôn cho cách hiểu "Studio" ở mục 7). Thẻ trùng: cộng dồn số lượng theo loại thẻ, không giới hạn kho. E-card C được xác nhận là **1 loại thưởng mission mới**, chạy song song Points/Star hiện có trong Daily Mission System (BRD F3.2) — RnD cần cập nhật lại thiết kế mission reward pool.

**⚠️ Còn treo (chặn phần lớn effort của mục này):**
- Nút "Đổi vật phẩm" trên chi tiết thẻ — đổi ra gì, tỷ lệ nào — chưa có câu trả lời.
- Quà rank-up cụ thể mỗi lần nâng hạng — chưa có câu trả lời.
- **File "Poin Star Card" vẫn chưa được gửi** — toàn bộ công thức gộp nâng hạng + cơ chế burn chi tiết vẫn thiếu, kể cả sau bản update này.

### FE
- Màn "Bộ Sưu Tập" trong My Space → tab "Thẻ sở hữu": lưới thẻ theo idol, hiển thị số lượng mỗi thẻ
- Chi tiết thẻ: xem full theo định dạng hạng (ảnh tĩnh C/R, motion SR, live photo 2s SSR, video 5-10s có âm thanh UR) + 3 nút Gộp / Đổi vật phẩm / Đặt làm nhạc nền (UR only)
- Popup nhận thẻ mới: "Bạn nhận được thẻ [hạng] – [idol]"
- Màn mua thẻ C/R bằng Star — chọn đúng idol + card cụ thể (fixed price, không random)

### BE
- Schema `user_card` (user_id, card_id, idol_id, rarity, quantity) — cộng dồn, không cap kho
- API mua thẻ trực tiếp: C = 10 Star, R = 100 Star (SR/SSR/UR không bán trực tiếp)
- Cập nhật Mission Service: hỗ trợ reward type = E-card (mission thường → C; mission sự kiện/campaign → SR/SSR/UR)
- **Chưa code được, chờ file Poin Star Card:** API gộp nâng hạng, API/logic quà rank-up, cơ chế burn chi tiết
- **Chưa code được, chờ trả lời:** logic "Đổi vật phẩm" (tỷ lệ + danh sách vật phẩm đổi được)

---

## 7. Quét QR merch offline → online

**Đã chốt:** Áp dụng cách hiểu "Studio" = My Space → Bộ Sưu Tập (đồng bộ mục 6). Kế thừa toàn bộ cơ chế QR unique/redeem 1 lần đã có trong BRD F4.1.

**⚠️ Còn treo (chưa thấy trả lời trong đợt update này):**
- "Nhân vật" (character có thể mặc skin) — có phải 1 sub-system mới hoàn toàn (character customization) hay dùng lại avatar sẵn có? Ảnh hưởng lớn effort, vẫn chưa xác nhận.
- Chi phí vận hành lặp lại: "Khi idol bán trang phục mới → cần brief product để vẽ trang phục tương ứng" — chưa xác nhận ai/quy trình nào đảm nhận.
- Câu hỏi gốc từ CR: merch nhỏ (keyring, badge) có gắn QR không — chưa có trả lời.

### FE
- Nút "Quét vật phẩm" (icon QR) trong My Space
- Màn quét: camera QR + nhập mã tay
- Popup trạng thái: xử lý / thành công / thất bại / mã đã dùng
- Popup thành công: "Bạn nhận được [tên vật phẩm]" → điều hướng Inventory hoặc Sổ E-card
- Inventory: nút "Place in studio" (= My Space/Bộ Sưu Tập) hoặc gán skin lên nhân vật
- Màn nhân vật: hiển thị/chọn skin đã mở khoá (**scope phụ thuộc câu hỏi "nhân vật" ở trên — ưu tiên xác nhận trước khi FE bắt tay thiết kế màn này**)

### BE
- API redeem QR: validate unique, 1 lần/tài khoản, gắn vĩnh viễn
- Mapping bảng merch type → reward (Photocard/Album/Lightstick/Trang phục/Merch nhỏ/Limited edition/Vé concert — theo bảng chi tiết trong CR)
- CMS: cấu hình QR code batch theo từng đợt merch + gán reward tương ứng
- API gán reward vào Inventory hoặc Sổ E-card tuỳ loại
- **Chờ xác nhận trước khi estimate đầy đủ:** phạm vi hệ thống "nhân vật"/skin

---

## 8. Âm thanh nền My Space

**Đã chốt:** Nhạc bản quyền idol được xác nhận **cho phép dùng làm nhạc nền** — vấn đề pháp lý đã giải quyết, không còn là rủi ro cần theo dõi. Bài hát hết sẽ tự loop lại; dừng khi thoát My Space.

**Đã chốt — flow chi tiết (cập nhật mới):**
- Trong My Space/Your Space có nút mở **Kho "Track"** — cùng cấu trúc với các kho vật phẩm khác (Loa, Stage, Skin...).
- Track gồm 2 loại nội dung: **voice** (thu âm của idol) hoặc **đoạn nhạc ngắn** (<30s, dạng giống sound trên IG Stories).
- Cách sở hữu track — chỉ có 2 đường, **không có track miễn phí mặc định** (giải quyết luôn mâu thuẫn free/paid đã nêu ở bản trước):
  1. **Mua bằng E-card idol** (burn E-card để đổi lấy track)
  2. **Tích hợp sẵn theo thẻ SSR/UR**: thẻ SSR/UR vốn đã là định dạng video/motion có âm thanh → sở hữu thẻ SSR/UR là tự động sở hữu luôn track gắn liền với thẻ đó, không cần mua thêm

**⚠️ Còn treo:**
- Số lượng track tối đa được lưu trữ — chưa có quyết định chính thức từ stakeholder.
  - **Đề xuất PD:** track gắn với SSR/UR vốn là vật phẩm hiếm, tỷ lệ ra thẻ thấp, và nội dung phát hành còn giới hạn trong 1-2 tháng đầu → nên đặt cap **mềm ở mức 10-15 track**, cấu hình qua CMS (không hardcode) để dễ tăng dần khi thư viện nội dung mở rộng, không cần deploy lại code.
- Tỷ lệ/loại E-card cần burn để "mua" 1 track thường — chưa có số liệu, cùng nhóm vướng mắc với file "Poin Star Card" ở mục 6.

### FE
- Nút mở Kho "Track" trong My Space (cùng nhóm với Loa/Stage/Skin)
- Màn Kho Track: danh sách theo idol, phân loại voice/nhạc ngắn, hiển thị đã sở hữu / cần mua bằng E-card / đi kèm thẻ SSR-UR nào
- Auto-play track mặc định khi vào My Space; voice chỉ phát 1 lần khi vừa vào phòng
- Track nhạc nền tự loop khi hết bài; dừng phát khi thoát My Space

### BE
- API mua track bằng cách burn E-card idol (loại/tỷ lệ E-card cần burn — chờ số liệu)
- Track gắn liền SSR/UR: tự động cấp quyền sở hữu khi user sở hữu thẻ tương ứng (trigger lúc nhận thẻ, không qua API mua riêng)
- Lưu sở hữu track theo từng idol riêng biệt (không gộp chung giữa các My Space)
- Storage/CDN cho audio (voice clips + đoạn nhạc ngắn) theo idol — áp cap tối đa khi có quyết định chính thức (xem đề xuất PD ở trên)
- **Chờ xác nhận:** số lượng track tối đa; tỷ lệ/loại E-card cần burn để mua track

---

## 9. Giảm 5% khi thanh toán MoMo

**Đã chốt (thông tin đầu tiên cho mục này):** Cơ chế = **trừ trực tiếp 5% vào số tiền nạp**, KHÔNG PHẢI tặng thêm Star bonus. Đây là điểm quan trọng nhất mới biết — trước đó mục này hoàn toàn không có nội dung.

**⚠️ Còn treo (vẫn là mục thiếu thông tin nhất CR, dù đã có 1 chi tiết mới):**
- Phạm vi áp dụng: chỉ nạp Star, hay mọi giao dịch qua MoMo?
- Áp dụng cho toàn bộ phương thức con của MoMo (Ví MoMo/Trả sau/ATM/thẻ quốc tế/QR) hay chỉ 1 số kênh?
- **Câu hỏi tài chính bắt buộc (CRITICAL):** ai tài trợ khoản giảm 5% — MoMo (chương trình co-marketing) hay Fanation tự bù? Fanation hiện đã trả phí giao dịch cho MoMo (1,5–3%, xem `context.md` mục 0.1) — nếu tự bù thêm 5%, biên lợi nhuận nạp Star qua MoMo âm khoảng 6,5–8%. Đây không phải quyết định BA/dev có thể tự suy đoán, cần Finance/stakeholder xác nhận trước khi code.

### FE
- Hiển thị giá đã trừ 5% ngay tại màn thanh toán khi chọn MoMo (trước khi user confirm)
- Badge "Giảm 5% khi thanh toán qua MoMo" tại điểm chọn phương thức thanh toán

### BE
- Logic: giá net = giá gốc × 0.95 khi `payment_provider = MOMO` (trừ thẳng trên số tiền thanh toán, không phải cộng Star)
- **Chưa code được, chờ xác nhận:** phạm vi áp dụng (loại giao dịch + kênh MoMo con) và nguồn tài trợ discount

---

## Tổng kết — việc cần làm trước khi giao dev full-scope

| # | Việc cần làm | Ai xử lý |
|---|---|---|
| 1 | Confirm chính xác tên 4 loại reaction (bị lỗi chính tả trong câu trả lời) | Design/Thuy Tien |
| 1 | Chốt có cho phép report comment hay không | Product/stakeholder |
| 2 | Giá Star cho bộ sticker trả phí + chi tiết mission sở hữu | Team Acc (đã hẹn gửi sau) |
| 3 | Có giới hạn số lượng khung avatar sở hữu không | PD (Hiếu) trả lời câu hỏi ngược của Thuy Tien |
| 4 | Tỷ lệ đổi E-card → Khung thư | Phụ thuộc file Poin Star Card (mục 6) |
| 5 | ~~Follow up với anh Đạt để chốt chính thức luồng CMS team QLNS~~ | ✅ Resolved — chỉ 2 platform YouTube/Spotify, cấu hình qua view riêng trong Idol Dashboard |
| 6 | **File Poin Star Card** (gộp nâng hạng, quà rank-up, burn) | Vẫn thiếu — chặn phần lớn effort mục 6 |
| 6 | Nút "Đổi vật phẩm" đổi gì/tỷ lệ | Chưa trả lời |
| 7 | Scope hệ thống "Nhân vật"/skin — feature mới hay tái dùng avatar | Chưa trả lời — ảnh hưởng effort lớn |
| 7 | Merch nhỏ (keyring, badge) có gắn QR không | Đội merchandise |
| 8 | Số lượng track tối đa/idol + làm rõ mâu thuẫn free/paid | Chưa trả lời |
| 9 | Phạm vi áp dụng + ai tài trợ 5% discount | **CRITICAL — Finance/stakeholder** |

Các mục 1, 2 (trừ giá), 3 (trừ cap kho), 4, 5 (trừ follow-up anh Đạt) đã đủ thông tin để BA viết AC chi tiết và dev bắt tay ngay. Mục 6, 7, 9 nên giữ lại chờ thêm thông tin trước khi estimate effort chính thức.
