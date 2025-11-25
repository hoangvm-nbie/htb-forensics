# Obsecure - Hack The Box  
**Độ khó:** Easy  

---

## 1. Giới thiệu Challenge  
- Hacker lợi dụng lỗi upload file tùy ý để đưa lên một webshell PHP (support.php). Dịch vụ HTTP được tắt và bạn có log tcpdump 2 phút trước đó. 

---

## 2. Mục tiêu  
- Phân tích log mạng để xác định các lệnh hacker đã thực thi và đánh giá mức độ hệ thống bị xâm phạm.
---

## 3. Phương pháp giải  
- Sau khi tải file zip ta nhận được 1 file Wireshark và 1 file PHP
  # Phần 1: Phân tích file PHP
- Sao chép file PHP vào https://www.onlinegdb.com/online_php_interpreter để chạy
![alt text](image.png)
- ở Dòng 6 hàm N thay thế FD sẽ cho dòng createfunction 
=> Dòng 9 sẽ tạo hàm N với code u và sau đó chạy hàm đó 
- Ta sẽ thử chạy hàm u trước 
![alt text](image-1.png)
![alt text](image-2.png)
- Chú ý 2 đoạn mã function x và preg-match cho ta thấy cách hoạt động của file php như sau
   --1. Khởi tạo Khóa và Hàm Mã hóa
- Các biến $k, $kh, $kf, $p là các chuỗi khóa (key) và định danh được sử dụng để mã hóa/giải mã và đánh dấu dữ liệu.
- Hàm x($t, $k) thực hiện phép toán XOR lặp (Repeating-key XOR). Phép toán này là đối xứng, có thể dùng cả để mã hóa và giải mã dữ liệu ($t) bằng khóa ($k).
   --2. Xử lý dữ liệu đến
- Đọc toàn bộ dữ liệu gửi đến qua HTTP POST bằng @file_get_contents("php://input"). 
- dùng hàm @preg_match để tìm một chuỗi con ($m[1]) nằm giữa hai chuỗi định danh $kh và $kf.Nếu tìm thấy dữ liệu hợp lệ:Giải mã: Dữ liệu $m[1] được giải mã theo chuỗi: Base64 -> XOR (bằng khóa $k$)  
- Thu thập kết quả: Bất kỳ đầu ra nào được tạo ra trong quá trình thực thi đều được thu thập bằng bộ đệm đầu ra (@ob_start(), @ob_get_contents()).
 --3. Gửi dữ liệu Phản hồi 
 - Mã hóa ngược lại theo chuỗi: Gzip 
 - In chuỗi phản hồi đã mã hóa ($r)  (dòng 21)

 => ta cần tìm input tức là  @file_get_contents("php://input") 

 # Phần 2: Phân tích file WireShark
 - Follow HTTP của stream 
 ![alt text](image-3.png)
 - Ta có đoạn mã input và output sau:
 input: 6f8af44abea0QKwu/Xr7GuFo50p4HuAZHBfnqhv7/+ccFfisfH4bYOSMRi0eGPgZuRd6SPsdGP//c+dVM7gnYSWvlINZmlWQGyDpzCowpzczRely/Q351039f4a7b5
 output: 0UlYyJHG87EJqEz66f8af44abea0QKxO/n6DAwXuGEoc5X9/H3HkMXv1Ih75Fx1NdSPRNDPUmHTy351039f4a7b5

 - Thay input trên vào @file_get ta có:
 ![alt text](image-4.png)
 - Không giống output bên trên
=> cần decrypt từ hàm x

- Ta sẽ đi xor ngược lại như sau:
![alt text](image-5.png)

- Từ đó ta có các kết quả sau:
+ Từ stream 1 ta có id và uid sau:
![alt text](image-6.png)
+ Từ stream 23 ta có:
![alt text](image-7.png)
+Stream 24
![alt text](image-8.png)
+ Stream 25:
![alt text](image-9.png)

Qua đó ta thấy được timeline là : lấy uid,id -> dùng ls/-lah/home để phân tích home -> chuyển địa chỉ đến /home/developer -> dump file pwdb.kdbx
-Lưu đoạn mã code vào file pwdb.kdbx
![alt text](image-11.png)

Bước 2: Đẩy file lên keypass2john và lấy hashcat
![alt text](image-12.png)

Bước 3 Dùng haschcat để crack được passwword
![alt text](image-13.png)
- Sau đó nhập pass và lấy flag
![alt text](image-14.png)
flag đã cho
HTB{pr0tect_y0_shellZ}

## 4. Kết luận
-Challenge này giúp ta thấy rõ quy trình phân tích một webshell/backdoor đã được mã hóa, kỹ thuật giải mã XOR lặp, cũng như kỹ năng tái tạo hành vi kẻ tấn công qua file pcap.
