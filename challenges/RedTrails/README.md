# Obsecure - Hack The Box  
**Độ khó:** Easy  

---

## 1. Giới thiệu Challenge  
- Đội SOC phát hiện hoạt động bất thường trên một Redis instance dù đã được bảo vệ bằng mật khẩu. Điều này cho thấy kẻ tấn công đã lợi dụng một kỹ thuật hoặc lỗ hổng để truy cập trái phép

---

## 2. Mục tiêu  
- Tìm hiểu cách attacker truy cập vào Redis.
- Phân tích các dấu vết và hành vi bất thường.
- Thu thập đủ 3 phần flag.
---

## 3. Phương pháp giải  
 ### Bước 1: Phân tích file capture
 - Ở stream 1 thấy 1 đoạn mã có chuỗi base64:
 ![alt text](image.png)
 - Có các chuỗi bị ngắt và ở cuối có eval
 -> ta sẽ phải reverse rồi mới decode:
 ![alt text](image-1.png) ![alt text](image-2.png)
 - Copy mã được file reverse
 - Đọc mã 2 hàm ta thấy khi ghép kí tự mỗi hàm vào và đổi sang base64 nhận được:
 ![alt text](image-3.png)  ![alt text](image-4.png)
 - ta đã nhận được phần 1 của flag: HTB{r3d15_1n574nc35
 ### Bước 2: Tìm flag trong RESP packet
 - Sau 1 hồi mò tìm được 1 phần flag ở packet 35
 ![alt text](image-5.png)
 FLAG_PART:_c0uld_0p3n_n3w
 ### Bước 3:
 - Trong khi tìm ở bước 2 ta thấy có 1 file có kichd thước bất thường:
 ![alt text](image-6.png)
 - Hơn nữa nó còn là binary ELF qua đoạn 7f 45 4c 46
 - Dịch ngược hàm elf ở ghidra được:
 ![alt text](image-7.png)
 - Thấy khóa h02B6aVgu09Kzu9QTvTOtgx9oER9WIoz và vecto YDP7ECjzuV7sagMN đã được tìm ra
 - Việc còn lại cần tìm ra đoạn code để decrypt:
 ![alt text](image-8.png)
- ở đây có x10SHp có thể là đoạn mã dài dưới kia thử với cyberchef:
![alt text](image-9.png)
tìm được đoạn flag cuối cùng:
FLAG_PART=_un3xp3c73d_7r41l5!}

-> giải thành công

## 4. Kết luận
- Cuộc tấn công lợi dụng lỗ hổng truy cập trái phép vào Redis để thực thi mã độc theo hai giai đoạn:

+ Giai đoạn 1: Thực thi mã lệnh che giấu (eval Base64) để tải xuống binary độc hại.

+ Giai đoạn 2: Binary ELF được giải mã (dùng Key/IV cứng từ hàm DoCommand) để lộ lệnh cài đặt ethminer.

- Kết luận thực tế: Sự cố này là một cuộc tấn công Cryptojacking (khai thác tiền điện tử). Nó nhấn mạnh sự cần thiết phải giới hạn truy cập mạng và vô hiệu hóa các lệnh nguy hiểm trong các Redis instance để ngăn chặn việc thực thi mã từ xa.