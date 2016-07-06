## Relipaサーバフロー

### 前提
* 本番環境と開発環境は一緒、あるいは異なる環境でも構いません。ソースパースのミスを避けるため
* サーバ：AWS
* Webサーバ：nginx

### 原則
* 本番環境のソースフォルダと開発環境のは異なること
* 本番環境のSQLのユーザと開発環境のユーザは異なること
  * 本番DBへアクセス権限は本番ユーザしかない
  * 開発DBとステージングDBへアクセス権限は開発ユーザしかない
* 開発サイトはBasic認証をインストールすること。グーグルのインデックスのミスを避けるため、情報リークもさけること
* エンドユーザに公開済みのサイトは、ステージング環境が必須

### 準備
#### MySQL

1. セキュリティを高め
 ```
 mysql_secure_installation
 ```

1. brochute.comというサイトを構築中の前提
1. `root`というユーザを使い, 本番用のユーザ`pro_brochute`を作成
 ```
 CREATE USER 'pro_brochute'@'localhost' IDENTIFIED BY '174ed9865123654f2b6211dd3a2ab42b';
 ```
 パスワードはランダムに作成されること。例えば、`openssl rand -hex 16`というコマンドを使う

1. 本番用のDBを作成して権限与える
 ```
 CREATE DATABASE pro_brochute;
 GRANT ALL PRIVILEGES ON `pro_brochute` . * TO 'pro_brochute'@'localhost';
 ```
1. 開発用とステージング用のDBとユーザを作成。共有のユーザを使う

 ```
 CREATE USER 'dev_brochute'@'localhost' IDENTIFIED BY 'd7b27090b88403c573b18a5c5f38c135';
 CREATE DATABASE dev_brochute;
 CREATE DATABASE sta_brochute;
 GRANT ALL PRIVILEGES ON `dev_brochute` . * TO 'dev_brochute'@'localhost';
 GRANT ALL PRIVILEGES ON `sta_brochute` . * TO 'dev_brochute'@'localhost';
 ```

1. クリア

 ```
 FLUSH PRIVILEGES;
 ```

### サーバ

1. 本番用のソースフォルダを作成して、権限与える。`nginx`というグループに権限与える

 ```
 sudo mkdir -p /var/www/vhosts/pro_brochute
 sudo chown -R ec2-user:nginx /var/www/vhosts/pro_brochute/*
 sudo chmod -R g+rw /var/www/vhosts/pro_brochute/*
 ```
1. 開発用とステージング用のソースフォルダを作成して、権限与える

 ```
 sudo mkdir -p /var/www/devhosts/dev_brochute /var/www/devhosts/sta_brochute
 sudo chown -R ec2-user:nginx /var/www/devhosts/dev_brochute/*
 sudo chown -R ec2-user:nginx /var/www/devhosts/sta_brochute/*
 sudo chmod -R g+rw /var/www/devhosts/dev_brochute/*
 sudo chmod -R g+rw /var/www/devhosts/sta_brochute/*
 ```
1. `nginx`の設定を調整する
