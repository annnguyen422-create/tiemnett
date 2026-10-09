# HANDOFF v5 — Tiệm Net

> Đọc file này trước khi sửa code. Viết cho người (hoặc AI) tiếp quản dự án.
> Cập nhật: 09/10/2026 · Bản: **v5** (bản sạch, thay cho v4 + Patch A→H2)

---

## 1. Tổng quan

Game quản lý tiệm net Việt Nam, chạy trên trình duyệt (điện thoại + máy tính).

| Mục | Giá trị |
|---|---|
| Link chơi | https://annnguyen422-create.github.io/tiemnett/ (hai chữ **t**) |
| Repo | `tiemnett` |
| File chính | `index.html` (= v5 sau khi test đạt) |
| Bản dự phòng | `v4.html` (v4 + patch A→H2, chạy được, không sửa nữa) |
| Công nghệ | 1 file HTML + CSS + JS thuần, không framework, không thư viện |
| Font | Nunito (Google Fonts) |
| Hình vẽ | SVG tự vẽ trong code, không dùng ảnh ngoài |
| Save | `localStorage["tiemnet_save_v3"]` |
| App phụ | Chi tiêu: https://annnguyen422-create.github.io/chitieu/ (repo `chitieu`, độc lập) |

**Trạng thái:** v5 đã viết xong đủ 9 phần. **Chưa xác nhận test 151/151** — việc đầu tiên khi tiếp quản là chạy test (mục 3).

---

## 2. Cấu trúc file v5 (9 phần, theo thứ tự trong `<script>`)

| Phần | Nội dung | Hàm / hằng chính |
|---|---|---|
| HTML + CSS | Giao diện, toàn bộ CSS đã gộp patch | `#top` `#vp/#world` `#bar` `#carry` `#kbar` `#sheet` `#toasts` |
| 1 Config + Data + Utils | Hằng số, dữ liệu món, kho, máy, khách | `CFG` `PAT` `COOK` `PREP` `UNLOCK` `ST_LOCK` `maxItemsOn` `ITEMS` `MENU` `KIT` `UPGRADES` `CTYPES` `budgetOf` |
| 2 Text | Toàn bộ chữ hiển thị + thoại khách | `STR.vi` `t()` `DIALOGUE.vi` `line()` |
| 3 State + Economy + Save | State, tiền, kho/lô, nhân viên, nâng cấp, lưu/tải | `newState` `applyUpgrades` `addTx` `recharge` `buyStock` `returnBottle` `hireStaff` `buyUpgrade` `deserialize` `migrateV1/V2/C` |
| 4 Customers + Day | Khách, hàng chờ, gọi món, kiên nhẫn, vòng ngày | `queueFront` `spawnCustomer` `assignPC` `orderPatienceFor` `maybeOrder` `seatedTick` `stepSim` `endDay` `nextDay` |
| 5 Kitchen + Matching + Delivery + Staff | Bếp, ghép món ↔ khách, giao món, AI nhân viên | `kBin` `kTap` `canFinish` `finishDish` `matchAll` `deliverTo` `cookStep` `cashierStep` |
| 6 Art + World | SVG nhân vật, máy, món, trạm; dựng bản đồ | `charSVG` `wiSVG` `binSVG` `KSVG` `buildWorld` `fit` |
| 7 Render | Vẽ mọi thứ mỗi khung hình | `renderTop` `renderMeta` `bubbleOf` `nextAction` `renderHL` `renderKBar` |
| 8 Input + Sheets + Loop | Chạm, các bảng, vòng lặp chính, khởi động | `onWorld` `startDelivery` `BUILD.*` `ACT.*` `frame` `bind` `init` |
| 9 Tests | Test tự động + bot tự chơi | `runTests` `simulateDay` `botAct` `checkInv` · cuối file: `init();` |

---

## 3. Chạy test

- Mở: `…/tiemnett/?test&v=N` (đổi `N` mỗi lần để không dính cache).
- Hoặc trong game: **Cài đặt → Chạy kiểm thử**.
- Kỳ vọng: **151/151** ✅.

| Nhóm | Nội dung | Số bài |
|---|---|---|
| Q | Hàng chờ (gồm: đầu hàng chưa tới thì chưa phục vụ) | 10 |
| K | Kiên nhẫn | 7 |
| M | Nấu mì | 16 |
| F | Đồ chiên | 6 |
| W | Pha chế + nước chai | 7 |
| G | Quầy ra món (khi đã mua) | 3 |
| E | Trường hợp đặc biệt | 7 |
| S | Nhân viên | 9 |
| T | Tiền | 10 |
| H | Hạn dùng | 2 |
| P | Giá bán | 1 |
| U | Nâng cấp | 3 |
| R | Báo cáo cuối ngày | 5 |
| V | Lưu / tải / chuyển save | 6 |
| A | Giao diện gọn | 5 |
| N | Chờ món, topping đúng y | 12 |
| C | Quầy ra món là nâng cấp | 8 |
| D | Mở món theo ngày | 11 |
| X | Thẻ xN, nút "−", tag "Tiếp theo" | 18 |
| BOT | Bot tự chơi trọn ngày | 5 |

**Quy tắc:** mọi tính năng mới phải thêm test vào `runTests()` (Phần 9), đặt mã nhóm mới, không trùng mã cũ.

---

## 4. Quy ước code (bắt buộc giữ)

1. **Mỗi hàm chỉ có MỘT bản.** Không ghi đè hàm (`foo=function…`), không bọc (`const _old=BUILD.x; BUILD.x=…`), không `MutationObserver`/`setInterval` để vá giao diện, không chồng `!important`. Muốn đổi → sửa thẳng hàm gốc.
2. **Logic không đụng DOM.** Hiệu ứng (toast, âm thanh, chữ bay) đi qua `fx.*`, tự tắt khi `TEST=true`. Lớp CSS theo trạng thái (ẩn Quầy ra món, khóa trạm, chế độ dạy) chỉ đặt trong `renderMeta()`.
3. **Hàm logic trả** `{ok:true,step}` hoặc `{ok:false,err,v}`; `err` là key trong `STR`.
4. **Tiền chỉ thay đổi qua** `addTx(amount,type,desc)` (để sổ sách luôn khớp — test R2, BOT3).
5. **Mọi chữ hiển thị nằm trong `STR.vi`** (Phần 2), không viết cứng tiếng Việt trong code (trừ vài nhãn SVG).
6. **Save giữ `VERSION = 3`.** Thêm trường mới vào state → thêm giá trị mặc định trong `newState()` và trong `deserialize()` (để save cũ không lỗi). Đổi cấu trúc lớn → viết `migrateX()` chạy 1 lần, có cờ đánh dấu (như `verC`).
7. **Gửi code theo kiểu "dán nối tiếp"** hoặc "thay nguyên hàm X" — người chủ dự án sửa trực tiếp trên GitHub, không dùng terminal.

---

## 5. State `S` (lưu vào save)

```
v=3 · verC=1 · phase PREP|OPEN|CLOSING|REPORT · day · time (phút game) · money · rating · paused
pcs[] · inv{} · lots{id:[{n,exp}]} · prices{} · upg{} · staff{type,hand,t,st,serving}|null
k{board[3],stove[2],fryer[2],bar[2]} · hand · sel{board,bar}
carry[] (khay, ≤4 hoặc 6) · pickup[6] (Quầy ra món — chỉ dùng khi upg.pickup)
customers[] · accounts{} (khách quen) · tx[] · history[] · seq{c,tx,order,dish}
player{x,y,flip} · today{} · report · settings{sound,hints} · tutorialDone
```

- **Khách:** `TO_QUEUE → QUEUE (đầu hàng có thể SERVING) → TO_PC → PLAYING ⇄ WAITING_FOOD → EATING`, thêm `RECHARGE` (hết số dư) và `LEAVING`.
- **Món đang làm (work item):** `raw` · `prep` · `bowl` · `pot` · `fry` · `fried` · `cup` · `trash`; `by:'cook'` = của đầu bếp, người chơi không đụng vào.
- **Món xong (dish):** `{id,r,top,q}` nằm trong `carry[]` hoặc `pickup[]`.
- **Đã bỏ khỏi v5:** `speed` (xóa khi tải save cũ).

### Chuyển save
| Hàm | Từ → đến | Khi nào chạy |
|---|---|---|
| `migrateV1` | v1 → v2 | `d.v===1` |
| `migrateV2` | v2 → v3 (bếp mới, khay → carry, hàng chờ mới) | `d.v===2` |
| `migrateC` | save trước Patch C | `verC` chưa có: có đầu bếp → tặng Quầy ra món (toast 🎁); không có → món trên quầy chuyển về khay |
| Nhập save rất cũ | `tiemnet_save_v1` → v3 | 1 lần, cờ `tiemnet_v3_imported` |
| Save hỏng / phiên bản lạ | sao lưu sang `tiemnet_save_v3_backup_<time>`, chơi mới | — |

---

## 6. Luật chơi đã chốt (KHÔNG đổi nếu chưa hỏi chủ dự án)

### Vòng ngày
CHUẨN BỊ 07:30 → MỞ CỬA 08:00 → 23:00 ngừng nhận khách → 24:00 đóng → BÁO CÁO → ngày mới.
1 giây thật = 2 phút game. **Không có nút tốc độ**, chỉ ⏸/▶ (phím Space).

### Mở món theo ngày
| Ngày | Món mở | Tối đa món/lần gọi |
|---|---|---|
| 1 | Nước chai + pha chế | 1 |
| 2 | + Đồ chiên | 1 |
| 3 | + Mì | 1 |
| 4–5 | Tất cả | 2 |
| 6+ | Tất cả | 3 |

- Trạm chưa mở: xám + nhãn "🔒 Ngày N" (Bàn sơ chế/Tủ đông/Chảo = ngày 2; Thùng mì/Bếp = ngày 3).
- Bảng chuẩn bị đầu ngày có khung "🆕 Món mới hôm nay" (các bước + công thức).
- **Chế độ dạy:** đơn đầu tiên của món mới trong ngày → viền đỏ đậm, tag "Tiếp theo" bấm được. Bán món đó 1 lần → tắt.

### Hàng chờ
- 6 chỗ, số thứ tự, người đầu có ★. Hàng đầy → khách mới về luôn.
- Chỉ phục vụ người đầu hàng **đã đứng tới quầy**; người đầu còn đang đi tới thì cả hàng chờ.
- Khách xếp hàng **không có bóng chat** (chỉ số + thanh kiên nhẫn). Bấm quầy để xem yêu cầu.

### Kiên nhẫn
- Xếp hàng: `80 × tính cách` (×1,3 nếu có máy lạnh), giảm 1/phút; đang được phục vụ = không giảm.
- Chờ món: `(40 + Σ thời gian món) × hệ số số món × tính cách` (×1,15 máy lạnh).
  Thời gian món: nước chai 15 · pha chế 35 · đồ chiên 50 · mì 60. Hệ số: 1 món ×1 · 2 món ×1,15 · 3 món ×1,3.
- Tốc độ giảm: đang chơi ×0,7 · **món xong mà chưa giao ×1,6** · ngồi chờ (hết giờ/hết tiền) ×1.
- 4 tâm trạng: 😊 >66% · 🙂 >40% · 😐 >15% · 😠.
- Hủy đơn lần 2 hoặc vui vẻ <20 → bỏ về.
- Chỉ gọi món khi thời gian chơi còn lại ≥ Σ thời gian món + 15 (bỏ bớt món chậm nhất cho vừa).
- **Đang chờ món thì không bỏ về** dù hết giờ hay hết tiền: ngồi chờ, không tính tiền máy; nhận món xong hoặc hết kiên nhẫn mới về.

### Nấu món
- **Mì:** Thùng mì → Bàn sơ chế (tự xé gói → tô → nêm, 3 phút, tốn 1 tô) → thêm topping → bưng tô → Bếp → vớt khi vạch vào vùng xanh.
- **Đồ chiên:** Tủ đông → Chảo → vớt khi vàng → Bàn sơ chế (tự bày đĩa + tương, 2 phút) → bấm đĩa.
- **Pha chế:** Quầy pha nước → Ly → Đá → Trà/Cà phê → Chanh/Đào/Sữa → tự lắc 2 phút → bấm ly. Sai công thức = hỏng.
- **Nước chai:** Tủ lạnh → khay. Mỗi lần bấm = +1 chai.
- **Topping phải ĐÚNG Y** (không thiếu, không dư). Sai → khách không nhận, nói "Không đúng món"; bảng khách hiện cảnh báo đỏ.
- Cầm loại khác cùng tủ → đổi; bấm lại đúng loại đang cầm → trả về kho.

### Giao món
- Món đã ghép với khách → khách có **vòng xanh + mũi tên ▼**.
- Bấm khách / bấm món trên khay → người chơi tự đi (lấy ở Quầy ra món nếu cần) rồi **bám theo đúng khách**. Khách về giữa chừng → món ở lại khay, tự ghép khách khác.

### Gợi ý (tắt được: Cài đặt → Gợi ý bước tiếp theo)
- Trạm tiếp theo: viền vàng nét đứt nhấp nháy. Ngăn cần lấy: nảy + nền xanh.
- Bàn thao tác: dòng "Cần làm", rồi tag xanh **"Tiếp theo: …"** — chỉ là nhãn, **chỉ bấm được khi đang dạy món mới**.
- Chưa có Quầy ra món → không bao giờ gợi ý "đặt lên Quầy ra món".

### Bàn thao tác (thẻ dưới màn hình)
- Thẻ đang chọn: sáng xanh + **badge xanh "xN"** góc trên phải.
- Tủ lạnh: thêm **nút đỏ "−"** góc trên trái (cùng cỡ badge) để trả 1 chai, ưu tiên chai chưa có khách nhận.
- **Ngăn tủ trên bản đồ: không badge, không sáng viền** (chỉ còn hiệu ứng nảy của gợi ý).

### Nhân viên (tối đa 1 người)
| Ai | Phí tuyển | Lương/ngày | Tốc độ | Việc |
|---|---|---|---|---|
| Thu ngân | 400k | 150k | 1 việc/phút | Gọi người đầu hàng, nạp đúng số khách xin, xếp đúng loại máy, tự nạp cho khách đang chơi |
| Đầu bếp | 700k | 200k | 1 việc/0,5 phút | Nấu trọn đơn (đúng topping), không đụng món người chơi đang làm. Món xong → Quầy ra món nếu có, **không thì lên khay người chơi**; khay đầy thì đứng chờ |

Lương trả cuối ngày cho người có làm trong ngày.

### Nâng cấp (mua 1 lần)
Bếp hẹn giờ 900k · Chảo tự ngắt 1,2tr · Bếp từ (6→4 phút) 700k · Chảo lớn (5→3,5 phút) 800k · Máy lắc (2→1 phút) 400k · Khay lớn (6 món) 500k · **Quầy ra món 800k (🔒 cần đang có đầu bếp)** · Màn hình 144Hz 2,4tr · Ghế gaming 1,6tr · Máy lạnh 1,5tr.
Quầy ra món ẩn trên bản đồ cho tới khi mua.

### Kinh tế
- Vốn đầu: 5.000.000đ.
- Chi phí cuối ngày: điện 40k + 2.500đ/giờ máy (+20k nếu có máy lạnh) · Internet 50k · Bảo trì 30k.
- Hạn dùng (ngày): cá viên 3 · bò viên 3 · tôm viên 2 · xúc xích 4 · hồ lô 3. Theo lô, dùng lô cũ trước, tự bỏ đầu ngày, báo ở bảng chuẩn bị.
- Giá bán: ±1.000đ, trong khoảng 50–200% giá gốc; giá cao hơn gốc → khách ít gọi món đó.

### Giao diện
- Thanh trên 1 hàng: `☀️ N7·11:00 | 💰 6,63tr | ⭐ 3.8 | 🟢 Mở | ⏸` (màn hẹp <600px rút gọn).
- Thanh dưới 7 nút: Máy · Đồ ăn · Kho · Doanh thu · Nhân viên · Nâng cấp · Cài đặt.
- Góc dưới trái: "Tay: … | Khay n/4: …" (ẩn khi bàn thao tác mở).
- Bản đồ 960×600: bếp hàng trên; quầy + hàng chờ bên trái; 8 máy ở giữa; Quầy ra món bên phải (khi đã mua).
- Cài đặt có dòng **"Bản: v5"** (`CFG.BUILD`).

---

## 7. Lỗi đã biết (chưa sửa)

| Mã | Mô tả |
|---|---|
| BUG-003 | Mở bảng quầy thì khách đầu hàng không mất kiên nhẫn → có thể lợi dụng |
| BUG-008 | Nút "Số khác" dùng `prompt()` → bị chặn trong trình duyệt nhúng |
| BUG-010 | Khách không đủ tiền mặt vẫn ghi đủ doanh thu món |
| BUG-012 | Mua nâng cấp tốc độ bếp/chảo, thanh thời gian trên bản đồ chưa cập nhật ngay |
| — | Không có tìm đường (nhân vật đi xuyên đồ vật) · tiền có thể âm |

## 8. Việc chờ quyết định

- Lộ trình phase chính thức (PROJECT_STATUS vs spec gốc).
- Ghi doanh thu nạp tiền lúc nạp hay lúc khách dùng giờ máy.
- Phí tuyển + nâng cấp tính là chi phí vận hành hay đầu tư.

## 9. Kế hoạch tiếp theo

**Bước ngay:** chạy test v5 → 151/151 → đổi tên `index.html` → `v4.html`, `v5.html` → `index.html` → mở game kiểm tra save cũ + "Bản: v5".

**v5.1 — "Chất net Việt Nam"** (đã đề xuất, chủ dự án chưa chọn):
- Dễ: thoại chất hơn ("máy 7 lag quá", "team ngu quá"), biệt danh kiểu in-game, bảng giá viết tay.
- Vừa: món net (mì xào bò, cơm chiên, trứng ốp la, bánh mì, nước tăng lực, trà đá, bim bim, hướng dương); học sinh mặc đồng phục bị phụ huynh đến đón; gói chơi đêm 22h–6h; khách quen ghi sổ nợ; bán thẻ game.
- Lớn: cúp điện (mua máy phát), đứt cáp quang (lag cả ngày), chuột/phím hỏng, mưa đông khách, mùa thi ít học sinh, giải đấu cuối tuần.
- Hình: biển LED "INTERNET · GAME ONLINE", quạt trần, ghế nhựa đỏ → ghế gaming, xe máy trước cửa, dép để ngoài.
- Dùng **tên chế / tên chung** cho game và thương hiệu (tránh bản quyền), như đã làm với nước chai (`BRAND`).

**Spec gốc chưa làm:** mạng & lag · sửa/bảo trì máy · dọn dẹp · nâng cấp từng máy · sự kiện ngẫu nhiên · thời tiết/giải đấu/mùa · mở rộng 12→64 máy · phòng VIP · lên hạng tiệm · khách VIP · trang trí · thành tựu · thống kê tuần/tháng · thanh toán QR/ví · vay nợ & thua cuộc · nhạc nền · đổi ngôn ngữ.

---

## 10. Lịch sử phiên bản

| Bản | Nội dung |
|---|---|
| v4 | Phần 1–9 + Patch A (thanh trên gọn, bỏ tốc độ, bỏ bóng chat khi xếp hàng) · B (kiên nhẫn theo số món, ngồi chờ món, topping đúng y) · B-fix (đầu hàng phải tới nơi) · C (Quầy ra món là nâng cấp) · D (mở món theo ngày, dạy món mới) · E/E2 (badge xN) · F (nút "−" trả chai) · G/G3 (màu, cỡ badge, gọn tủ bản đồ) · H/H2 (tag "Tiếp theo") |
| **v5** | Viết lại sạch toàn bộ v4 + patch: mỗi hàm 1 bản, không observer/interval vá UI, `renderMeta` gom lớp CSS trạng thái, `migrateC` chạy lúc tải save, 1 hàm `runTests` 151 bài. Hành vi giữ nguyên v4. |
