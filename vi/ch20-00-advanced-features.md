# Các Tính Năng Nâng Cao

Đến nay, bạn đã tìm hiểu những phần cốt lõi và được sử dụng phổ biến nhất của ngôn
ngữ lập trình Rust. Trước khi cùng nhau thực hiện dự án cuối cùng trong Chương 21,
chúng ta sẽ xem xét một vài khía cạnh nâng cao của ngôn ngữ mà bạn có thể thỉnh
thoảng bắt gặp, nhưng không nhất thiết phải dùng mỗi ngày. Bạn có thể sử dụng chương
này như một tài liệu tham khảo khi gặp những cú pháp hoặc khái niệm mới chưa rõ. Các
tính năng được đề cập ở đây đặc biệt hữu ích trong các tình huống chuyên biệt. Mặc dù
có thể bạn không thường xuyên sử dụng chúng, chúng tôi muốn đảm bảo rằng bạn nắm bắt
được bức tranh toàn diện về sức mạnh mà Rust cung cấp.

Trong chương này, chúng ta sẽ tìm hiểu:

- **Unsafe Rust**: Cách chủ động bỏ qua một số đảm bảo an toàn nghiêm ngặt của Rust và
  tự chịu trách nhiệm duy trì an toàn bộ nhớ một cách thủ công (tương tự như khi bạn viết
  mã C/C++, trái ngược với Python luôn có bộ thu gom rác Garbage Collector ẩn phía sau).
- **Trait nâng cao**: Kiểu liên kết (associated types), tham số kiểu mặc định (default
  type parameters), cú pháp định danh đầy đủ (fully qualified syntax), supertraits, và
  mẫu newtype (newtype pattern) trong trait.
- **Hệ thống kiểu nâng cao**: Hiểu sâu hơn về mẫu newtype, bí danh kiểu (type aliases -
  tương tự như Type Alias trong Python typing), kiểu never (`!`), và các kiểu có kích
  thước động (dynamically sized types / DST).
- **Hàm và closure nâng cao**: Con trỏ hàm (function pointers) và cách trả về closure
  từ một hàm.
- **Macro**: Kỹ thuật siêu lập trình (metaprogramming) cho phép định nghĩa mã nguồn để
  sinh ra thêm mã nguồn khác tại thời điểm biên dịch (compile time - khác với decorator
  hay `eval()` trong Python vốn chạy tại runtime).

Đây là một tập hợp phong phú các tính năng mạnh mẽ của Rust với nhiều điều thú vị dành
cho tất cả mọi người! Hãy cùng bắt đầu nào!
