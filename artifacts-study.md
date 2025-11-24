**Challenge:** Fishy-HTTP - Hack The Box


---

##1. Artifact được đề cập trong Challenge
### Artifact 1: Malicious DOC File (diagnostic.doc / layoffs.doc)
1. Định nghĩa & vai trò

Đây là một tài liệu Office chứa OLE Object được dùng làm mồi nhử trong chiến dịch phishing.

Vai trò: artefact gốc khởi nguồn toàn bộ chuỗi tấn công, dẫn người dùng tới nội dung độc hại từ server.

2. Cách trích xuất & công cụ

Dùng oleid để xác định file có chứa OLE object nguy hiểm.

Dùng oleobj để extract các object và đường dẫn bên trong.

3. Chỉ dấu & IOC quan tâm

High-risk OLE object được oleid cảnh báo.

IOC file: diagnostic.doc, laysoff.doc.

4. Ý nghĩa pháp chứng

Cho phép xác định kỹ thuật tấn công dựa trên OLE redirect.

Cung cấp bằng chứng về file gốc attacker gửi đến nạn nhân.

### Artifact 2: OLE Embedded Object (URL Link)
1. Định nghĩa & vai trò

Là đối tượng nhúng bên trong file DOC, trỏ đến một URL độc hại trên máy chủ attacker.

Vai trò: dẫn nạn nhân đến payload thực sự.

2. Cách trích xuất & công cụ

oleobj diagnostic.doc để trích ra URL nhúng.

Inspect nội dung OLE để lấy đường dẫn tài nguyên.

3. IOC quan tâm

URL độc hại:

http://94.237.122.36:44671/laysoff.doc

http://94.237.122.36:44671/223_index_style_fancy.html

4. Ý nghĩa pháp chứng

Xác định máy chủ attacker điều khiển.

Là bằng chứng cho việc tài liệu không chỉ chứa nội dung mà còn kích hoạt hành vi truy cập từ xa.

### Artifact 3: HTML Payload từ Server
1. Định nghĩa & vai trò

File HTML trả về từ URL OLE chứa payload ẩn, cụ thể là đoạn mã Base64 được che giấu trong mã nguồn.

Vai trò: cung cấp payload script độc hại dùng trong giai đoạn tấn công tiếp theo.

2. Cách trích xuất & công cụ

Inspect source (F12) để tìm đoạn mã màu cam chứa Base64.

Dùng Base64 decoding để trích payload.

3. IOC quan tâm

HTML file: 223_index_style_fancy.html

Base64 encoded data bên trong HTML.

4. Ý nghĩa pháp chứng

Cho thấy server host nội dung độc hại thực sự.

Là bằng chứng rõ ràng của sự xáo trộn (obfuscation) để né phát hiện.

### Artifact 4: PowerShell Obfuscated Script
1. Định nghĩa & vai trò

Là đoạn mã PowerShell được tạo từ payload Base64, sử dụng string formatting obfuscation để che giấu chuỗi thật.

Vai trò: thực thi hoặc hiển thị nội dung ẩn (flag), mô phỏng kỹ thuật mã độc thường dùng trong thực tế.

2. Cách trích xuất & công cụ

Giải Base64 → lấy đoạn PowerShell.

Dán vào PowerShell để giải obfuscation.

3. IOC quan tâm

Định dạng script đặc trưng:

"{7}{1}{6}{8}{5}{3}{2}{4}{0}" -f ...

4. Ý nghĩa pháp chứng

Là bằng chứng về việc attacker cố tình che giấu payload.

Thể hiện kỹ thuật obfuscation phổ biến trong malware/phishing.

Kết quả giải mã dẫn trực tiếp đến flag và completion của case.