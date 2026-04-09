## `RefCell<T>` và mô hình “có thể thay đổi từ bên trong” (Interior Mutability Pattern)

_Interior mutability_ (interior mutability) là một mẫu thiết kế trong Rust cho phép bạn thay đổi
dữ liệu ngay cả khi có các tham chiếu bất biến đến dữ liệu đó; thông thường,
hành động này bị cấm bởi các quy tắc mượn. Để thay đổi dữ liệu, mô hình này sử dụng
mã `unsafe` bên trong một cấu trúc dữ liệu để lách các quy tắc thông thường của Rust quản lý
việc thay đổi và mượn. Mã không an toàn (unsafe code) chỉ ra cho trình biên dịch rằng chúng ta đang
kiểm tra các quy tắc một cách thủ công thay vì dựa vào trình biên dịch để kiểm tra chúng
cho chúng ta; chúng ta sẽ thảo luận về mã không an toàn nhiều hơn trong Chương 20.

Chúng ta chỉ có thể sử dụng các kiểu sử dụng mô hình interior mutability khi chúng ta có thể
đảm bảo rằng các quy tắc mượn sẽ được tuân thủ tại thời điểm chạy, ngay cả khi
trình biên dịch không thể đảm bảo điều đó. Mã `unsafe` liên quan sau đó được gói trong một
API an toàn, và kiểu bên ngoài vẫn là bất biến.

Hãy cùng khám phá khái niệm này bằng cách xem xét kiểu `RefCell<T>` tuân theo
mô hình interior mutability.

### Thực thi các quy tắc mượn tại thời điểm chạy với `RefCell<T>`

Không giống như `Rc<T>`, kiểu `RefCell<T>` đại diện cho quyền sở hữu đơn lẻ đối với dữ liệu
mà nó nắm giữ. Vậy điều gì làm cho `RefCell<T>` khác biệt với một kiểu như `Box<T>`?
Hãy nhớ lại các quy tắc mượn bạn đã học ở Chương 4:

- Tại bất kỳ thời điểm nào, bạn có thể có _hoặc_ một tham chiếu có thể thay đổi hoặc bất kỳ số lượng
  tham chiếu bất biến nào (nhưng không phải cả hai).
- Các tham chiếu phải luôn hợp lệ.

Với các tham chiếu và `Box<T>`, các bất biến của quy tắc mượn được thực thi tại
thời điểm biên dịch. Với `RefCell<T>`, các bất biến này được thực thi _tại thời điểm chạy_.
Với các tham chiếu, nếu bạn vi phạm các quy tắc này, bạn sẽ nhận được lỗi trình biên dịch. Với
`RefCell<T>`, nếu bạn vi phạm các quy tắc này, chương trình của bạn sẽ panic và thoát.

Ưu điểm của việc kiểm tra các quy tắc mượn tại thời điểm biên dịch là các lỗi
sẽ được phát hiện sớm hơn trong quá trình phát triển, và không có tác động nào đến
hiệu suất thời gian chạy vì tất cả các phân tích đã được hoàn thành trước đó. Vì những
lý do đó, kiểm tra các quy tắc mượn tại thời điểm biên dịch là lựa chọn tốt nhất trong
phần lớn các trường hợp, đó là lý do tại sao đây là mặc định của Rust.

Ưu điểm của việc kiểm tra các quy tắc mượn tại thời điểm chạy thay thế là
một số kịch bản an toàn về bộ nhớ nhất định sau đó được cho phép, nơi mà chúng đáng lẽ đã bị
cấm bởi các kiểm tra tại thời điểm biên dịch. Phân tích tĩnh, giống như trình biên dịch Rust,
vốn dĩ có tính bảo thủ. Một số thuộc tính của mã là không thể phát hiện được bằng cách
phân tích mã: ví dụ nổi tiếng nhất là Bài toán dừng (Halting Problem), nằm ngoài
phạm vi của cuốn sách này nhưng là một chủ đề thú vị để nghiên cứu.

Bởi vì một số phân tích là không thể, nếu trình biên dịch Rust không thể chắc chắn rằng
mã tuân thủ các quy tắc quyền sở hữu, nó có thể từ chối một chương trình đúng; theo
cách này, nó có tính bảo thủ. Nếu Rust chấp nhận một chương trình sai, người dùng
sẽ không thể tin tưởng vào những đảm bảo mà Rust đưa ra. Tuy nhiên, nếu Rust
từ chối một chương trình đúng, lập trình viên sẽ gặp bất tiện, nhưng không có gì
thảm khốc có thể xảy ra. Kiểu `RefCell<T>` hữu ích khi bạn chắc chắn rằng mã của mình
tuân thủ các quy tắc mượn nhưng trình biên dịch không thể hiểu và
đảm bảo điều đó.

Tương tự như `Rc<T>`, `RefCell<T>` chỉ được sử dụng trong các tình huống đơn luồng
và sẽ báo lỗi biên dịch nếu bạn cố gắng sử dụng nó trong một ngữ cảnh đa luồng.
Chúng ta sẽ nói về cách có được chức năng của `RefCell<T>` trong một
chương trình đa luồng ở Chương 16.

Dưới đây là bản tóm tắt các lý do để chọn `Box<T>`, `Rc<T>`, hoặc `RefCell<T>`:

- `Rc<T>` cho phép nhiều chủ sở hữu của cùng một dữ liệu; `Box<T>` và `RefCell<T>`
  có chủ sở hữu đơn lẻ.
- `Box<T>` cho phép mượn bất biến hoặc có thể thay đổi được kiểm tra tại thời điểm biên dịch; `Rc<T>`
  chỉ cho phép mượn bất biến được kiểm tra tại thời điểm biên dịch; `RefCell<T>` cho phép
  mượn bất biến hoặc có thể thay đổi được kiểm tra tại thời điểm chạy.
- Bởi vì `RefCell<T>` cho phép mượn có thể thay đổi được kiểm tra tại thời điểm chạy, bạn có thể
  thay đổi giá trị bên trong `RefCell<T>` ngay cả khi `RefCell<T>` là
  bất biến.

Việc thay đổi giá trị bên trong một giá trị bất biến là mô hình _interior mutability_.
Hãy cùng xem xét một tình huống mà interior mutability hữu ích và
tìm hiểu cách nó có thể thực hiện được.

### Interior mutability: Một lần mượn có thể thay đổi đến một giá trị bất biến

Một hệ quả của các quy tắc mượn là khi bạn có một giá trị bất biến,
bạn không thể mượn nó một cách có thể thay đổi. Ví dụ, mã này sẽ không biên dịch được:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Nếu bạn cố gắng biên dịch mã này, bạn sẽ nhận được lỗi sau:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Tuy nhiên, có những tình huống mà việc một giá trị tự thay đổi
chính nó trong các phương thức của nó nhưng lại có vẻ bất biến đối với mã khác là rất hữu ích. Mã bên ngoài
các phương thức của giá trị sẽ không thể thay đổi giá trị đó. Sử dụng `RefCell<T>` là
một cách để có được khả năng interior mutability, nhưng `RefCell<T>`
không hoàn toàn lách được các quy tắc mượn: bộ kiểm tra mượn (borrow checker) trong
trình biên dịch cho phép interior mutability này, và các quy tắc mượn được kiểm tra
tại thời điểm chạy thay thế. Nếu bạn vi phạm các quy tắc, bạn sẽ nhận được một `panic!` thay vì
một lỗi trình biên dịch.

Hãy cùng thực hiện một ví dụ thực tế nơi chúng ta có thể sử dụng `RefCell<T>` để thay đổi
một giá trị bất biến và xem tại sao điều đó lại hữu ích.

#### Một trường hợp sử dụng cho interior mutability: Mock Objects

Đôi khi trong quá trình thử nghiệm, một lập trình viên sẽ sử dụng một kiểu thay thế cho một kiểu khác,
để quan sát hành vi cụ thể và khẳng định rằng nó được thực thi
chính xác. Kiểu giữ chỗ này được gọi là một _test double_. Hãy nghĩ về nó theo nghĩa
của một diễn viên đóng thế (stunt double) trong điện ảnh, nơi một người bước vào và thay thế
cho một diễn viên để thực hiện một cảnh quay đặc biệt khó khăn. Test doubles thay thế cho các
kiểu khác khi chúng ta đang chạy các bài kiểm tra. _Mock objects_ là các loại test
doubles cụ thể ghi lại những gì xảy ra trong một bài kiểm tra để bạn có thể khẳng định rằng
các hành động chính xác đã diễn ra.

Rust không có các đối tượng (objects) theo cùng nghĩa như các ngôn ngữ khác có các đối tượng,
và Rust không có chức năng mock object được tích hợp sẵn trong thư viện tiêu chuẩn
như một số ngôn ngữ khác. Tuy nhiên, bạn chắc chắn có thể tạo một struct
phục vụ cùng mục đích như một mock object.

Đây là kịch bản chúng ta sẽ kiểm tra: chúng ta sẽ tạo một thư viện theo dõi một giá trị
so với một giá trị tối đa và gửi các thông báo dựa trên việc giá trị hiện tại
gần với giá trị tối đa như thế nào. Thư viện này có thể được sử dụng để theo dõi
quota của người dùng về số lượng lời gọi API mà họ được phép thực hiện, chẳng hạn.

Thư viện của chúng ta sẽ chỉ cung cấp chức năng theo dõi mức độ gần
với mức tối đa của một giá trị và các thông báo nên là gì vào thời điểm nào. Các ứng dụng
sử dụng thư viện của chúng ta sẽ được mong đợi cung cấp cơ chế gửi các
thông báo: ứng dụng có thể đặt một thông báo trong ứng dụng, gửi email,
gửi tin nhắn văn bản, hoặc làm điều gì đó khác. Thư viện không cần biết
chi tiết đó. Tất cả những gì nó cần là một cái gì đó thực thi một trait mà chúng ta sẽ cung cấp gọi là
`Messenger`. Liệt kê 15-20 cho thấy mã thư viện.

<Listing number="15-20" file-name="src/lib.rs" caption="Một thư viện để theo dõi mức độ gần của một giá trị với giá trị tối đa và cảnh báo khi giá trị ở các mức nhất định">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Một phần quan trọng của mã này là trait `Messenger` có một phương thức
tên là `send` nhận một tham chiếu bất biến đến `self` và văn bản của
thông báo. Trait này là giao diện mà mock object của chúng ta cần thực thi để
mock có thể được sử dụng theo cùng cách mà một đối tượng thật được sử dụng. Phần quan trọng khác
là chúng ta muốn kiểm tra hành vi của phương thức `set_value` trên
`LimitTracker`. Chúng ta có thể thay đổi những gì chúng ta truyền vào cho tham số `value`, nhưng
`set_value` không trả về bất cứ thứ gì để chúng ta thực hiện các khẳng định. Chúng ta muốn có thể
nói rằng nếu chúng ta tạo một `LimitTracker` với một cái gì đó thực thi
trait `Messenger` và một giá trị cụ thể cho `max`, khi chúng ta truyền các số khác nhau
cho `value`, messenger được yêu cầu gửi các thông báo thích hợp.

Chúng ta cần một mock object mà thay vì gửi email hoặc tin nhắn văn bản khi chúng ta
gọi `send`, nó sẽ chỉ theo dõi các thông báo mà nó được yêu cầu gửi. Chúng ta có thể
tạo một instance mới của mock object, tạo một `LimitTracker` sử dụng
mock object đó, gọi phương thức `set_value` trên `LimitTracker`, và sau đó kiểm tra xem
mock object có các thông báo mà chúng ta mong đợi hay không. Liệt kê 15-21 cho thấy một nỗ lực
thực thi một mock object để làm điều đó, nhưng bộ kiểm tra mượn sẽ không cho phép.

<Listing number="15-21" file-name="src/lib.rs" caption="Một nỗ lực thực thi một `MockMessenger` không được phép bởi bộ kiểm tra mượn">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Mã kiểm tra này định nghĩa một struct `MockMessenger` có một trường `sent_messages`
với một `Vec` các giá trị `String` để theo dõi các thông báo mà nó được yêu cầu
gửi. Chúng ta cũng định nghĩa một hàm liên kết `new` để thuận tiện cho việc
tạo các giá trị `MockMessenger` mới bắt đầu với một danh sách thông báo trống. Sau đó chúng ta
thực thi trait `Messenger` cho `MockMessenger` để chúng ta có thể đưa một
`MockMessenger` cho một `LimitTracker`. Trong định nghĩa của phương thức `send`, chúng ta
lấy thông báo được truyền vào dưới dạng tham số và lưu trữ nó trong danh sách `sent_messages`
của `MockMessenger`.

Trong bài kiểm tra, chúng ta đang kiểm tra điều gì xảy ra khi `LimitTracker` được yêu cầu đặt
`value` thành một giá trị lớn hơn 75 phần trăm của giá trị `max`. Đầu tiên, chúng ta
tạo một `MockMessenger` mới, nó sẽ bắt đầu với một danh sách thông báo trống.
Sau đó chúng ta tạo một `LimitTracker` mới và đưa cho nó một tham chiếu đến `MockMessenger`
mới và một giá trị `max` là `100`. Chúng ta gọi phương thức `set_value` trên
`LimitTracker` với giá trị `80`, lớn hơn 75 phần trăm của 100.
Sau đó chúng ta khẳng định rằng danh sách các thông báo mà `MockMessenger` đang theo dõi
bây giờ nên có một thông báo trong đó.

Tuy nhiên, có một vấn đề với bài kiểm tra này, như được hiển thị ở đây:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Chúng ta không thể sửa đổi `MockMessenger` để theo dõi các thông báo, vì
phương thức `send` nhận một tham chiếu bất biến đến `self`. Chúng ta cũng không thể thực hiện theo
gợi ý từ văn bản lỗi để sử dụng `&mut self` trong cả phương thức `impl` và
định nghĩa `trait`. Chúng ta không muốn thay đổi trait `Messenger` chỉ vì
mục đích kiểm tra. Thay vào đó, chúng ta cần tìm một cách để làm cho mã kiểm tra của chúng ta
hoạt động chính xác với thiết kế hiện tại của mình.

Đây là một tình huống mà interior mutability có thể giúp ích! Chúng ta sẽ lưu trữ
`sent_messages` bên trong một `RefCell<T>`, và sau đó phương thức `send` sẽ có thể
sửa đổi `sent_messages` để lưu trữ các thông báo chúng ta đã thấy. Liệt kê 15-22
cho thấy điều đó trông như thế nào.

<Listing number="15-22" file-name="src/lib.rs" caption="Sử dụng `RefCell<T>` để thay đổi một giá trị bên trong trong khi giá trị bên ngoài được coi là bất biến">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

Trường `sent_messages` bây giờ thuộc kiểu `RefCell<Vec<String>>` thay vì
`Vec<String>`. Trong hàm `new`, chúng ta tạo một instance `RefCell<Vec<String>>`
mới bao quanh vector trống.

Đối với việc thực thi phương thức `send`, tham số đầu tiên vẫn là một
lần mượn bất biến của `self`, khớp với định nghĩa trait. Chúng ta gọi
`borrow_mut` trên `RefCell<Vec<String>>` trong `self.sent_messages` để lấy một
tham chiếu có thể thay đổi đến giá trị bên trong `RefCell<Vec<String>>`, chính là
vector. Sau đó chúng ta có thể gọi `push` trên tham chiếu có thể thay đổi đến vector để theo dõi
các thông báo được gửi trong quá trình kiểm tra.

Thay đổi cuối cùng chúng ta phải thực hiện là trong câu lệnh khẳng định: để xem có bao nhiêu mục
trong vector bên trong, chúng ta gọi `borrow` trên `RefCell<Vec<String>>` để lấy một
tham chiếu bất biến đến vector.

Bây giờ bạn đã thấy cách sử dụng `RefCell<T>`, hãy cùng tìm hiểu cách nó hoạt động!

#### Theo dõi các lần mượn tại thời điểm chạy với `RefCell<T>`

Khi tạo các tham chiếu bất biến và có thể thay đổi, chúng ta sử dụng cú pháp `&` và `&mut`
tương ứng. Với `RefCell<T>`, chúng ta sử dụng các phương thức `borrow` và `borrow_mut`,
là một phần của API an toàn thuộc về `RefCell<T>`. Phương thức
`borrow` trả về kiểu con trỏ thông minh `Ref<T>`, và `borrow_mut`
trả về kiểu con trỏ thông minh `RefMut<T>`. Cả hai kiểu đều thực thi `Deref`, vì vậy chúng ta
có thể đối xử với chúng như các tham chiếu thông thường.

`RefCell<T>` theo dõi có bao nhiêu con trỏ thông minh `Ref<T>` và `RefMut<T>`
hiện đang hoạt động. Mỗi khi chúng ta gọi `borrow`, `RefCell<T>`
tăng số lượng các lần mượn bất biến đang hoạt động. Khi một giá trị `Ref<T>`
ra khỏi phạm vi, số lượng mượn bất biến giảm đi 1. Giống như
các quy tắc mượn tại thời điểm biên dịch, `RefCell<T>` cho phép chúng ta có nhiều lần mượn bất biến
hoặc một lần mượn có thể thay đổi tại bất kỳ thời điểm nào.

Nếu chúng ta cố gắng vi phạm các quy tắc này, thay vì nhận được lỗi trình biên dịch như chúng ta
thường thấy với các tham chiếu, thực thi của `RefCell<T>` sẽ panic tại
thời điểm chạy. Liệt kê 15-23 cho thấy một sửa đổi của việc thực thi `send` trong
Liệt kê 15-22. Chúng ta đang cố tình tạo ra hai lần mượn có thể thay đổi hoạt động
trong cùng một phạm vi để minh họa rằng `RefCell<T>` ngăn chúng ta làm điều này
tại thời điểm chạy.

<Listing number="15-23" file-name="src/lib.rs" caption="Tạo hai tham chiếu có thể thay đổi trong cùng một phạm vi để thấy rằng `RefCell<T>` sẽ panic">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Chúng ta tạo một biến `one_borrow` cho con trỏ thông minh `RefMut<T>` được trả về
từ `borrow_mut`. Sau đó chúng ta tạo một lần mượn có thể thay đổi khác theo cùng cách trong
biến `two_borrow`. Điều này tạo ra hai tham chiếu có thể thay đổi trong cùng một phạm vi,
điều này không được phép. Khi chúng ta chạy các bài kiểm tra cho thư viện của mình, mã trong Liệt kê
15-23 sẽ biên dịch mà không có bất kỳ lỗi nào, nhưng bài kiểm tra sẽ thất bại:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Lưu ý rằng mã đã bị panic với thông báo `already borrowed:
BorrowMutError`. Đây là cách `RefCell<T>` xử lý các vi phạm của các quy tắc mượn
tại thời điểm chạy.

Việc chọn bắt các lỗi mượn tại thời điểm chạy thay vì thời điểm biên dịch, như
chúng ta đã làm ở đây, có nghĩa là bạn có khả năng tìm thấy các lỗi trong mã của mình muộn hơn
trong quá trình phát triển: có thể cho đến khi mã của bạn được triển khai lên
môi trường production. Ngoài ra, mã của bạn sẽ chịu một hình phạt nhỏ về hiệu suất thời gian chạy như
kết quả của việc theo dõi các lần mượn tại thời điểm chạy thay vì thời điểm biên dịch.
Tuy nhiên, sử dụng `RefCell<T>` giúp có thể viết một mock object có thể
tự sửa đổi để theo dõi các thông báo mà nó đã thấy trong khi bạn đang sử dụng nó
trong một ngữ cảnh mà chỉ các giá trị bất biến được cho phép. Bạn có thể sử dụng `RefCell<T>`
bất chấp những đánh đổi của nó để có được nhiều chức năng hơn những gì các tham chiếu thông thường
cung cấp.

<!-- Old link, do not remove -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>

### Cho phép Đa sở hữu Dữ liệu có thể thay đổi với `Rc<T>` và `RefCell<T>`

Một cách phổ biến để sử dụng `RefCell<T>` là kết hợp với `Rc<T>`. Hãy nhớ rằng
`Rc<T>` cho phép bạn có nhiều chủ sở hữu đối với một số dữ liệu, nhưng nó chỉ cung cấp
quyền truy cập bất biến vào dữ liệu đó. Nếu bạn có một `Rc<T>` nắm giữ một `RefCell<T>`, bạn có thể
có được một giá trị có thể có nhiều chủ sở hữu _và_ mà bạn có thể thay đổi!

Ví dụ, hãy nhớ lại ví dụ cons list trong Liệt kê 15-18 nơi chúng ta đã sử dụng `Rc<T>`
để cho phép nhiều danh sách chia sẻ quyền sở hữu một danh sách khác. Bởi vì `Rc<T>`
chỉ nắm giữ các giá trị bất biến, chúng ta không thể thay đổi bất kỳ giá trị nào trong danh sách một khi
chúng ta đã tạo ra chúng. Hãy thêm `RefCell<T>` vào vì khả năng thay đổi các
giá trị trong danh sách của nó. Liệt kê 15-24 cho thấy rằng bằng cách sử dụng một `RefCell<T>` trong
định nghĩa `Cons`, chúng ta có thể sửa đổi giá trị được lưu trữ trong tất cả các danh sách.

<Listing number="15-24" file-name="src/main.rs" caption="Sử dụng `Rc<RefCell<i32>>` để tạo một `List` mà chúng ta có thể thay đổi">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Chúng ta tạo một giá trị là một instance của `Rc<RefCell<i32>>` và lưu trữ nó trong một
biến tên là `value` để chúng ta có thể truy cập trực tiếp sau này. Sau đó chúng ta tạo một
`List` trong `a` với một biến thể `Cons` nắm giữ `value`. Chúng ta cần clone
`value` để cả `a` và `value` đều có quyền sở hữu giá trị `5` bên trong thay vì
chuyển giao quyền sở hữu từ `value` sang `a` hoặc để `a` mượn từ
`value`.

Chúng ta bao bọc danh sách `a` trong một `Rc<T>` để khi chúng ta tạo các danh sách `b` và `c`,
chúng đều có thể tham chiếu đến `a`, đó là những gì chúng ta đã làm trong Liệt kê 15-18.

Sau khi chúng ta đã tạo các danh sách trong `a`, `b`, và `c`, chúng ta muốn cộng thêm 10 vào
giá trị trong `value`. Chúng ta thực hiện điều này bằng cách gọi `borrow_mut` trên `value`, nó sử dụng
tính năng tự động giải tham chiếu (automatic dereferencing) mà chúng ta đã thảo luận trong Chương 4 để
giải tham chiếu `Rc<T>` thành giá trị `RefCell<T>` bên trong. Phương thức `borrow_mut`
trả về một con trỏ thông minh `RefMut<T>`, và chúng ta sử dụng toán tử giải tham chiếu
trên nó và thay đổi giá trị bên trong.

Khi chúng ta in `a`, `b`, và `c`, chúng ta có thể thấy rằng tất cả chúng đều có giá trị đã sửa đổi
là `15` thay vì `5`:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Kỹ thuật này khá gọn gàng! Bằng cách sử dụng `RefCell<T>`, chúng ta có một giá trị `List` trông bề ngoài có vẻ
bất biến. Nhưng chúng ta có thể sử dụng các phương thức trên `RefCell<T>` cung cấp
quyền truy cập vào interior mutability của nó để chúng ta có thể sửa đổi dữ liệu của mình khi cần thiết.
Các kiểm tra tại thời điểm chạy của các quy tắc mượn bảo vệ chúng ta khỏi việc chạy đua dữ liệu, và đôi khi
đánh đổi một chút tốc độ lấy sự linh hoạt này trong các cấu trúc dữ liệu của chúng ta là xứng đáng. Lưu ý rằng `RefCell<T>` không hoạt động cho mã đa luồng!
`Mutex<T>` là phiên bản an toàn luồng (thread-safe) của `RefCell<T>`, và chúng ta sẽ thảo luận về
`Mutex<T>` trong Chương 16.

{{#quiz ../quizzes/ch15-05-interior-mutability.toml}}

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
