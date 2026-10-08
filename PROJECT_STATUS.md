# Project Status

> **Phạm vi kiểm tra:** Repository không được truy cập trực tiếp. Trạng thái dưới đây dựa trên việc đọc lại toàn bộ code `index.html` (ghép từ 3 đoạn, viết ngày 2026-10-08). Code **chưa được chạy trên trình duyệt** và **chưa có kết quả self-test**.
> Nếu code trong repo khác mô tả ở đây → tin code trong repo và cập nhật file này.
>
> **Ký hiệu:** `Code: ✅` = đã đọc code và xác nhận logic tồn tại · `Runtime: ⚠️` = chưa chạy thực tế để xác minh.

## 1. Current Phase

- **Current Phase:** Phase 1 — Core Game Loop (Playable MVP)
- **Phase Status:** Needs Verification
- **Last Updated:** 2026-10-08
- **Overall Progress:** khoảng 25% toàn bộ spec (ước lượng thô theo số feature). Phase 1 đã có đủ code, chưa xác minh runtime.
- **Current Development Focus:** Xác minh runtime Phase 1 (self-test `?test` + chơi tay 1 ngày), sửa lỗi phát sinh. Không làm feature mới.

## 2. Phase Progress

> ⚠️ **Lệch giữa các tài liệu:** Tên phase trong bảng này (theo yêu cầu tạo file) **khác** với spec gốc. Spec gốc chia: Phase 1 MVP · Phase 2 (PC upgrade, Internet, Cleaning, Repair, Reputation, Random events) · Phase 3 (Staff, Expansion, VIP room, Analytics, Achievements) · Phase 4 (Seasonal, Decoration, Advanced AI, More content). Bảng dưới giữ tên theo yêu cầu. Cần chủ project chốt roadmap nào là chính thức.

| Phase | Status | Progress | Notes |
|---|---|---:|---|
| Phase 1 — Core Game Loop | Needs Verification | ~90% (code) | Code đủ vòng lặp ngày. Chưa chạy runtime. BUG-001 có thể chặn khởi động. |
| Phase 2 — Food Preparation | Not Started | ~40% (nền) | Đã có bếp/chảo/quầy nước/tủ lạnh dạng hẹn giờ từ MVP. Chưa có thao tác từng bước như spec. |
| Phase 3 — Vietnamese Atmosphere | Not Started | ~30% (nền) | Đã có thoại tiếng Việt, art SVG chibi, tên khách VN. Chưa có trang trí, mùa lễ, nhạc. |
| Phase 4 — Gameplay & Economy | Not Started | ~20% (nền) | Đã có economy cơ bản + rating. Chưa có upgrade, staff, event, internet, vay vốn. |
| Phase 5 — QA & Polish | Not Started | ~10% (nền) | Đã có bộ self-test tích hợp, chưa chạy. |

## 3. Implemented Features

Tất cả nằm trong một file: `index.html`. Vị trí ghi theo tên hàm/hằng số, vì số dòng phụ thuộc cách ghép file.

### Core Game

- **Game loop** — Code: ✅ · Runtime: ⚠️
  - *Location:* `frame()`, `stepSim(dt)`
  - *Behavior:* `requestAnimationFrame`. 1 giây thật = 2 phút game ở tốc độ 1x (`CFG.MIN_PER_SEC`). Mô phỏng chia bước ≤ 1 phút game. Có tạm dừng, 1x/2x/4x.
  - *Limitation:* Không có fixed timestep tách khỏi render. Tab chạy nền bị trình duyệt giảm RAF → game gần như đứng.
- **Time system / ngày** — Code: ✅ · Runtime: ⚠️
  - *Location:* `CFG.OPEN/CLOSE/LAST_ENTRY`, `period()`, `PERIODS`
  - *Behavior:* Mở cửa 08:00, ngừng nhận khách 23:00, đóng 24:00. Chia 5 khung Sáng/Trưa/Chiều/Tối/Đêm, ảnh hưởng tỉ lệ khách và loại khách. Một ngày ≈ 8 phút thật ở 1x.
  - *Limitation:* Không có hiệu ứng sáng/tối trên bản đồ, chỉ đổi icon trên top bar.
- **Phases trong ngày** — Code: ✅ · Runtime: ⚠️
  - *Location:* `S.phase`, các giá trị `PREP → OPEN → CLOSING → REPORT`; `openShop()`, `endDay()`, `nextDay()`
- **Customer spawning** — Code: ✅ · Runtime: ⚠️
  - *Location:* `maybeSpawn()`, `spawnCustomer()`, `CTYPES`
  - *Behavior:* Xác suất theo khung giờ × cuối tuần (×1.3) × rating. Giới hạn 6 khách chờ quầy, tổng ≤ số PC + 6. Khách đầu tiên xuất hiện ngay khi mở cửa. 35% là khách quen (lấy từ `S.accounts`).
- **Customer AI (state machine)** — Code: ✅ · Runtime: ⚠️ — xem mục 9.
- **PC system** — Code: ✅ · Runtime: ⚠️
  - *Location:* `PC_TIERS`, `PC_LAYOUT`, `assignPC()`, `checkout()`, `seatedTick()`
  - *Behavior:* 8 PC (3 Gaming, 1 VIP, 4 Thường). Trạng thái thực dùng: `AVAILABLE`, `RESERVED`, `OCCUPIED`. Trừ số dư khách theo phút với giá 8k/12k/18k mỗi giờ. Màn hình sáng khi có người chơi. Panel PC hiện thời gian, khách và doanh thu.
  - *Limitation:* `MAINTENANCE/BROKEN/CLEANING` chưa dùng. Trường `condition` có trong data nhưng không ảnh hưởng gì.
- **Player avatar + di chuyển** — Code: ✅ · Runtime: ⚠️
  - *Location:* `walkTo()`, `movePlayer()`, `onWorld()`
  - *Behavior:* Bấm vào đối tượng → nhân vật chạy tới → làm hành động khi tới nơi.
  - *Limitation:* Đi thẳng, xuyên qua bàn ghế (không tìm đường). Mục tiêu đang đi không được lưu khi reload.
- **Money + transaction log** — Code: ✅ · Runtime: ⚠️
  - *Location:* `addTx()`
  - *Behavior:* Mọi thay đổi tiền đều đi qua `addTx`, có đủ `transactionId, type, amount, timestamp, description` (kèm `day, gameTime`).
- **Recharge (nạp tiền)** — Code: ✅ · Runtime: ⚠️
  - *Location:* `recharge()`, `BUILD.counter`, `BUILD.cust`, `ACT.rc/rcx`
  - *Behavior:* Các mức 10k–200k hoặc nhập số khác. Hiện phép cộng `trước + nạp = sau`. Chặn nạp vượt tiền mặt của khách (đã trừ phần tiền món đang gọi chưa giao).
- **Save / Load** — Code: ✅ · Runtime: ⚠️ — xem mục 10.
- **End-day report** — Code: ✅ · Runtime: ⚠️
  - *Location:* `endDay()`, `BUILD.report`
  - *Behavior:* Doanh thu (nạp, đồ ăn, nước), chi phí (nhập hàng, điện, internet, bảo trì), lợi nhuận, lượt khách, bỏ về, % hài lòng. Có idempotent guard: gọi lại không trừ phí lần hai.

### Food

- **Bếp (mì), Chảo chiên, Quầy pha nước** — Code: ✅ · Runtime: ⚠️
  - *Location:* `STATIONS`, `startCook()`, `stationTick()`, `pickUp()`, `interactStation()`
  - *Behavior:* Bếp có 2 slot, chảo 2 slot, quầy nước 1 slot. Chọn món → trừ nguyên liệu → đếm giờ, hiện tên bước đang làm → "Xong!" → bấm để lấy lên tay.
- **Tủ lạnh (nước chai)** — Code: ✅ · Runtime: ⚠️ — `takeFridge()`, lấy thẳng lên tay.
- **Tay cầm (tray)** — Code: ✅ · Runtime: ⚠️ — `S.tray`, tối đa 4 món, bấm vào món để bỏ.
- **Giao món** — Code: ✅ · Runtime: ⚠️ — `deliverTo()`, `finishOrderIfDone()`
  - *Behavior:* Giao từng món khớp với order, thu tiền từng món, cờ `it.done` chống thu hai lần.
- **Báo hết món** — Code: ✅ · Runtime: ⚠️ — `markOOS()`, khách bị trừ 5 điểm hài lòng.

### Inventory
- **Tồn kho, nhập hàng, cảnh báo sắp hết** — Code: ✅ · Runtime: ⚠️ — `ITEMS`, `S.inv`, `buyStock()`, `BUILD.inv`. Xem mục 8.

### UI
- **Top bar** (ngày/giờ, tiền, khách đang ngồi/số PC, rating, trạng thái, tốc độ), **hint bar** có nhân vật hướng dẫn, **bottom bar** (Máy, Đồ ăn, Kho, Doanh thu, Cài đặt), **bottom sheet**, **toast**, **số tiền bay lên**, **bong bóng thoại**, **thanh kiên nhẫn** — Code: ✅ · Runtime: ⚠️

### QA
- **Self-test tích hợp** — Code: ✅ · Runtime: ⚠️ **chưa từng chạy**
  - *Location:* `runTests()`, `simulateFullDay()`, `botTick()`, `checkInv()`
  - *Cách chạy:* Cài đặt → "Chạy kiểm thử core loop", hoặc mở URL kèm `?test`.
  - *Nội dung:* 10 kịch bản spec + kiểm tra lỗi (hết hàng, thiếu tiền, thu tiền hai lần) + bot chơi trọn 1 ngày, kiểm tra invariant mỗi phút game (sổ sách khớp, không trùng máy, kho không âm).

## 4. Partially Implemented Features

### Food preparation flow
**Status:** ⚠️ Partial
**Implemented:** Chọn món → hẹn giờ có hiện tên bước → lấy → giao.
**Missing:** Spec yêu cầu người chơi tự thao tác từng bước (TAKE → OPEN_PACK → … → SERVE). Thiếu topping tự chọn (trứng, xúc xích, cá viên, rau); thiếu mì bò/gà/cay; thiếu bò viên, tôm viên, hồ lô; thiếu Pepsi/7Up/trà đào/cà phê; không có món bị cháy hay quá giờ.
**Files:** `index.html` — `MENU`, `startCook`, `stationTick`, `BUILD.station`
**Next step:** Chốt thiết kế mini-step (bấm từng bước hay giữ hẹn giờ) trước khi code.

### Reputation
**Status:** ⚠️ Partial
**Implemented:** `S.rating` (1–5 sao), cập nhật theo trung bình trượt mức hài lòng mỗi khách khi rời đi (`rateVisit`). Ảnh hưởng tỉ lệ khách tới.
**Missing:** Không ảnh hưởng khách VIP, mức chi tiêu hay tỉ lệ quay lại. Không có hiệu ứng ⭐ +0.1.
**Next step:** Thuộc Phase 2 (theo spec gốc).

### Demand forecast
**Status:** ⚠️ Partial
**Implemented:** `forecast()` dựa trên tỉ lệ khách theo khung giờ × rating × cuối tuần. Hiện trong bảng chuẩn bị đầu ngày.
**Missing:** Thời tiết, ngày lễ, event, doanh thu lịch sử. Dự báo món ăn chỉ là tỉ lệ cố định.

### Tutorial / Onboarding
**Status:** ⚠️ Partial
**Implemented:** Bảng chào mừng 4 dòng + hint bar đổi theo ngữ cảnh (`hintText`).
**Missing:** Không highlight đối tượng, không có các bước bắt buộc. `tutorialDone` chỉ đổi lời gợi ý.

### Localization
**Status:** ⚠️ Partial
**Implemented:** `STR.vi`, `t()`, `DIALOGUE.vi`, `BRAND` (đổi tên thương hiệu nước không ảnh hưởng gameplay).
**Missing:** Còn chữ viết thẳng trong code: tên món và bước nấu (`MENU`), tên loại khách (`CTYPES`), chữ trong SVG ("QUẦY THU NGÂN", "THỰC ĐƠN"…), nhãn khu vực ("KHU MÁY", "KHU CHỜ"), tên các bài test.

### Customer types
**Status:** ⚠️ Partial — 5/6 loại: Học sinh, Sinh viên, Game thủ, Văn phòng, Khách đêm. **Thiếu Khách VIP.**

### Analytics
**Status:** ⚠️ Partial
**Implemented:** Doanh thu hôm nay, top 5 món bán chạy, 30 giao dịch gần nhất, lịch sử 7 ngày.
**Missing:** Tuần/tháng, biểu đồ, khách mới vs khách quay lại.

### Settings
**Status:** ⚠️ Partial — có Âm thanh, Tốc độ, Lưu, Test, Reset. Thiếu Nhạc nền, bật/tắt Animation, đổi Ngôn ngữ.

### Sound
**Status:** ⚠️ Partial — có hook `SFX.play(name)` + `SOUND_FILES` để gắn file. Hiện chỉ phát beep tổng hợp bằng WebAudio.

### Mobile
**Status:** ⚠️ Partial — scale tối thiểu 0.75, kéo ngang để xem. Không có pinch-zoom; nút ≥ 44px ở UI chính.

### Accessibility
**Status:** ⚠️ Partial — trạng thái PC có biểu tượng ○◐● kèm chữ (không chỉ dựa vào màu), `aria-label`, Enter/Space trên đối tượng, phím Esc, Space, 1/2/4. Chưa kiểm tra tương phản và trình đọc màn hình.

### Returning customers
**Status:** ⚠️ Partial — số dư còn lại được lưu theo tên (`S.accounts`, tối đa 40). Chưa có thống kê "khách quay lại".

## 5. Not Implemented

Đã đối chiếu với code: không có hàm hay data nào cho các mục sau.

- PC upgrade (CPU/GPU/RAM…), Internet packages / lag / disconnect
- PC hỏng, sửa máy, bảo trì (`MAINTENANCE/BROKEN`), dọn bàn, dọn rác (`CLEANING`)
- Random events (5 khách cùng lúc, máy đứng hình, đổ nước, xin nợ, dẫn bạn)
- Weather / special events / seasonal events / decorations
- Staff (thu ngân, bếp, dọn dẹp, kỹ thuật), staff level
- Expansion (12–64 PC), VIP room, progression levels (Góc phố → Cyber Arena)
- Achievements
- Payment methods (tiền mặt / QR / ví) — hiện chỉ có một hình thức ngầm định
- Loan system, game over, xử lý tiền âm (tiền có thể âm sau chi phí cuối ngày, không có cảnh báo hay hậu quả)
- Animation xe giao hàng (nhập hàng là tức thì)
- Order UI dạng hóa đơn (`ORDER #1024 … TỔNG`) — order chỉ hiện icon trong bong bóng
- Spoilage, maxStock, dailyUsage
- Music
- Kiến trúc React/TypeScript/Vite theo cấu trúc thư mục trong spec

## 6. Current Game Loop

```
PREP (07:30, đồng hồ đứng)
→ Xem dự báo, nhập hàng (Kho)
→ Bấm MỞ CỬA → 08:00, khách đầu tiên vào
→ Khách vào → xếp hàng ở quầy (WAITING, giảm kiên nhẫn)
   → quá kiên nhẫn → bỏ về [FRUSTRATED là hành vi, không phải state riêng]
→ Người chơi bấm khách → nạp tiền → chọn máy (bảng hoặc bấm máy trên bản đồ)
→ Khách đi tới máy (TO_PC, PC=RESERVED) → ngồi (PLAYING, PC=OCCUPIED)
→ Trừ số dư theo phút; đói/khát tăng
→ Gọi món (WAITING_FOOD) → người chơi nấu/lấy → giao → khách trả tiền mặt → EATING → PLAYING
   → quá kiên nhẫn → hủy order, trừ hài lòng
→ Số dư sắp hết → xin nạp; hết hẳn → RECHARGE (máy dừng) → nạp → chơi tiếp / quá hạn → về
→ Đủ giờ chơi → CHECKOUT → PC=AVAILABLE → lưu số dư → ra cửa
→ 24:00 → CLOSING → mọi khách về
→ REPORT: trừ điện/internet/bảo trì → doanh thu/chi phí/lợi nhuận
→ Sang ngày mới → PREP
[NOT IMPLEMENTED] dọn bàn sau khi khách về · máy hỏng · sự kiện ngẫu nhiên · nâng cấp
```

## 7. Economy Status

### Money
- **Starting money:** 5.000.000đ (`CFG.START_MONEY`)
- **Money source:** Nạp tiền (`RECHARGE`), bán đồ ăn (`FOOD_SALE`), bán nước (`DRINK_SALE`)
- **Money sink:** Nhập hàng (`INVENTORY_PURCHASE`), điện (`ELECTRICITY`), internet (`INTERNET`), bảo trì (`REPAIR`)
- **Currency:** VND, số nguyên. Số dư khách là số thực, chỉ làm tròn khi hiển thị.
- **Transaction handling:** Mọi thay đổi `S.money` đều qua `addTx()`. Log reset mỗi ngày (`S.tx`), lịch sử ngày lưu trong `S.history` (tối đa 60).

### Revenue
- **PC revenue:** Không tính trực tiếp. Doanh thu = **tiền nạp**. Tiền máy đã dùng (`pcUsage`) chỉ hiển thị để tham khảo.
- **Food revenue:** Mì 12–20k, đồ chiên 12–18k (lãi gộp 51–60%).
- **Drink revenue:** 6–13k (lãi gộp 38–50%).
- **Other:** Không có.

### Expenses
- **Inventory:** Tính theo lần nhập.
- **Electricity:** 40.000đ + 2.500đ × giờ PC thực dùng.
- **Internet:** 50.000đ/ngày cố định.
- **Staff:** Not implemented.
- **Repairs:** 30.000đ/ngày cố định (ghi type `REPAIR`, tên "Bảo trì").
- **Other:** Not implemented.

> ⚠️ **Design decision cần chốt:** Ghi nhận doanh thu khi nạp nghĩa là số dư khách chưa dùng vẫn được tính vào lợi nhuận hôm nay. Ngày sau khách quen dùng số dư cũ thì không có doanh thu. Spec mẫu lại liệt kê cả "Tiền máy" và "Nạp tiền" là doanh thu, cách này sẽ **tính trùng**. Cần chọn một mô hình.

## 8. Inventory Status

| Item | Implemented | Purchase | Consume | Stock Tracking | Notes |
|---|---|---|---|---|---|
| Mì gói | Code ✅ | ✅ `buyStock` | ✅ 3 món mì | ✅ `S.inv.mi` | Runtime ⚠️ |
| Trứng | Code ✅ | ✅ | ✅ Mì trứng | ✅ | |
| Xúc xích | Code ✅ | ✅ | ✅ Mì xúc xích, xúc xích chiên | ✅ | |
| Cá viên | Code ✅ | ✅ | ✅ Cá viên chiên | ✅ | |
| Nước tăng lực "Sấm Sét" | Code ✅ | ✅ | ✅ Tủ lạnh | ✅ | Tên đổi qua `BRAND.energy` |
| "Cola Mát" | Code ✅ | ✅ | ✅ | ✅ | `BRAND.cola` |
| "Suối Mát" | Code ✅ | ✅ | ✅ | ✅ | `BRAND.water` |
| Bột trà chanh | Code ✅ | ✅ | ✅ Trà chanh | ✅ | |
| Tô giấy | Code ✅ | ✅ | ✅ mọi món mì | ✅ | Cá viên không dùng đĩa |
| Ly nhựa | Code ✅ | ✅ | ✅ Trà chanh | ✅ | |

Item data thiếu so với spec: `maxStock`, `sellPrice` (giá bán nằm ở `MENU`), `dailyUsage`, `spoilageRate`. Món nấu xong chưa giao sẽ bị hủy cuối ngày, nguyên liệu mất.

## 9. Customer AI Status

| Behavior | Status | Ghi chú |
|---|---|---|
| Spawn | Implemented | `spawnCustomer`, ngoại hình ngẫu nhiên (da, tóc, 4 kiểu tóc, áo, quần, phụ kiện) |
| Enter shop | Implemented | `ENTER` → đi từ cửa tới vị trí xếp hàng |
| Find PC | Partial | Người chơi chọn máy; khách không tự tìm. Khách có máy ưa thích, xếp sai hạng bị trừ hài lòng |
| Sit | Implemented | `TO_PC` → `PLAYING`, hiển thị quay lưng |
| Use PC | Implemented | Trừ số dư theo phút |
| Order food | Implemented | 1–3 món theo đói/khát và sở thích, không vượt tiền mặt |
| Wait | Implemented | Thanh kiên nhẫn ở quầy / chờ món / chờ nạp |
| Receive food | Implemented | `deliverTo`, ăn 15 phút |
| Pay | Implemented | Tiền mặt cho món, tài khoản cho giờ máy |
| Recharge | Implemented | Xin nạp khi số dư < 12 phút; hết hẳn thì dừng chờ |
| Leave | Implemented | Hết giờ chơi, hết tiền, quá kiên nhẫn, đóng cửa |
| Satisfaction | Implemented | `happiness` 0–100, mặt đổi 3 mức, ảnh hưởng rating |
| Random behavior | Partial | Thời gian chơi, đói, sở thích ngẫu nhiên; câu đùa "chơi 5 phút". Không có event |
| FRUSTRATED / CHECKOUT / IDLE | Partial | Là hành vi trong hàm, không phải state riêng như spec |
| Pathfinding | Not Implemented | Đi theo waypoint cố định, đi xuyên đồ vật |

## 10. Save / Load Status

- **Save format:** JSON của toàn bộ object `S` (`serialize()`), có trường `v` (version).
- **Save location:** `localStorage["tiemnet_save_v1"]`
- **Thời điểm lưu:** mỗi 30 giây, khi nhập hàng, mở cửa, kết thúc ngày, sang ngày mới, khi ẩn tab, `beforeunload`, bấm "Lưu ngay".
- **Data saved / loaded:** ngày, giờ, phase, tốc độ, pause, tiền, rating, toàn bộ PC, kho, slot đang nấu, món trên tay, **toàn bộ khách đang có mặt** (vị trí, đường đi, order, số dư), tài khoản khách quen, giao dịch hôm nay, lịch sử, bộ đếm ID, vị trí nhân vật, thống kê ngày, báo cáo, cài đặt.
- **Không lưu (theo thiết kế):** đích đang đi của nhân vật + hành động chờ, bảng đang mở, `SERVING`.
- **Mất dữ liệu?** Theo code thì money, inventory, customers, PC state, day, reputation đều được lưu. Upgrades/progression chưa tồn tại. **Runtime ⚠️ chưa xác minh** (self-test 10a/10b sẽ kiểm tra).
- **Known issues:** xem BUG-004. `deserialize` chỉ merge nông (shallow), save cũ thiếu key lồng nhau mới sẽ không được bổ sung. Nếu localStorage bị chặn (chế độ riêng tư, trình duyệt nhúng) thì lưu thất bại âm thầm, không báo người chơi.

## 11. Technical Architecture

**Thực tế:** một file `index.html` (HTML + CSS + JS thuần, không build, không framework). Chỉ phụ thuộc bên ngoài là Google Fonts (Nunito). Các khối được đánh dấu bằng comment `/* ===== … ===== */`.

| System | Location (trong `index.html`) | Responsibility | Dependencies |
|---|---|---|---|
| Config | `CFG` | Hằng số thời gian, kinh tế, vị trí | — |
| i18n | `STR`, `t()`, `DIALOGUE`, `line()` | Chuỗi UI, thoại | — |
| Data | `PC_TIERS`, `PC_LAYOUT`, `ITEMS`, `MENU`, `CTYPES`, `STATIONS`, `BRAND` | Data-driven content | — |
| Game state | `S`, `newState()`, `freshToday()` | Nguồn dữ liệu duy nhất | Data |
| Game manager / loop | `frame()`, `stepSim()`, `openShop/endDay/nextDay` | Thời gian, phase, gọi các hệ thống | Mọi hệ thống |
| Customer system | `spawnCustomer`, `updCustomer`, `seatedTick`, `maybeOrder`, `checkout`, `leave` | AI, nhu cầu, di chuyển | State, Data, Economy |
| PC system | `assignPC`, `checkout`, `seatOf` | Trạng thái máy, giá giờ | State |
| Food system | `startCook`, `stationTick`, `pickUp`, `takeFridge`, `deliverTo`, `pendingNeeds` | Nấu, tay cầm, giao | Inventory, Economy |
| Inventory | `buyStock`, `hasIng`, `consume` | Tồn kho | Economy |
| Economy | `addTx`, `recharge`, `endDay` | Tiền, giao dịch, báo cáo | State |
| Save | `serialize`, `deserialize`, `saveGame`, `loadGame` | localStorage | State, `UI.noSave` |
| FX hooks | `fx`, `SFX`, `SOUND_FILES` | Toast, số tiền bay, âm thanh; tắt khi `TEST` | DOM |
| Render | `buildWorld`, `render*`, `bubbleOf`, `fit` | Bản đồ DOM + SVG, cập nhật theo key thay đổi | State |
| Art | `charSVG`, `icon`, `pcDeskSVG`, `chairSVG`, `SVG_ST`… | Asset SVG nguyên bản | — |
| UI sheets | `BUILD`, `openSheet`, `renderSheet`, `ACT` | Bottom sheet, hành động | Logic functions |
| Input | `bind()`, `onWorld`, phím tắt | Click/tap/bàn phím | UI, logic |
| Self-test | `runTests`, `simulateFullDay`, `botTick`, `checkInv` | Kiểm thử logic trên state tạm | Logic (không DOM) |

**Nguyên tắc đang giữ:** logic game không đụng DOM. Mọi side-effect UI đi qua `fx` (bị tắt khi `TEST=true`).

## 12. Important Files

| File | Purpose | Status |
|---|---|---|
| `index.html` | Toàn bộ game | Code đủ Phase 1, runtime ⚠️, kiểm tra BUG-001 |
| `PROJECT_STATUS.md` | Trạng thái project (file này) | Mới tạo |
| Specification (đang nằm trong chat) | Yêu cầu gốc | ⚠️ Chưa xác nhận có trong repo. Nên lưu thành `SPEC.md` |

## 13. Known Bugs

### BUG-001 — Đoạn code 1 có lời dẫn bị lẫn vào khối code
**Severity:** Critical (nếu bị dán vào file)
**Status:** Open — Needs verification
**Description:** Khi gửi đoạn 1, khối code không được đóng đúng chỗ. Lời dẫn tiếng Việt và dòng ` ```js ` bị dính vào cuối khối code. Nếu dán nguyên khối, JavaScript bị lỗi cú pháp → game trắng màn hình.
**Reproduction:** Trong `index.html`, tìm dòng chứa `const slots=S.stations[st];if(!slots)return{ok:false,err:'err_generic'};`. Nếu ngay sau `};` có chữ "Mình chia phần còn lại…", tức là đã dính lỗi.
**Fix:** Xóa từ chữ "Mình chia…" đến hết dòng ` ```js ` (bao gồm cả dòng đó). Dòng code tiếp theo phải là `const i=slots.findIndex(s=>s&&s.done);…`.
**Affected files:** `index.html`

### BUG-002 — Có thể mất cú bấm trong bảng khách/đơn/PC
**Severity:** Medium
**Status:** Open — Needs verification
**Description:** `frame()` render lại bảng mỗi 1 giây nếu nội dung đổi. Bảng khách có thời gian chơi đổi liên tục → nút bị thay mới đúng lúc người chơi bấm → cú bấm có thể bị mất.
**Reproduction:** Mở bảng của khách đang chơi, bấm nút nạp nhiều lần.
**Affected:** `frame()`, `renderSheet()`, `BUILD.cust/orders/pc/station`

### BUG-003 — Mở bảng quầy làm đóng băng kiên nhẫn của khách
**Severity:** Low
**Status:** Open
**Description:** Khi bảng quầy đang mở cho khách nào (`SERVING`), khách đó không mất kiên nhẫn. Mở bảng rồi để đó thì khách không bao giờ bỏ về.
**Affected:** `updCustomer` (case WAITING), `openSheet`

### BUG-004 — Đổi version save sẽ xóa save cũ âm thầm
**Severity:** Low (tiềm ẩn, sẽ thành High khi đổi `CFG.VERSION`)
**Status:** Open
**Description:** `deserialize` throw khi version khác → `loadGame` trả `null` → game mới → autosave ghi đè save cũ. Không có migration, không báo người chơi.
**Affected:** `deserialize`, `loadGame`, `init`

### BUG-005 — Khách đang xếp hàng lúc đóng cửa vẫn bị tính điểm
**Severity:** Low
**Status:** Open
**Description:** `leave()` gọi `rateVisit()` cho khách chưa được phục vụ lúc 24:00 → kéo rating về khoảng 3.8 sao dù không phải lỗi người chơi.
**Affected:** `sendHome`, `leave`, `rateVisit`

### BUG-006 — Khách đang đi tới máy lúc đóng cửa bị tính "đã ngồi máy"
**Severity:** Low
**Status:** Open
**Description:** `checkout()` gọi từ trạng thái `TO_PC` vẫn tăng `S.today.served`.
**Affected:** `checkout`, `sendHome`

### BUG-007 — Bỏ món trên tay không cần xác nhận
**Severity:** Low (UX)
**Status:** Open
**Description:** Một chạm vào món trên tay là mất món, nguyên liệu đã bị trừ.
**Affected:** `bind()` (trayItems), `discardTray`

### BUG-008 — Nút "Số khác" dùng `prompt()`
**Severity:** Low
**Status:** Open — Needs verification
**Description:** Một số trình duyệt nhúng (Zalo, Facebook) chặn `prompt()` → nút không có tác dụng.
**Affected:** `ACT.rcx`

## 14. Known Technical Debt

- **Một file duy nhất (~1.400 dòng):** lệch với kiến trúc React/TS/Vite trong spec. Khó review, dễ lỗi khi ghép (xem BUG-001).
- **Hardcoded values:** tọa độ bản đồ (`STATIONS`, `PC_LAYOUT`, `queueSpot`, `CFG.AISLE_X`), tỉ lệ đói/khát, kiên nhẫn 50/45/40, thưởng/phạt hài lòng nằm rải trong hàm, chưa gom vào data.
- **Localization chưa sạch:** xem mục 4.
- **Doanh thu ghi khi nạp:** xem mục 7, cần chốt mô hình kế toán trước khi làm analytics.
- **Không có migration save.**
- **Không có pathfinding:** nhân vật đi xuyên đồ vật.
- **Render DOM:** đủ cho ~15 khách. Chưa có object pooling. Chưa đo FPS trên điện thoại yếu.
- **Logic đọc biến UI:** `SERVING` là biến UI nhưng ảnh hưởng logic kiên nhẫn.
- **Tiền có thể âm,** không có xử lý nào.
- **Self-test dựa trên ngẫu nhiên:** bot test phụ thuộc `Math.random`, kết quả mỗi lần khác nhau. Chưa có seed.
- **Spec lệch nhẹ:** spec Level 1 ghi 6 PC, mục Economy ghi 8 PC. Code đang dùng 8.

## 15. Current Blockers

### BLOCKER-001
**Problem:** Phase 1 chưa được xác minh runtime.
**Why blocked:** Không thể đánh dấu `Completed` cho bất kỳ feature nào khi chưa chạy. Không môi trường nào trong quá trình phát triển chạy được trình duyệt.
**Required action (chủ project):**
1. Kiểm tra và sửa BUG-001 trong `index.html`.
2. Mở `…/tiemnet/?test` và chụp kết quả.
3. Chơi tay trọn 1 ngày (khuyến nghị tốc độ 4x).
4. Mở DevTools → Console, ghi lại lỗi đỏ nếu có.
5. Gửi kết quả để cập nhật file này.

## 16. Last Completed Work

### 2026-10-08
- Viết code Phase 1 trong một file `index.html` (gửi thành 3 đoạn): game loop, bản đồ SVG, 8 PC, 5 loại khách, AI, nạp tiền, 9 món, kho, báo cáo ngày, save/load, self-test.
- Đọc lại toàn bộ code để lập file này. Phát hiện BUG-001 đến BUG-008.
- Tạo `PROJECT_STATUS.md`.
- **Chưa có:** chạy runtime, chạy self-test, sửa bug.

## 17. Next Recommended Tasks

*(Chỉ đề xuất, chưa thực hiện. Chờ chỉ đạo.)*

1. **Critical:** Xác minh và sửa BUG-001. Chạy self-test + chơi tay (BLOCKER-001).
2. **Bug từ kết quả test:** sửa mọi ❌ và lỗi console phát sinh.
3. **BUG-002:** chỉ cập nhật phần chữ thay đổi trong bảng, không thay cả nút.
4. **Chốt quyết định:** roadmap phase nào là chính thức (mục 2), mô hình doanh thu (mục 7).
5. **Nền cho phase sau:** migration save (BUG-004), tách hằng số AI vào data, xử lý tiền âm.
6. **UX nhỏ:** BUG-003, BUG-005, BUG-006, BUG-007, BUG-008.
7. **Polish:** chỉ làm sau khi Phase 1 được xác nhận `Completed`.

## 18. Verification Checklist

Ghi chú: `(auto)` = có bài tự kiểm thử tương ứng; `(manual)` = cần chơi tay.

### Core Game
- [ ] Game starts (manual)
- [ ] Player can move (manual)
- [ ] Customer can enter (auto 1, 1b)
- [ ] Customer can use PC (auto 2a–2d)
- [ ] Customer can pay (auto 5, 7)
- [ ] Money updates correctly (auto 7, 9d)
- [ ] Day can end (auto 9a)
- [ ] Save works (auto 10a, 10b)
- [ ] Load works (manual: reload giữa ngày, kiểm tra khách và tiền còn nguyên)

### Food
- [ ] Customer can order food (auto 3)
- [ ] Player can prepare food (auto 4 + manual)
- [ ] Inventory decreases (auto 4)
- [ ] Customer receives food (auto 5)
- [ ] Food transaction works (auto 5, 5b)

### Economy
- [ ] Revenue calculated (auto 9b)
- [ ] Expenses calculated (auto 9c)
- [ ] Profit calculated (auto 9c)
- [ ] Inventory purchasing works (auto + manual)

### Stability
- [ ] No critical console errors (manual)
- [ ] No duplicate transactions (auto 5b, 9e)
- [ ] No obvious dead-end (auto bot + manual)
- [ ] Save/load does not corrupt progression (manual qua nhiều ngày)

---

## Quy tắc duy trì file này
1. Đọc file này và code thực tế trước mỗi phiên làm việc.
2. Code thực tế mâu thuẫn với file → tin code, cập nhật file.
3. Spec mâu thuẫn với code → giải thích chênh lệch trước, không âm thầm đổi hành vi.
4. Chỉ đánh dấu `Completed` sau khi đã xác minh runtime.
5. Mỗi thay đổi: ghi file bị sửa, cập nhật mục 16 và 17.
6. Không tự bắt đầu phase mới khi chưa có chỉ đạo.
