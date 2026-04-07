# Viết các Bài kiểm tra Tự động

Trong bài tiểu luận năm 1972 “The Humble Programmer,” Edsger W. Dijkstra đã nói rằng “kiểm thử chương trình
có thể là một cách rất hiệu quả để chỉ ra sự hiện diện của lỗi, nhưng nó
hoàn toàn không đủ để chỉ ra sự vắng mặt của chúng.” Điều đó không có nghĩa là chúng ta không nên
cố gắng kiểm thử nhiều nhất có thể!

Tính đúng đắn trong các chương trình của chúng ta là mức độ mà mã của chúng ta thực hiện những gì chúng ta muốn
nó làm. Rust được thiết kế với mức độ quan tâm cao về tính đúng đắn
của các chương trình, nhưng tính đúng đắn rất phức tạp và không dễ chứng minh. Hệ thống type của Rust
gánh vác một phần lớn gánh nặng này, nhưng hệ thống type không thể bắt được
mọi thứ. Do đó, Rust bao gồm hỗ trợ viết các bài kiểm tra phần mềm tự động.

Giả sử chúng ta viết một hàm `add_two` để cộng 2 vào bất kỳ số nào được truyền cho
nó. Chữ ký của hàm này chấp nhận một số nguyên làm tham số và trả về một
số nguyên làm kết quả. Khi chúng ta triển khai và biên dịch hàm đó, Rust thực hiện tất cả
việc kiểm tra type và borrow checking mà bạn đã học cho đến nay để đảm bảo
rằng, ví dụ, chúng ta không truyền một giá trị `String` hoặc một reference không hợp lệ
cho hàm này. Nhưng Rust _không thể_ kiểm tra rằng hàm này sẽ làm chính xác
những gì chúng ta muốn, đó là trả về tham số cộng 2 thay vì, chẳng hạn,
tham số cộng 10 hoặc tham số trừ 50! Đó là lúc các bài kiểm tra phát huy tác dụng.

Chúng ta có thể viết các bài kiểm tra khẳng định, ví dụ, rằng khi chúng ta truyền `3` cho
hàm `add_two`, giá trị trả về là `5`. Chúng ta có thể chạy các bài kiểm tra này bất cứ khi nào
chúng ta thực hiện thay đổi đối với mã của mình để đảm bảo bất kỳ hành vi đúng đắn hiện có nào
chưa thay đổi.

Kiểm thử là một kỹ năng phức tạp: mặc dù chúng ta không thể trình bày trong một chương mọi chi tiết
về cách viết các bài kiểm tra tốt, trong chương này chúng ta sẽ thảo luận về cơ chế
của các tiện ích kiểm thử của Rust. Chúng ta sẽ nói về các annotation và macro
có sẵn cho bạn khi viết các bài kiểm tra của mình, hành vi mặc định và các tùy chọn
được cung cấp để chạy các bài kiểm tra của bạn, và cách tổ chức các bài kiểm tra thành unit tests và
integration tests.
