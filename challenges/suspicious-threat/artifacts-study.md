# Nghiên cứu về các loại Bằng Chứng Số (Digital Artifacts)  
**Challenge:** Suspicious Threat – Hack The Box  



---

## 1. Khái niệm về Artifact trong Điều tra số

**Artifact (bằng chứng số)** là mọi dữ liệu được tạo ra trong quá trình một hệ thống, ứng dụng hoặc người dùng hoạt động.  
Trong điều tra số (digital forensics), artifact là các dấu vết giúp điều tra viên:

- Xác định hành vi của người dùng hoặc mã độc  
- Tái dựng lại chuỗi sự kiện  
- Phát hiện xâm nhập, rootkit, hoặc thao túng hệ thống  

### Các nhóm artifact phổ biến
| Nhóm | Mô tả | Ví dụ |
|------|-------|--------|
| **Hệ thống (System Artifacts)** | Dữ liệu phản ánh cấu trúc hệ điều hành, thư viện, tiến trình | `ld.so.preload`, `/etc/passwd`, `.bash_history` |
| **Tập tin & Thư viện (File / Binary Artifacts)** | File thực thi, thư viện động, script… có thể bị sửa đổi hoặc chèn mã độc | `libc.hook.so.6`, `libproc.so` |
| **Nhật ký (Log Artifacts)** | Dấu vết hoạt động được hệ thống ghi lại | `/var/log/auth.log`, `dmesg`, `journalctl` |
| **Bộ nhớ & tiến trình (Memory Artifacts)** | Phần dữ liệu trong RAM khi chương trình đang chạy | Dump tiến trình, thông tin process |
| **Người dùng (User Artifacts)** | Hoạt động, lệnh, file của người dùng | `~/.bash_history`, `Downloads/`, config cá nhân |

---

## 2. Artifact được đề cập trong Challenge *Suspicious Threat*

### 2.1. `/etc/ld.so.preload`
- **Loại:** System configuration artifact  
- **Mô tả:**  
  File cấu hình này cho phép nạp thêm thư viện động (*shared library*) trước khi chạy bất kỳ chương trình nào.  
  Kẻ tấn công thường lợi dụng nó để “preload” một thư viện độc hại (rootkit userland).  
- **Ý nghĩa điều tra:**  
  - Là chỉ dấu quan trọng để phát hiện rootkit cấp người dùng.  
  - Phân tích đường dẫn bên trong giúp xác định vị trí thư viện độc hại.  

---

### 2.2. `libc.hook.so.6`
- **Loại:** Binary / Library artifact  
- **Mô tả:**  
  Đây là thư viện giả mạo `libc.so.6` – một thư viện hệ thống chuẩn của Linux.  
  Trong challenge, kẻ tấn công thay thế liên kết thư viện này để giấu tiến trình và file (rootkit dạng preload).  
- **Kỹ thuật phân tích:**  
  - Sử dụng `ldd`, `readelf`, `strings` để xác định hành vi bất thường.  
  - Tính hash SHA256 để so sánh với bản gốc.  

---

### 2.3. `/var/pr3l04d_/flag.txt`
- **Loại:** User / File artifact  
- **Mô tả:**  
  Là file bị ẩn bởi rootkit, nằm trong thư mục bị thao túng (`pr3l04d_`).  
- **Ý nghĩa điều tra:**  
  - Chứng minh rõ việc rootkit đã được gỡ bỏ.  
  - Là bằng chứng kết luận về hành vi che giấu dữ liệu (anti-forensics).

---

## 3. Cách thu thập và bảo quản artifact
| Bước | Mục tiêu | Công cụ gợi ý |
|------|-----------|---------------|
| **1. Xác định và cô lập hệ thống** | Tránh bị rootkit xóa dấu vết | `ssh`, `netstat`, `ps aux` |
| **2. Thu thập file và thư viện liên quan** | Lưu bản sao không thay đổi | `cp`, `dd`, `tar` |
| **3. Phân tích thư viện và cấu hình** | Phát hiện hành vi hook | `ldd`, `strings`, `readelf`, `objdump` |
| **4. Ghi nhận hash & metadata** | Đảm bảo tính toàn vẹn bằng chứng | `sha256sum`, `file`, `stat` |

---

## 4. Kết luận

Trong challenge *Suspicious Threat*, ta gặp hai loại artifact điển hình:
1. **System Artifact:** `/etc/ld.so.preload`
2. **Binary Artifact:** `libc.hook.so.6`

Chúng đại diện cho kỹ thuật **Userland Rootkit Injection** – thao túng hành vi hệ thống bằng preload library.  
Hiểu và phân tích đúng các artifact này giúp điều tra viên:
- Phát hiện sự hiện diện của mã độc ở tầng userland  
- Khôi phục các hành vi bị che giấu  
- Củng cố chuỗi bằng chứng kỹ thuật trong báo cáo pháp chứng.

---
