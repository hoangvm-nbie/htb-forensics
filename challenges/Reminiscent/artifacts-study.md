**Challenge:** Reminiscent - Hack The Box


---

## 1. Artifact được đề cập trong Challenge
### 1.  Memory Dump (flounder-pc-memdump.elf)
- Bản chất: Là ảnh chụp nhanh (snapshot) của bộ nhớ RAM của hệ thống tại một thời điểm cụ thể.

- Giá trị Pháp y:

Dữ liệu dễ bay hơi (Volatile Data): Đây là nguồn duy nhất để tìm kiếm dữ liệu không được ghi vào ổ đĩa cứng, đặc biệt là trong các cuộc tấn công không cần file (fileless).

Tiến trình (Processes) và Kết nối mạng: Nó tiết lộ các tiến trình đang chạy (kể cả tiến trình ẩn), các DLL được nạp, và các kết nối TCP/UDP đang hoạt động, giúp xác định máy chủ Chỉ huy & Điều khiển (C2) của mã độc.

Giải mã Payload: Thường là nơi tìm thấy các chuỗi lệnh (shellcode) hoặc lệnh độc hại (payload) đã được giải mã, như lệnh PowerShell Base64 mà bạn đã tìm thấy.

Hành vi của Mã độc: Cung cấp thông tin chi tiết về các hoạt động như tiêm mã (code injection), móc nối (hooking), và thao tác với hệ thống.

### 2.  Tệp EML (Resume.eml)
Bản chất: Là định dạng tệp chuẩn để lưu trữ một thông điệp email, bao gồm Header (tiêu đề) và Body (nội dung).

Giá trị Pháp y:

Vector Tấn công Ban đầu (Initial Access): Là bằng chứng đầu tiên xác định cách kẻ tấn công xâm nhập.

Header: Chứa thông tin quan trọng về nguồn gốc:

Địa chỉ IP Gửi: Giúp lần theo dấu vết máy chủ gửi ban đầu.

Mail Path: Đường đi của email qua các máy chủ.

Kết quả SPF/DKIM: Cho biết kẻ tấn công có giả mạo thành công tên miền hay không.

Body & Attachments: Cho biết mồi nhử (lure) được sử dụng ("Resume") và tệp độc hại được phân phối (resume.zip, sau đó là đường dẫn tải xuống).

### 3. Tệp LNK (resume.pdf.lnk)
Bản chất: Là một tệp Shortcut (Phím tắt) của Windows. Mặc dù có vẻ vô hại, tệp .lnk chứa cấu trúc dữ liệu riêng biệt.

Giá trị Pháp y:

Kỹ thuật Khai thác (Exploitation Technique): .lnk được sử dụng để che giấu việc thực thi một lệnh phức tạp. Thay vì chỉ trỏ đến một tệp, nó có thể chứa một đối số dòng lệnh (command-line argument) dài và độc hại.

Lệnh PowerShell Mã hóa: Trong trường hợp này, tệp .lnk được dùng để kích hoạt powershell -encodedcommand <chuỗi Base64>. Việc phân tích tệp .lnk đã giúp bạn trích xuất được chuỗi mã hóa này.

Ngụy trang (Masquerading): Tên tệp resume.pdf.lnk được thiết lập để lừa người dùng rằng họ đang mở một tệp PDF an toàn.