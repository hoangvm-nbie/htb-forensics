**Challenge:** EMO - Hack The Box


---

##1. Artifact được đề cập trong Challenge
### Artifact 1 – Malicious Word Document (.doc/.docx)
1. Định nghĩa & Vai trò

File Word là vật chứng ban đầu trong challenge. Đây là vector tấn công chính, bên trong chứa macro độc hại được dùng để thực thi PowerShell khi người dùng mở file.

2. Cách trích xuất & Công cụ

oletools (oleid, olevba): phân tích cấu trúc OLE, xem có macro hay không.

ANY.RUN: chạy file trong sandbox để quan sát hành vi runtime.

oledump.py: trích macro thủ công.

3. Chỉ dấu & IOC quan tâm

File Word có macro auto-run.

Khi mở sẽ spawn PowerShell → dấu hiệu tài liệu độc hại.

File chứa mã hóa XOR payload.

4. Ý nghĩa pháp chứng

Là bằng chứng gốc cho thấy người dùng đã nhận và mở file lừa đảo.

Chứa macro → chứng minh ý đồ thực thi mã độc.

Dùng để truy xuất nguồn gốc tấn công và tái tạo hành vi tấn công.
### Artifact 2 – Macro VBA / Embedded Script
1. Định nghĩa & Vai trò

Macro VBA là đoạn script được nhúng trong file Word, đóng vai trò loader, chứa mã độc PowerShell bị obfuscate. Khi nạn nhân mở tài liệu, macro sẽ kích hoạt.

2. Cách trích xuất & Công cụ

olevba file.doc → trích toàn bộ macro.

ANY.RUN → tự động hiển thị macro và chuỗi code.

oledump.py -V → liệt kê stream macro.

3. Chỉ dấu & IOC quan tâm

Tên biến bị obfuscate:

$FN5ggmsH, $Odb3hf3, $Zhcnaux


Sử dụng backticks trong PowerShell:

"dO`WnLOA`dfILe"


Macro thực thi PowerShell — hành vi cực kỳ đáng ngờ.

4. Ý nghĩa pháp chứng

Giải thích cơ chế tấn công nội tại của file Word.

Giúp tái hiện quá trình decode payload.

Cung cấp bằng chứng kỹ thuật để xây dựng YARA rule phát hiện macro malware.

### Artifact 3 – Obfuscated PowerShell Payload
1. Định nghĩa & Vai trò

Đây là đoạn PowerShell được sinh ra bởi macro. Code bị obfuscate bằng backticks, trick string, XOR…
Vai trò của nó là giải mã payload ẩn, trong challenge chính là flag.

2. Cách trích xuất & Công cụ

Copy từ ANY.RUN hoặc từ macro VBA.

Làm sạch code → chạy thử trong PowerShell.

Dùng CyberChef để giải mã XOR các dãy byte.

3. Chỉ dấu & IOC quan tâm

XOR 0xDF → kỹ thuật mã hóa payload:

([byte][char]$_ -bxor 0xdf)


Dữ liệu mã hóa dưới dạng mảng số:

(186,141,228,...)
(185,179,190,...)


Hành vi đọc byte và build chuỗi vào biến $FN5ggmsH.

4. Ý nghĩa pháp chứng

Chỉ ra payload cuối cùng mà attacker muốn ẩn.

Giúp phân tích được nội dung thực (flag trong bài, C2 hoặc shellcode trong thực tế).

Cho phép xác định mức độ nguy hiểm và kỹ thuật obfuscation được sử dụng.