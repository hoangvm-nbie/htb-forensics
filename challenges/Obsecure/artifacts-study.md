**Challenge:** Obsecure - Hack The Box


---
## 1. Artifact được đề cập trong Challenge

### 1.1 Artifact A — File PCAP / HTTP Streams (Network Artifact)
1. Định nghĩa & vai trò

Định nghĩa:
File PCAP ghi lại toàn bộ traffic mạng trong thời điểm xảy ra sự kiện. Trong challenge này, PCAP chứa một chuỗi các HTTP streams, trong đó attacker sử dụng POST request để truyền payload đã được mã hóa nhằm giao tiếp với webshell.

Vai trò forensics:
PCAP là bằng chứng mạng quan trọng nhất vì nó:

Lưu lại các gói tin HTTP chứa payload mã hóa.

Cho phép khôi phục toàn bộ lệnh attacker gửi qua webshell (uid, id, ls, dump file...).

Tái dựng timeline dựa trên timestamp trong từng stream.

Xác định rõ nguồn – đích – nội dung trao đổi (C2 traffic).

2. Cách trích & công cụ

GUI – Wireshark:

File → Open → chọn pcap

Chuột phải một packet → Follow → HTTP Stream

Chuyển sang Raw/ASCII để xem payload

Export stream để phân tích nội dung
3. Chỉ dấu & IOC cần quan tâm

POST requests có payload dài và Base64-like.

Encoded payload giữa hai marker đặc biệt (kh/kf).

Không có tham số HTTP thông thường → dấu hiệu C2.

Timestamp cho thấy chuỗi lệnh liên tiếp phục vụ lateral movement.

HTTP replies nén gzip, cấu trúc bất thường.

4. Ý nghĩa pháp chứng

Cho thấy rõ attacker sử dụng webshell thông qua HTTP POST.

Cho phép giải mã và tái dựng toàn bộ lệnh đã chạy.

Chứng minh file pwdb.kdbx đã bị exfiltrate.

Là nền tảng để kết nối các artifact khác (PHP payload, XOR decoding, timeline).
### 1.2 Artifact B — Backdoor PHP File (Malicious Code Artifact)
1. Định nghĩa & vai trò

Định nghĩa:
File PHP trong challenge là malware/backdoor dùng để nhận lệnh từ attacker. Nó sử dụng XOR + Base64 + gzip để che giấu dữ liệu và thực thi payload thông qua hàm createfunction.

Vai trò forensics:

Giải thích cơ chế mã hóa của traffic bên trong PCAP.

Là mẫu mã độc giúp giải mã tất cả HTTP payloads.

Chứng minh server bị cấy webshell → pivot point để attacker điều khiển.

2. Cách trích & công cụ
Xem code:
Editor (VSCode)
Online PHP sandbox (OnlineGDB, 3v4l.org)
Reverse XOR:
Sử dụng script Python hoặc chạy trực tiếp trong PHP.

Bảo toàn mẫu malware:

Copy file dạng read-only

Hash SHA-256 để phục vụ chain-of-custody

3. Chỉ dấu & IOC
Hàm x() (XOR lặp) — dấu hiệu obfuscated C2.
Sử dụng preg_match() để tách dữ liệu giữa hai marker ($kh, $kf).
createfunction() — hàm nguy hiểm dùng để thực thi mã tùy ý.
Base64 + Gzip — kỹ thuật che giấu payload.

4. Ý nghĩa pháp chứng
Là artifact giúp giải mã toàn bộ traffic trong PCAP.
Cho phép tái dựng chính xác nội dung attacker gửi → phục vụ timeline.
Dùng làm bằng chứng malware implant trên hệ thống.
### 1.3 Artifact C — KeePass Database (pwdb.kdbx)
1. Định nghĩa & vai trò
Định nghĩa:
pwdb.kdbx là file cơ sở dữ liệu mật khẩu KeePass mà attacker đã truy cập và dump từ /home/developer. Đây là file bị đánh cắp trong quá trình tấn công.
Vai trò forensics:
Cho thấy dấu hiệu rõ ràng của data exfiltration.
Là mục tiêu cuối cùng mà attacker nhắm tới.
Dùng để kiểm chứng việc attacker có truy cập được thông tin nhạy cảm hay không.
2. Cách trích & công cụ

Trích từ PCAP:

Follow HTTP Stream → Save payload → lưu đúng dạng binary .kdbx

Crack mật khẩu:
Convert KDBX → John format
Mở file:
KeePassXc
KeePass2
3. IOC cần quan tâm
Tên file và đường dẫn: /home/developer/pwdb.kdbx
Dung lượng file trong payload HTTP
Hash của file để đối chiếu: SHA-256
4. Ý nghĩa pháp chứng
Xác nhận attacker truy cập và đánh cắp dữ liệu nhạy cảm.

Việc crack thành công mật khẩu chứng minh integrity của quá trình phân tích.

Lấy flag từ file giúp xác nhận chuỗi tấn công hoàn chỉnh.
