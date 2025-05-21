# WordPress-Privilege-Escalation

**Author:** Dang Duy Hau <br>
**Affiliation:** Hanoi University of Science and Technology <br>

Thank you, Dr. Vu Thanh Nam, for helping me develop this project.

## Set up environment

1. Cài WordPress thủ công trên Ubuntu

- Máy ảo VirtualBox (dùng Ubuntu 20.04LTS)
- Cài Apache, MySQL, PHP:
  sudo apt update && sudo apt install apache2 mysql-server php php-mysql libapache2-mod-php php-curl php-gd php-mbstring php-xml php-zip unzip wget -y
- Cấu hình MySQL:
  sudo mysql_secure_installation
- Tạo database:
  sudo mysql -u root -p
  CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
  CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongPass123!';
  GRANT ALL PRIVILEGES ON wordpress.\* TO 'wpuser'@'localhost';
  FLUSH PRIVILEGES;
  EXIT;

2. Tải WordPress và giải nén
   cd /tmp
   wget https://wordpress.org/latest.tar.gz
   tar -xvzf latest.tar.gz
   sudo mv wordpress /var/www/html/
   sudo chown -R www-data:www-data /var/www/html/
   sudo chmod -R 755 /var/www/html/

3. Cấu hình Apache Virtual Host (nếu cần)
