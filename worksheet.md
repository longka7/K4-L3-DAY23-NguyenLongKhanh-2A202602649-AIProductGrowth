# Worksheet — VFO2O-06 · AI lập kế hoạch bảo dưỡng EV

Họ tên: **Nguyễn Long Khánh** · MSSV: **2A202602649** · Ngày làm: **09/10/2026**

> **Nguồn số liệu đầu vào:** mô hình tài chính Day 22 của sản phẩm VFO2O-06 (`Day22_model.xlsx`, tab `1_Cost_Job`, `2_Pricing`, `4_Channel_Fit`, `5_90Day_Plan`, `6_Benchmarks`) và one-pager Day 22 (ngày 08/10/2026). Ô Excel được ghi kèm khi trích, ví dụ `1_Cost_Job!B64`.
> **Trạng thái:** sản phẩm **chưa có pilot và chưa có eval thật**. Mọi số "hiện tại" là giả định của mô hình, không phải số đo được. Đơn vị VND.

---

## Trạm 1 — Loại mô hình

**Ba câu hỏi, trả lời theo thực tế hôm nay:**

| Câu hỏi | Trả lời |
|---|---|
| Ai trả tiền? | **Xưởng dịch vụ EV / mạng lưới xưởng**, trả phí gói tháng (Starter 4.690.000 ₫, gồm 190 booking) cộng phí vượt quota, từ ngân sách vận hành hậu mãi → doanh nghiệp |
| Ai dùng sản phẩm? | **Chủ xe EV**, tức khách của xưởng, dùng miễn phí: nhận nhắc bảo dưỡng đến hạn, xem hạng mục + dự toán, chọn ca và gửi yêu cầu đặt lịch. CVDV của xưởng chỉ bấm xác nhận |
| Có chạm được người dùng cuối không? | **Có.** Chủ xe thao tác trực tiếp trên mini-flow đặt lịch của mình, mở từ link trong Zalo OA/inbox của xưởng. Mình giữ dữ liệu xe (ODO, chu kỳ bảo dưỡng) và sự kiện `booking_request_created` / `booking_confirmed` theo `booking_id` + `maintenance_cycle_id` |

**Câu chốt loại:** Chúng tôi là **B2B2C** vì tiền đến từ **xưởng dịch vụ EV** (phí gói + overage), người dùng thật là **chủ xe, khách của xưởng**, và chúng tôi chạm được họ qua **mini-flow đặt lịch bảo dưỡng mở từ Zalo OA của xưởng**. Ở đó chủ xe tự xem hạng mục, dự toán và chọn ca, còn chúng tôi có dữ liệu xe và log booking của họ.

> **Điều kiện để giữ nhãn B2B2C:** nếu xưởng chỉ cho tích hợp white-label (CVDV tự nhập hộ, chủ xe không bao giờ thấy flow của mình) thì mình mất quyền chạm end-user. Khi đó phải chuyển sang **bảng B2B**. Điều này sẽ được kiểm ở cổng ngày 30.

**Bảng đèn §3.3 (B2B2C)**, đánh dấu cho **mọi đèn**:

| Đèn (HANDBOOK §3.3) | ✅ / 🔧 / ❌ | Số nằm ở đâu / cần gì để đo |
|---|---|---|
| L · Partner activation rate ⭐ | 🔧 | Event `booking_confirmed` đã thiết kế (`3_Value_Metric!B11`). Cần thêm cờ `is_test` để loại xe của nhân viên xưởng, và cần ít nhất 1 xưởng go-live (pilot từ 07/11/2026) |
| L · End-user reach trong partner | 🔧 | Cần mẫu số là **số xe xưởng đang quản lý có hạn bảo dưỡng trong tháng**. Phải ghi vào thoả thuận pilot rằng xưởng cung cấp danh sách xe (ẩn danh) |
| L · Time-to-first-end-user | 🔧 | Cần lưu 2 timestamp: ngày ký pilot và `booking_confirmed` thật đầu tiên. Làm được trong 2 tuần |
| O · Volume volatility | ❌ | Cần ≥3 tháng volume/xưởng mới có độ lệch chuẩn. Sớm nhất 02/2027. **Không chọn** vào dashboard 90 ngày |
| O · GM sau rev-share | 🔧 | Mô hình hiện **không có rev-share** (xưởng trả thẳng cho mình). Phí nền tảng Zalo (ZNS/OA) **chưa có trong Cost/Job**, cần báo giá chính thức từ Zalo. Thay bằng đèn "GM tiền gói" (G7) |
| O · Chi phí inference ÷ doanh thu, theo TỪNG partner | 🔧 | Log API Gemini đã có token count. Cần gắn `tenant_id` vào mỗi call để tách theo xưởng. Thay bằng đèn Cost/Job theo xưởng (O5) |
| O · Tập trung volume | 🔧 | Tính được từ log khi có ≥2 xưởng. Trong 90 ngày chỉ có 1–2 xưởng pilot nên đèn luôn đỏ 100%, vô nghĩa. **Không chọn**, mở lại khi có ≥5 xưởng |
| O · Chất lượng nhìn từ end-user | 🔧 | Event `rejected` / `rescheduled` từ dashboard CVDV (`5_90Day_Plan!A17`). Kênh khiếu nại của chủ xe **chưa có** (❌), cần thêm nút "báo sai dự toán" trong mini-flow |
| G · Doanh thu/partner · partner NRR · GM tổng | ❌ / 🔧 | Partner NRR cần 12 tháng hợp đồng (❌, sớm nhất 11/2027). GM tổng tính được từ mô hình (🔧, số thật cần hoá đơn cloud + API) |

---

## Trạm 2 — Thẻ đèn

**North Star:** **Partner activation rate**, tức % xưởng go-live có ≥1 booking được CVDV xác nhận từ chủ xe thật trong 30 ngày. **Hiện tại:** chưa đo (0 xưởng go-live). **Mục tiêu:** xưởng pilot đầu tiên activate trong ≤12 ngày sau go-live. Khi có ≥5 xưởng go-live: ≥60%.

| # | Tầng | Đèn | Định nghĩa (đếm gì · **không** đếm gì) | Công thức | Nhịp · ai lấy số | Báo trước cho |
|---|---|---|---|---|---|---|
| 1 | L | **Partner activation rate** ⭐ | Đếm xưởng đã go-live (link mini-flow đã bật cho chủ xe) có ≥1 `booking_confirmed` từ **chủ xe thật** trong 30 ngày kể từ go-live. **Không** đếm: xe của nhân viên xưởng hoặc tài khoản `is_test`, booking CVDV tự tạo hộ ngoài flow, booking trùng `booking_id`+`cycle_id`, request còn pending/rejected | Số xưởng có ≥1 booking thật ÷ số xưởng đã go-live ≥30 ngày | Tuần · Product (Khánh) lấy từ bảng `booking_confirmed` | Booking/xưởng/tháng (O4) → GM tiền gói (G7) và CAC/xưởng activated (G8) |
| 2 | L | **Time-to-first-end-user (TTFEU)** | Số ngày lịch từ ngày ký thoả thuận pilot đến `booking_confirmed` thật đầu tiên (cùng loại trừ như đèn 1). **Không** tính từ ngày go-live, vì chậm tích hợp cũng là chậm | Ngày(`first_real_booking_confirmed`) − Ngày(`pilot_signed`) | Mỗi xưởng · Product | Partner activation (L1) → số eligible case trong cửa sổ pilot → cổng ngày 60 |
| 3 | L | **AI-to-request rate (containment)** — *đèn chi phí AI* | Đếm case thử của **xưởng trả phí** mà Agent đưa tới `booking_request_created` **không cần vendor escalation**. CVDV xác nhận theo quy trình chuẩn **không** bị tính là thất bại. **Không** đếm traffic free ngoài tenant, case test, case retry trùng | `booking_request_created` không escalation ÷ case thử (attempted) | Ngày · DEV Data/Eval từ trace log | Cost/Job (O5) → GM tiền gói (G7). Mỗi 1 điểm % containment mất đi làm tăng chi phí escalation (`2_Pricing!B48`) |
| 4 | O | **Booking confirmed / xưởng activated / tháng** | Đếm `booking_confirmed` thật, unique theo `booking_id`+`maintenance_cycle_id`, của **từng** xưởng trong tháng lịch. **Không** đếm đổi ca (re-schedule) thành booking mới, **không** đếm booking xưởng nhận qua điện thoại ngoài flow | Σ `booking_confirmed` của xưởng i trong tháng | Tháng (theo dõi tuần) · Backend | Giá trị xưởng nhận được → quyết định gia hạn → Partner NRR. Volume thấp cũng đẩy Cost/Job (O5) lên vì chi phí tenant cố định |
| 5 | O | **Cost/Job trực tiếp theo TỪNG xưởng** — *đèn chi phí AI* | Toàn bộ chi phí trực tiếp phục vụ case thử của xưởng i (API + retry + infra biến đổi + phần infra chung phân bổ + tenant 300k + QA/escalation vendor) ÷ booking confirmed của xưởng i. **Không** lấy trung bình toàn hệ thống, **không** gộp overhead R&D/sales (`1_Cost_Job!B62`) | Direct cost xưởng i ÷ `booking_confirmed` xưởng i | Tháng · DEV Backend (API log có `tenant_id` + hoá đơn cloud) | GM tiền gói (G7), lộ ra ở P&L 1 quý sau |
| 6 | O | **Tỷ lệ request bị CVDV từ chối** — *chất lượng nhìn từ end-user* | Đếm request do Agent tạo mà CVDV **reject** hoặc **sửa hạng mục/dự toán** trước khi xác nhận. **Không** đếm việc đổi ca do xưởng hết chỗ hay chủ xe tự huỷ. Thêm **sự cố cứng**: dự toán/hạng mục không có trong Rule Engine hoặc DB giá ("bịa") | (rejected + sửa hạng mục) ÷ `booking_request_created` | Tuần · Backend/Workshop từ dashboard CVDV | Xưởng mất tin → tắt link khỏi Zalo OA → Partner activation (L1) và gia hạn. Đồng thời giảm job confirmed → Cost/Job (O5) |
| 7 | G | **GM tiền gói** | (Doanh thu gói + overage − COGS trực tiếp, gồm cả chi phí traffic free và phí nền tảng Zalo khi có) ÷ doanh thu. **Không** trừ overhead R&D/sales | 1 − COGS ÷ Doanh thu (`2_Pricing!B19`) | Quý (xem tháng) · Product/Finance | (kết quả) |
| 8 | G | **CAC / xưởng activated** | Toàn bộ chi phí founder-led để có 1 xưởng **activated** (không phải "đã ký"): thời gian founder quy tiền, demo, đi lại, setup/tích hợp. Mẫu số là xưởng activated, **không** phải xưởng đã ký | Tổng chi phí bán + setup ÷ số xưởng activated | Quý · Khánh | (kết quả) → CAC payback |

**Đèn chi phí AI:** số **3** (AI-to-request, bắt được sớm, tính theo ngày) và số **5** (Cost/Job theo xưởng). Cả hai báo trước GM (G7) khoảng 1 quý.
**Đếm tầng:** 3 Leading · 3 Operating · 2 Lagging (25% lagging).

---

## Trạm 3 — Ngưỡng

| # | Đèn | 🟢 | 🟡 | 🔴 | Nguồn | Lý do (1 câu) · ngày kiểm tra nếu [BM] |
|---|---|---|---|---|---|---|
| 1 | Partner activation rate | ≥60% | 30–60% | <30% | **[TB]** | Chưa có chuẩn ngành, nên lấy mức khởi điểm của HANDBOOK §3.3, đo 2 chu kỳ 30 ngày (dự kiến có số 06/12/2026 và 05/01/2027) rồi đặt lại baseline. Khi <5 xưởng go-live, đọc theo từng xưởng: xưởng có 0 booking thật sau 30 ngày = 🔴 |
| 2 | Time-to-first-end-user | ≤12 ngày | 13–18 ngày | >18 ngày | **[MH]** | Pilot chỉ có 30 ngày (07/11–06/12) để gom ≥100 eligible case, với tốc độ 250 case/xưởng/tháng ≈ 8,3 case/ngày. Quá 18 ngày là không còn kịp đủ 100 case (phụ lục MH-1) |
| 3 | AI-to-request rate | ≥78% | 67,5–78% | <67,5% | **[MH]** | Dưới 67,5% thì GM/job quy đổi <60% (`2_Pricing!B50`). 78% là giả định mà giá Starter đang dựa vào, nên xanh = đạt đúng giả định (phụ lục MH-2) |
| 4 | Booking confirmed / xưởng / tháng | ≥179 | 108–178 | <108 | **[MH]** | Dưới 108 booking/tháng thì giá Starter vượt 25% giá trị xưởng nhận được, tức vượt giá trần. 179 là volume mô hình đang giả định (phụ lục MH-3) |
| 5 | Cost/Job theo xưởng | ≤8.228 ₫ | 8.229–9.874 ₫ | >9.874 ₫ | **[MH]** | Giá quy đổi 24.684 ₫/job. ≤8.228 ₫ giữ được giá ≥3× cost (giá sàn Day 22). >9.874 ₫ thì GM/job <60% (phụ lục MH-4) |
| 6 | Tỷ lệ request bị CVDV từ chối | ≤8% **và** 0 sự cố "bịa" | 8–25,5% | >25,5% **hoặc** ≥1 sự cố "bịa" dự toán/hạng mục | **[MH]** + **[TB]** | Mốc 8% là giả định acceptance 92% (`1_Cost_Job!B15`). >25,5% thì GM/job <60% ngay cả khi containment đạt 78% (phụ lục MH-5). Sự cố "bịa" là đỏ tuyệt đối theo bài học Klarna: lỗi của mình là khủng hoảng thương hiệu của xưởng. Baseline reject thật đo 2 chu kỳ ([TB], `5_90Day_Plan!B17`) |
| 7 | GM tiền gói | ≥60% | 53–60% | <53% | **[MH]** + **[BM]** | 60% là mục tiêu GM mà toàn bộ giá Day 22 dựa vào (`6_Benchmarks!B19`). 53% là GM trung vị AI-native 2026P (ICONIQ *State of AI 2026*, 07/2026, theo HANDBOOK §8 mục 7, số chốt 27/08/2026, **kiểm tra 09/10/2026**), dưới mức đó là kém hơn mặt bằng ngành |
| 8 | CAC / xưởng activated | <29,3 tr ₫ | 29,3–39,07 tr ₫ | >39,07 tr ₫ | **[MH]** + **[BM]** | >39,07 tr ₫ thì payback >12 tháng, vượt ngưỡng SMB <12 tháng (Bessemer, theo HANDBOOK §8 mục 11, **kiểm tra 09/10/2026**). Dưới 29,3 tr ₫ (payback ≤9 tháng) thì còn đệm 3 tháng cho churn năm đầu chưa đo được (phụ lục MH-6) |

**Ngưỡng tôi nghĩ dễ sai nhất:** đèn 4 (≥108 booking). Ngưỡng này chỉ đứng được nếu giả định **24 booking tăng thêm/xưởng/tháng** là thật (`2_Pricing!B29`). Bỏ uplift đi thì cần **888 booking/tháng**, không xưởng nào đạt. Vì vậy uplift phải được đo bằng nhóm đối chứng trong pilot (xem "Chưa đo được" ở dashboard).

### Phụ lục [MH] — phép tính

**[MH] 1 — Time-to-first-end-user**

```
Đầu vào: 250 case thử/xưởng/tháng (1_Cost_Job!B17) · cửa sổ pilot 30 ngày (07/11–06/12/2026)
         · mục tiêu ≥100 eligible case (5_90Day_Plan!E7)
Phép tính: tốc độ = 250 / 30 = 8,33 case/ngày
           số ngày cần để đủ 100 case = 100 / 8,33 = 12 ngày
           ngày muộn nhất phải có end-user thật = 30 − 12 = ngày 18
           giữ đệm 1,5× (150 case, phòng khi traffic thật thấp hơn giả định) → 150 / 8,33 = 18 ngày → 30 − 18 = ngày 12
Kết quả → 🟢 ≤12 ngày · 🟡 13–18 ngày · 🔴 >18 ngày
```

**[MH] 2 — AI-to-request rate (containment tối thiểu)**

```
Đầu vào: N = 1.500 case thử/tháng · a = 92% CVDV chấp nhận · giá quy đổi = 4.690.000 / 190 = 24.684 ₫/job
         C0 = chi phí trực tiếp không phụ thuộc containment = 7.368.671 ₫/tháng (2_Pricing!B49)
         K  = chi phí escalation nếu AI thất bại toàn bộ = 5.625.000 ₫/tháng (2_Pricing!B48)
Điều kiện GM/job ≥ 60%:  (C0 + K·(1−p)) / (N·p·a) ≤ 40% × 24.684 = 9.874 ₫
Giải theo p: p ≥ (C0 + K) / (N·a·9.874 + K)
           = (7.368.671 + 5.625.000) / (1.500 × 0,92 × 9.874 + 5.625.000)
           = 12.993.671 / 19.250.684 = 67,5%
Kiểm tra bằng stress test (2_Pricing!A57:E62): p = 70% → GM/job 62,0% · p = 60% → 52,9%
Kết quả → 🟢 ≥78% (giả định giá đang dựa vào) · 🟡 67,5–78% · 🔴 <67,5%
```

**[MH] 3 — Booking confirmed / xưởng / tháng (sàn giá trị)**

```
Đầu vào: giá Starter 4.690.000 ₫ · giá trần = 25% giá trị (2_Pricing!B38)
         giá trị/xưởng = tiết kiệm thao tác + margin booking tăng thêm + cứu no-show
         tiết kiệm thao tác/booking = 12 phút × 100.000 ₫/giờ = 20.000 ₫ (2_Pricing!B27:B28)
         margin booking tăng thêm = 24 × 650.000 = 15.600.000 ₫ · no-show = 2 × 500.000 = 1.000.000 ₫
Điều kiện: 4.690.000 ≤ 25% × giá trị  →  giá trị ≥ 18.760.000 ₫
           20.000 × B + 16.600.000 ≥ 18.760.000  →  B ≥ 108 booking/tháng
Đối chiếu: ở 179 booking (giả định mô hình) giá trị = 20.186.667 ₫ (2_Pricing!B36), phí chiếm 23,2%
Độ nhạy: nếu uplift 24 booking = 0 → B ≥ (18.760.000 − 1.000.000) / 20.000 = 888, bất khả thi
Kết quả → 🟢 ≥179 · 🟡 108–178 · 🔴 <108
```

**[MH] 4 — Cost/Job theo xưởng**

```
Đầu vào: giá quy đổi Starter = 24.684 ₫/job · quy tắc giá sàn 3× cost · GM mục tiêu 60%
Phép tính: cost tối đa để giá ≥ 3× cost = 24.684 / 3 = 8.228 ₫
           cost tối đa để GM/job ≥ 60%  = 24.684 × 40% = 9.874 ₫
           mô hình hiện tại: 7.998 ₫ (1_Cost_Job!B64) → 🟢
Kiểm tra chéo theo volume 1 xưởng:
  chi phí cố định/xưởng = 2.700.000/6 (infra chung) + 300.000 (tenant) = 750.000 ₫
  chi phí biến đổi/case thử = 255 × 1,08 (LLM + retry) + 900 (infra) + 667 (QA) + 825 (escalation) = 2.667 ₫
  case thử cần cho 1 booking = 1 / (78% × 92%) = 1,39 → biến đổi/job = 3.717 ₫
  Cost/Job(B) = 750.000 / B + 3.717  →  B = 179: 7.907 ₫ · B = 122: 9.874 ₫ · B = 108: 10.662 ₫
  → volume thấp (đèn 4) tự kéo đèn 5 sang đỏ: hai đèn nhất quán với nhau
Kết quả → 🟢 ≤8.228 ₫ · 🟡 8.229–9.874 ₫ · 🔴 >9.874 ₫
```

**[MH] 5 — Tỷ lệ request bị CVDV từ chối**

```
Đầu vào: giữ containment p = 78% · chi phí trực tiếp ở p = 78% = 8.606.171 ₫/tháng (1_Cost_Job!B63)
         cost/job tối đa cho GM 60% = 9.874 ₫ (MH-4)
Phép tính: job confirmed = 1.500 × 0,78 × a = 1.170 × a
           8.606.171 / (1.170 × a) ≤ 9.874  →  a ≥ 8.606.171 / 11.552.210 = 74,5%
           → tỷ lệ từ chối tối đa = 1 − 74,5% = 25,5%
Kết quả → 🟢 ≤8% (giả định 92% acceptance) · 🟡 8–25,5% · 🔴 >25,5% hoặc ≥1 sự cố "bịa"
```

**[MH] 6 — CAC / xưởng activated**

```
Đầu vào: ARPU 4.690.000 ₫/tháng · GM tiền gói 69,4% (2_Pricing!B19) · payback tối đa 12 tháng (SMB)
Phép tính: lãi gộp/xưởng/tháng = 4.690.000 × 69,4% = 3.255.638 ₫
           CAC tối đa (payback 12 tháng) = 3.255.638 × 12 = 39.067.658 ₫ (khớp 4_Channel_Fit!B10)
           CAC cho payback 9 tháng       = 3.255.638 × 9  = 29.300.743 ₫
Đối chiếu: kịch bản CPO VN 8 tr ÷ win 25% = 32 tr ₫ (4_Channel_Fit!B20) → đang 🟡, chưa đo
           CPO global 6.300 USD → CAC ≈ 660 tr ₫ = 16,9× ngân sách (4_Channel_Fit!B24) → không dùng được
Kết quả → 🟢 <29,3 tr ₫ · 🟡 29,3–39,07 tr ₫ · 🔴 >39,07 tr ₫
```

---

## Trạm 4 — 5 luật quyết định

⏹ đánh dấu luật dừng (có 3 luật).

1. ⏹ **R1 · Partner activation**: **NẾU** một xưởng đã go-live mà có **0 booking_confirmed từ chủ xe thật** (hoặc, khi đã có ≥5 xưởng go-live, activation rate <30%) **TRONG** 30 ngày kể từ go-live **THÌ** **dừng tiếp cận và ký xưởng mới**. Founder ngồi tại xưởng đó 2 buổi ca sáng (08:15) để xem CVDV và chủ xe đi qua Zalo OA thế nào, rồi sửa đúng **một** điểm chặn lớn nhất trong đường đi tới chủ xe (link, quyền OA, bước xác nhận) trong 2 tuần. **KHÔNG THÌ** không được báo cáo "số xưởng đã ký" ở bất kỳ đâu nếu không kèm số xưởng activated, và không được ký thêm xưởng để bù doanh thu.
2. **R2 · Containment**: **NẾU** AI-to-request rate <67,5% **TRONG** 2 tuần liên tiếp **VÀ** mỗi tuần ≥50 case thử **THÌ** lấy 30 case thất bại gần nhất, phân loại nguyên nhân (Rule Engine / RAG / dữ liệu ODO / bước UI), sửa nhóm lỗi lớn nhất, và chạy lại golden set 50 case đạt 100% Rule Engine trước khi deploy. **KHÔNG THÌ** không đổi sang model đắt hơn để "chữa cháy" (Gemini 3.7 Flash giá list đắt hơn 5× input, 3× output, `6_Benchmarks!B10:B11`), và không cắt QA sample 8% để kéo Cost/Job xuống.
3. ⏹ **R3 · Chất lượng nhìn từ end-user**: **NẾU** tỷ lệ request bị CVDV từ chối hoặc sửa hạng mục >25,5% **TRÊN** 50 request gần nhất của một xưởng, **HOẶC** xuất hiện ≥1 dự toán/hạng mục không có trong Rule Engine/DB giá **THÌ** trong 24 giờ **tắt chế độ tự tạo request** ở xưởng đó, chuyển sang chế độ "CVDV duyệt bản nháp", sửa rule và chạy lại golden set đến 100% rồi mới bật lại. **KHÔNG THÌ** không gửi thêm nhắc bảo dưỡng hàng loạt tới chủ xe của xưởng đó để giữ volume, vì mỗi lỗi lọt ra là lỗi trên thương hiệu của xưởng.
4. **R4 · Volume / xưởng**: **NẾU** booking_confirmed của một xưởng activated <108/tháng **TRONG** 2 tháng liên tiếp **THÌ** cùng chủ xưởng nhập danh sách xe khách cũ đến hạn bảo dưỡng vào hệ thống trong 2 tuần (vấn đề là luồng xe đến hạn, không phải tính năng). Nếu tháng tiếp theo vẫn <108, chuyển xưởng sang báo giá theo booking (24.684 ₫/job, không phí nền) và ngừng chào Starter cho phân khúc xưởng cùng cỡ. **KHÔNG THÌ** không giảm giá Starter và không làm tính năng tuỳ biến riêng cho xưởng đó để kéo volume.
5. ⏹ **R5 · CAC**: **NẾU** CAC/xưởng activated >39,07 tr ₫ **TRÊN** 2 xưởng activated gần nhất **THÌ** **dừng mở rộng tiếp cận sang khu vực mới**, chỉ bán vào cụm xưởng quanh xưởng pilot qua giới thiệu kèm Pilot Report, và tách chi phí setup/tích hợp thành phí triển khai một lần. **KHÔNG THÌ** không tuyển AE và không chạy quảng cáo B2B để "tăng tốc". Thêm người bán vào một kênh payback >12 tháng chỉ làm lỗ nhanh hơn.

**Mỗi đèn đỏ dẫn tới luật nào:** L1, L2 → R1 · L3 → R2 · O4 → R4 · O5 → R2 (nếu containment là nguyên nhân) hoặc R4 (nếu do volume thấp, xem MH-4) · O6 → R3 · G7 → R2 / R4 · G8 → R5.
