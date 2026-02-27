# Quản lý các Dự án đang Phát triển với Package, Crate, và Module

Khi bạn viết các chương trình lớn, việc tổ chức mã nguồn sẽ trở nên ngày càng
quan trọng. Bằng cách nhóm các chức năng liên quan và tách biệt mã nguồn với các
tính năng riêng biệt, bạn sẽ làm rõ được nơi tìm mã nguồn triển khai một
tính năng cụ thể và nơi cần đến để thay đổi cách một tính năng hoạt động.

Các chương trình chúng ta đã viết cho đến nay đều nằm trong một module trong một file. Khi một
dự án phát triển, bạn nên tổ chức mã nguồn bằng cách chia nó thành nhiều module
và sau đó là nhiều file. Một package có thể chứa nhiều binary crate và
tùy chọn một library crate. Khi một package phát triển, bạn có thể trích xuất các phần thành
các crate riêng biệt để trở thành các phụ thuộc bên ngoài (external dependencies). Chương này bao gồm tất cả
những kỹ thuật này. Đối với các dự án rất lớn bao gồm một tập hợp các
package có liên quan với nhau và cùng phát triển, Cargo cung cấp _workspaces_, chúng ta sẽ đề cập trong
[“Cargo Workspaces”][workspaces]<!-- ignore --> ở Chương 14.

Chúng ta cũng sẽ thảo luận về việc đóng gói các chi tiết triển khai (encapsulating implementation details), điều này cho phép bạn tái
sử dụng mã nguồn ở cấp độ cao hơn: một khi bạn đã triển khai một thao tác, mã nguồn khác có thể
gọi mã của bạn thông qua giao diện công khai (public interface) của nó mà không cần biết
cách thức triển khai hoạt động như thế nào. Cách bạn viết mã xác định phần nào là công khai để
mã khác sử dụng và phần nào là chi tiết triển khai riêng tư (private implementation details) mà bạn
giữ quyền thay đổi. Đây là một cách khác để giới hạn lượng chi tiết
bạn phải ghi nhớ trong đầu.

Một khái niệm liên quan là phạm vi (scope): ngữ cảnh lồng nhau trong đó mã được viết có một
tập hợp các tên được định nghĩa là “trong phạm vi” (in scope). Khi đọc, viết và
biên dịch mã, lập trình viên và trình biên dịch cần biết liệu một
tên cụ thể tại một vị trí cụ thể đề cập đến một biến, hàm, struct, enum, module,
hằng số hoặc mục khác và mục đó có ý nghĩa gì. Bạn có thể tạo các phạm vi và
thay đổi tên nào nằm trong hoặc ngoài phạm vi. Bạn không thể có hai mục có
cùng tên trong cùng một phạm vi; các công cụ có sẵn để giải quyết xung đột tên.

Rust có một số tính năng cho phép bạn quản lý việc tổ chức mã nguồn của mình,
bao gồm chi tiết nào được để lộ ra, chi tiết nào là riêng tư,
và tên nào nằm trong mỗi phạm vi trong chương trình của bạn. Những tính năng này, đôi khi
được gọi chung là _hệ thống module_ (module system), bao gồm:

- **Packages**: Một tính năng của Cargo cho phép bạn xây dựng, kiểm thử và chia sẻ các crate
- **Crates**: Một cây các module tạo ra một thư viện hoặc tệp thực thi
- **Modules** và **use**: Cho phép bạn kiểm soát việc tổ chức, phạm vi và tính riêng tư của
  các đường dẫn (paths)
- **Paths**: Một cách để đặt tên cho một mục, chẳng hạn như struct, hàm hoặc module

Trong chương này, chúng ta sẽ đề cập đến tất cả các tính năng này, thảo luận về cách chúng tương tác và
giải thích cách sử dụng chúng để quản lý phạm vi. Đến cuối chương, bạn sẽ có một
hiểu biết vững chắc về hệ thống module và có thể làm việc với các phạm vi như một chuyên gia!

[workspaces]: ch14-03-cargo-workspaces.html-
