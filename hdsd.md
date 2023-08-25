Install EMD-ADminTool

Created Wednesday 26 July 2023
Hướng dẫn deploy lavarel: https://laravel.com/docs/10.x/deployment

Cài nginx  # https://www.nginx.com/resources/wiki/start/topics/tutorials/install/
#vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
#yum install nginx-1.14.1
#vim /etc/nginx/conf.d/test.conf					#tạo 1 file cấu hình mới cho lavarel
Nội dung:
server {
	listen 80;
	listen [::]:80;
		server_name example.com;				                 #Tên web truy cập vào LVREL'
	   root /usr/share/nginx/emd-admin-master/public;		#Move code vào thư mục /usr/share/nginx

	add_header X-Frame-Options "SAMEORIGIN";
	add_header X-Content-Type-Options "nosniff";

	index index.php;

	charset utf-8;

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

	location = /favicon.ico { access_log off; log_not_found off; }
	location = /robots.txt  { access_log off; log_not_found off; }

	error_page 404 /index.php;

	location ~ \.php$ {
		fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;			#Tạo folder này để php-fpm work''**__#
		fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
		include fastcgi_params;
	}

	location ~ /\.(?!well-known).* {
		deny all;
	}
}
#mkdir -p /var/run/php
# chown -R nginx:nginx /usr/share/nginx/emd-admin-master							# Change own cho thư mục code
--------------------

Cài mysql 8.       #  https://www.mysqltutorial.org/install-mysql-centos/
# vim /etc/yum.repos.d/mysql-community.repo
nội dung:
# Enable to use MySQL 5.5
[mysql55-community]
name=MySQL 5.5 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.5-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=0
gpgcheck=0
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

[mysql-tools-preview]
name=MySQL Tools Preview
baseurl=http://repo.mysql.com/yum/mysql-tools-preview/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

[mysql-cluster-7.5-community]
name=MySQL Cluster 7.5 Community
baseurl=http://repo.mysql.com/yum/mysql-cluster-7.5-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

[mysql-cluster-7.6-community]
name=MySQL Cluster 7.6 Community
baseurl=http://repo.mysql.com/yum/mysql-cluster-7.6-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

[mysql-cluster-8.0-community]
name=MySQL Cluster 8.0 Community
baseurl=http://repo.mysql.com/yum/mysql-cluster-8.0-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql#

--------------------
#yum install mysql-community-server-8.0.34								Install mysql#
#service mysqld start													Start dich vụ#
#grep "A temporary password" /var/log/mysqld.log						lấy password gen mặc định#
#mysql_secure_installation												cài đặt secure cho mysql#
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
#service mysqld restart
#chkconfig mysqld on													#enable
#mysql -u root -p														#đăng nhập
mysql> show databases;												#check databases hiện có
mysql> CREATE DATABASE laravel;										#Tạo database tên laravelmysql> FLUSH PRIVILEGES;
mysql> FLUSH PRIVILEGES;											#update quyền
--------------------
Cài php, php-fpm và các exentions
#sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm	#Cài kho remi có các repo php mới#
#yum install yum-utils -y												#Cài công cụ quản lý yum#
#sudo yum-config-manager --disable remi-php54							#Disabled kho php54#
#sudo yum-config-manager --enable remi-php81							#enabled kho php 81#
#yum install php php-fpm php-{zip,ctype,curl,dom,fileinfo,filter,hash,mbstring,openssl,pcre,pdo,session,tokenizer,xml,mysql}
#cài php, php-fpm và các extension 
Sửa cấu hình php-fpm
#vim /etc/php-fpm.d/www.conf
user = apache  => user = nginx
group = apache => group = nginx
listen = 12.0.0.1:9000 => listen = /var/run/php/php8.1-fpm.sock
Uncomment các dòng sau:
;listen.owner = nginx
;listen.group = nginx
;listen.mode = 0660

# cd /usr/share/nginx/emd-admin-master/public									vào thư muc chứa code lavarel#
# vim test.php																Tạo file test php#
Nội dung: 
<?php

phpinfo();

?>

Truy cập vào  example.com/test.php 										#Check có hiển thị thông tin php là php có hoạt động
--------------------
Cai composer ver mới:  https://linux.how2shout.com/how-to-install-composer-on-ubuntu-22-04-20-04-lts/
#curl -sS https://getcomposer.org/installer -o composer-setup.php						Tải bộ cài composer về máy#
#sudo php composer-setup.php --install-dir=/usr/bin --filename=composer				Cài composer bằng bộ cài vừa tải về#
hoặc:
#sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
#composer --version																Check version composer#
Cài đặt các gói phụ thuộc:
#cd /usr/share/nginx/emd...										#thư mục chứa code
# composer install --optimize-autoloader --prefer-dist --no-dev			(xoa file composer.lock- nếu có trước khi chạy lệnh này)
Cấu hình .env:
# cp .env.example .env
#vim .env
Chỉnh các thông số sau:
./pasted_image.png
--------------------

APP_NAME="Email Mass Delivery"							Tên hiển thị ( lưu ý sau khi đổi tên cần chạy lệnh xóa cache php artisan config:cache)#
APP_URL=http://example.com								#Link web lavarel
DB_CONNECTION=mysql										#Điền các thông số để kết nối vào database
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel										#cần tạo database này trước
DB_USERNAME=root
DB_PASSWORD=Lovingu1%									#pass có kí tự đặc biệt cần cho vào ""
--------------------
#cd /usr/share/nginx/emd-admin-master						vào thư mục chưa code#
Generate an application key. Re-cache:
# php artisan key:generate
# php artisan config:cache

Run database migrations:
# php artisan migrate:refresh

Create a new user account:
# php artisan make:filament-user
=> điền các thông số của account được tạo để đăng nhập vào lavarel
#systemctl restart php-fpm
truy cập vào example.com/admin


Cấu hình transport sang 1 instance mới
cú pháp : "smtp:ip:port"
Sau khi chỉnh transport cần update thông số của client
![anh](https://imgur.com/a/jooZXFq)
