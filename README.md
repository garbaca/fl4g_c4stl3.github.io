# fl4g_c4stl3.github.io
fl4g_c4stl3 never die
## WARMUP
![alt text](image-1.png)

![alt text](image-2.png)
Đầu tiên cần đăng ký cho mình một tài khoản

Tiếp theo là tạo thử 1 post xem sao
![alt text](image-3.png)
![alt text](image-4.png)
Có vẻ như phàn `content` dính **XSS**
Còn thêm có nút `Report to admin` nữa nên ban đầu mình đã đi theo hướng steal cookie
Sau một hồi làm không được thì mình nhìn lại đề, `Flag nằm ở post của admin` và nhìn lên url thì thấy khả năng cao là dính **IDOR**

url: `http://159.223.41.73:8087/posts/view/admin`
![alt text](image-6.png)

Flag: ~~**ISPCTF{131593365355344022914563712420631936324}**~~
