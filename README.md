<img width="1919" height="655" alt="image" src="https://github.com/user-attachments/assets/5bba8a2f-1755-4507-adcb-aa7a10285ed3" /># WU-26-27-12-25

Có 3 bài và mình làm được bài 1 và bài 3

## Bài 1:

<img width="671" height="280" alt="image" src="https://github.com/user-attachments/assets/a82964aa-e2f4-489d-9e65-67c3e4e9ad84" />

Thư mục bài 1 gồm các file sau, ta đoán file output.enc là kết quả khi chạy file src,py

Mở file src.py, ta được như sau:

<img width="1861" height="987" alt="image" src="https://github.com/user-attachments/assets/ac002be2-4746-4cfd-829a-cd19e2a689e1" />

Ở đây ta thấy file sẽ lấy đoạn flag ta cần tìm từ file flag.txt, cắt làm 3, rồi XOR 3 đoạn với nhau theo thứ tự trong đoạn mã

Vì tính chất của XOR là A xor B = B xor A; A xor B = C -> B xor C = A

Nên chúng ta chỉ cần viết đoạn mã để:
  - Lấy đoạn xâu/byte trong file output.enc
  - Chia làm 3
  - Xor chúng với nhau theo thứ tự ngược lại so với đoạn mã gốc

Từ đây, chúng ta viết mã giải như sau:

```
from pwn import *

with open('output.enc', 'rb') as (f):
    key = f.read()

a = key[0:len(key) // 3]
b = key[len(key) // 3:2 * len(key) // 3]
c = key[2 * len(key) // 3:]

c = xor(c, int(str(len(key))[0]) * int(str(len(key))[1]))
c = xor(b, c)
b = xor(a, b)
a = xor(c, a)
c = xor(b, c)
b = xor(a, b)
a = xor(a, int(str(len(key))[0]) + int(str(len(key))[1]))

flag = a + b + c

print (flag)
```

Output sẽ ra một chuỗi kí tự như thế này, giải mã base64 nhiều lần sẽ ra flag

<img width="1855" height="1008" alt="image" src="https://github.com/user-attachments/assets/631c5123-a9bb-4a3a-b217-d2f334aa7a77" />

## Bài 3:
Đến với bài 3, thứ chúng ta thấy là một file tên warmup.exe

Mở lên bằng IDA, thứ đầu tiên mình chú ý là biến v12 và v13 được khai báo ngay từ đầu

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/872ddc09-9dfb-41cb-b00e-1599a466591a" />

Đọc một số dòng ở dưới này

<img width="697" height="489" alt="image" src="https://github.com/user-attachments/assets/690ddcec-0fee-4547-98fc-b45998b411ac" />

Mình rút ra kết luận: Chương trình này lấy input người dùng và mã hóa nó, rồi so sánh với 1 key gốc. Nếu trùng nhau sẽ thông báo Correct.

Mình nghi ngờ các byte được khai báo ở v12 và v13 chính là key được mang đi so sánh với input. Sử dụng Little Endian, mình nối v12 và v13 lại thành một chuỗi như sau:

```
48 FC 88 86 89 AB 6E 8B 74 94 51 82 A4 51 DA A7 A0 EF 27 98 02 03 D3 E4 D6 B9 ED FA 51
```

Giờ đến lúc giải quyết bài này.

Bước 1: Tìm lệnh gọi hàm CryptEncrypt, đặt breakpoint cho nó

<img width="1518" height="830" alt="image" src="https://github.com/user-attachments/assets/18a1bfc6-20cb-456c-8c6e-7fef26fb45ad" />

Bước 2: Thiết lập môi trường cho nó và bắt đầu debug

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/bd1d04ec-162d-4a79-9eb4-708d4400a6ce" />

Bước 3: Vì chuỗi nối lại có 29 byte, vì vậy hãy nhập input 30 kí tự, có thể là 30 chữ A, rồi Enter

<img width="1919" height="655" alt="image" src="https://github.com/user-attachments/assets/9a38f86c-2b8a-434f-956f-8566ba4c0321" />

Nhìn ở đây ta thấy được chuỗi 30 chữ A này được lưu ở thanh RBX

<img width="572" height="467" alt="image" src="https://github.com/user-attachments/assets/11708f3e-9eda-4fc6-806a-3c2326e9dcb2" />

Dưới cửa sổ Hex View, chuột phải rồi chọn Synchronize with -> RBX

Ở đây ta thấy được đoạn hex:

<img width="656" height="158" alt="image" src="https://github.com/user-attachments/assets/79bc9ea1-06fa-47e8-a898-1b7c6474f2be" />

Bước 4: Sửa chúng thành đoạn byte ta vừa tìm được, bỏ qua byte 41 cuối vì chuỗi byte ghép được có 29 byte

<img width="955" height="124" alt="image" src="https://github.com/user-attachments/assets/55ea71b1-eed4-474c-bd14-5d02117ed598" />

Bước 5: Nhấn F8 một lần nữa ra kết quả như sau:

<img width="655" height="142" alt="image" src="https://github.com/user-attachments/assets/2d7b730c-6c55-46c8-b0af-45f747d6d5e4" />

Cộng với tên file, ta suy luận được flag sẽ là CIS2024{900dw0rk_foR_w4RmUp}

Bước 6: Mở cmd lên để chạy chương trình với input là flag vừa tìm được

<img width="416" height="73" alt="image" src="https://github.com/user-attachments/assets/b93333f6-c3f8-438f-8b56-3e73acc8a11f" />

Hoàn thành bài
