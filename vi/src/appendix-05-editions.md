## Phụ lục E - Các phiên bản

Trong Chương 1, bạn đã thấy rằng `cargo new` thêm một chút siêu dữ liệu vào tệp _Cargo.toml_ của bạn về một phiên bản (_edition_). Phụ lục này nói về ý nghĩa của điều đó!

Ngôn ngữ và trình biên dịch Rust có chu kỳ phát hành sáu tuần, nghĩa là người dùng nhận được một luồng liên tục các tính năng mới. Các ngôn ngữ lập trình khác phát hành các thay đổi lớn hơn ít thường xuyên hơn; Rust phát hành các bản cập nhật nhỏ hơn thường xuyên hơn. Sau một thời gian, tất cả những thay đổi nhỏ này cộng lại. Nhưng từ phiên bản này sang phiên bản khác, khó để nhìn lại và nói, “Wow, giữa Rust 1.10 và Rust 1.31, Rust đã thay đổi rất nhiều!”

Khoảng ba năm một lần, nhóm Rust tạo ra một _edition_ Rust mới. Mỗi edition gom các tính năng đã được thêm vào thành một gói rõ ràng với tài liệu và công cụ được cập nhật đầy đủ. Các edition mới được phát hành như một phần của quy trình phát hành sáu tuần thông thường.

Các edition phục vụ những mục đích khác nhau cho những người khác nhau:

- Đối với người dùng Rust đang hoạt động, một edition mới gom các thay đổi dần dần lại thành một gói dễ hiểu.
- Đối với những người chưa dùng, một edition mới báo hiệu rằng đã có những tiến bộ lớn, có thể khiến họ cân nhắc quay lại tìm hiểu Rust.
- Đối với những người đang phát triển Rust, một edition mới cung cấp một điểm quy tụ cho toàn bộ dự án.

Tại thời điểm viết tài liệu này, có bốn edition Rust có sẵn: Rust 2015, Rust 2018, Rust 2021 và Rust 2024. Cuốn sách này được viết theo các phong cách (idioms) của edition Rust 2024.

Khóa `edition` trong _Cargo.toml_ cho biết trình biên dịch nên sử dụng edition nào cho mã của bạn. Nếu khóa không tồn tại, Rust sử dụng `2015` làm giá trị edition vì lý do tương thích ngược.

Mỗi dự án có thể chọn tham gia một edition khác với edition mặc định 2015. Edition có thể chứa những thay đổi không tương thích, chẳng hạn như thêm một từ khóa mới xung đột với các định danh trong mã. Tuy nhiên, trừ khi bạn chọn tham gia những thay đổi đó, mã của bạn sẽ tiếp tục biên dịch ngay cả khi bạn nâng cấp phiên bản trình biên dịch Rust mà bạn sử dụng.

Tất cả các phiên bản trình biên dịch Rust đều hỗ trợ bất kỳ edition nào tồn tại trước khi phiên bản trình biên dịch đó được phát hành, và chúng có thể liên kết các crate thuộc bất kỳ edition được hỗ trợ nào với nhau. Thay đổi edition chỉ ảnh hưởng cách trình biên dịch phân tích cú pháp ban đầu mã nguồn. Do đó, nếu bạn đang dùng Rust 2015 và một trong các phụ thuộc của bạn dùng Rust 2018, dự án của bạn sẽ biên dịch và có thể sử dụng phụ thuộc đó. Tình huống ngược lại, khi dự án của bạn dùng Rust 2018 và một phụ thuộc dùng Rust 2015, cũng hoạt động bình thường.

Để làm rõ: hầu hết các tính năng sẽ có sẵn trên tất cả các edition. Những nhà phát triển dùng bất kỳ edition Rust nào sẽ tiếp tục thấy các cải tiến khi có các bản phát hành ổn định mới. Tuy nhiên, trong một số trường hợp, chủ yếu khi thêm từ khóa mới, một số tính năng mới có thể chỉ có trong các edition sau. Bạn sẽ cần chuyển đổi edition nếu muốn tận dụng những tính năng như vậy.

Để biết chi tiết hơn, [_Hướng dẫn về các phiên bản_](https://doc.rust-lang.org/stable/edition-guide/) là một cuốn sách hoàn chỉnh về các edition, liệt kê sự khác biệt giữa các edition và giải thích cách tự động nâng cấp mã của bạn lên edition mới thông qua `cargo fix`.
