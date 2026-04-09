# Các tính năng nâng cao

Đến nay, bạn đã tìm hiểu những phần được sử dụng phổ biến nhất của ngôn ngữ lập trình
Rust. Trước khi chúng ta thực hiện thêm một dự án nữa trong Chương 21, chúng ta sẽ xem xét một vài
khía cạnh của ngôn ngữ mà bạn có thể thỉnh thoảng bắt gặp, nhưng có thể không
sử dụng hàng ngày. Bạn có thể sử dụng chương này như một tài liệu tham khảo khi bạn gặp
bất kỳ điều gì chưa biết. Các tính năng được đề cập ở đây hữu ích trong các tình huống rất cụ thể.
Mặc dù bạn có thể không thường xuyên sử dụng chúng, chúng tôi muốn đảm bảo rằng bạn đã
nắm bắt được tất cả các tính năng mà Rust cung cấp.

Trong chương này, chúng ta sẽ đề cập đến:

- Unsafe Rust: cách chọn loại bỏ một số đảm bảo của Rust và tự
  chịu trách nhiệm duy trì các đảm bảo đó một cách thủ công
- Các trait nâng cao: các kiểu liên kết (associated types), tham số kiểu mặc định, cú pháp định danh đầy đủ (fully qualified syntax),
  supertraits, và mẫu newtype liên quan đến các trait
- Các kiểu nâng cao: tìm hiểu thêm về mẫu newtype, bí danh kiểu (type aliases), kiểu never,
  và các kiểu có kích thước động (dynamically sized types)
- Các hàm và closure nâng cao: con trỏ hàm (function pointers) và trả về closure
- Macro: các cách để định nghĩa mã nguồn mà sẽ định nghĩa thêm mã nguồn khác tại thời điểm biên dịch

Đây là một tập hợp phong phú các tính năng của Rust với những điều thú vị dành cho tất cả mọi người! Hãy cùng bắt đầu nào!
