# OPERATING DASHBOARD — VFO2O-06 · AI lập kế hoạch bảo dưỡng EV

**Loại mô hình:** B2B2C (xưởng EV trả phí gói, chủ xe dùng miễn phí qua mini-flow đặt lịch từ Zalo OA của xưởng) · **Cập nhật:** 09/10/2026 · Nguyễn Long Khánh – 2A202602649\
**NORTH STAR:** Partner activation rate — hiện tại **chưa đo (0 xưởng go-live)** — mục tiêu: xưởng pilot đầu activate ≤12 ngày; khi ≥5 xưởng go-live thì ≥60%\
*Trạng thái: chưa có pilot. Cột "Hiện" ghi giả định mô hình Day 22, không phải số đo. Phép tính [MH] ở trang 2.*

### Đèn báo sớm (Leading — nhìn hằng ngày/tuần)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 | Nguồn | Báo trước cho |
|---|---|---|---|---|
| **Partner activation** ⭐ — % xưởng go-live có ≥1 booking_confirmed từ chủ xe thật trong 30 ngày (không tính xe nhân viên/test, booking CVDV tạo hộ) | chưa đo | ≥60% / 30–60% / <30% | [TB] mốc HANDBOOK §3.3, đo 2 chu kỳ tới 05/01/2027 | Booking/xưởng → GM, CAC |
| **Time-to-first-end-user** — ngày từ ký pilot → booking thật đầu tiên | chưa đo | ≤12 / 13–18 / >18 ngày | [MH] >18 ngày không kịp đủ 100 case trong pilot 30 ngày | Activation → cổng ngày 60 |
| **AI-to-request rate** 💰 — case thử tới `booking_request_created` không cần vendor escalation | giả định 78% | ≥78% / 67,5–78% / <67,5% | [MH] <67,5% → GM/job <60% | Cost/Job → GM |

### Đèn vận hành (Operating — nhìn hằng tuần/tháng)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 | Nguồn | Báo trước cho |
|---|---|---|---|---|
| **Booking confirmed / xưởng activated / tháng** (unique booking_id + cycle_id) | giả định 179 | ≥179 / 108–178 / <108 | [MH] <108 → giá Starter > 25% giá trị xưởng nhận | Gia hạn xưởng → NRR; Cost/Job |
| **Cost/Job theo TỪNG xưởng** 💰 — direct cost xưởng ÷ booking confirmed của xưởng đó | giả định 7.998 ₫ | ≤8.228 / 8.229–9.874 / >9.874 ₫ | [MH] 3× cost và GM/job 60% trên giá 24.684 ₫/job | GM tiền gói |
| **Request bị CVDV từ chối/sửa hạng mục** (chất lượng nhìn từ end-user) | giả định 8% | ≤8% / 8–25,5% / >25,5% hoặc ≥1 lỗi "bịa" giá/hạng mục | [MH] >25,5% → GM/job <60%; lỗi "bịa" đỏ ngay (bài học Klarna) | Xưởng gỡ link → activation |

### Đèn kết quả (Lagging — nhìn hằng quý)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 | Nguồn |
|---|---|---|---|
| **GM tiền gói** (gồm traffic free + phí Zalo khi có; không gồm overhead) | giả định 69,4% | ≥60% / 53–60% / <53% | [MH] mục tiêu giá Day 22 · [BM] ICONIQ AI-native 53% 2026P, kiểm tra 09/10/2026 |
| **CAC / xưởng activated** (mẫu số là xưởng activated, không phải xưởng đã ký) | kịch bản 32 tr ₫ | <29,3 / 29,3–39,07 / >39,07 tr ₫ | [MH] ARPU×GM×12 tháng · [BM] SMB payback <12 tháng (Bessemer), kiểm tra 09/10/2026 |

💰 = đèn chi phí AI · 3 Leading · 3 Operating · 2 Lagging

### 5 luật quyết định (⏹ = luật dừng)

1. ⏹ **NẾU** xưởng go-live có 0 booking thật (hoặc activation <30% khi ≥5 xưởng) **TRONG** 30 ngày **THÌ** dừng ký xưởng mới; founder ngồi 2 ca sáng tại xưởng, sửa đúng 1 điểm chặn trên đường tới chủ xe trong 2 tuần **KHÔNG THÌ** không báo cáo "số xưởng đã ký" nếu thiếu số xưởng activated, không ký thêm xưởng để bù.
2. **NẾU** AI-to-request <67,5% **TRONG** 2 tuần liên tiếp **VÀ** mỗi tuần ≥50 case **THÌ** phân loại 30 case lỗi gần nhất, sửa nhóm lỗi lớn nhất, chạy lại golden set 50 case đạt 100% rồi mới deploy **KHÔNG THÌ** không đổi sang model đắt hơn để chữa cháy, không cắt QA 8% để hạ Cost/Job.
3. ⏹ **NẾU** request bị từ chối/sửa >25,5% **TRÊN** 50 request gần nhất của một xưởng, hoặc có ≥1 dự toán/hạng mục "bịa" **THÌ** trong 24h tắt tự tạo request, chuyển sang "CVDV duyệt bản nháp" đến khi golden set đạt 100% **KHÔNG THÌ** không gửi nhắc bảo dưỡng hàng loạt để giữ volume.
4. **NẾU** booking/xưởng activated <108 **TRONG** 2 tháng liên tiếp **THÌ** cùng chủ xưởng nhập danh sách xe đến hạn trong 2 tuần; tháng sau vẫn <108 thì chuyển xưởng sang giá theo booking 24.684 ₫/job **KHÔNG THÌ** không giảm giá Starter, không làm tuỳ biến riêng cho xưởng đó.
5. ⏹ **NẾU** CAC/xưởng activated >39,07 tr ₫ **TRÊN** 2 xưởng activated gần nhất **THÌ** dừng mở khu vực mới, chỉ bán vào cụm xưởng quanh pilot bằng giới thiệu + Pilot Report, tách phí setup một lần **KHÔNG THÌ** không tuyển AE, không chạy quảng cáo B2B.

### Cổng gác 90 ngày

| Ngày | Metric (1) | Ngưỡng qua cổng | Bằng chứng | Nếu trượt |
|---|---|---|---|---|
| 30 · 06/11/2026 (HỌC) | Số xưởng đã xác nhận bằng văn bản **đường đi tới chủ xe** (link mini-flow trong Zalo OA, ai bấm gì ở màn hình nào) kèm danh sách ≥80 xe đến hạn | ≥1 xưởng | Biên bản/LOI pilot · sơ đồ luồng · file danh sách xe ẩn danh | **FIX**: vướng quyền OA thì dùng link web dự phòng · **PIVOT** sang bảng B2B nếu xưởng chỉ nhận white-label |
| 60 · 06/12/2026 | **Cost/Job đo thật** tại xưởng pilot, trên ≥100 eligible case | ≤9.874 ₫ | Trace log + hoá đơn API theo `tenant_id` + Eval Results | **FIX** 1 lần theo R1 (thiếu case) hoặc R2 (containment); trượt lần 2 thì **PIVOT** |
| 90 · 05/01/2027 | **Booking confirmed thật** của xưởng pilot trong 30 ngày gần nhất | ≥108 | Export booking_confirmed đã dedupe + Pilot Report có nhóm đối chứng | **PIVOT** sang giá theo booking / xưởng volume lớn nếu 54–107 · **KILL** nếu <54 |

**KILL CRITERIA:** Đến **05/01/2027**, nếu Cost/Job đo thật vẫn **>9.874 ₫** sau 1 lần FIX, **hoặc** không xưởng pilot nào đạt **≥108 booking thật/tháng**, thì dừng mô hình thuê bao xưởng của VFO2O-06 và không ký thêm xưởng nào trên mô hình này.

**CHƯA ĐO ĐƯỢC:** ① **Uplift 24 booking/xưởng/tháng**: cả giá trần lẫn ngưỡng 108 dựa vào nó; thiếu nó thì cần 888 booking. Cần nhóm đối chứng (xe đến hạn không nhận nhắc) ở xưởng pilot, có số 05/01/2027. ② **Containment, tỷ lệ từ chối, Cost/Job thật**: cần trace log + `tenant_id` trên mọi call API, có số 06/12/2026. ③ **CPO founder-led VN** (đang giả định 8 tr ₫): cần chấm công founder + chi phí demo, có số sau 2 xưởng (~01/2027). ④ **Phí Zalo OA/ZNS** chưa vào Cost/Job: cần báo giá chính thức từ Zalo, trước 06/11/2026. ⑤ **Partner NRR, volume volatility**: cần 12 tháng và ≥3 tháng dữ liệu, sớm nhất 11/2027 và 02/2027.
