# Danh sách Test Case chức năng Đăng ký mở tài khoản

Bảng phân tích Test Case cho các trường `fullName`, `email`, `phone`, `citizenId`.

| ID | Trường (Field) | Mô tả Test Case (Description) | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected) | Loại (Type) |
|---|---|---|---|---|---|
| TC_01 | **fullName** | Tên đầy đủ hợp lệ, có dấu | `Nguyễn Văn A` | Trả về 201 Created, tạo TK thành công | Positive |
| TC_02 | **fullName** | Tên để trống hoặc rỗng | `""` hoặc null | Trả về 400 Bad Request, "Họ và tên không được để trống" | Negative |
| TC_03 | **fullName** | Tên quá dài (trên 255 ký tự) | Chuỗi gồm 256 ký tự 'a' | Trả về 400/500 tuỳ DB Schema | Boundary |
| TC_04 | **email** | Email đúng định dạng phổ biến | `nguyenvana@gmail.com` | Trả về 201 Created | Positive |
| TC_05 | **email** | Email thiếu ký tự `@` | `nguyenvanagmail.com` | Trả về 400 Bad Request, "Định dạng email không hợp lệ" | Negative |
| TC_06 | **email** | Email chứa khoảng trắng | `nguyen vana@gmail.com` | Trả về 400 Bad Request | Negative |
| TC_07 | **email** | Trùng email đã đăng ký (Tuỳ logic) | `nguyenvana@gmail.com` (Đã có trong DB) | Trả về lỗi 400/409 (Email đã tồn tại) | Negative |
| TC_08 | **phone** | Số điện thoại hợp lệ (10 số, đầu 09) | `0912345678` | Trả về 201 Created | Positive |
| TC_09 | **phone** | Số điện thoại sai đầu số (đầu 01) | `0123456789` | Trả về 400 Bad Request, "Số điện thoại không hợp lệ" | Negative |
| TC_10 | **phone** | Số điện thoại có chứa chữ cái | `0912abc678` | Trả về 400 Bad Request | Negative |
| TC_11 | **phone** | Số điện thoại thừa số (11 số) | `09123456789` | Trả về 400 Bad Request | Boundary / Negative |
| TC_12 | **phone** | Số điện thoại thiếu số (9 số) | `091234567` | Trả về 400 Bad Request | Boundary / Negative |
| TC_13 | **citizenId** | Số CCCD hợp lệ đúng 12 chữ số | `030012345678` | Trả về 201 Created | Positive |
| TC_14 | **citizenId** | Số CCCD dưới 12 chữ số (11 số) | `03001234567` | Trả về 400 Bad Request, "Số CCCD phải bao gồm đúng 12 chữ số" | Boundary / Negative |
| TC_15 | **citizenId** | Số CCCD có chứa ký tự đặc biệt | `0300-2345678` | Trả về 400 Bad Request | Negative |
| TC_16 | **citizenId** | Số CCCD đã được đăng ký | `030012345678` (Đã tồn tại) | Báo lỗi Runtime "CCCD đã được sử dụng..." | Negative |
