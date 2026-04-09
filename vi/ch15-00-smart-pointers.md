# Con trỏ thông minh (Smart Pointers)

Một _con trỏ_ (pointer) là một khái niệm chung cho một biến chứa một địa chỉ trong
bộ nhớ. Địa chỉ này tham chiếu đến, hoặc “trỏ vào,” một vài dữ liệu khác. Loại
con trỏ phổ biến nhất trong Rust là tham chiếu (reference), cái mà bạn đã học ở
Chương 4. Các tham chiếu được chỉ định bởi ký hiệu `&` và mượn giá trị mà chúng
trỏ tới. Chúng không có bất kỳ khả năng đặc biệt nào khác ngoài việc tham chiếu đến
dữ liệu, và chúng không có chi phí bổ sung (overhead).

_Con trỏ thông minh_ (Smart pointers), mặt khác, là các cấu trúc dữ liệu hoạt động giống như
một con trỏ nhưng cũng có thêm các siêu dữ liệu (metadata) và các khả năng bổ sung.
Khái niệm con trỏ thông minh không chỉ có duy nhất ở Rust: con trỏ thông minh có nguồn gốc từ C++ và cũng
tồn tại trong các ngôn ngữ khác. Rust có nhiều loại con trỏ thông minh khác nhau được định nghĩa trong
thư viện tiêu chuẩn nhằm cung cấp các chức năng vượt xa những gì tham chiếu cung cấp.
Để khám phá khái niệm chung này, chúng ta sẽ xem xét một vài ví dụ khác nhau về
con trỏ thông minh, bao gồm một loại con trỏ thông minh _đếm tham chiếu_ (reference counting). Loại
con trỏ này cho phép bạn cho phép dữ liệu có nhiều chủ sở hữu bằng cách theo dõi
số lượng chủ sở hữu và khi không còn chủ sở hữu nào, nó sẽ dọn dẹp dữ liệu đó.

Rust, với khái niệm quyền sở hữu (ownership) và mượn (borrowing), có thêm một điểm khác biệt
giữa tham chiếu và con trỏ thông minh: trong khi tham chiếu chỉ mượn dữ liệu, thì trong
nhiều trường hợp con trỏ thông minh _sở hữu_ dữ liệu mà chúng trỏ tới.

Mặc dù chúng ta không gọi chúng như vậy vào thời điểm đó, nhưng chúng ta đã gặp một vài
con trỏ thông minh trong cuốn sách này, bao gồm `String` và `Vec<T>` ở Chương 8. Cả hai
loại này đều được coi là con trỏ thông minh vì chúng sở hữu một số vùng nhớ và cho phép
bạn thao tác trên đó. Chúng cũng có siêu dữ liệu và các khả năng hoặc đảm bảo bổ sung.
`String`, ví dụ, lưu trữ dung lượng (capacity) của nó dưới dạng siêu dữ liệu và có
khả năng bổ sung để đảm bảo dữ liệu của nó luôn là UTF-8 hợp lệ.

Con trỏ thông minh thường được thực thi bằng cách sử dụng các struct. Không giống như một
struct thông thường, con trỏ thông minh thực thi các trait `Deref` và `Drop`.
Trait `Deref` cho phép một instance của struct con trỏ thông minh hành xử giống như một tham chiếu
để bạn có thể viết mã của mình hoạt động với cả tham chiếu hoặc con trỏ thông minh.
Trait `Drop` cho phép bạn tùy chỉnh mã được chạy khi một instance
của con trỏ thông minh ra khỏi phạm vi (scope). Trong chương này, chúng ta sẽ thảo luận về cả hai
trait này và chứng minh tại sao chúng quan trọng đối với con trỏ thông minh.

Bởi vì mô hình con trỏ thông minh là một mô hình thiết kế chung được sử dụng
thường xuyên trong Rust, chương này sẽ không bao gồm mọi con trỏ thông minh hiện có. Nhiều
thư viện có các con trỏ thông minh của riêng chúng, và bạn thậm chí có thể tự viết con trỏ của riêng mình. Chúng ta sẽ
đề cập đến các con trỏ thông minh phổ biến nhất trong thư viện tiêu chuẩn:

- `Box<T>`, để cấp phát các giá trị trên heap
- `Rc<T>`, một kiểu đếm tham chiếu cho phép đa sở hữu
- `Ref<T>` và `RefMut<T>`, được truy cập thông qua `RefCell<T>`, một kiểu thực thi
  các quy tắc mượn tại thời điểm chạy (runtime) thay vì thời điểm biên dịch (compile time)

Ngoài ra, chúng ta sẽ đề cập đến mô hình _tính đột biến nội thất_ (interior mutability) nơi một
kiểu không biến đổi (immutable) để lộ ra một API để thay đổi một giá trị bên trong. Chúng ta cũng sẽ thảo luận về
_chu kỳ tham chiếu_ (reference cycles): cách chúng có thể làm rò rỉ bộ nhớ và cách ngăn chặn chúng.

Hãy cùng bắt đầu nào!
