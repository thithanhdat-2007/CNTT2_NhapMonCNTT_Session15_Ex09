# Session 15 - Ex09: Thiết kế lưu trữ Google Drive cho CampusPay

Họ và tên: Thi Thành Đạt  
MSSV: N26DTCN041  
Lớp: CNTT2  
Email: thithanhdat111@gmail.com

## Phần 1. Phân tích bài toán (I/O)

### 1. Đầu vào và đầu ra

- Đầu vào: CampusPay có Ban Kỹ thuật (3 người), Ban Thiết kế (2 người), Ban Kinh doanh (2 người) và một nhà đầu tư bên ngoài. Tài liệu gồm mã nguồn và kiểm thử API, bản thiết kế UI, Business Plan, Slide Pitch và dữ liệu nhạy cảm như API keys, thông tin khách hàng.
- Đầu ra: một cấu trúc thư mục Google Drive hai cấp và bảng phân quyền cho bốn vai trò. Tài liệu nhạy cảm chỉ chia sẻ ở chế độ Restricted; nhà đầu tư chỉ được xem đúng tài liệu cần thiết trong 14 ngày.

### 2. Cấu trúc thư mục hai cấp

```text
CampusPay/
├── 01_KyThuat/     Mã nguồn, tài liệu kỹ thuật, kết quả kiểm thử API
├── 02_ThietKe/     Bản thiết kế UI, hình ảnh và tài sản đồ họa
├── 03_KinhDoanh/   Business Plan và tài liệu làm việc của Ban Kinh doanh
├── 04_LienBan/     Slide Pitch và Business Summary dùng khi cộng tác
└── 05_BaoMat/      Tài liệu có API keys hoặc thông tin khách hàng
```

Thư mục `CampusPay` chỉ để trưởng nhóm quản lý. Em không cấp quyền cho tất cả thành viên ở thư mục gốc, vì quyền ở thư mục cha có thể được kế thừa xuống các thư mục con. Mỗi ban được quyền làm việc ở thư mục của mình. Hai thư mục `04_LienBan` và `05_BaoMat` cũng chỉ để trưởng nhóm quản lý; khi cần cộng tác, trưởng nhóm chia sẻ từng file cụ thể. Cách này tránh việc ai được vào một thư mục thì tự động thấy mọi file trong đó.

Với API keys, nếu dự án có công cụ quản lý bí mật riêng thì nên lưu khóa ở đó. Nếu bắt buộc lưu tài liệu liên quan trên Drive, file phải để Restricted và chỉ cấp cho người thật sự cần.

### 3. Các bước lập cấu hình

1. Trưởng nhóm tạo thư mục `CampusPay`, sau đó tạo năm thư mục cấp hai với tên như trên.
2. Đặt chế độ truy cập mặc định là Restricted. Không dùng “Anyone with the link” cho Business Plan, dữ liệu khách hàng hoặc API keys.
3. Cấp quyền cho từng ban ở thư mục riêng. Với tài liệu cần dùng chéo, chia sẻ từng file theo bảng ở Phần 2, không mở quyền cho cả thư mục.
4. Với nhà đầu tư, chỉ chia sẻ Business Plan đã duyệt và bản PDF của Slide Pitch bằng tài khoản email cụ thể, quyền Viewer, đặt ngày hết hạn sau 14 ngày. Cần dùng tài khoản Google Drive có hỗ trợ tính năng đặt ngày hết hạn và kiểm tra ngày hết hạn đã lưu thành công.
5. Tải tài liệu quan trọng lên Drive và kiểm tra lịch sử phiên bản. Với Docs/Slides, dùng Version History và đặt tên các bản chốt. Với PDF, ảnh hoặc file tải lên, đánh dấu “Keep forever” cho bản quan trọng để tránh mất phiên bản khi có nhiều lần cập nhật; kiểm tra định kỳ để đáp ứng yêu cầu lưu ít nhất 30 ngày.
6. Đăng nhập thử bằng tài khoản đại diện cho từng vai trò để kiểm tra ai xem được, ai sửa được và ai không thấy file. Kiểm tra riêng việc tải bản PDF của nhà đầu tư.
7. Khi đổi nhiệm vụ hoặc có người nghỉ việc, thu hồi quyền ở nhóm và từng file/thư mục đã chia sẻ trực tiếp, rồi kiểm tra lại quyền truy cập.

## Phần 2. Triển khai và chống lỗi

### 4. Bảng phân quyền

Trong bảng, **Editor** được sửa, **Viewer** chỉ xem, **Không cấp** là không có quyền truy cập. Quyền chỉ áp dụng cho người thuộc vai trò cần dùng tài liệu đó.

| Loại tài liệu | Ban KT | Ban TK | Ban KD | Nhà đầu tư | Ghi chú bảo mật |
|---|---|---|---|---|---|
| 1. Mã nguồn, tài liệu kỹ thuật và kiểm thử API | Editor | Không cấp | Không cấp | Không cấp | Lưu ở `01_KyThuat`; chỉ thành viên kỹ thuật và trưởng nhóm quản lý. |
| 2. Bản thiết kế UI và tài sản đồ họa | Viewer | Editor | Không cấp | Không cấp | Lưu ở `02_ThietKe`; Ban KT xem để triển khai giao diện. |
| 3. Business Plan | Viewer | Không cấp | Editor | Viewer, hết hạn sau 14 ngày | Lưu ở `03_KinhDoanh`; chỉ chia sẻ riêng file đã duyệt cho Ban KT và nhà đầu tư, không chia sẻ cả thư mục. |
| 4. Slide Pitch và Business Summary | Viewer | Editor với Slide Pitch, Viewer với Summary | Editor | Viewer bản PDF Slide Pitch trong 14 ngày; không cấp Summary | Lưu ở `04_LienBan`; chia sẻ riêng từng file. Nhà đầu tư không được sửa bản gốc. |
| 5. API keys và thông tin khách hàng | Editor chỉ với file API keys được giao; không cấp file khách hàng | Không cấp | Editor chỉ với file khách hàng được giao; không cấp file API keys | Không cấp | Lưu ở `05_BaoMat`, Restricted. Không cấp quyền chung cho cả ban hoặc cả thư mục; mỗi file chỉ cấp đúng người phụ trách. |

Trưởng nhóm là người quản lý cấu trúc và có quyền xử lý các trường hợp thay đổi nhân sự. Trong hàng 5, người phụ trách chỉ được quyền cần cho công việc của mình; không vì cùng thuộc một ban mà được xem tất cả file nhạy cảm.

### 5. Xử lý ba bẫy dữ liệu

**Bẫy 1 - Ban Kỹ thuật cần đọc Business Plan:** Em cấp quyền **Viewer** trên đúng file Business Plan. Ban Kỹ thuật xem được bối cảnh để viết code nhưng không sửa được nội dung. File vẫn ở chế độ Restricted, không bật “Anyone with the link”.

**Bẫy 2 - Nhà đầu tư cần tải Slide Pitch để xem offline:** Em xuất Slide Pitch thành **PDF**, chia sẻ bản PDF đó cho đúng email nhà đầu tư với quyền **Viewer** và hạn 14 ngày. Không cấp quyền vào bản slide gốc có thể chỉnh sửa. **Không tắt** tính năng cho Viewer tải xuống ở bản PDF, vì tắt thì nhà đầu tư không thể tải file để xem offline. Chỉ nên chia sẻ bản đã duyệt; việc cho tải xuống đồng nghĩa nhà đầu tư có một bản lưu trên máy và hết hạn quyền Drive không xóa được bản đã tải.

**Bẫy 3 - Một thành viên nghỉ việc:** Trưởng nhóm xóa người đó khỏi các nhóm chia sẻ, gỡ quyền được cấp trực tiếp trên mọi thư mục và file, đồng thời thu hồi tài khoản làm việc theo quy định của dự án. Nếu người đó sở hữu file công việc, cần chuyển quyền sở hữu cho người tiếp quản trước khi đóng tài khoản. Cuối cùng, kiểm tra bằng danh sách người có quyền truy cập để bảo đảm tài khoản cũ không còn thấy tài liệu.

## Kết luận

Thiết kế này chia tài liệu theo công việc, còn tài liệu dùng chung hoặc nhạy cảm được chia sẻ theo từng file. Nhờ vậy, mỗi người chỉ có quyền cần thiết; nhóm vẫn có thể cộng tác và thu hồi quyền khi công việc kết thúc.

