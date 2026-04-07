# Dự Án I/O: Xây Dựng Chương Trình Dòng Lệnh

Chương này là phần ôn tập về nhiều kỹ năng bạn đã học cho đến nay và là một
cuộc khám phá thêm một vài tính năng của thư viện chuẩn. Chúng ta sẽ xây dựng một
công cụ dòng lệnh tương tác với tệp và đầu vào/đầu ra dòng lệnh để thực hành một số
khái niệm Rust mà bạn hiện đã nắm vững.

> **Ghi chú:** không có các bài trắc nghiệm trong chương này, vì nó chỉ nhằm mục đích là một bài thực hành hướng dẫn trực tiếp.

Tốc độ, sự an toàn, đầu ra một tệp nhị phân duy nhất và hỗ trợ đa nền tảng của Rust làm cho nó
trở thành một ngôn ngữ lý tưởng để tạo các công cụ dòng lệnh, vì vậy đối với dự án của chúng ta, chúng ta sẽ
tạo phiên bản riêng của công cụ tìm kiếm dòng lệnh cổ điển `grep`
(**g**lobally search a **r**egular **e**xpression and **p**rint - tìm kiếm toàn cầu một biểu thức chính quy và in ra). Trong
trường hợp sử dụng đơn giản nhất, `grep` tìm kiếm một chuỗi được chỉ định trong một tệp được chỉ định. Để
làm như vậy, `grep` nhận các đối số là một đường dẫn tệp và một chuỗi. Sau đó, nó đọc
tệp, tìm các dòng trong tệp đó có chứa đối số chuỗi và in
những dòng đó ra.

Trong quá trình thực hiện, chúng tôi sẽ chỉ ra cách làm cho công cụ dòng lệnh của chúng ta sử dụng các
tính năng terminal mà nhiều công cụ dòng lệnh khác sử dụng. Chúng ta sẽ đọc giá trị của một
biến môi trường (environment variable) để cho phép người dùng cấu hình hành vi của công cụ của chúng ta.
Chúng ta cũng sẽ in các thông báo lỗi ra luồng bảng điều khiển lỗi chuẩn (`stderr`)
thay vì đầu ra chuẩn (`stdout`) để, ví dụ, người dùng có thể
chuyển hướng đầu ra thành công vào một tệp trong khi vẫn thấy các thông báo lỗi trên màn hình.

Một thành viên cộng đồng Rust, Andrew Gallant, đã tạo ra một phiên bản
đầy đủ tính năng, rất nhanh của `grep`, được gọi là `ripgrep`. Để so sánh,
phiên bản của chúng ta sẽ khá đơn giản, nhưng chương này sẽ cung cấp cho bạn một số
kiến thức nền tảng cần thiết để hiểu một dự án thực tế như
`ripgrep`.

Dự án `grep` của chúng ta sẽ kết hợp một số khái niệm bạn đã học cho đến nay:

- Tổ chức mã ([Chương 7][ch7]<!-- ignore -->)
- Sử dụng vector và chuỗi ([Chương 8][ch8]<!-- ignore -->)
- Xử lý lỗi ([Chương 9][ch9]<!-- ignore -->)
- Sử dụng trait và lifetime ở những nơi thích hợp ([Chương 10][ch10]<!-- ignore -->)
- Viết kiểm thử ([Chương 11][ch11]<!-- ignore -->)

Chúng ta cũng sẽ giới thiệu ngắn gọn về closure, iterator và trait object, những thứ mà
[Chương 13][ch13]<!-- ignore --> và [Chương 18][ch18]<!-- ignore --> sẽ
đề cập chi tiết.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
