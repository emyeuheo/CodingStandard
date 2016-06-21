## Relipa Server Flow

### Giả định
* Môi trường production và dev là chung 1 server
* Trường hợp production và dev khác server nhau thì cũng áp dụng được, nhằm tránh nhầm lẫn đường dẫn
* Dùng server AWS, SSH bằng user ec2-user

### Nguyên tắc
* Thư mục chứa source của production và dev phải khác nhau
* User MySQL/PostgreSQL của production và dev phải khác nhau
  * Chỉ có User SQL của production mới có quyền với DB production
  * User SQL của dev có quyền với dev  

### Chuẩn bị

#### MySQL
1. Dùng tài khoản `root`, tạo user `prod_{service_name}`
```
CREATE USER 'prod_{service_name}'@'localhost' IDENTIFIED BY 'password';
```
`password` nên được tạo ra ngẫu nhiên, ví dụ sử dụng http://passwordsgenerator.net

1. Tạo DB cho production và gán quyền
```
CREATE DATABASE {service_name};
GRANT ALL PRIVILEGES ON `{service_name}` . * TO '{service_name}'@'localhost';
```
2. Tạo user `dev_{service_name}`
```
CREATE USER 'dev_{service_name}'@'localhost' IDENTIFIED BY 'password';
```
3. Tạo DB cho dev và gán quyền
```
 CREATE DATABASE dev_{service_name};
GRANT ALL PRIVILEGES ON `dev_{service_name}` . * TO 'dev_{service_name}'@'localhost';
```
4. Flush
```
FLUSH PRIVILEGES;
```

### Server

1. Tạo source code folder cho production và set quyền
```
sudo mkdir /var/www/vhosts/{service_name}
sudo chmod -R ec2-user:nginx /var/www/vhosts/{service_name}/*
sudo chown -R g+xw /var/www/vhosts/{service_name}/*
```
2. Tạo source code folder cho dev và set quyền
```
sudo mkdir /var/www/devhosts/dev_{service_name}
sudo chmod -R ec2-user:nginx /var/www/vhosts/dev_{service_name}/*
sudo chown -R g+xw /var/www/vhosts/dev_{service_name}/*
```
3. Sau đó setting lại cấu hình `nginx` cho phù hợp
