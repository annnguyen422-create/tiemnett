# TIỆM NET — HANDOFF v5 (đọc file này TRƯỚC khi làm bất cứ việc gì)

> Cập nhật: 2026-10-09 · Thay thế HANDOFF.md cũ (đã lỗi thời).
> **Nguồn sự thật về hành vi game = `v4.html` (đã dán đủ Patch A → H2).**
> Nếu file này mâu thuẫn với code → tin code, rồi sửa file này.
> Việc đang làm dở: **gộp v4 + patch thành file sạch `v5.html`** (mới gửi Phần 1/9).

---

## 0. Quy tắc cho AI / developer

1. Đọc hết file → mở `v4.html` → đối chiếu mục 5–6 trước khi đề xuất.
2. **Không tự làm phase/feature mới.** Chủ project yêu cầu: *đưa checklist/đề xuất trước → được OK → mới code → chủ project test → mới qua bước sau.*
3. Trả lời **tiếng Việt**, ngắn gọn, từng bước. Chủ project **không phải lập trình viên**, dùng **iPhone + Mac/Chrome**, sửa code bằng **github.dev** (nhấn phím `.` ở trang repo) hoặc trình sửa web GitHub.
4. Khi gửi code:
   - Ghi rõ **chỗ dán** bằng một mốc duy nhất (vd: "ngay trước dòng `init();`").
   - **Mỗi khối code phải tự đóng ```**. Code dài → **chia nhiều tin nhắn, đánh số phần**, kết thúc mỗi phần bằng "nhắn *tiếp*".
   - Kèm **checklist kiểm tra** + số bài test mong đợi.
5. **Bộ nhớ đệm GitHub Pages ~10 phút**: luôn bảo chủ project mở bằng URL có số mới, vd `v4.html?v=12`, `v4.html?test&v=12`. Có dòng **"Bản: …"** trong ⚙️ Cài đặt để biết đã cập nhật chưa.
6. Trong trình sửa web GitHub, Ctrl/Cmd+F chỉ chạy khi **bấm vào vùng code trước**; nếu không thì ô tìm của Chrome không thấy dòng ngoài màn hình. github.dev thì luôn tìm được.
7. Chỉ đánh dấu **Completed** khi: self-test đạt **và** chủ project đã chơi tay.

---

## 1. Tổng quan

| Mục | Giá trị |
|---|---|
| Game | Tiệm Net — quản lý tiệm net Việt Nam, chibi, ấm áp |
| Repo | `annnguyen422-create/tiemnett` (**2 chữ t**) |
| Bản đang chơi | `https://annnguyen422-create.github.io/tiemnett/v4.html` |
| Self-test | thêm `?test` (vd `v4.html?test&v=12`) hoặc ⚙️ Cài đặt → Chạy kiểm thử |
| Bản cũ | `index.html` = v3 (3 lớp ghi đè, 98 test) — **chưa thay**, giữ dự phòng |
| Bản đang làm | `v5.html` — bản sạch, mới gửi **Phần 1/9** (khung + CSS + config + data) |
| Công nghệ | 1 file HTML + CSS + JS thuần, không build. Font Nunito. Toàn bộ hình là SVG tự vẽ |
| Save | `localStorage`: `tiemnet_save_v3` (v4/v5 dùng chung) · `tiemnet_save_v1` (index.html cũ) · cờ `tiemnet_v3_imported` (đã nhập save cũ 1 lần) |
| Repo khác | `chitieu` — app chi tiêu, không liên quan |

---

## 2. Trạng thái

- **v4 + Patch A→H2**: chủ project đã chơi tay và duyệt từng patch. Test mong đợi **139/139** (81 gốc + A5 + B12 + C8 + D11 + E4 + E2·5 + F5 + H4 + H2·4). Con số cuối cùng **chưa có ảnh xác nhận** → chạy lại `v4.html?test&v=…` để chốt.
- **v5**: đã gửi Phần 1/9. Chưa rõ chủ project đã dán chưa. Các phần 2→9 **chưa viết**.
- Chủ project đang ở save **ngày 7**, có/không có đầu bếp tuỳ save.

---

## 3. Gameplay hiện tại (v4 + patch) — v5 PHẢI giữ nguyên

### 3.1 Vòng lặp ngày
PREP (07:30, bảng chuẩn bị: dự báo, món mới hôm nay, nhập hàng, nhân viên, nâng cấp) → **Mở cửa** 08:00 → 23:00 ngừng nhận khách → 24:00 CLOSING (mọi khách về) → REPORT → Ngày mới (bỏ hàng hết hạn, tính lương).
1 giây thật = 2 phút game. **Không còn tốc độ 2x/4x**; chỉ có ⏸ (đổi thành ▶ khi dừng). Khi dừng **không thao tác được trên bản đồ**.

### 3.2 Mở món theo ngày + số món mỗi lần gọi
| Ngày | Món mở | Món/lần |
|---|---|---|
| 1 | Nước chai + pha chế | 1 |
| 2 | + Đồ chiên | 1 |
| 3 | + Mì | 1 |
| 4–5 | tất cả | ≤2 |
| 6+ | tất cả | ≤3 |
- Trạm chưa mở: **xám + "🔒 Ngày N"**, không bấm được (`ST_LOCK`: board/freezer/fryer → ngày 2; noodlebox/stove → ngày 3).
- Ngày ra món mới: khách **ưu tiên 60%** gọi món mới; bảng chuẩn bị có khung **"🆕 Món mới hôm nay"** (hình món + các bước + công thức pha) và dòng "Ngày mai mở thêm…".
- **Teach mode**: đơn đầu tiên của món mới trong ngày → viền gợi ý **đỏ đậm, nhấp nhanh**; tắt khi đã bán món đó 1 lần.

### 3.3 Khách & hàng chờ
- Trạng thái: `TO_QUEUE → QUEUE (→ SERVING) → TO_PC → PLAYING ⇄ WAITING_FOOD → EATING`, `RECHARGE`, `LEAVING`.
- Hàng dọc trước quầy, tối đa 6 chỗ (`QUEUE_SPOTS`); có số thứ tự, người đang được phục vụ hiện ★. Hàng đầy → khách mới về luôn ("Đông quá…").
- **Phục vụ đúng thứ tự đến**: `queueFront()` = người đầu `queueList()` **và phải đã tới nơi** (QUEUE/SERVING); nếu chưa tới thì cả hàng chờ.
- Khi xếp hàng **không có bóng chat** (chỉ số thứ tự + thanh kiên nhẫn). Yêu cầu khách xem ở bảng quầy.
- Bấm quầy / khách trong hàng → luôn mở **khách đầu hàng** (SERVING khi bảng mở; đóng bảng → về QUEUE).

### 3.4 Kiên nhẫn (đã duyệt là "ổn")
- Quầy: `80 × pat` phút game (×1,3 máy lạnh), giảm ×1/phút; SERVING không giảm.
- Chờ món: `(40 + Σ phần theo món) × hệ số số món × pat` (×1,15 máy lạnh). Phần: nước chai 15, pha chế 35, đồ chiên 50, mì 60. Hệ số: 1 món ×1, 2 món ×1,15, 3 món ×1,3.
- Tốc độ giảm: đang chơi ×0,7 · **món xong mà chưa giao ×1,6** · hết giờ/hết tiền nhưng còn chờ món ×1.
- 4 mức tâm trạng: 😊 >66% · 🙂 >40% · 😐 >15% · 😠. Hủy đơn lần 2 (hoặc vui <20) → bỏ về.
- **Khách chỉ gọi món nếu thời gian chơi còn lại ≥ Σ phần + 15** (bớt món lâu nhất cho vừa).
- **Còn món đang chờ → khách KHÔNG về** dù hết giờ chơi hoặc hết tiền máy: ngồi chờ, không tính tiền máy. Nhận xong mới về.

### 3.5 Làm món (đã duyệt)
- **Mì**: Thùng mì (chọn loại) → Bàn sơ chế (**tự** xé gói → cho vào tô → nêm, 3′, có thanh tiến độ; tốn 1 tô) → thêm topping khách gọi → bấm tô để bưng → Bếp → vớt khi vạch vào vùng xanh.
- **Đồ chiên**: Tủ đông → Chảo → vớt khi vàng giòn → Bàn sơ chế (**tự** bày đĩa + rưới tương, 2′; tốn đĩa + tương) → bấm đĩa.
- **Pha chế**: Quầy pha nước → Ly → Đá → Trà/Cà phê → Chanh/Đào/Sữa → **tự lắc** (2′) → bấm ly. Sai công thức → hỏng.
- **Nước chai**: Tủ lạnh → lên khay.
- Món xong (người chơi làm) → **khay đang bưng** (`carry`, 4 món, 6 nếu có khay lớn).
- Chất lượng: mì chưa chín 60/quá lửa 70; chiên 55/70; không đá 80; hỏng → thùng rác.
- **Topping phải ĐÚNG Y** (thiếu hoặc dư đều không nhận). Khách nói "Không đúng món em gọi"; bảng khách hiện dòng đỏ cảnh báo.
- Đang cầm gói mì/đồ chiên: bấm loại khác = **đổi**, bấm lại đúng loại = **trả về kho**.
- **Tủ lạnh**: mỗi bấm = +1 chai lên khay; thẻ có **badge xanh xN** (góc phải) + **nút "−" đỏ** (góc trái, cùng cỡ badge) để trả 1 chai về tủ (ưu tiên chai **chưa có khách nhận**).

### 3.6 Ghép món ↔ khách + giao món
- `matchAll()`: mỗi món khách gọi (theo thứ tự gọi) được ghép với món/công đoạn tiến xa nhất **đúng loại**. Món đã xong (khay/quầy, tô/nồi) phải **đúng y topping**; tô đang ở Bàn sơ chế được ghép nếu chưa dư topping (để hiện nhãn "+rau").
- Nhãn **"PC n"** trên món; khách có món sẵn sàng → **vòng xanh + mũi tên ▼**.
- Bấm khách có vòng xanh / bấm món trên khay → nhân vật **tự đi lấy (nếu ở Quầy ra món) rồi bám theo đúng khách** để giao. Khách về giữa chừng → báo, món còn trên khay và tự ghép cho khách khác cùng món.
- Bảng khách: mỗi món có trạng thái (Chưa làm / Đang sơ chế / Đang nấu / Đang bưng / Ở quầy ra món) + nút **"Giao món ➜"** khi sẵn sàng. "Báo hết hàng" chỉ hiện khi kho thật sự hết.

### 3.7 Gợi ý bước tiếp theo (Next Valid Action)
- Trạm cần bấm tiếp: **viền vàng nhấp nháy**; hũ nguyên liệu khách cần: **nảy + nền xanh** (trên bản đồ và bàn thao tác).
- Bàn thao tác (thanh dưới, hiện khi đứng ở một trạm): dòng "Cần làm: …", các thẻ, rồi **tag xanh "Tiếp theo: …"** (vd "Mang đi chiên", "Mang tô lên Bếp nấu", "Giao cho PC 5", "Bỏ vào Thùng rác").
  - Tag **không bấm được**, trừ khi đang **teach mode** (làm món mới lần đầu) → thành nút có ➜, bấm thì đi tới trạm.
  - Chưa có Quầy ra món → **không** hiện câu "đặt lên Quầy ra món".
- Tắt toàn bộ gợi ý: ⚙️ Cài đặt → "Gợi ý bước tiếp theo".
- **Đã bỏ hẳn**: thanh luồng món/chị Hai ở trên cùng; hàng tiêu đề bàn thao tác (vẫn đóng được bằng bấm ra ngoài); số đỏ "cần làm" trên thẻ; badge/hiệu ứng chọn trên các tủ **ở bản đồ**; phần "Đường đi của món" trong thực đơn.

### 3.8 Nhân viên (chỉ 1 người)
- **Thu ngân** (phí 400k, lương 150k/ngày, 1 việc/phút): gọi khách đầu hàng (SERVING) → nạp đúng số khách xin → xếp máy đúng loại; tự nạp khi khách đang chơi xin.
- **Đầu bếp** (phí 700k, lương 200k/ngày, 1 thao tác/0,5 phút): tự làm trọn món (cả topping), không đụng món người chơi đang làm. Món xong: **có Quầy ra món → đặt lên quầy; chưa có → đặt lên khay của người chơi**; khay đầy → đứng chờ.
- Lương tính cho mọi người đã làm trong ngày (kể cả cho nghỉ giữa ngày).

### 3.9 Nâng cấp (mua 1 lần)
Bếp hẹn giờ 900k · Chảo tự ngắt 1,2tr · Bếp từ (mì 6→4′) 700k · Chảo lớn (5→3,5′) 800k · Máy lắc (2→1′) 400k · Khay lớn (6 món) 500k · **Quầy ra món 800k (🔒 cần đang có đầu bếp)** · Màn hình 144Hz 2,4tr (+vui, +2k/giờ) · Ghế gaming 1,6tr (+20% giờ chơi, +vui) · Máy lạnh 1,5tr (+kiên nhẫn, +20k điện).
- Quầy ra món: **ẩn trên bản đồ cho tới khi mua**. Save cũ đang có đầu bếp → **tặng sẵn** (chạy 1 lần, cờ `S.verC`). Save cũ không có đầu bếp mà quầy còn món → chuyển món về khay.

### 3.10 Kho, hạn dùng, giá
- Đồ chiên quản lý **theo lô** (cá viên 3 ngày, bò viên 3, tôm viên 2, xúc xích 4, hồ lô 3), lô cũ dùng trước, hết hạn tự bỏ đầu ngày (báo ở bảng chuẩn bị).
- Đổi giá trong 🍜 Đồ ăn: −/+ 1.000đ, giới hạn 50–200% giá gốc; hiện vốn + % lãi; giá cao hơn gốc → khách có xác suất bỏ món (`min(0,9; (tỉ lệ−1)×1,2)`).

### 3.11 Giao diện
- **Thanh trên luôn 1 hàng**: `☀️ N7 · 11:00 | 💰 6,63tr | ⭐ 3.8 | 🟢 Mở | ⏸` (màn hẹp <600px dùng dạng rút gọn; màn rộng dùng chữ đầy đủ).
- Thanh dưới 7 nút: Máy · Đồ ăn · Kho · Doanh thu · Nhân viên · Nâng cấp · Cài đặt.
- Góc dưới trái: "Tay: … | Khay n/4: …" (ẩn khi bàn thao tác mở).
- Bản đồ 960×600: kho/bếp hàng trên; quầy thu ngân + làn "🧍 XẾP HÀNG" bên trái; 8 máy giữa; Quầy ra món bên phải (khi đã mua).

---

## 4. Số liệu cân bằng

| | |
|---|---|
| Tiền đầu | 5.000.000đ |
| Giá giờ | Thường 8k · Gaming 12k · VIP 18k (+2k màn hình 144Hz). 8 máy: 3 Gaming + 1 VIP hàng trên, 4 Thường hàng dưới |
| Chi phí ngày | Điện 40k + 2.500đ/giờ máy (+20k máy lạnh) · Internet 50k · Bảo trì 30k · Lương nhân viên |
| Nấu | Mì chín 6′ (4′ bếp từ), vùng ngon 6′, hỏng ở +14′ · Chiên 5′ (3,5′), vùng ngon 4′, cháy ở +10′ · Sơ chế mì 3′, đồ chiên 2′ · Lắc 2′ (1′) |
| Giá món | Mì tôm 12k · Mì bò/gà 14k · Mì cay 15k · Cá viên 18k · Bò viên 20k · Tôm viên 22k · Xúc xích chiên 12k · Hồ lô 14k · Trà chanh 8k · Trà đào/Cà phê sữa 12k · Nước chai 6–13k |
| Topping | Trứng +5k · Xúc xích +8k · Cá viên +12k · Rau +2k |
| Hài lòng khi giao | q≥90 +8 · ≥70 +4 · ≥50 0 · <50 −8 · giao nhanh +4 |
| Thương hiệu hư cấu | `BRAND`: Sấm Sét, Cola Mát, Suối Mát, Chanh Sủi |

---

## 5. Kiến trúc v4.html (bản đang chạy)

**Cấu trúc file**: Phần 1 (HTML+CSS+config+data) → 2 (STR/DIALOGUE) → 3 (state, fx, economy, save) → 4 (khách, ngày) → 5 (bếp, matching, giao món, AI nhân viên) → 6 (art SVG, buildWorld) → 7 (render) → 8 (input, sheets, ACT, bind, frame, init) → 9 (runTests 81 bài, bot) → **Patch A…H2** → `init();`

**Cơ chế patch**: khai báo lại `function` cùng tên (bản cuối theo thứ tự trong file thắng) **hoặc** gán lại lúc chạy (`maybeOrder=function…`, `renderKBar=…`, `syncNext=…`, thắng bất kể vị trí). Test bổ sung đăng ký vào `EXTRA_TESTS` (`try{push}catch{setTimeout(push)}`), chạy bằng `runAllTests()`.

| Patch | Nội dung | Ghi đè / thêm |
|---|---|---|
| A | Thanh trên 1 hàng, bỏ tốc độ & ô số khách, ẩn thanh luồng món & tiêu đề bàn thao tác (CSS), không bóng chat khi xếp hàng | `renderTop`, `bubbleOf`, `init`, `EXTRA_TESTS`, `runAllTests` |
| B | Kiên nhẫn nhiều món, gọi món theo giờ còn lại, không về khi còn chờ món, topping đúng y | `orderPatienceFor`, `maybeOrder`, `seatedTick`, `matchAll`, `deliverTo`, bọc `BUILD.cust` |
| B-fix | Sửa tay 2 dòng: `queueFront` (người đầu hàng phải đã tới) · `orderReady` dùng `refreshMatch()` | sửa trực tiếp Phần 4/5 |
| C | Quầy ra món thành nâng cấp (cần đầu bếp), đầu bếp → khay khi chưa có quầy, tặng quầy cho save cũ | `applyUpgrades` (+class `nopu`), `canFinish`, `finishDish`, `buyUpgrade`, `BUILD.upg`, `hasPickup`; sửa test Phần 9 (`S.upg.pickup=true;` trước `const sc=seat();`) |
| D | Mở món theo ngày, số món/lần, khóa trạm, món mới hôm nay, teach mode | gán lại `maybeOrder`; `updateLocks`, `teachActive` (setInterval); bọc `BUILD.prep`; sửa test Phần 9 (`S.day=3;` trong `simulateDay`) |
| E | Bỏ số "cần làm" trên thẻ; x1 khi chọn; thực đơn bỏ "Đường đi" | gán lại `renderKBar`; `BUILD.orders` |
| E2 | Badge xN + viền chọn trên tủ ở bản đồ (sau đó bị G ẩn) | `binQty`, `updateBinQty` (setInterval) |
| F v2 | Nút "−" trả chai | `returnBottle`, `syncFridgeMinus` (MutationObserver + setInterval) |
| G v2 / G3 | Badge xanh, "−" đỏ cùng cỡ badge, ẩn badge tủ trên bản đồ, nhãn "Bản: …" | chỉ CSS + bọc `BUILD.set` |
| H / H2 | Tag "Tiếp theo" dưới hàng thẻ; chỉ bấm được ở teach mode; bỏ "đặt lên Quầy" khi chưa có quầy | `syncNext` (observer+interval), `nextActionTag`, `STR.nx_*` |

---

## 6. Kế hoạch v5 (bản sạch) — đang làm

**Mục tiêu**: 1 file `v5.html`, **mỗi hàm 1 bản**, không patch/observer/interval vá, không `!important` chồng; hành vi **y hệt mục 3**; giữ save (`tiemnet_save_v3`, VERSION 3); dòng "Bản: v5" trong Cài đặt. Sau khi đạt test → đổi `v5.html` thành `index.html`, giữ `v4.html` dự phòng.

**Đã gửi — Phần 1/9** (khung + CSS gộp + config + data):
- HTML: `#top` gồm `#cDay #cMoney #cRate #bStatus #bPause` (không còn `#speeds`, `#cPeople`, `#hint`); `#vp>#wrap>#world`; `#bar` 7 nút có `data-t`; `#carry`, `#kbar`, `#sheetBg`, `#sheet`, `#toasts`.
- CSS đã gộp mọi patch; class dùng tiếp: `.lockst .lockb` (khóa trạm, `.lockb` chỉ hiện khi cha có `.lockst`) · `.nopu` (ẩn quầy) · `.teach` · `.kbs .qty` (badge xanh) · `.kbs .minus` (nút "−" đỏ) · `.knext .tg(.btn)` · `.ndcard .ndic` · `.oitem`.
- Config/data mới: `CFG.BUILD='v5'`; `PAT.multi=[1,1,1.15,1.3]`, `PAT.drainWait=1`; `UNLOCK`, `ST_LOCK`, `maxItemsOn(d)`; `UPGRADES` có `{id:'pickup',price:800000,req:'cook'}`; `CABINETS`; tiện ích `budgetOf(it)`. Đã bỏ `ROUTE`.

**Còn phải viết** (mỗi phần dán nối tiếp cuối file):
| Phần | Nội dung (gộp từ v4 + patch) |
|---|---|
| 2 | `STR.vi` gộp + `DIALOGUE`. Bỏ key không dùng: `rt_* fl_* h_* guide_* nx_park nx_pickup btn_*?` (giữ `btn_deliver`, `btn_oos`). Thêm: `sts_*`, `upg_pickup…`, `err_need_cook`, `err_wrong_top`, `err_nothing_return`, `kb_return_one`, `nd_*`, `lock_day`, `nx_prefix`, `toast_pickup_gift`, `wrong_top`, `pricey`… |
| 3 | state (`newState` có `verC:1`), fx, economy (`recharge`, `buyStock`, lô hàng, `setPrice`, `hireStaff/fireStaff`, `buyUpgrade` có `req`), `returnBottle`, save + `migrateV1/V2` + **migrateC** (tặng quầy) trong `deserialize`, `applyUpgrades` **thuần logic** |
| 4 | khách: `queueFront` đã sửa, `spawnCustomer`, `maybeOrder` (bản D: mở theo ngày + ưu tiên món mới + giới hạn số món + cắt theo giờ còn lại), `orderPatienceFor` (bản B), `seatedTick` (bản B), checkout/leave, ngày (`endDay`, `nextDay` có `expireLots`) |
| 5 | bếp `kBin/kTap/…`, `canFinish/finishDish` (bản C), `matchAll` (bản B), `orderReady` dùng `refreshMatch()`, `deliverTo` (bản B), `cookStep` dùng `canFinish()` thay vì "quầy còn chỗ", `cashierStep` |
| 6 | art SVG + `buildWorld` (gắn sẵn `.lockb` vào trạm khóa được; không tạo badge trên tủ bản đồ) |
| 7 | render: `renderTop` (bản A), `bubbleOf` (bản A), `renderKitchen`, `renderCustomers`, `nextAction` + `nextActionTag` (bản H2), `renderHL`, `renderCarry`, `renderKBar` (gộp E+F+G3+H2: badge xN, nút "−", tag "Tiếp theo" **vẽ trực tiếp**, không observer), `renderMeta` (class `nopu`, `teach`, khóa trạm) |
| 8 | input (`onWorld`, `startDelivery`, kbar xử lý `ret:` và `nx:`), `BUILD.*` (orders không có "Đường đi"; prep có khung món mới; upg có khóa; set có "Bản: v5"), `ACT`, `bind`, `frame`, `init` (không còn `EXTRA_TESTS`) |
| 9 | **một** `runTests()` gộp ≈134 bài (81 gốc đã sửa + A/B/C/D/E/F/H/H2; bỏ 5 bài E2 vì đã bỏ badge trên bản đồ), bot (`simulateDay` bắt đầu ngày 3), `init();</script></body></html>` |

---

## 7. Nhật ký quyết định của chủ project

- Kiên nhẫn & luồng nấu v4: **ổn**, giữ nguyên.
- Topping: **đúng y**.
- Đầu bếp chưa có quầy: **như bản cũ** → món lên khay người chơi.
- Ngày 1: **cả nước chai lẫn pha chế**. Ngày 2 đồ chiên, ngày 3 mì.
- Số món/lần theo bảng mục 3.2; đơn 3 món kiên nhẫn chậm hơn.
- Nút trả chai: **phương án A** (nút "−" trên thẻ), **không** làm "trả từ khay".
- "−" **đỏ**, badge **xanh**, **cùng kích thước**, nằm 2 góc trên.
- Tủ trên bản đồ: **không badge, không hiệu ứng chọn**, chỉ hình chai/gói/xiên. (Gợi ý nảy của Next Valid Action **vẫn giữ** — chưa có yêu cầu bỏ.)
- Tag "Tiếp theo": **chỉ là tag**, chỉ bấm được khi làm món mới lần đầu.
- Cách giao code: **mỗi phase 1 khối dán**, rồi gộp thành bản sạch.

### Còn chờ chốt (từ trước)
- Roadmap phase chính thức (PROJECT_STATUS vs spec gốc).
- Doanh thu ghi khi nạp (số dư chưa dùng vẫn tính lợi nhuận) — có đổi không.
- Phí tuyển/nâng cấp tính vào chi phí ngày mua — có tách "đầu tư" không.

---

## 8. Lỗi & nợ kỹ thuật còn lại (sẽ xử lý trong/ sau v5)
- BUG-003: bảng quầy đang mở → khách đầu hàng không giảm kiên nhẫn (cố ý, nhưng có thể lạm dụng).
- BUG-008: nút "Số khác" dùng `prompt()` — có thể bị chặn trong trình duyệt nhúng (Zalo/Facebook).
- BUG-010: khách thiếu tiền mặt vẫn ghi đủ doanh thu món.
- BUG-012: vạch vùng ngon trên bản đồ không đổi ngay sau khi mua bếp/chảo nhanh.
- Nhân vật đi xuyên đồ vật; tiền có thể âm không hậu quả.
- Đã sửa trong v4: hàng chờ sai thứ tự, khách về khi món đang làm, topping thiếu vẫn nhận, test thiếu vốn, kiểm tra món sẵn sàng dùng dữ liệu cũ.

---

## 9. Spec gốc — chưa làm
Internet & lag · máy hỏng/sửa máy · dọn bàn/rác · nâng cấp linh kiện từng PC · sự kiện ngẫu nhiên (5 khách cùng lúc, máy đứng hình, đổ nước, xin nợ, dẫn bạn) · thời tiết/giải đấu/mùa lễ · mở rộng 12→64 máy, phòng VIP, cấp độ tiệm · khách VIP · trang trí · thành tích · analytics tuần/tháng · thanh toán QR/ví · vay vốn & game over · nhạc nền · đổi ngôn ngữ.

---

## 10. Bước tiếp theo
1. Chạy `v4.html?test&v=…` → xác nhận **139/139**.
2. Làm tiếp **v5 Phần 2 → 9** theo mục 6 (Phần 1 đã có). Đạt test → đổi thành `index.html`.
3. Cập nhật file này (mục 2, 5, 6, 8).
4. Chờ chủ project chọn hướng mới (mục 7 "còn chờ chốt" hoặc mục 9).

---

## 11. Mẫu tin nhắn mở đầu phiên mới
Đính kèm **`HANDOFF_v5.md` + `v4.html`** (và `v5.html` nếu đã dán Phần 1), rồi gửi:

> Đây là game "Tiệm Net". Hãy đọc HANDOFF_v5.md trước, đối chiếu với v4.html (bản đang chạy, có Patch A→H2).
> Việc hôm nay: [vd: viết tiếp v5 từ Phần 2 / sửa lỗi X / tính năng Y].
> Nhớ: đề xuất + checklist trước, mình OK rồi mới code; chia phần, mỗi phần nhắn "tiếp".
