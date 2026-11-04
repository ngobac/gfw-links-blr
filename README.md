# gfw-links-blr

Pipeline tự động tạo và duy trì link bản đồ Global Forest Watch (GFW) cho các lô cao su của layer `20161104_master_lo_cty2` (BLR), phục vụ truy xuất nguồn gốc EUDR. Cùng cơ chế với `gfw-links-donaruco` và `gfw-links-dnk`.

## Sản phẩm bàn giao

`docs/links.json` — danh sách link GFW theo mã lô (`ProductionPlace_sub`), tự cập nhật hằng đêm.

Quy tắc đọc cho hệ thống truy xuất: lấy `long_url` theo `id_lo`; chỉ hiển thị khi `status` là `alive`/`healed`; `last_checked` thể hiện độ tươi dữ liệu.

Cùng link được ghi ngược vào field `gfw_link` (string 4000) của Feature Layer, để xem trực tiếp trên ArcGIS.

## Cơ chế

- **SYNC** (01:45 VN hằng đêm): query toàn bộ lô từ AGOL Feature Layer, POST geostore GFW cho lô mới/đổi ranh, cập nhật `docs/links.json`. Bản ghi trống mã lô hoặc không có hình học bị bỏ qua và báo Telegram; layer không có field hiện trạng nên trùng mã lô thì lấy bản ghi đầu.
- Cuối SYNC: ghi `long_url` vào `gfw_link` cho các bản ghi đang có giá trị khác (lần đầu ghi toàn bộ, sau đó chỉ lô đổi). Layer phải bật **Editing (Update)**; nếu ghi lỗi, `links.json` vẫn được commit, job báo đỏ + Telegram.
- **HEAL** (08h45/16h45/00h45 VN): kiểm tra từng geostore còn sống; nếu bị xóa thì POST lại đúng GeoJSON — nhờ tính determinism của geostore v1, ID (và URL) không đổi.
- Biến động báo qua Telegram, tiền tố `[BLR]`. Lịch chạy lệch so với Donaruco (:00) và DNK (:30) để không gọi GFW cùng lúc.

## Cấu hình

Secrets cần khai báo trong Settings → Secrets and variables → Actions:
`AGOL_USERNAME`, `AGOL_PASSWORD`, `AGOL_LAYER_URL`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`.

`AGOL_LAYER_URL` = `https://services-ap1.arcgis.com/<org>/arcgis/rest/services/20161104_master_lo_cty2/FeatureServer/0`.

`map_template.txt` chứa phần query string cấu hình layer (Tree cover loss 2021→nay + Tree plantations). Khi GFW ra dữ liệu TCL năm mới (~tháng 4): cập nhật template (ở cả 3 repo), xóa `docs/links.json`, chạy tay SYNC để rebuild toàn bộ URL.
