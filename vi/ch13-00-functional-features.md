# Các tính năng của ngôn ngữ lập trình hàm: Iterator và Closure

Thiết kế của Rust đã lấy cảm hứng từ nhiều ngôn ngữ và
kỹ thuật hiện có, và một ảnh hưởng đáng kể là _lập trình hàm_ (functional programming).
Lập trình theo phong cách hàm thường bao gồm việc sử dụng các hàm như là các giá trị bằng cách
truyền chúng làm đối số, trả về chúng từ các hàm khác, gán chúng
vào các biến để thực thi sau này, và vân vân.

Trong chương này, chúng ta sẽ không tranh luận về vấn đề lập trình hàm là gì hay
không phải là gì mà thay vào đó sẽ thảo luận về một số tính năng của Rust tương tự như
các tính năng trong nhiều ngôn ngữ thường được gọi là lập trình hàm.

Cụ thể hơn, chúng ta sẽ đề cập đến:

- _Closure_, một cấu trúc giống như hàm mà bạn có thể lưu trữ trong một biến
- _Iterator_, một cách để xử lý một chuỗi các phần tử
- Cách sử dụng closure và iterator để cải thiện dự án I/O trong Chương 12
- Hiệu suất của closure và iterator (tiết lộ nội dung: chúng nhanh hơn
  bạn nghĩ đấy!)

Chúng ta đã đề cập đến một số tính năng khác của Rust, chẳng hạn như khớp mẫu (pattern matching)
và enum, những tính năng này cũng chịu ảnh hưởng của phong cách hàm. Bởi vì việc nắm vững
closure và iterator là một phần quan trọng để viết mã Rust đúng chuẩn (idiomatic) và nhanh,
chúng ta sẽ dành toàn bộ chương này cho chúng.
