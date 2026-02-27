# Các Bộ sưu tập Phổ biến

Thư viện tiêu chuẩn của Rust bao gồm một số cấu trúc dữ liệu rất hữu ích được gọi là
_các bộ sưu tập_ (collections). Hầu hết các kiểu dữ liệu khác đại diện cho một giá trị cụ thể, nhưng
các bộ sưu tập có thể chứa nhiều giá trị. Không giống như các kiểu mảng (array) và bộ (tuple)
tích hợp sẵn, dữ liệu mà các bộ sưu tập này trỏ đến được lưu trữ trên heap, điều đó có
nghĩa là lượng dữ liệu không cần phải được biết tại thời điểm biên dịch và có thể tăng lên
hoặc thu nhỏ lại khi chương trình chạy. Mỗi loại bộ sưu tập có các khả năng
và chi phí khác nhau, và việc chọn một loại phù hợp cho tình huống hiện tại
của bạn là một kỹ năng bạn sẽ phát triển theo thời gian. Trong chương này, chúng ta sẽ thảo luận về
ba bộ sưu tập được sử dụng rất thường xuyên trong các chương trình Rust:

- Một _vector_ cho phép bạn lưu trữ một số lượng biến đổi các giá trị cạnh nhau.
- Một _chuỗi_ (string) là một bộ sưu tập các ký tự. Trước đây chúng ta đã đề cập đến kiểu `String`,
  nhưng trong chương này chúng ta sẽ thảo luận sâu hơn về nó.
- Một _bảng băm_ (hash map) cho phép bạn liên kết một giá trị với một khóa cụ thể. Nó là
  một triển khai cụ thể của cấu trúc dữ liệu tổng quát hơn được gọi là _map_.

Để tìm hiểu về các loại bộ sưu tập khác do thư viện tiêu chuẩn cung cấp,
hãy xem [tài liệu][collections].

Chúng ta sẽ thảo luận về cách tạo và cập nhật vector, chuỗi và bảng băm, cũng như
điều gì làm cho mỗi loại trở nên đặc biệt.

[collections]: https://doc.rust-lang.org/std/collections/index.html
