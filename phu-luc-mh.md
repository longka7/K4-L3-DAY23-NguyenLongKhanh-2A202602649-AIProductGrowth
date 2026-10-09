# PHỤ LỤC [MH] — phép tính ngưỡng suy từ mô hình

Nguồn đầu vào: mô hình tài chính Day 22 của VFO2O-06 (ô Excel ghi trong ngoặc). Giá quy đổi Starter = 4.690.000 ₫ ÷ 190 booking = **24.684 ₫/job**. Toàn bộ là giả định, chưa có pilot. Bản đầy đủ nằm ở `worksheet.md`, Trạm 3.

**MH-1 · Time-to-first-end-user.** Tốc độ 250 case/xưởng/tháng (`1_Cost_Job!B17`) = 8,33 case/ngày. Pilot dài 30 ngày và cần ≥100 eligible case (`5_90Day_Plan!E7`): 100 ÷ 8,33 = 12 ngày → end-user thật đầu tiên chậm nhất là ngày 30 − 12 = **18**. Thêm đệm 1,5× (150 case = 18 ngày) → 30 − 18 = **12**. → 🟢 ≤12 · 🟡 13–18 · 🔴 >18 ngày.

**MH-2 · AI-to-request rate.** C0 = 7.368.671 ₫ (`2_Pricing!B49`), K = 5.625.000 ₫ (`B48`), N = 1.500, a = 92%. GM/job ≥60% ⇔ (C0 + K(1−p)) ÷ (N·p·a) ≤ 40% × 24.684 = 9.874 ₫. Suy ra p ≥ (C0+K) ÷ (N·a·9.874 + K) = 12.993.671 ÷ 19.250.684 = **67,5%**. Stress test: p = 70% → GM 62,0%; p = 60% → GM 52,9%. → 🟢 ≥78% (giả định giá đang dựa vào) · 🟡 67,5–78% · 🔴 <67,5%.

**MH-3 · Booking/xưởng/tháng.** Giá trần = 25% giá trị (`2_Pricing!B38`), nên giá trị phải ≥ 4.690.000 ÷ 25% = 18.760.000 ₫. Giá trị = 20.000 ₫ × B (12 phút × 100.000 ₫/giờ) + 15.600.000 (24 booking tăng thêm × 650.000) + 1.000.000 (no-show). Suy ra 20.000·B ≥ 2.160.000 ⇒ **B ≥ 108**. Ở mức giả định 179 booking, giá trị = 20,19 tr ₫ và phí chiếm 23,2%. Độ nhạy: nếu uplift = 0 thì cần B ≥ 888. → 🟢 ≥179 · 🟡 108–178 · 🔴 <108.

**MH-4 · Cost/Job theo xưởng.** Để giữ 3× cost: 24.684 ÷ 3 = **8.228 ₫**. Để GM/job ≥60%: 24.684 × 40% = **9.874 ₫**. Mô hình hiện tại 7.998 ₫ (`1_Cost_Job!B64`). Kiểm tra chéo trên 1 xưởng: cố định 750.000 ₫ (450k infra chung + 300k tenant); biến đổi 2.667 ₫/case thử ÷ (78% × 92%) = 3.717 ₫/job. Suy ra Cost/Job(B) = 750.000/B + 3.717: B = 179 → 7.907 ₫ · B = 122 → 9.874 ₫ · B = 108 → 10.662 ₫. Volume thấp tự đẩy đèn này sang đỏ. → 🟢 ≤8.228 · 🟡 8.229–9.874 · 🔴 >9.874 ₫.

**MH-5 · Tỷ lệ request bị CVDV từ chối.** Giữ p = 78%, chi phí 8.606.171 ₫/tháng (`1_Cost_Job!B63`), job = 1.170 × a. Điều kiện 8.606.171 ÷ (1.170·a) ≤ 9.874 ⇒ a ≥ **74,5%**, tức tỷ lệ từ chối ≤ **25,5%**. → 🟢 ≤8% (giả định acceptance 92%) · 🟡 8–25,5% · 🔴 >25,5% hoặc ≥1 lỗi "bịa".

**MH-6 · CAC/xưởng activated.** Lãi gộp/xưởng/tháng = 4.690.000 × 69,4% = 3.255.638 ₫. Payback 12 tháng (ngưỡng SMB) → CAC tối đa **39.067.658 ₫** (`4_Channel_Fit!B10`). Payback 9 tháng → **29.300.743 ₫**. Đối chiếu: kịch bản CPO VN 8 tr ÷ win 25% = 32 tr ₫, đang 🟡. Benchmark CPO global 6.300 USD → ≈660 tr ₫, gấp 16,9× ngân sách, không dùng được. → 🟢 <29,3 · 🟡 29,3–39,07 · 🔴 >39,07 tr ₫.

**[BM] dùng trong dashboard:** GM sản phẩm AI 53% (2026P), ICONIQ *State of AI 2026* (07/2026, iconiq.com/growth/reports/state-of-ai-2026) · CAC payback SMB <12 tháng, Bessemer *Scaling to $100 Million* (bvp.com/atlas/scaling-to-100-million). Cả hai **đã mở nguồn gốc kiểm tra ngày 09/10/2026**, số khớp HANDBOOK §8. Giá Gemini 2.5 Flash và tỷ giá NCB 26.190 ₫/USD lấy từ `6_Benchmarks` (kiểm tra 08/10/2026).
