# Xử lý Lỗi

Lỗi là một phần tất yếu trong phần mềm, vì vậy Rust có một số tính năng để
xử lý các tình huống mà ở đó có điều gì đó không ổn xảy ra. Trong nhiều trường hợp, Rust yêu cầu
bạn phải thừa nhận khả năng xảy ra lỗi và thực hiện một số hành động trước khi
mã của bạn có thể biên dịch. Yêu cầu này làm cho chương trình của bạn mạnh mẽ hơn bằng cách đảm bảo
rằng bạn sẽ phát hiện lỗi và xử lý chúng một cách thích hợp trước khi triển khai
mã của mình lên môi trường thực tế (production)!

Rust nhóm các lỗi thành hai loại chính: lỗi _có thể phục hồi_ (recoverable) và _không thể phục hồi_
(unrecoverable). Đối với một lỗi có thể phục hồi, chẳng hạn như lỗi _không tìm thấy file_, chúng ta
nhiều khả năng chỉ muốn báo cáo sự cố cho người dùng và thử lại thao tác đó.
Các lỗi không thể phục hồi luôn là dấu hiệu của các bug, chẳng hạn như cố gắng truy cập một
vị trí vượt quá giới hạn của một mảng, và vì vậy chúng ta muốn dừng ngay lập tức
chương trình.

Hầu hết các ngôn ngữ không phân biệt giữa hai loại lỗi này và xử lý
cả hai theo cùng một cách, bằng cách sử dụng các cơ chế như ngoại lệ (exceptions). Rust không có
ngoại lệ. Thay vào đó, nó có kiểu `Result<T, E>` cho các lỗi có thể phục hồi và
macro `panic!` để dừng thực thi khi chương trình gặp một
lỗi không thể phục hồi. Chương chương này sẽ đề cập đến việc gọi `panic!` trước và sau đó nói
về việc trả về các giá trị `Result<T, E>`. Ngoài ra, chúng ta sẽ khám phá
các cân nhắc khi quyết định nên thử phục hồi từ một lỗi hay dừng
thực thi.
