# History
JavaScript được sinh ra với cái tên mocha vào những năm 1995.
Dần dần nó trở thành client-side script language khá là thành công. Nhưng mà server-side thì lại khác, ngày xưa có mỗi PHP với Java là ngon nghẻ.
Vì JS quá ngon, hỗ trợ được hết những trình duyệt phổ biến nên GG, Apple và nhiều công ty nhảy vào cải thiện JS, khiến cho nó ngoài phổ biến thì lại ngon, chạy không bị lag nữa.

Năm 2019 thì NodeJS bắt đầu được tạo ra cho mục đích **scalable server cho web**. Được viết bằng C++, JS. dùng 8 JS engine

# Execution model
Ngày xưa có bộ Linux, Apache, MySQL, and PHP. Trong đó Apache là chịu trách nhiệm process HTTP request, còn logic ứng dụng thì dùng PHP/Java.
Node thì **gộp chung HTTP req và app logic**, ngắn gọn súc tích.

Về mô hình thì cơ bản những **thằng khác chạy 1 pool of threads** để handling client connections. Threads thì xài tốn tài nguyên, mà server chạy tải lớn thì sẽ cần nhiều thread thế là scale lên thì cost.
Node chạy **single thread**, thay vì block server chờ I/O thì nó sẽ handle req khác. Gọi là **nonblocking I/O**. Triger async callback khi có kết quả.
I/O ví dụ như DB query.

# Read-Eval-Print-Loop (REPL)
Cái này là shell y chang mở devtools chạy.
Vô terminal gõ node sau đó gõ lệnh như thường.

# Install
Nó là open source nên thích chọc code thì lấy về mà mò: https://github.com/nodejs/node
Còn bình thường thì xài npm cho nhanh gọn lẹ.

