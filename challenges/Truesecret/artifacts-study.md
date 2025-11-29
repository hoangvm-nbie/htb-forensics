**Challenge:** TrueSecret - Hack The Box


---

## 1. Artifact được đề cập trong Challenge
### 1.File System Artifacts (Hệ thống Tệp) 
 1. Định nghĩa & Vai trò
 a) Định nghĩa: 
 - File System Artifacts là các bằng chứng nằm trên hệ thống lưu trữ (ổ cứng), không phải trong RAM hay Network. Chúng bao gồm các tệp dữ liệu được tạo ra, truy cập hoặc sửa đổi trong quá trình tấn công
 b) Vai trò Forensics:
 + Chứa Payload: Lưu trữ các tệp mã độc, công cụ tấn công (như $7zFM.exe$, $DumpIt.exe$) và đặc biệt là tệp dữ liệu bị mã hóa ($archive.zip$, $backup.dat.tc$).
 + Chứng minh Hành động: Metadata của tệp (MAC times: Modified, Accessed, Created) cho biết khi nào kẻ tấn công tương tác với tệp.
 + Tìm kiếm Dữ liệu Bí mật: Là nơi cuối cùng chứa Volume mã hóa TrueCrypt và sau đó là Mã nguồn C2 sau khi giải mã.
 2. Cách trích & Công cụ
 - Các artifact này được trích xuất từ hình ảnh ổ đĩa vật lý (disk image) hoặc trong trường hợp này, thông qua việc giải nén và mount volume:
 + Tệp Nén (archive.zip): Trích xuất từ File System của Image hoặc được xác định qua $cmdline$.
 + Volume TrueCrypt (backup.dat.tc): Tệp nhị phân được tìm thấy sau khi giải nén.
 + Mã nguồn C2 (.cs files): Được trích xuất sau khi mount thành công tệp $backup.dat.tc$ bằng mật khẩu khôi phục từ RAM.
 3. Chỉ dấu & IOC cần quan tâm
 + Tên tệp đáng ngờ: $backup.dat$, $TrueCrypt.exe$, tên tệp Mã nguồn C2.
 + Phần mở rộng lạ: $.tc$ (TrueCrypt Volume).Đường dẫn: $C:\Users\User\Documents\Backup\archive.zip$ (chỉ ra vị trí lưu trữ).
 + MAC Times: Nếu thời gian tạo ($Creation Time$) của $archive.zip$ trùng khớp với thời gian $7zFM.exe$ được chạy trong $cmdline$ của RAM.
 4. Ý nghĩa Pháp chứngBằng chứng Hiện hữu: 
 + Chứng minh dữ liệu nhạy cảm (Mã nguồn C2) đã được lưu trữ trên máy tính của thủ lĩnh APT.
 + Tái tạo Kịch bản: Cho thấy chuỗi hành động rõ ràng: Mã hóa $\rightarrow$ Nén $\rightarrow$ Lưu trữ.
 + Chứng minh Giả định: Việc tìm thấy tệp $backup.dat.tc$ xác nhận giả định về việc sử dụng TrueCrypt, điều này củng cố tính chính xác của Artifact RAM (TrueCrypt Passphrase).

 ### 2.Volatile Memory Artifacts (Bộ nhớ RAM)
 1. Định nghĩa & Vai trò
 - Định nghĩa: Volatile Memory Artifacts là các bằng chứng chỉ tồn tại khi hệ thống đang hoạt động và sẽ bị mất khi tắt nguồn. Chúng được thu thập thông qua file Memory Dump ($TrueSecrets.raw$).
 - Vai trò Forensics:
 + Khóa Mã hóa & Mật khẩu: RAM lưu trữ các khóa bí mật được sử dụng bởi các ứng dụng đang hoạt động (ví dụ: mật khẩu TrueCrypt).
 + Hoạt động Hiện tại: Cung cấp danh sách các tiến trình, kết nối mạng, các lệnh đã thực thi mà chưa được ghi vào ổ đĩa.
 + Dữ liệu Fileless: Có thể chứa các mã độc được tiêm vào tiến trình hoặc các script chỉ chạy trong bộ nhớ.
 2. Cách trích & Công cụ
 - Tất cả các artifacts này được trích xuất từ file $TrueSecrets.raw$.
 + Công cụ chính: Volatility Frameworkpslist/pstree: Trích xuất danh sách tiến trình ($TrueCrypt.exe$, $7zFM.exe$, $DumpIt.exe$).
 + cmdline: Trích xuất các đối số dòng lệnh đầy đủ (Full Command Line) cho tiến trình $7zFM.exe$.
 + truecryptpassphrase: Module chuyên biệt để quét các cấu trúc dữ liệu của TrueCrypt trong RAM và khôi phục mật khẩu.
 + filescan/hashdump: (Không dùng trong challenge này, nhưng là chức năng quan trọng) Trích xuất các handle tệp đang mở và hash mật khẩu hệ thống.
 3. Chỉ dấu & IOC cần quan tâm
 + Tiến trình Độc hại: $7zFM.exe$ (hoạt động trong ngữ cảnh đáng ngờ).
 + Key Material: Chuỗi mật khẩu TrueCrypt $X2Hk2XbEJqWYsh8VdbSYg6WpG9g7$.
 + Chuỗi Hoạt động: Chuỗi lệnh đầy đủ của $7zFM.exe$ (Artifact C4).
 4. Ý nghĩa Pháp chứng
 + Vô hiệu hóa Mã hóa: Việc khôi phục mật khẩu TrueCrypt từ RAM là bằng chứng quyết định cho phép nhà phân tích vượt qua lớp bảo mật mạnh mẽ nhất.
 + Chứng minh Thời điểm: Dữ liệu RAM (tiến trình đang chạy) xác nhận thời điểm hành động cuối cùng của đối tượng (chạy $7zFM.exe$) đã xảy ra ngay trước khi bị thu giữ.
 + Khôi phục Dữ liệu Cố định: Các artifacts trong RAM (như chuỗi lệnh) cung cấp các thông tin liên kết trực tiếp đến các artifact trên ổ đĩa (như tên file $archive.zip$), tạo ra một chuỗi bằng chứng không thể chối cãi.