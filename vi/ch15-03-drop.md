## Thực thi mã khi dọn dẹp (giải phóng) với trait `Drop`

Trait quan trọng thứ hai đối với mô hình con trỏ thông minh là `Drop`, nó cho phép
bạn tùy chỉnh những gì xảy ra khi một giá trị sắp ra khỏi phạm vi (scope). Bạn có thể
cung cấp một thực thi cho trait `Drop` trên bất kỳ kiểu nào, và mã đó có thể
được sử dụng để giải phóng các tài nguyên như tệp hoặc kết nối mạng.

Chúng tôi giới thiệu `Drop` trong ngữ cảnh của con trỏ thông minh vì
chức năng của trait `Drop` hầu như luôn được sử dụng khi thực thi một
con trỏ thông minh. Ví dụ, khi một `Box<T>` bị hủy (dropped), nó sẽ giải phóng
không gian trên heap mà box đó trỏ tới.

Trong một số ngôn ngữ, đối với một số kiểu, lập trình viên phải gọi mã để giải phóng bộ nhớ
hoặc tài nguyên mỗi khi họ sử dụng xong một instance của các kiểu đó. Các ví dụ
bao gồm file handles, sockets, và locks. Nếu họ quên, hệ thống có thể
trở nên quá tải và bị treo. Trong Rust, bạn có thể chỉ định một đoạn mã
cụ thể sẽ được chạy bất cứ khi nào một giá trị ra khỏi phạm vi, và trình biên dịch sẽ tự động
chèn mã này. Kết quả là, bạn không cần phải cẩn thận về việc
sử dụng code dọn dẹp ở khắp mọi nơi trong chương trình khi một instance của một kiểu cụ thể
đã kết thúc—tài nguyên vẫn được giải phóng an toàn, không bị rò rỉ!

Bạn chỉ định mã sẽ chạy khi một giá trị ra khỏi phạm vi bằng cách thực thi
trait `Drop`. Trait `Drop` yêu cầu bạn thực thi một phương thức tên là
`drop` nhận một tham chiếu có thể thay đổi (mutable reference) đến `self`. Để xem khi nào Rust gọi `drop`,
hãy thực thi `drop` với các câu lệnh `println!` ngay bây giờ.

Liệt kê 15-14 cho thấy một struct `CustomSmartPointer` có chức năng tùy chỉnh
duy nhất là nó sẽ in ra `Dropping CustomSmartPointer!` khi
instance ra khỏi phạm vi, để cho thấy khi nào Rust chạy phương thức `drop`.

<Listing number="15-14" file-name="src/main.rs" caption="Một struct `CustomSmartPointer` thực thi trait `Drop` nơi chúng ta sẽ đặt mã dọn dẹp của mình">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

Trait `Drop` được bao gồm trong prelude, vì vậy chúng ta không cần phải đưa nó vào
phạm vi. Chúng ta thực thi trait `Drop` trên `CustomSmartPointer` và cung cấp một
thực thi cho phương thức `drop` có gọi `println!`. Thân của phương thức
`drop` là nơi bạn sẽ đặt bất kỳ logic nào mà bạn muốn chạy khi một
instance của kiểu của bạn ra khỏi phạm vi. Chúng ta đang in một số văn bản ở đây để
minh họa trực quan khi nào Rust sẽ gọi `drop`.

Trong `main`, chúng ta tạo hai instance của `CustomSmartPointer` và sau đó in
`CustomSmartPointers created`. Ở cuối `main`, các instance của
`CustomSmartPointer` của chúng ta sẽ ra khỏi phạm vi, và Rust sẽ gọi mã chúng ta đã đặt
trong phương thức `drop`, in ra thông báo cuối cùng của chúng ta. Lưu ý rằng chúng ta không cần phải
gọi phương thức `drop` một cách rõ ràng.

Khi chúng ta chạy chương trình này, chúng ta sẽ thấy kết quả sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust đã tự động gọi `drop` cho chúng ta khi các instance của chúng ta ra khỏi phạm vi,
chạy mã mà chúng ta đã chỉ định. Các biến được hủy theo thứ tự ngược lại
với lúc chúng được tạo ra, vì vậy `d` đã bị hủy trước `c`. Mục đích của ví dụ này là
để cung cấp cho bạn một hướng dẫn trực quan về cách phương thức `drop` hoạt động; thông thường bạn sẽ
chỉ định mã dọn dẹp mà kiểu của bạn cần chạy thay vì một thông báo in ra.

<!-- Old link, do not remove -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

Thật không may, không dễ dàng để vô hiệu hóa chức năng `drop`
tự động. Việc vô hiệu hóa `drop` thường không cần thiết; toàn bộ ý nghĩa của
trait `Drop` là nó được tự động xử lý. Tuy nhiên, đôi khi,
bạn có thể muốn dọn dẹp một giá trị sớm hơn. Một ví dụ là khi sử dụng các con trỏ
thông minh quản lý các khóa (locks): bạn có thể muốn ép buộc phương thức `drop`
giải phóng khóa để mã khác trong cùng phạm vi có thể lấy được khóa đó.
Rust không cho phép bạn gọi phương thức `drop` của trait `Drop` một cách thủ công; thay vào đó,
bạn phải gọi hàm `std::mem::drop` được cung cấp bởi thư viện tiêu chuẩn
nếu bạn muốn ép buộc một giá trị bị hủy trước khi kết thúc phạm vi của nó.

Nếu chúng ta cố gắng gọi phương thức `drop` của trait `Drop` một cách thủ công bằng cách sửa đổi
hàm `main` từ Liệt kê 15-14, như được hiển thị trong Liệt kê 15-15, chúng ta sẽ nhận được một
lỗi trình biên dịch.

<Listing number="15-15" file-name="src/main.rs" caption="Cố gắng gọi phương thức `drop` từ trait `Drop` một cách thủ công để dọn dẹp sớm">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

Khi chúng ta cố gắng biên dịch mã này, chúng ta sẽ nhận được lỗi này:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Thông báo lỗi này cho biết rằng chúng ta không được phép gọi `drop` một cách rõ ràng.
Thông báo lỗi sử dụng thuật ngữ _destructor_ (hàm hủy), đây là thuật ngữ lập trình chung
cho một hàm dọn dẹp một instance. Một _destructor_ tương tự như một
_constructor_ (hàm khởi tạo), cái tạo ra một instance. Hàm `drop` trong Rust là một
destructor cụ thể.

Rust không cho phép chúng ta gọi `drop` một cách rõ ràng vì Rust vẫn sẽ
tự động gọi `drop` trên giá trị đó ở cuối `main`. Điều này sẽ gây ra một
lỗi _double free_ (giải phóng hai lần) vì Rust sẽ cố gắng dọn dẹp cùng một giá trị
hai lần.

Chúng ta không thể vô hiệu hóa việc tự động chèn `drop` khi một giá trị ra khỏi
phạm vi, và chúng ta không thể gọi phương thức `drop` một cách rõ ràng. Vì vậy, nếu chúng ta cần ép buộc
một giá trị được dọn dẹp sớm, chúng ta sử dụng hàm `std::mem::drop`.

Hàm `std::mem::drop` khác với phương thức `drop` trong trait `Drop`.
Chúng ta gọi nó bằng cách truyền vào dưới dạng một đối số giá trị mà chúng ta muốn ép buộc hủy.
Hàm này nằm trong prelude, vì vậy chúng ta có thể sửa đổi `main` trong Liệt kê 15-15 để
gọi hàm `drop`, như được hiển thị trong Liệt kê 15-16.

<Listing number="15-16" file-name="src/main.rs" caption="Gọi `std::mem::drop` để hủy một giá trị một cách rõ ràng trước khi nó ra khỏi phạm vi">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

Chạy mã này sẽ in ra những nội dung sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Dòng chữ ``Dropping CustomSmartPointer with data `some data`!`` được in ra
giữa văn bản `CustomSmartPointer created.` và `CustomSmartPointer dropped
before the end of main.`, cho thấy mã phương thức `drop` được gọi để
hủy `c` tại thời điểm đó.

Bạn có thể sử dụng mã được chỉ định trong một thực thi trait `Drop` theo nhiều cách để
làm cho việc dọn dẹp trở nên thuận tiện và an toàn: ví dụ, bạn có thể sử dụng nó để tạo
trình cấp phát bộ nhớ (memory allocator) của riêng mình! Với trait `Drop` và hệ thống quyền sở hữu của Rust,
bạn không cần phải nhớ dọn dẹp vì Rust thực hiện việc đó một cách tự động.

Bạn cũng không phải lo lắng về các vấn đề phát sinh từ việc vô tình
dọn dẹp các giá trị vẫn còn đang được sử dụng: hệ thống quyền sở hữu đảm bảo
các tham chiếu luôn hợp lệ cũng đảm bảo rằng `drop` chỉ được gọi một lần khi
giá trị không còn được sử dụng nữa.

Bây giờ chúng ta đã xem xét `Box<T>` và một số đặc điểm của con trỏ
thông minh, hãy cùng xem xét một vài con trỏ thông minh khác được định nghĩa trong thư viện
tiêu chuẩn.

{{#quiz ../quizzes/ch15-03-drop.toml}}
