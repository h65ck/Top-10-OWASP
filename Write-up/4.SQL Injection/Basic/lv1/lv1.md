# SQL INJECTION localhost:24001/basic/level1.php
### 1. Tổng quan
lv1 là website để login 
![alt text](image.png)
### 2. Phạm vi
### 3. Lỗ hổng
#### Description
localhost:24001/basic/level1.php cho phép thực hiện login
#### Impact
Kẻ xấu có thể thực thi 1 phần/toàn bộ SQL query trên server
#### Root-cause analysis

Trong mã nguồn `/src/basic/level1.php`, anh dev đã nối chuỗi SQL query với user input là biến  `$_POST["username"]` và biến `$_POST["password"]` mà không hề có sự kiểm tra.

    $sql = "SELECT username FROM users WHERE username='$username' AND password='$password'";
	$query = $database->query($sql);

`$row = $query->fetch_assoc()` sẽ lấy hàng đầu tiên từ kết quả trả về. Và hiện tại username đang được bọc trong dấu nháy đơn `'username'` .

	$row = $query->fetch_assoc(); // Get the first row

	if ($row === NULL)
		return "Wrong username or password"; // No result

	$login_user = $row["username"];
	if ($login_user === "admin")
		return "Wow you can log in as admin";
	else
		return "You log in as $login_user, but then what? You are not an admin";

Vậy sẽ ra sao nếu nhập username là `admin'` ?

#### Step to reproduce

1. Kiểm tra có tồn tại SQL Injection

Lúc này SQL query bị lỗi syntax vì dấu `'` không được đóng lại

![alt text](image-1.png)

##### Cách 1
2. Login tài khoản admin

Dùng dấu `--` hoặc `#` để comment phần đằng sau cửa SQL
Payload: username = `admin'-- -` hoặc `admin'#`

![alt text](image-3.png)

##### Cách 2
2.Login tài khoản admin

Dùng toán tử logic `OR`
 
Payload: username = `admin' or 1='1 ` 
Lúc này câu SQL query sẽ thực hiện câu lệnh:

`SELECT username FROM users WHERE username='admin' OR 1='1' AND password=123`

![alt text](image-4.png)