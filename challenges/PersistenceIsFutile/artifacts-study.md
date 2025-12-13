**Challenge:** Persistence - Hack The Box


---
## 1. Artifact được đề cập trong Challenge
### 1. File SUID (Set User ID) Độc hại và Shell Backdoor Cục bộ

| Mục | Chi tiết |
| :--- | :--- |
| **Định nghĩa & Vai trò** | SUID là bit đặc biệt cho phép tệp thực thi chạy với quyền của chủ sở hữu (thường là **root**), cho phép leo thang đặc quyền. Kẻ tấn công lợi dụng điều này. |
| **Công cụ Trích xuất** | `find / -perm -04000 2>/dev/null` để tìm SUID. `rm -rf` để xóa tệp. |
| **Chỉ dấu Quan trọng** | **Tệp `.backdoor`** với quyền SUID root (`rwsr-xr-x`) đã được phát hiện và xóa. Sau đó, các tệp SUID khác như tệp được tạo bởi script `access-up` cũng được loại bỏ. |
| **Ý nghĩa Pháp chứng** | Chứng minh kẻ tấn công đã tạo **cơ chế leo thang đặc quyền** và **cửa hậu (backdoor)** cục bộ. Việc xóa SUID là cần thiết để ngăn chặn việc giành lại quyền root dễ dàng. |

### 2. File Cấu hình Shell (`.bashrc`) và Command Alias

| Mục | Chi tiết |
| :--- | :--- |
| **Định nghĩa & Vai trò** | Tệp cấu hình shell, được thực thi mỗi khi shell được khởi tạo. Kẻ tấn công sử dụng nó để chèn **alias** độc hại nhằm duy trì quyền truy cập. |
| **Công cụ Trích xuất** | `cat .bashrc` để kiểm tra nội dung và `nano/vi` để xóa dòng alias. |
| **Chỉ dấu Quan trọng** | **Alias độc hại:** Dòng alias thay thế lệnh `cat` bằng một **Reverse Shell** kết nối đến IP `172.17.0.1` trên port `443`. |
| **Ý nghĩa Pháp chứng** | Xác định một kỹ thuật **Persistence** tinh vi, biến lệnh phổ biến thành **cơ chế kích hoạt payload**. Việc xóa alias đã vô hiệu hóa phương thức này. |

### 3. Tiến trình Đang chạy (Running Processes) và File Thực thi Độc hại

| Mục | Chi tiết |
| :--- | :--- |
| **Định nghĩa & Vai trò** | Các chương trình đang chạy. Tiến trình độc hại là dấu hiệu của việc kiểm soát hệ thống theo thời gian thực. Các file như `alertd` và `connectivity-check` là tệp thực thi của tiến trình độc hại. |
| **Công cụ Trích xuất** | `ps auxef` để xem danh sách tiến trình, `kill -9 <PID>` để chấm dứt. `find / -type f -name...` để định vị tệp thực thi. |
| **Chỉ dấu Quan trọng** | **Tiến trình lạ:** `alertd` chạy trên cổng **`4444`** và `connectivity-check` được phát hiện đang chạy ngầm bằng `nohup`. Việc xóa chúng là hành động ngăn chặn ngay lập tức. |
| **Ý nghĩa Pháp chứng** | Cung cấp bằng chứng về **quyền kiểm soát hệ thống đang hoạt động**. Đồng thời, việc tìm thấy các file thực thi như `alertd` và `connectivity-check` là cần thiết để loại bỏ **công cụ (tools)** của kẻ tấn công. |

### 4. Cron Jobs (Lịch trình Tác vụ) và DNS Persistence

| Mục | Chi tiết |
| :--- | :--- |
| **Định nghĩa & Vai trò** | Cơ chế lên lịch tác vụ. Là nơi kẻ tấn công thiết lập cơ chế Persistence để đảm bảo mã độc được chạy định kỳ. |
| **Công cụ Trích xuất** | `crontab -l` và kiểm tra các thư mục `/etc/cron.*`. |
| **Chỉ dấu Quan trọng** | **Lệnh độc hại:** Lệnh sử dụng `dig` để lấy mã độc từ bản ghi **TXT** DNS của tên miền **`imforce.HTB`**. |
| **Ý nghĩa Pháp chứng** | Chứng minh kẻ tấn công đã thiết lập cơ chế **Persistence** sử dụng **kỹ thuật DNS** để che giấu payload, khiến việc dò tìm bằng các công cụ mạng truyền thống khó khăn hơn. |

### 5. Authorized Keys (Khóa SSH) và Tệp `pyssh/ssh_import_id_update`

| Mục | Chi tiết |
| :--- | :--- |
| **Định nghĩa & Vai trò** | Chứa khóa công khai SSH được ủy quyền. Nếu bị thỏa hiệp, cho phép đăng nhập từ xa không cần mật khẩu. |
| **Công cụ Trích xuất** | Kiểm tra script `pyssh/ssh_import_id_update` và nội dung tệp `/root/.ssh/authorized_keys`. |
| **Chỉ dấu Quan trọng** | **Mã Base64:** Script `ssh_import_id_update` chứa mã base64 được dùng để chèn khóa SSH vào `/root/.ssh/authorized_keys`. Khóa này cho phép kẻ tấn công đăng nhập trực tiếp vào tài khoản **root**. |
| **Ý nghĩa Pháp chứng** | Xác định một **cửa hậu (backdoor)** có độ tin cậy cao, cho phép kẻ tấn công duy trì quyền truy cập cấp root.

### 6. Cấu hình Tài khoản Người dùng (User Account Configuration)

| Mục | Chi tiết |
| :--- | :--- |
| **Định nghĩa & Vai trò** | Cấu hình trong `/etc/passwd` và `/etc/shadow`. Kẻ tấn công sửa đổi chúng để tạo user backdoor với đặc quyền cao. |
| **Công cụ Trích xuất** | `cat /etc/passwd` và `cat /etc/shadow`. |
| **Chỉ dấu Quan trọng** | **Tài khoản `gnats`:** User hệ thống được gán **shell đăng nhập (`/bin/bash`)** và có **UID 0** (root). |
| **Ý nghĩa Pháp chứng** | Chứng minh kẻ tấn công đã thực hiện **thao túng tài khoản (Account Manipulation)** để tạo một **tài khoản backdoor cấp root** khó bị phát hiện hơn. Việc sửa đổi lại shell và UID là cần thiết để khóa tài khoản này. |