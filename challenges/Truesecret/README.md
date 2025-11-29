# TrueSecret - Hack The Box  
**Độ khó:** Easy  

---

## 1. Giới thiệu Challenge  
- Tình huống: Bắt giữ thủ lĩnh nhóm APT và thu được bản chụp bộ nhớ (memory dump) từ máy tính đang chạy của hắn.
- Vấn đề: Nhóm APT này sử dụng máy chủ Command & Control (C2) tùy chỉnh.
---

## 2. Mục tiêu  
- phân tích bản chụp bộ nhớ để tìm và trích xuất (hoặc tái tạo) mã nguồn, cấu hình, và giao thức giao tiếp của máy chủ C2 tùy chỉnh.
---

## 3. Phương pháp giải
 ### Bước 1. Phân tích file zip
- Unzip file được cấp ta được:
![alt text](image.png)

- Dùng file Truesecret.raw chỉ cho ta biết đó là 1 file data
-> dùng vol2 để phân tích:
![alt text](image-1.png)
- Trước hết ta thử với file thứ nhất Win7SP1x86_23418
- liệt kê các file bản sao trong đó bằng $ vol -f TrueSecrets.raw --profile=Win7SP1x86_23418 pslist
![alt text](image-2.png) ![alt text](image-3.png)
- Ta có thể thấy 1 số file khả nghi:
+ TrueCrypt.exe
+ DumpIt.exe: dùng để tạo file RAM
+ 7zFM.exe: có thể nén/giải nén gì đó
+ WiMPrvSE.exe: đôi khi bị giả mạo
- Ta sẽ xem các cmd line của chúng:
$ vol -f TrueSecrets.raw --profile=Win7SP1x86_23418 cmdline
![alt text](image-4.png)
- file 7zFm có 1 đường dẫn đến 1 file zip backup
- Trích xuất file zip ta có:
![alt text](image-5.png)
- Sau khi trích xuất vào file dump ta có 2 file dat và vacb 
![alt text](image-6.png)
- Sau khi unzip 2 file trên ta được file có đuôi .tc:
![alt text](image-7.png)
 ### Bước 2. Bẻ khóa file tc:
 - Dùng lệnh: vol -f TrueSecrets.raw --profile=Win7SP1x86_23418 truecryptpassphrase
![alt text](image-8.png)
 - Ta có pass của file tc là: X2Hk2XbEJqWYsh8VdbSYg6WpG9g7
 - Dùng veracrypt bản 1.25 để mở file tc ta được 1 đường dẫn vào file đó:
 ![alt text](image-9.png) ![alt text](image-10.png)
 - Vào file agent server có 1 đoạn code Encrypt lạ:
 ![alt text](image-11.png)
 - mã code này dùng để mã hóa (encrypt) một chuỗi bằng DES, dùng key và IV cố định.
 - Khi  truyền plaintext vào sẽ  nhận được ciphertext dạng Base64.
 - Giờ ta cần viết file decrypt để đưa file session còn lại vào:
 ![alt text](image-13.png)
 - Từ đó ta nhận được flag trong các mã code:
 ![alt text](image-12.png)

 ## 4. Kết luận

 - Thử thách này cho thấy chiến thuật che giấu dữ liệu của nhóm tấn công APT dựa trên hai lớp bảo mật: mã hóa Volume bằng TrueCrypt và sử dụng C2 tùy chỉnh với mã hóa DES cố định.
 - Tuy nhiên, đối tượng tấn công đã mắc một lỗi hoạt động nghiêm trọng: Mặc dù đã mã hóa tệp tĩnh, họ không đảm bảo xóa sạch dữ liệu nhạy cảm khỏi bộ nhớ RAM. Bằng chứng then chốt: Công cụ Volatility đã lợi dụng lỗ hổng này, thành công trích xuất mật khẩu TrueCrypt ($truecryptpassphrase$) từ RAM.Giá trị cốt lõi: Việc khôi phục mật khẩu cho phép nhà phân tích truy cập vào mã nguồn C2, từ đó đảo ngược thuật toán mã hóa DES và giải mã các phiên giao tiếp.
 - Rút ra: Phân tích pháp y bộ nhớ là phương pháp không thể thiếu để vô hiệu hóa các lớp mã hóa tĩnh, nhấn mạnh tầm quan trọng của việc thu thập memory dump ngay lập tức để truy tìm các "chìa khóa" và giao thức C2 bị bỏ sót trong RAM.