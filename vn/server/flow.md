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

1. Giả định ta có dự án cho website là http://relipasoft.com
1. Dùng tài khoản `root`, tạo user `pro_relipa`
 ```
 CREATE USER pro_relipa@localhost IDENTIFIED BY '174ed9865123654f2b6211dd3a2ab42b';
 ```
 `password` nên được tạo ra ngẫu nhiên, ví dụ sử dụng lệnh `openssl rand -hex 16`

1. Tạo DB cho production và gán quyền
 ```
 CREATE DATABASE pro_relipa;
 GRANT ALL PRIVILEGES ON pro_relipa . * TO pro_relipa@localhost;
 ```
1. Tạo DB và user cho dev và staging. Ở đây ta dùng chung 1 user

 ```
 CREATE USER dev_relipa@localhost IDENTIFIED BY d7b27090b88403c573b18a5c5f38c135;
 CREATE DATABASE dev_relipa;
 CREATE DATABASE sta_relipa;
 GRANT ALL PRIVILEGES ON dev_relipa . * TO dev_relipa@localhost;
 GRANT ALL PRIVILEGES ON sta_relipa . * TO dev_relipa@localhost;
 ```

1. Flush

 ```
 FLUSH PRIVILEGES;
 ```

### Server
#### Tạo source code folder
1. Tạo source code folder cho production và set quyền. Ở đây ta sẽ cho group nginx quyền đọc ghi

 ```
 sudo mkdir -p /var/www/vhosts/pro_relipa
 sudo chown -R ec2-user:nginx /var/www/vhosts/pro_relipa/
 sudo chmod -R g+rw /var/www/vhosts/pro_relipa/
 ```
1. Tạo source code folder cho dev và set quyền

 ```
 sudo mkdir -p /var/www/devhosts/dev_relipa /var/www/devhosts/sta_relipa
 sudo chown -R ec2-user:nginx /var/www/devhosts/dev_relipa/
 sudo chown -R ec2-user:nginx /var/www/devhosts/sta_relipa/
 sudo chmod -R g+rw /var/www/devhosts/dev_relipa/
 sudo chmod -R g+rw /var/www/devhosts/sta_relipa/
 ```
1. Thiết lập lại cấu hình `nginx` để nhận 2 folder trên ứng với dev và staging

#### Thiết lập Basic Auth cho dev và staging site
1. Tạo file chứa user và password
 ```
 sudo htpasswd -c /etc/nginx/.htpasswd relipa
 ```
1. Thiết lập lại cấu hình `nginx` bằng cách thêm 2 dòng sau vào file `.conf` của dev và staging
 ```
 auth_basic "Relipa Awesome";
 auth_basic_user_file /etc/nginx/.htpasswd;
 ```
