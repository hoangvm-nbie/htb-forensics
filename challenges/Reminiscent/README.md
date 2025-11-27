# Reminiscent - Hack The Box  
**Độ khó:** Easy  

---

## 1. Giới thiệu Challenge  
-Một máy tính ảo (VM) của nhân viên tuyển dụng đã bị nhiễm mã độc, có khả năng thông qua một email lừa đảo (phishing email) được ngụy trang thành một hồ sơ xin việc
---

## 2. Mục tiêu  
- Điều tra nguồn gốc của mã độc để tìm flag.
---

## 3. Phương pháp giải
- Sau khi tải file được cấp ta được 3 file:![alt text](image.png)
+ flounder-pc-memdump.elf:
Là một bản sao (dump) của toàn bộ bộ nhớ truy cập ngẫu nhiên (RAM) của máy tính ảo (flounder-pc) tại thời điểm được ghi lại. 
Định dạng .elf (Executable and Linkable Format) thường được sử dụng cho các tệp nhị phân trên hệ thống Linux, nhưng cũng có thể được dùng để lưu trữ dữ liệu bộ nhớ.
+ imageinfo.txt
![alt text](image-1.png)
tệp đầu ra (output) từ công cụ Volatility Framework sau khi nó phân tích tệp memdump.
+ Resume.eml:
Là bản sao lưu hoàn chỉnh của email mà nhân viên tuyển dụng đã nhận được, được lưu ở định dạng EML
   ### Bước 1. Phân tích file resume.eml:
![alt text](image-2.png)
- Email lừa nhân viên click vào file Resume.zip được host ở địa chỉ http://10.10.99.55:8080/resume.zip
-> cần phân tích file Resume.zip 
   ### Bước 2. Phân tích file flounder-pc-memdump.elf:
- Dùng volatility3 xem các plugin của file ta có:
![alt text](image-3.png)
- DÙng plugin windows.filescan.Filescan để tìm ra địa chỉ file resume.zip
![alt text](image-4.png)
-> lúc này artifact chỉ còn là file resume.pdf.lnk

- DÙng windows.dumpfiles.DumpFiles dump file đó ta có
![alt text](image-5.png)
- Dùng strings tìm các keywword liên quan
![alt text](image-6.png)
- có 1 đoạn base64 encoded string
-> dùng cyberchef để decode: dùng deocde text UTF-16LE có
![alt text](image-7.png)
tiếp tục decode đoạn dưới powershell ta có
![alt text](image-8.png)
- vậy là đã thu được flag

## 4. Kết luận
-Thử thách này cho thấy tầm quan trọng của việc phân tích bộ nhớ trong việc khám phá các hoạt động của mã độc đã được xóa khỏi ổ đĩa cứng, đặc biệt là khi chúng sử dụng các kỹ thuật không ghi vào ổ đĩa (fileless) hoặc mã hóa phức tạp.