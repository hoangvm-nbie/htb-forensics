**Challenge:** Redtrails - Hack The Box


---
## 1. Artifact được đề cập trong Challenge
###  1. 🌐 Artifact Mạng (Network Artifacts)

#### 1.1. TCP/IP & RESP Packet
| Mục | Nội dung |
|-----|----------|
| **Định nghĩa & Vai trò** | Giao thức TCP/IP và Redis Serialization Protocol (RESP) được sử dụng trong truyền tải dữ liệu giữa kẻ tấn công và hệ thống. |
| **Công cụ & Cách trích** | Wireshark – Follow TCP Stream để trích toàn bộ luồng dữ liệu. |
| **Chỉ dấu quan trọng** | IP, Port, timestamp, luồng dữ liệu bất thường. |
| **Ý nghĩa pháp chứng** | Xác định thời điểm, nguồn gốc và phương thức truy cập trái phép. |

#### 1.2. Bulk String (Chứa ELF)
| Mục | Nội dung |
|-----|----------|
| **Định nghĩa & Vai trò** | Bulk String chứa file ELF – mã độc được truyền qua Redis. |
| **Công cụ & Cách trích** | Wireshark (RESP), Hex Editor. |
| **Chỉ dấu quan trọng** | Ký tự `$`, độ dài lớn, magic number ELF `\x7fELF`. |
| **Ý nghĩa pháp chứng** | Chứng minh kẻ tấn công sử dụng Redis để truyền payload độc hại. |

#### 1.3. Mã Base64 / eval
| Mục | Nội dung |
|-----|----------|
| **Định nghĩa & Vai trò** | Payload giai đoạn đầu, mã Base64 được thực thi qua eval. |
| **Công cụ & Cách trích** | Wireshark, Hex Editor, CyberChef. |
| **Chỉ dấu quan trọng** | Lệnh `eval` hoặc `system`, chuỗi Base64 dài. |
| **Ý nghĩa pháp chứng** | Bằng chứng thực thi lệnh từ xa (RCE). |

---

### 2. 💻 Artifact Mã Nhị Phân & Hệ Thống (Binary & System Artifacts)

#### 2.1. File ELF Độc Hại
| Mục | Nội dung |
|-----|----------|
| **Định nghĩa & Vai trò** | File thực thi chính của mã độc được tải lên hệ thống. |
| **Công cụ & Cách trích** | Ghidra, IDA Pro. |
| **Chỉ dấu quan trọng** | Magic number `\x7fELF`, kích thước bất thường. |
| **Ý nghĩa pháp chứng** | Xác nhận mã độc được dùng để chiếm quyền thực thi. |

#### 2.2. Hàm DoCommand
| Mục | Nội dung |
|-----|----------|
| **Định nghĩa & Vai trò** | Hàm cốt lõi giải mã dữ liệu, xử lý payload tiếp theo. |
| **Công cụ & Cách trích** | Ghidra/IDA Pro Decompiler. |
| **Chỉ dấu quan trọng** | Hàm OpenSSL, chuỗi mã hóa hard-coded. |
| **Ý nghĩa pháp chứng** | Chứng minh logic hoạt động của mã độc. |

#### 2.3. Key & IV (AES-256-CBC)
| Mục | Nội dung |
|-----|----------|
| **Định nghĩa & Vai trò** | Key/IV phục vụ giải mã payload được mã hóa AES-256-CBC. |
| **Công cụ & Cách trích** | Phân tích mã giả C trong DoCommand. |
| **Chỉ dấu quan trọng** | Key 32 byte và IV 16 byte. |
| **Ý nghĩa pháp chứng** | Cho phép giải mã các payload tiếp theo để truy vết tấn công. |

#### 2.4. Payload Cuối Cùng
| Mục | Nội dung |
|-----|----------|
| **Định nghĩa & Vai trò** | Payload cuối sau khi giải mã — thường chứa lệnh cài ethminer. |
| **Công cụ & Cách trích** | CyberChef với Key/IV đã thu được. |
| **Chỉ dấu quan trọng** | Lệnh đào tiền ảo, địa chỉ ví. |
| **Ý nghĩa pháp chứng** | Xác nhận mục đích tấn công: Cryptojacking. |