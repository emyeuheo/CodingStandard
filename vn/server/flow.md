## Relipa Server Flow

### Giả định
* Môi trường production và dev là chung 1 server
* Trường hợp production và dev khác server nhau thì cũng áp dụng được, nhằm tránh nhầm lẫn đường dẫn
* Dùng server AWS, SSH bằng user ec2-user
* Web server: nginx

### Nguyên tắc
* Thư mục chứa source của production và dev phải khác nhau
* User MySQL/PostgreSQL của production và dev phải khác nhau
  * Chỉ có User SQL của production mới có quyền với DB production
  * User SQL của dev có quyền với dev và staging  
* Các site dev cần phải cài đặt để Google không đánh index, hoặc cài đặt Basic Auth để tránh bị dòm ngó
* Bất kỳ site nào đang ở trạng thái công khai với user thì đều cần staging

### Chuẩn bị
#### MySQL

1. Tăng bảo mật cho MySQL
 ```
 mysql_secure_installation
 ```

1. Giả định ta có tên website là brochute.com
1. Dùng tài khoản `root`, tạo user `pro_brochute`
 ```
 CREATE USER pro_brochute@localhost IDENTIFIED BY '174ed9865123654f2b6211dd3a2ab42b';
 ```
 `password` nên được tạo ra ngẫu nhiên, ví dụ sử dụng lệnh `openssl rand -hex 16`

1. Tạo DB cho production và gán quyền
 ```
 CREATE DATABASE pro_brochute;
 GRANT ALL PRIVILEGES ON pro_brochute . * TO pro_brochute@localhost;
 ```
1. Tạo DB và user cho dev và staging. Ở đây ta dùng chung 1 user

 ```
 CREATE USER dev_brochute@localhost IDENTIFIED BY d7b27090b88403c573b18a5c5f38c135;
 CREATE DATABASE dev_brochute;
 CREATE DATABASE sta_brochute;
 GRANT ALL PRIVILEGES ON dev_brochute . * TO dev_brochute@localhost;
 GRANT ALL PRIVILEGES ON sta_brochute . * TO dev_brochute@localhost;
 ```

1. Flush

 ```
 FLUSH PRIVILEGES;
 ```

### Server

1. Tạo source code folder cho production và set quyền. Ở đây ta sẽ cho group nginx quyền đọc ghi

 ```
 sudo mkdir -p /var/www/vhosts/pro_brochute
 sudo chown -R ec2-user:nginx /var/www/vhosts/pro_brochute/
 sudo chmod -R g+rw /var/www/vhosts/pro_brochute/
 ```
1. Tạo source code folder cho dev và set quyền

 ```
 sudo mkdir -p /var/www/devhosts/dev_brochute /var/www/devhosts/sta_brochute
 sudo chown -R ec2-user:nginx /var/www/devhosts/dev_brochute/
 sudo chown -R ec2-user:nginx /var/www/devhosts/sta_brochute/
 sudo chmod -R g+rw /var/www/devhosts/dev_brochute/
 sudo chmod -R g+rw /var/www/devhosts/sta_brochute/
 ```
1. Sau đó setting lại cấu hình `nginx` cho phù hợp
