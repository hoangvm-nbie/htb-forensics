**Challenge:** Fishy-HTTP - Hack The Box


---

##1. Artifact được đề cập trong Challenge

###1.1 Artifact A — PCAP / HTTP stream (Network artifact)

1. Định nghĩa & vai trò

Định nghĩa: file PCAP lưu lại toàn bộ lưu lượng mạng (packet). Trong challenge, PCAP chứa nhiều luồng HTTP; một luồng HTTP mang theo payload HTML được attacker dùng để giấu dữ liệu (encoding/obfuscation).

Vai trò forensics: PCAP là nguồn chứng cứ thô — nó chứng minh dữ liệu di chuyển qua mạng, cung cấp timestamp, địa chỉ IP, user-agent, SNI/Host header, payload gốc. Quan trọng vì nó cho thấy kênh truyền (exfiltration, C2, payload transfer).

2. Cách trích & công cụ

GUI: Wireshark → File > Open → chọn packet → Analyze > Follow > HTTP Stream → chọn Show data as (ASCII/Raw) → Save as (body).

CLI: tshark/tcpdump

Liệt kê flows:
tshark -r fishy_http.pcap -qz conv,ip

Liệt kê HTTP hosts/URIs:
tshark -r fishy_http.pcap -Y http -T fields -e http.host -e http.request.uri

Lấy toàn bộ stream N:
tshark -r fishy_http.pcap -Y "tcp.stream eq N" -V > streamN.txt

Trích payloads: tshark -r fishy_http.pcap -Y http -T fields -e http.file_data > raw_payloads.txt

Lưu ý bảo toàn bằng chứng: luôn copy pcap gốc read-only, làm checksum (sha256sum fishy_http.pcap) và lưu manifest.

3. Chỉ dấu & IOC cần quan tâm

IP nguồn/đích, port, Host header, User-Agent khác thường.

HTTP method bất thường (POST chứa payload lớn).

Nội dung body có cấu trúc lặp/HTML tag lạ (dấu hiệu stego/encoding).

Thời gian (timestamps) để ghép timeline với các artifact khác.

4. Ý nghĩa pháp chứng

Chứng minh dữ liệu bị truyền/nhận qua network tại thời điểm cụ thể.

Cho phép tái dựng chuỗi sự kiện: ai/đâu/bao giờ gửi payload.

Nếu payload chứa direct evidence (flag/part), pcap là nguồn nguyên gốc để xác minh người thu gửi/nhận.

---

###1.2.Artifact B — Windows binary (.NET) (Binary / Static analysis artifact)
1. Định nghĩa & vai trò

Định nghĩa: file thực thi Windows được cung cấp trong ZIP. Phân tích tĩnh (Detect It Easy) cho thấy .NET assembly; decompile (dotPeek/dnSpy) tiết lộ logic mã hóa/giải mã (ví dụ mapping tag → hex).

Vai trò forensics: binary chỉ ra thuật toán mà attacker dùng để encode dữ liệu; giúp viết decoder chính xác, chứng minh “cơ chế” (attacker methodology).

2. Cách phân tích & công cụ

Phân loại file: Detect It Easy (DIE) để biết type (PE, .NET, obfuscation).

Decompile: dotPeek/dnSpy/ILSpy — tìm các class tên lạ, method xử lý HTML hoặc base conversion. Tìm mapping (dictionary, array) hoặc chuỗi hằng (hardcoded).

Static checks: strings, r2, ghidra (nếu C++) — tìm hardcoded domain/IP, key, hàm decode.

Ghi chép: chụp màn hình phần code quan trọng, copy đoạn mapping vào file text (tag_hex_mapping.txt), ghi chú dòng lệnh để reproduce.

3. Chỉ dấu & IOC cần quan tâm

Mapping tag → nibble/hex (vd: <cite> = 0, <h1> = 1, ...).

Bất kỳ domain/IP hoặc URL cứng trong string table (C2).

Thủ thuật obfuscation (string encryption, anti-decompile) — ghi chú mất mát/khó phân tích.

4. Ý nghĩa pháp chứng

Binary là bằng chứng về phương pháp: nó cho thấy attacker đã thiết kế encoding cụ thể — từ đó ta chứng minh rằng decode ta làm là cùng thuật toán người tấn công dùng.

Nếu binary chứa IOC (C2), đó là mỏ thông tin để kết nối chiến dịch.

---

###1.3.Artifact C — Intermediate encoded data (Extracted payloads / Derived data)
1. Định nghĩa & vai trò

Định nghĩa: các file/chuỗi được trích xuất từ payload gốc và đã trải qua xử lý tạm (ví dụ: lấy chữ cái đầu mỗi từ → chuỗi Base64 → giải Base64 → phần flag/fragment). Đây là artifact đã qua xử lý, còn gọi derived artifacts.

Vai trò forensics: những file này cho thấy quá trình chuyển đổi dữ liệu từ raw → intermediate → decoded. Chúng giúp chứng minh chain-of-custody và reproducibility.

2. Cách trích & công cụ

Trích từ HTTP body: copy text từ Follow HTTP Stream → lưu zup.txt.

Chạy script trích ký tự đầu: python3 extract_first_letters.py zup.txt > extracted_base64.txt.

Giải mã: base64 --decode extracted_base64.txt > part_flag.txt hoặc CyberChef.

Nếu tiếp tục cần decode bằng mapping: feed phần HTML vào decode_html.py (mapping từ binary) để có decoded_part.txt.

3. Chỉ dấu & IOC cần quan tâm

Kết quả intermediate có thể chứa padding/marker (ví dụ } hoặc HTB{ fragments), giúp xác nhận flag.

Kiểm tra encoding layers (Base64, hex, custom mapping) để không bỏ sót bước.

Lưu hash cho từng intermediate file để chứng minh không bị giả mạo.

4. Ý nghĩa pháp chứng

Cho thấy chuỗi xử lý từ raw → flag; scripts + intermediate files là bằng chứng phương pháp (reproducible).

Nếu ai đó tranh cãi với kết luận, bạn có thể tái tạo exact transformation bằng script + files.

---

###2. Kết luận

Trong challenge **Fishy HTTP** chúng ta đã gặp ba loại artifact then chốt:

1. **Network Artifact (PCAP / HTTP stream):** file `fishy_http.pcap` chứa các luồng HTTP; một HTTP stream chứa phần body (HTML) được attacker dùng làm container để ẩn dữ liệu.  
2. **Binary Artifact (Windows binary .NET):** file nhị phân đi kèm chứa logic mã hoá—decompile cho thấy một mapping `tag -> hex` dùng để giải mã payload HTML.  
3. **Derived / Intermediate Data:** các file text được trích từ HTTP body và các output của script (`zup.txt` → `extracted_base64.txt` → `part_flag.txt` → `decoded_part.txt`) biểu diễn chuỗi chuyển đổi từ dữ liệu thô đến fragment của flag.

Tập hợp các artifact này cho thấy một **kỹ thuật hai lớp**: attacker chèn payload vào traffic HTTP (network steganography/obfuscation), rồi dùng một chương trình (binary) để định nghĩa phương thức mã hoá/giải mã (tag-to-hex mapping). Phân tích chi tiết các artifact (PCAP stream, extracted payloads, scripts, và mã nguồn decompiled) cho phép ta:

- Xác nhận nguồn gốc và con đường truyền payload (network evidence);  
- Phục hồi thuật toán giải mã và tái tạo chính xác các bước decode (reproducible methodology);  
- Cung cấp chuỗi bằng chứng kỹ thuật rõ ràng để hỗ trợ báo cáo forensics.

**Khuyến nghị ngắn:**  
- Bảo toàn pcap gốc (read-only) và lưu checksum (sha256) để chứng minh tính toàn vẹn;  
- Lưu scripts và intermediate outputs (text, screenshots, mapping) để người khác có thể tái dựng quy trình;  
- Không công khai flag hoặc file nhị phân nhạy cảm trên repo public — thay bằng `HTB{REDACTED}` nếu cần;  
- Với góc độ phòng ngừa: giám sát traffic HTTP bất thường (payloads dài / nhiều HTML tags lạ), và xác minh binaries lạ bằng phân tích tĩnh trước khi chạy.
