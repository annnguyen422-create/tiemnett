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

## 1. Current Phase

- **Current Phase:** Phase 2 — Food Preparation
- **Phase Status:** Needs Verification
- **Last Updated:** 2026-10-08
- **Overall Progress:** ~35% toàn bộ spec (ước lượng thô)
- **Current Development Focus:** Xác minh runtime Phase 1 + Phase 2 qua `?test` (57 bài) và chơi tay. Không làm feature mới.

## 2. Phase Progress

| Phase | Status | Progress | Notes |
|---|---|---:|---|
| Phase 1 — Core Game Loop | Needs Verification | ~90% (code) | Chưa có kết quả runtime |
| Phase 2 — Food Preparation | Needs Verification | ~85% (code) | Thao tác từng bước trên map, chất lượng món, save v2. Chưa chạy runtime |
| Phase 3 — Vietnamese Atmosphere | Not Started | ~30% (nền) | |
| Phase 4 — Gameplay & Economy | Not Started | ~20% (nền) | |
| Phase 5 — QA & Polish | Not Started | ~15% (nền) | Self-test mở rộng 57 bài + bot làm món từng bước |

## 3. Implemented Features → thay mục "### Food" bằng:

### Food (Phase 2) — Code: ✅ · Runtime: ⚠️
- *Location:* `index.html`, khối `PHASE 2 — FOOD PREPARATION` (dán trước `init();`), các hàm `kBin`, `kTap`, `kTrash`, `kDiscard`, `stationTick`, `cookState`, `deliverTo`, `renderKitchen`, `renderKBar`
- *Trạm trên map:* Kho, Thùng mì, Tủ đông, Bàn sơ chế (3 chỗ + 5 hũ), Bếp (2), Chảo (2), Quầy pha nước (2 ly + 7 hũ), Tủ lạnh, Thùng rác
- *Mì (4 loại):* lấy gói → đặt → bóc → cho vào tô → topping (trứng/xúc xích/cá viên/rau, tối đa 3) → nêm → bưng → bắc bếp → vớt. Chín 6′, vùng ngon 6′, hỏng ở 20′ (phút game)
- *Đồ chiên (5 loại):* lấy → thả chảo → vớt → bày đĩa → rưới tương → khay. Chín 5′, vùng ngon 4′, cháy ở 15′
- *Pha chế (3 món):* ly → đá → trà/cà phê → chanh/đào/sữa → lắc 2′ → khay. Sai công thức = hỏng
- *Nước chai (4 loại):* tủ lạnh → khay
- *Chất lượng:* chưa chín 60/55, quá lửa 70, không đá 80, thiếu mỗi topping −20 → ảnh hưởng hài lòng (+8/+4/0/−8)
- *Tay 1 món, khay 4 món; bấm món trên khay 2 lần mới bỏ; tạm dừng thì không thao tác được*
- *Bàn thao tác* (thanh dưới màn hình) hiện nút to của trạm đang đứng
- *Known limitation:* đá không giới hạn; khách không yêu cầu "ít đá/không đá"; giá không giảm khi món kém

## 4. Partially Implemented → XÓA mục "Food preparation flow" (đã làm). Thêm:

### Phase 1 code bị thay thế nhưng vẫn còn trong file
**Status:** ⚠️ Partial (technical debt)
**Implemented:** Phase 2 ghi đè bằng các function cùng tên.
**Missing:** Code cũ (`startCook`, `pickUp`, `takeFridge`, `renderStations`, `interactStation`, `BUILD.station`, `ACT.cook/take`, `STATIONS`, `SVG_ST`, `menuBoardSVG`) không còn được gọi nhưng chưa xóa.
**Next step:** Dọn dẹp khi gộp file hoặc chuyển sang React/TS.

## 8. Inventory Status — thay bảng bằng:

| Nhóm | Item | Purchase | Consume | Stock |
|---|---|---|---|---|
| Mì | mi, mi_bo, mi_ga, mi_cay | ✅ | Thùng mì | ✅ |
| Topping | trung, rau (+ xucxich, cavien) | ✅ | Bàn sơ chế | ✅ |
| Đồ chiên | cavien, bovien, tomvien, xucxich, holo | ✅ | Tủ đông | ✅ |
| Nước chai | energy, cola, water, soda | ✅ | Tủ lạnh | ✅ |
| Pha chế | tra, caphe, chanh, dao, sua | ✅ | Quầy pha nước | ✅ |
| Bao bì | to, dia, ly, tuong | ✅ | theo bước | ✅ |
Tất cả: Code ✅ · Runtime ⚠️. Đá không có trong kho (không giới hạn).

## 10. Save / Load — thêm:
- **Save version:** v2. Save v1 tự chuyển (`migrateV1`): đổi món cũ (Mì trứng → Mì tôm + trứng…), giữ kho, tặng số lượng khởi điểm cho nguyên liệu mới. Món đang nấu dở trong save v1 bị bỏ.
- Save lỗi/phiên bản lạ → sao lưu sang `tiemnet_save_v1_backup_<thời gian>` thay vì ghi đè.
- Lưu thêm: `k` (bếp), `hand`, `sel`, khay dạng món có chất lượng.

## 13. Known Bugs — cập nhật trạng thái:
- BUG-002 → **Fixed (needs verification)**: không làm mới bảng khi đang chạm.
- BUG-004 → **Fixed (needs verification)**: có migration + sao lưu.
- BUG-007 → **Fixed (needs verification)**: bỏ món cần bấm 2 lần.
- Thêm **BUG-009 — Hũ nguyên liệu nhỏ trên điện thoại** · Low · Open: hũ trên map chỉ khoảng 17–22px khi thu nhỏ; đã có thanh bàn thao tác thay thế.
- Thêm **BUG-010 — Khách có thể trả thiếu khi tiền mặt không đủ** · Low · Open: `deliverTo` ghi đủ doanh thu nhưng chỉ trừ tiền khách tới 0 (hiếm, do `availableCash` cũ còn được dùng ở chỗ khác).

## 14. Known Technical Debt — thêm:
- Phase 2 dùng cơ chế ghi đè function, nên file có code chết của Phase 1.
- Hằng số thời gian nấu (`COOK`), chất lượng (`QUAL`), giá topping (`TOPPINGS`) chưa gom chung với data Phase 1.

## 16. Last Completed Work — thêm:
### 2026-10-08 (Phase 2)
- Hệ thống làm món từng bước: mì, đồ chiên, pha chế, nước chai; chất lượng theo thời điểm vớt; thùng rác; bàn thao tác.
- Save v2 + migration v1. Sửa BUG-002, BUG-004, BUG-007.
- Self-test mở rộng lên 57 bài, bot tự làm món từng bước.
- File đổi: `index.html` (thêm CSS trước `</style>`, thêm JS trước `init();`).
- **Chưa có:** kết quả runtime.

## 17. Next Recommended Tasks
1. Chạy `?test`, gửi kết quả; sửa mọi ❌.
2. Chơi tay 1 ngày ở 1x, xem nhịp độ làm món có quá dồn không (đơn chờ 70′ game ≈ 35 giây thật).
3. Chốt mô hình doanh thu (mục 7) và roadmap phase chính thức.
4. Dọn code chết của Phase 1.

## 1. Current Phase
- **Current Phase:** Phase 4 — Gameplay & Economy (một phần: nhân viên, nâng cấp, hạn dùng, giá) + cải thiện UX Phase 2
- **Phase Status:** Needs Verification
- **Last Updated:** 2026-10-08
- **Overall Progress:** ~45% toàn bộ spec (ước lượng thô)
- **Current Development Focus:** Chạy `?test` (98 bài) và gửi kết quả. Không làm feature mới.
- **Runtime signal:** Chủ project đã chơi tay bản Phase 2 và gửi góp ý gameplay, nghĩa là game chạy được và nấu ăn từng bước hoạt động. Chưa có kết quả self-test.

## 2. Phase Progress
| Phase | Status | Progress | Notes |
|---|---|---:|---|
| Phase 1 — Core Game Loop | Needs Verification | ~90% | Đã chơi tay; chưa có kết quả test tự động |
| Phase 2 — Food Preparation | Needs Verification | ~95% | Đã chơi tay; đã sửa theo góp ý (đổi nguyên liệu, bảng thao tác, thanh chiên to) |
| Phase 3 — Vietnamese Atmosphere | Not Started | ~30% (nền) | |
| Phase 4 — Gameplay & Economy | In Progress | ~45% | Đã có: nhân viên, 9 nâng cấp, hạn dùng theo lô, đổi giá. Chưa có: internet, máy hỏng/sửa, dọn bàn, sự kiện, mở rộng tiệm, vay vốn |
| Phase 5 — QA & Polish | In Progress | ~25% | 98 bài self-test + 2 bot (tự chơi, đầu bếp ảo) |

## 3. Implemented Features — thêm:
### Staff — Code ✅ · Runtime ⚠️
- *Location:* khối `PHASE 3`: `STAFF`, `hireStaff`, `fireStaff`, `staffTick`, `cookStep`, `cookAct`, `cashierStep`, `BUILD.staff`
- Thu ngân (phí 400k, lương 150k/ngày, 1 việc mỗi 2 phút game): nạp tiền khi khách xin, xếp máy đúng loại.
- Đầu bếp (phí 700k, lương 200k/ngày, 1 thao tác mỗi phút game): tự làm mì có topping, chiên, pha, nước chai theo đơn → khay. Dùng chung bếp, có tay riêng, không đụng món người chơi đang làm trên Bàn sơ chế.
- Chỉ 1 người cùng lúc. Lương tính cho mọi người đã làm trong ngày, kể cả khi cho nghỉ giữa ngày.
### Upgrades — Code ✅ · Runtime ⚠️
- *Location:* `UPGRADES`, `buyUpgrade`, `applyUpgrades`, `BUILD.upg`
- Bếp: bếp hẹn giờ, chảo tự ngắt (giữ ở vùng ngon, không hỏng), bếp từ (6→4′), chảo lớn (5→3,5′), máy lắc (2→1′), khay 6 món.
- Phòng máy: màn hình 144Hz (+vui, +2.000đ/giờ), ghế (+20% thời gian chơi, +vui), máy lạnh (+30%/+15% kiên nhẫn, +20k điện/ngày).
### Expiry — Code ✅ · Runtime ⚠️
- *Location:* `SHELF`, `syncLots`, `takeStock`, `returnStock`, `expireLots`, `buyStock`
- Áp dụng cho cá viên (3 ngày), bò viên (3), tôm viên (2), xúc xích (4), hồ lô (3). Lô cũ dùng trước. Hết hạn tự bỏ đầu ngày mới, hiện ở bảng chuẩn bị.
### Menu pricing — Code ✅ · Runtime ⚠️
- *Location:* `priceOf`, `setPrice`, `recipeCost`, `BUILD.orders`. Bước 1.000đ, giới hạn 50–200% giá gốc. Giá cao hơn gốc thì khách có xác suất bỏ món đó.
### Tutorial & UX — Code ✅ · Runtime ⚠️
- Thẻ hướng dẫn lần đầu cho mì, chiên, pha (`chainTut`). Viền xanh chỉ trạm tiếp theo (`coachTarget`, `renderCoach`). Bật/tắt và xem lại trong Cài đặt.
- Đổi hoặc trả nguyên liệu đang cầm (`kBin` mới). Bảng thao tác có nút ✕, bấm ra ngoài để đóng, hiện "Cần làm" theo đơn và số đếm trên hũ. Thanh tiến độ chiên/nấu to hơn và có thêm trong bảng.
- Kiên nhẫn: chờ quầy ×1,6, chờ món ×1,5 (`CFG.PAT_WAIT`, `CFG.PAT_ORDER`).

## 7. Economy — thêm:
- Chi phí mới: lương (`STAFF_SALARY`), phí tuyển (`STAFF_HIRE`), nâng cấp (`UPGRADE`), máy lạnh +20k điện.
- Báo cáo tính phí tuyển và nâng cấp vào chi phí của ngày mua, nên lợi nhuận ngày đó sẽ thấp. Cần chốt có nên tách "đầu tư" khỏi lợi nhuận vận hành hay không.

## 13. Known Bugs — thêm:
- **BUG-011 — Đầu bếp có thể đứng yên** · Low · Open: nếu cả 3 ô Bàn sơ chế bị món của người chơi chiếm, đầu bếp không làm mì hoặc đồ chiên được.
- **BUG-012 — Vạch vùng ngon trên bản đồ chưa đổi ngay sau khi mua bếp/chảo nhanh** · Low · Open: chỉ cập nhật khi đặt món mới vào ô.
- **BUG-013 — Thời gian chờ nạp tiền chưa được nới** · Low · Open: `seatedTick` (Phase 1) vẫn dùng 40–45′.
- **BUG-010** — giữ nguyên Open.

## 14. Known Technical Debt — thêm:
- File có 3 lớp ghi đè (Phase 1 → 2 → 3), nhiều code chết. Nên gộp lại thành một bản sạch trước khi làm tiếp.
- Đầu bếp đánh dấu món bằng tiền tố `ws:'c:…'`; bot test dùng `ws` không có tiền tố.
- `applyUpgrades()` sửa trực tiếp `COOK` và `CFG.TRAY_MAX` (giá trị toàn cục).

## 16. Last Completed Work — thêm:
### 2026-10-08 (theo góp ý chơi thử)
- Nhân viên (thu ngân/đầu bếp), 9 nâng cấp, hạn dùng theo lô, đổi giá thực đơn, hướng dẫn lần đầu + viền chỉ dẫn, đổi/trả nguyên liệu, đóng bảng thao tác khi bấm ra ngoài, hiện món cần làm, thanh chiên to, giảm tốc độ mất kiên nhẫn.
- Thêm 33 bài test (tổng 98). Đính chính: Phase 2 có 65 bài, không phải 57.
- File đổi: `index.html` (CSS trước `</style>`; JS phần C1 + C2 trước `init();`).

## 17. Next Recommended Tasks
1. Chạy `?test`, sửa mọi ❌.
2. **Gộp file thành một bản sạch** (bỏ code chết, bỏ cơ chế ghi đè). Sau đó các lần sửa sẽ dễ dán hơn rất nhiều.
3. Cân bằng: phí tuyển/lương so với lợi nhuận ngày; giá nâng cấp.
4. Chốt: có tách "đầu tư" khỏi lợi nhuận ngày không.
