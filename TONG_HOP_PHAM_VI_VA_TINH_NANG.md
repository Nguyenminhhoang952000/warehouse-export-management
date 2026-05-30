# Warehouse Export — Tổng hợp phạm vi & tính năng hệ thống

> Tài liệu mô tả **phạm vi đã triển khai**, **quy mô kỹ thuật**, **hạ tầng vận hành** và **định hướng phát triển** của hệ thống quản lý kho / xuất hàng.

**Công nghệ:** React + Vite + TypeScript (giao diện) · NestJS + PostgreSQL (hệ thống xử lý dữ liệu)

---

## 1. Tổng quan sản phẩm

Hệ thống hỗ trợ doanh nghiệp bán hàng B2B trong các hoạt động:

- Quản lý **nhân viên** (phân quyền: nhân viên kinh doanh / kế toán / quản lý)
- Quản lý **khách hàng**, **sản phẩm** (ngành hàng, nhóm, đơn vị, giá, tồn kho)
- Quản lý **đơn hàng** (trạng thái, chiết khấu, ghi chú từng dòng, in / xuất PDF)
- **Bảng điều khiển (Dashboard)** — doanh thu, sản phẩm và nhân viên bán chạy, so sánh theo năm
- **Địa chỉ & bản đồ** — gắn tọa độ GPS cho khách hàng và đơn hàng
- **Đăng nhập Google** — đăng nhập nhanh, bảo mật phiên làm việc
- **Nhập dữ liệu từ Excel** — khách hàng và sản phẩm số lượng lớn
- **Triển khai trên máy chủ (VPS)** — cơ sở dữ liệu, API, giao diện web, tên miền, HTTPS

---

## 2. Các tính năng đã triển khai

### 2.1. Giao diện người dùng (Web)

| Module | Mô tả |
|--------|--------|
| **Đăng nhập** | Email/mật khẩu và **đăng nhập bằng tài khoản Google** |
| **Phân quyền** | Menu và chức năng hiển thị theo vai trò (ví dụ: nhân viên kinh doanh không truy cập quản lý nhân viên) |
| **Dashboard** | Thống kê tổng quan, biểu đồ (cột, đường, tròn), lọc theo thời gian / nhân viên / ngành hàng |
| **Nhân viên** | Thêm, sửa, xem, tìm kiếm, phân trang |
| **Khách hàng** | Quản lý thông tin, gán nhân viên phụ trách, **nhập Excel**, địa chỉ kèm bản đồ |
| **Ngành hàng / Nhóm sản phẩm** | Phân loại sản phẩm theo ngành và nhóm |
| **Sản phẩm** | Nhiều đơn vị (bán lẻ / bán sỉ), hình ảnh, theo dõi giá, **nhập Excel** theo từng đợt |
| **Đơn hàng** | Nhiều dòng sản phẩm, đơn vị, giá, chiết khấu từng dòng và tổng đơn, trạng thái, địa chỉ giao hàng |
| **In / PDF đơn** | Tải file PDF hóa đơn, in trực tiếp từ trình duyệt |
| **Đa ngôn ngữ** | Tiếng Việt và English |
| **Giao diện thiết bị di động** | Sidebar thu gọn, bảng dữ liệu dạng thẻ trên màn hình nhỏ |

### 2.2. Hệ thống xử lý (API & cơ sở dữ liệu)

| Module | Mô tả |
|--------|--------|
| **Xác thực** | Phiên đăng nhập an toàn (JWT), làm mới token, mật khẩu mã hóa, **đăng nhập Google** |
| **API nghiệp vụ** | Quản lý nhân viên, khách hàng, sản phẩm, đơn hàng — kiểm tra dữ liệu, phân trang, sắp xếp |
| **Phân quyền API** | Nhân viên kinh doanh chỉ thao tác dữ liệu thuộc phạm vi được phân (đơn hàng, PDF, …) |
| **Khách hàng** | Nhập hàng loạt, xử lý trùng số điện thoại, gán nhân viên phụ trách hợp lệ |
| **Sản phẩm** | Nhập Excel: tự tạo ngành/nhóm khi cần; **tồn kho** được tính từ các đơn đã hoàn thành |
| **Đơn hàng** | Cập nhật tồn kho trong giao dịch an toàn; tính tiền theo quy tắc nghiệp vụ |
| **Dashboard** | Tổng hợp doanh thu, xếp hạng sản phẩm / nhân viên, so sánh năm |
| **PDF hóa đơn** | Tạo file PDF trên máy chủ (mẫu HTML + trình duyệt headless) |
| **Cơ sở dữ liệu** | 9 phiên bản cấu trúc dữ liệu (migration), dữ liệu mẫu khi cài đặt |
| **Bảo mật** | Giới hạn số request, header bảo vệ, cấu hình CORS theo môi trường |

### 2.3. Bản đồ & địa chỉ (điểm nhấn kỹ thuật)

Phần bản đồ được xây dựng **chuyên sâu cho thị trường Việt Nam**, không chỉ hiển thị bản đồ tĩnh.

**Chức năng người dùng thấy được:**

- Gõ địa chỉ → gợi ý tự động (autocomplete)
- Chọn gợi ý hoặc **chọn trực tiếp trên bản đồ** (kéo ghim)
- Nút **「Dùng vị trí hiện tại」** — lấy GPS thiết bị, tự điền địa chỉ
- Đồng bộ giữa ô địa chỉ, tọa độ và bản đồ khi tạo/sửa **khách hàng** và **đơn hàng**

**Giải pháp kỹ thuật (tóm tắt):**

| Thành phần | Vai trò |
|------------|---------|
| **Goong Maps API** | Gợi ý địa chỉ và geocoding chi tiết tại Việt Nam (ưu tiên) |
| **OpenStreetMap / Nominatim / Photon** | Nguồn dự phòng khi cần |
| **Leaflet + bản đồ OSM** | Hiển thị và tương tác bản đồ trên web |
| **GPS trình duyệt** | Xác định vị trí hiện tại, chuyển ngược thành địa chỉ |
| **Lưu tọa độ** | Vĩ độ / kinh độ lưu trên hệ thống cho khách hàng và đơn hàng |

**Đăng nhập Google:** dùng cho **xác thực người dùng** (OAuth), tách biệt với lớp bản đồ/geocoding phía trên.

**Lợi ích vận hành:**

- Giảm sai địa chỉ giao hàng nhờ gợi ý và chọn trên bản đồ
- Hỗ trợ nhiều nguồn dữ liệu địa lý — ổn định hơn khi chỉ dùng một dịch vụ
- Trải nghiệm mượt khi nhập liệu, kể cả form đơn hàng phức tạp

### 2.4. Sao lưu cơ sở dữ liệu (máy chủ production)

Hệ thống production được bố trí **sao lưu PostgreSQL định kỳ** trên VPS:

| Hạng mục | Mô tả |
|----------|--------|
| **Phương thức** | Xuất backup cơ sở dữ liệu (`pg_dump`) |
| **Lịch chạy** | Tự động theo lịch (cron), ví dụ hàng ngày lúc 2:00 |
| **Lưu trữ** | Thư mục backup riêng trên máy chủ, giữ nhiều bản theo chính sách (ví dụ 7–30 ngày) |
| **Khôi phục** | Quy trình restore khi cần phục hồi dữ liệu |
| **Kiểm tra** | Khuyến nghị thử khôi phục trên môi trường thử định kỳ |

*Mục tiêu: bảo vệ dữ liệu nghiệp vụ trước sự cố phần cứng, cập nhật lỗi hoặc thao tác nhầm.*

### 2.5. Triển khai, tên miền & vận hành

**Quy trình cập nhật phần mềm**

- **Giao diện web:** build tự động → đồng bộ lên máy chủ (`/var/www/frontend`)
- **Hệ thống API:** build Docker image → triển khai trên VPS qua script deploy

**Kiến trúc production (Docker Compose)**

- **PostgreSQL 16** — cơ sở dữ liệu
- **NestJS** — API nghiệp vụ
- **Nginx** — phục vụ file giao diện + chuyển tiếp `/api/` tới backend

**Tên miền & bảo mật truy cập (đã / khuyến nghị triển khai)**

1. Trỏ bản ghi **A** của tên miền về IP máy chủ  
2. Bật **HTTPS** (Let's Encrypt / Certbot)  
3. Cấu hình URL API, CORS và callback đăng nhập Google theo tên miền production  
4. Đăng ký API key Goong (gợi ý địa chỉ)  
5. Firewall: chỉ mở cổng web (80/443), **không** mở PostgreSQL ra internet  

---

## 3. Quy mô kỹ thuật theo nhóm tính năng

Thang mức độ: **Cơ bản** · **Trung bình** · **Cao** · **Rất cao**  
*(phản ánh khối lượng thiết kế, lập trình và kiểm thử — không phải độ “khó dùng” với người dùng)*

| # | Tính năng | Mức độ | Giải thích ngắn |
|---|-----------|--------|-----------------|
| 1 | Đăng nhập email + phiên làm việc | Trung bình | Token, làm mới phiên, bảo vệ trang sau đăng nhập |
| 2 | Đăng nhập Google | Cao | OAuth hai phía web + API, liên kết tài khoản, cấu hình tên miền |
| 3 | Quản lý danh mục (nhân viên, KH, SP) | Trung bình | Form, lọc, phân trang, quy tắc nghiệp vụ |
| 4 | Phân quyền 3 vai trò | Cao | Khác giao diện + khác quyền API (đơn, PDF, …) |
| 5 | **Bản đồ & địa chỉ** | **Rất cao** | Đa nguồn geocoding, GPS, đồng bộ form, lưu tọa độ |
| 6 | Form đơn hàng | **Rất cao** | Nhiều dòng, đơn vị/giá/CK, chọn KH, bản đồ, tồn kho |
| 7 | Tồn kho & trạng thái đơn | Cao | Giao dịch DB, trừ tồn khi hoàn thành |
| 8 | Nhập Excel KH + SP | Cao | Đọc file, xem trước, gửi theo đợt, xử lý trùng lặp phía server |
| 9 | Dashboard thống kê | Cao | Tổng hợp SQL, nhiều biểu đồ, lọc đa chiều |
| 10 | Xuất PDF hóa đơn | Cao | Mẫu in, tạo PDF trên Linux server, phân quyền tải |
| 11 | In đơn từ trình duyệt | Trung bình | Định dạng in, lấy dữ liệu từ API |
| 12 | Song ngữ VN/EN | Trung bình | Toàn bộ nhãn và thông báo lỗi |
| 13 | Giao diện mobile | Trung bình | Layout thích ứng, bảng → thẻ |
| 14 | Triển khai CI/CD | Cao | Build tự động, deploy FE + BE |
| 15 | Docker + Nginx production | Cao | Stack container, proxy, healthcheck DB |
| 16 | Sao lưu DB tự động | Trung bình – Cao | Cron, retention, quy trình phục hồi |
| 17 | Migration cơ sở dữ liệu | Trung bình | 9 phiên bản schema, nâng cấp an toàn |

**Tổng thể:** Các hạng mục **Rất cao** và **Cao** chiếm phần lớn effort kỹ thuật — đặc biệt **bản đồ**, **đơn hàng**, **nhập Excel**, **dashboard**, **PDF** và **hạ tầng production** — cho thấy đây là **hệ thống nghiệp vụ đầy đủ**, không phải website giới thiệu đơn giản.

---

## 4. Định hướng phát triển (đề xuất)

| Ưu tiên | Hướng phát triển | Lợi ích |
|---------|------------------|---------|
| Cao | Báo cáo xuất Excel/PDF (doanh thu, công nợ, tồn kho) | Ban lãnh đạo / kế toán tra cứu nhanh |
| Cao | Thông báo khi đơn đổi trạng thái (email, Zalo, Telegram, …) | Giảm sót đơn giao hàng |
| Cao | Quản lý công nợ chi tiết | Phù hợp bán chịu B2B |
| Trung bình | Lịch sử thay đổi giá / nhật ký thao tác | Truy vết, minh bạch nội bộ |
| Trung bình | Phân vùng / tuyến giao từ tọa độ bản đồ | Tối ưu logistics |
| Trung bình | Mã vạch / QR khi nhập–xuất kho | Giảm nhập sai mã |
| Trung bình | Ứng dụng mobile / PWA cho nhân viên hiện trường | GPS, ảnh giao hàng |
| Trung bình | Tích hợp phần mềm kế toán | Giảm nhập liệu trùng |
| Thấp – TB | Giao diện tối, logo / màu theo thương hiệu | Nhận diện công ty |
| Thấp – TB | Xác thực hai lớp (2FA) | Tăng cường bảo mật |
| Thấp | Chỉ đường Google Maps Platform (nếu cần) | Tuyến đường chính thức (có phí API) |

---

## 5. Tóm tắt giá trị hệ thống

Hệ thống Warehouse Export đã bao gồm:

- **Quản trị bán hàng B2B** — từ danh mục đến đơn hàng, tồn kho và báo cáo tổng quan  
- **Bản đồ & địa chỉ Việt Nam** — gợi ý Goong, GPS, chọn trên bản đồ, lưu tọa độ  
- **Nhập liệu hàng loạt** — Excel khách hàng và sản phẩm  
- **Xuất PDF hóa đơn** — in ấn chuyên nghiệp từ server  
- **Đăng nhập Google** — thuận tiện cho người dùng  
- **Triển khai production** — VPS, tên miền, HTTPS, sao lưu database, cập nhật tự động  

---

*Tài liệu phục vụ trao đổi phạm vi dự án với Quý khách hàng. Cập nhật khi có tính năng hoặc thay đổi hạ tầng mới.*
