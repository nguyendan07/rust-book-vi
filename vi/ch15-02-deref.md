## Xử lý Smart Pointer giống như tham chiếu thông thường với `Deref`

<!-- Old link, do not remove -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>

Việc thực thi trait `Deref` cho phép bạn tùy chỉnh hành vi của
_toán tử giải tham chiếu_ (dereference operator) `*` (đừng nhầm lẫn với toán tử nhân hoặc toán tử glob).
Bằng cách thực thi `Deref` theo cách mà một con trỏ thông minh có thể được xử lý như một
tham chiếu thông thường, bạn có thể viết mã hoạt động trên các tham chiếu và sử dụng mã đó
với cả con trỏ thông minh.

Trước tiên hãy xem cách toán tử giải tham chiếu hoạt động với các tham chiếu thông thường.
Sau đó, chúng ta sẽ cố gắng định nghĩa một kiểu tùy chỉnh hoạt động giống như `Box<T>`, và xem tại sao
toán tử giải tham chiếu không hoạt động như một tham chiếu trên kiểu mới được định nghĩa của chúng ta.
Chúng ta sẽ khám phá cách thực thi trait `Deref` làm cho các con trỏ thông minh có thể
hoạt động theo những cách tương tự như các tham chiếu. Sau đó, chúng ta sẽ xem xét tính năng
_ép kiểu deref_ (deref coercion) của Rust và cách nó cho phép chúng ta làm việc với cả tham chiếu
hoặc con trỏ thông minh.

> Ghi chú: Có một điểm khác biệt lớn giữa kiểu `MyBox<T>` mà chúng ta sắp xây dựng
> và kiểu `Box<T>` thật sự: phiên bản của chúng ta sẽ không lưu trữ dữ liệu của nó trên heap.
> Chúng ta tập trung ví dụ này vào `Deref`, vì vậy nơi dữ liệu thực sự được lưu trữ
> ít quan trọng hơn hành vi giống như con trỏ.

<!-- Old links, do not remove -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### Đi theo Tham chiếu đến Giá trị

Một tham chiếu thông thường là một loại con trỏ, và một cách để nghĩ về một con trỏ là
như một mũi tên chỉ đến một giá trị được lưu trữ ở nơi khác. Trong Liệt kê 15-6, chúng ta tạo ra một
tham chiếu đến một giá trị `i32` và sau đó sử dụng toán tử giải tham chiếu để đi theo
tham chiếu đến giá trị đó.

<Listing number="15-6" file-name="src/main.rs" caption="Sử dụng toán tử giải tham chiếu để đi theo một tham chiếu đến một giá trị `i32` ">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

Biến `x` giữ một giá trị `i32` là `5`. Chúng ta gán `y` bằng một tham chiếu đến
`x`. Chúng ta có thể khẳng định rằng `x` bằng `5`. Tuy nhiên, nếu chúng ta muốn thực hiện một
khẳng định về giá trị trong `y`, chúng ta phải sử dụng `*y` để đi theo tham chiếu
đến giá trị mà nó đang trỏ tới (do đó gọi là _giải tham chiếu_) để trình biên dịch có thể so sánh
giá trị thực tế. Một khi chúng ta giải tham chiếu `y`, chúng ta có quyền truy cập vào giá trị số nguyên
mà `y` đang trỏ tới mà chúng ta có thể so sánh với `5`.

Nếu chúng ta cố gắng viết `assert_eq!(5, y);` thay vào đó, chúng ta sẽ nhận được lỗi biên dịch này:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Việc so sánh một số và một tham chiếu đến một số là không được phép vì chúng là
các kiểu khác nhau. Chúng ta phải sử dụng toán tử giải tham chiếu để đi theo tham chiếu
đến giá trị mà nó đang trỏ tới.

### Sử dụng `Box<T>` giống như một Tham chiếu

Chúng ta có thể viết lại mã trong Liệt kê 15-6 để sử dụng một `Box<T>` thay vì một
tham chiếu; toán tử giải tham chiếu được sử dụng trên `Box<T>` trong Liệt kê 15-7
hoạt động theo cùng một cách như toán tử giải tham chiếu được sử dụng trên tham chiếu trong
Liệt kê 15-6:

<Listing number="15-7" file-name="src/main.rs" caption="Sử dụng toán tử giải tham chiếu trên một `Box<i32>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

Sự khác biệt chính giữa Liệt kê 15-7 và Liệt kê 15-6 là ở đây chúng ta gán
`y` là một instance của một box trỏ đến một giá trị được sao chép của `x` thay vì một
tham chiếu trỏ đến giá trị của `x`. Trong khẳng định cuối cùng, chúng ta có thể sử dụng
toán tử giải tham chiếu để đi theo con trỏ của box theo cùng cách mà chúng ta đã làm
khi `y` là một tham chiếu. Tiếp theo, chúng ta sẽ khám phá điều gì đặc biệt ở `Box<T>`
mà cho phép chúng ta sử dụng toán tử giải tham chiếu bằng cách tự định nghĩa kiểu của riêng mình.

### Định nghĩa Con trỏ thông minh của riêng chúng ta

Hãy xây dựng một con trỏ thông minh tương tự như kiểu `Box<T>` được cung cấp bởi
thư viện tiêu chuẩn để trải nghiệm cách các con trỏ thông minh hành xử khác với
các tham chiếu theo mặc định. Sau đó chúng ta sẽ xem xét cách thêm khả năng sử dụng
toán tử giải tham chiếu.

Kiểu `Box<T>` cuối cùng được định nghĩa là một tuple struct với một thành phần, vì vậy
Liệt kê 15-8 định nghĩa một kiểu `MyBox<T>` theo cùng một cách. Chúng ta cũng sẽ định nghĩa một
hàm `new` để khớp với hàm `new` được định nghĩa trên `Box<T>`.

<Listing number="15-8" file-name="src/main.rs" caption="Định nghĩa kiểu `MyBox<T>` ">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

Chúng ta định nghĩa một struct tên là `MyBox` và khai báo một tham số generic `T` vì
chúng ta muốn kiểu của mình giữ các giá trị của bất kỳ kiểu nào. Kiểu `MyBox` là một tuple struct
với một thành phần kiểu `T`. Hàm `MyBox::new` nhận một tham số kiểu `T`
và trả về một instance `MyBox` giữ giá trị được truyền vào.

Hãy thử thêm hàm `main` trong Liệt kê 15-7 vào Liệt kê 15-8 và
thay đổi nó để sử dụng kiểu `MyBox<T>` chúng ta vừa định nghĩa thay vì `Box<T>`. Mã
trong Liệt kê 15-9 sẽ không biên dịch được vì Rust không biết cách giải tham chiếu
`MyBox`.

<Listing number="15-9" file-name="src/main.rs" caption="Cố gắng sử dụng `MyBox<T>` theo cách chúng ta đã sử dụng các tham chiếu và `Box<T>` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

Đây là lỗi biên dịch kết quả:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Kiểu `MyBox<T>` của chúng ta không thể được giải tham chiếu vì chúng ta chưa thực thi
khả năng đó trên kiểu của mình. Để cho phép giải tham chiếu với toán tử `*`, chúng ta
thực thi trait `Deref`.

<!-- Old link, do not remove -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### Thực thi trait `Deref`

Như đã thảo luận trong [“Thực thi một Trait trên một Kiểu”][impl-trait]<!-- ignore --> ở
Chương 10, để thực thi một trait, chúng ta cần cung cấp các thực thi cho các
phương thức bắt buộc của trait đó. Trait `Deref`, được cung cấp bởi thư viện tiêu chuẩn,
yêu cầu chúng ta thực thi một phương thức tên là `deref` mượn `self` và
trả về một tham chiếu đến dữ liệu bên trong. Liệt kê 15-10 chứa một thực thi
của `Deref` để thêm vào định nghĩa của `MyBox<T>`.

<Listing number="15-10" file-name="src/main.rs" caption="Thực thi `Deref` trên `MyBox<T>` ">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

Cú pháp `type Target = T;` định nghĩa một kiểu liên kết (associated type) cho trait `Deref`
sử dụng. Các kiểu liên kết là một cách hơi khác để khai báo một tham số generic,
nhưng bạn không cần lo lắng về chúng bây giờ; chúng ta sẽ đề cập đến chúng
chi tiết hơn ở Chương 20.

Chúng ta điền vào thân của phương thức `deref` với `&self.0` để `deref` trả về một
tham chiếu đến giá trị mà chúng ta muốn truy cập với toán tử `*`; nhớ lại từ
[“Sử dụng Tuple Structs Không có các Trường được đặt tên để Tạo ra các Kiểu Khác nhau”][tuple-structs]<!-- ignore --> ở Chương 5 rằng `.0` truy cập vào giá trị đầu tiên trong một tuple struct. Hàm `main` trong Liệt kê 15-9 gọi `*` trên
giá trị `MyBox<T>` bây giờ đã biên dịch được, và các khẳng định đã vượt qua!

Nếu không có trait `Deref`, trình biên dịch chỉ có thể giải tham chiếu các tham chiếu `&`.
Phương thức `deref` cung cấp cho trình biên dịch khả năng lấy một giá trị của bất kỳ kiểu nào
có thực thi `Deref` và gọi phương thức `deref` để lấy một tham chiếu `&` mà
nó biết cách giải tham chiếu.

Khi chúng ta nhập `*y` trong Liệt kê 15-9, đằng sau hậu trường Rust thực sự đã chạy mã này:

```rust,ignore
*(y.deref())
```

Rust thay thế toán tử `*` bằng một lời gọi đến phương thức `deref` và sau đó là một
giải tham chiếu thông thường để chúng ta không phải suy nghĩ về việc liệu chúng ta có cần
gọi phương thức `deref` hay không. Tính năng này của Rust cho phép chúng ta viết mã hoạt động
giống hệt nhau cho dù chúng ta có một tham chiếu thông thường hay một kiểu có thực thi
`Deref`.

Lý do phương thức `deref` trả về một tham chiếu đến một giá trị, và việc
giải tham chiếu thông thường bên ngoài dấu ngoặc đơn trong `*(y.deref())` vẫn cần thiết,
có liên quan đến hệ thống quyền sở hữu. Nếu phương thức `deref` trả về giá trị
trực tiếp thay vì một tham chiếu đến giá trị, giá trị đó sẽ bị di chuyển (move) ra khỏi
`self`. Chúng ta không muốn lấy quyền sở hữu của giá trị bên trong `MyBox<T>` trong
trường hợp này hoặc trong hầu hết các trường hợp chúng ta sử dụng toán tử giải tham chiếu.

Lưu ý rằng toán tử `*` được thay thế bằng một lời gọi đến phương thức `deref` và
sau đó là một lời gọi đến toán tử `*` chỉ một lần, mỗi lần chúng ta sử dụng một `*` trong mã của mình.
Bởi vì việc thay thế toán tử `*` không đệ quy vô hạn, chúng ta
kết thúc với dữ liệu kiểu `i32`, khớp với số `5` trong `assert_eq!` ở
Liệt kê 15-9.

### Ép kiểu Deref ngầm định với các Hàm và Phương thức

_Ép kiểu Deref_ (Deref coercion) chuyển đổi một tham chiếu đến một kiểu có thực thi trait `Deref`
thành một tham chiếu đến một kiểu khác. Ví dụ, ép kiểu deref có thể chuyển đổi
`&String` thành `&str` bởi vì `String` thực thi trait `Deref` sao cho nó
trả về `&str`. Ép kiểu Deref là một sự tiện lợi mà Rust thực hiện trên các đối số truyền vào
các hàm và phương thức, và chỉ hoạt động trên các kiểu có thực thi trait `Deref`.
Nó xảy ra tự động khi chúng ta truyền một tham chiếu đến giá trị của một kiểu cụ thể
như một đối số cho một hàm hoặc phương thức mà không khớp với kiểu tham số trong
định nghĩa hàm hoặc phương thức. Một chuỗi các lời gọi đến phương thức `deref` chuyển đổi
kiểu chúng ta cung cấp thành kiểu mà tham số cần.

Ép kiểu Deref đã được thêm vào Rust để các lập trình viên khi viết các lời gọi hàm và
phương thức không cần phải thêm nhiều tham chiếu và giải tham chiếu rõ ràng
với `&` và `*`. Tính năng ép kiểu deref cũng cho phép chúng ta viết nhiều mã hơn
có thể hoạt động cho cả tham chiếu hoặc con trỏ thông minh.

Để thấy ép kiểu deref hoạt động, hãy sử dụng kiểu `MyBox<T>` chúng ta đã định nghĩa trong
Liệt kê 15-8 cũng như việc thực thi `Deref` mà chúng ta đã thêm vào trong Liệt kê
15-10. Liệt kê 15-11 cho thấy định nghĩa của một hàm có một tham số string slice.

<Listing number="15-11" file-name="src/main.rs" caption="Một hàm `hello` có tham số `name` thuộc kiểu `&str` ">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

Chúng ta có thể gọi hàm `hello` với một string slice như một đối số, ví dụ như
`hello("Rust");`. Ép kiểu Deref giúp chúng ta có thể gọi `hello`
với một tham chiếu đến một giá trị kiểu `MyBox<String>`, như được hiển thị trong Liệt kê 15-12.

<Listing number="15-12" file-name="src/main.rs" caption="Gọi `hello` với một tham chiếu đến một giá trị `MyBox<String>`, điều này hoạt động nhờ vào ép kiểu deref">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

Ở đây chúng ta đang gọi hàm `hello` với đối số `&m`, vốn là một
tham chiếu đến một giá trị `MyBox<String>`. Bởi vì chúng ta đã thực thi trait `Deref`
trên `MyBox<T>` trong Liệt kê 15-10, Rust có thể biến `&MyBox<String>` thành `&String`
bằng cách gọi `deref`. Thư viện tiêu chuẩn cung cấp một thực thi của `Deref`
trên `String` trả về một string slice, và điều này có trong tài liệu API
cho `Deref`. Rust gọi `deref` một lần nữa để biến `&String` thành `&str`, điều này
khớp với định nghĩa của hàm `hello`.

Nếu Rust không thực thi ép kiểu deref, chúng ta sẽ phải viết mã trong
Liệt kê 15-13 thay vì mã trong Liệt kê 15-12 để gọi `hello` với một giá trị
kiểu `&MyBox<String>`.

<Listing number="15-13" file-name="src/main.rs" caption="Mã chúng ta sẽ phải viết nếu Rust không có ép kiểu deref">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)` giải tham chiếu `MyBox<String>` thành một `String`. Sau đó `&` và
`[..]` lấy một string slice của `String` bằng với toàn bộ chuỗi để
khớp với chữ ký của `hello`. Mã này khi không có ép kiểu deref thì khó
đọc, viết và hiểu hơn với tất cả các ký hiệu liên quan. Ép kiểu Deref
cho phép Rust xử lý các chuyển đổi này cho chúng ta một cách tự động.

Khi trait `Deref` được định nghĩa cho các kiểu liên quan, Rust sẽ phân tích các
kiểu đó và sử dụng `Deref::deref` nhiều lần nếu cần thiết để lấy một tham chiếu
khớp với kiểu của tham số. Số lần mà `Deref::deref` cần được
chèn vào được giải quyết tại thời điểm biên dịch, vì vậy không có hình phạt về hiệu năng khi chạy (runtime penalty) khi
tận dụng ép kiểu deref!

### Cách Ép kiểu Deref tương tác với Tính đột biến

Tương tự như cách bạn sử dụng trait `Deref` để ghi đè toán tử `*` trên
các tham chiếu bất biến, bạn có thể sử dụng trait `DerefMut` để ghi đè toán tử `*`
trên các tham chiếu có thể thay đổi (mutable references).

Rust thực hiện ép kiểu deref khi nó tìm thấy các kiểu và thực thi trait trong ba
trường hợp:

1. Từ `&T` sang `&U` khi `T: Deref<Target=U>`
2. Từ `&mut T` sang `&mut U` khi `T: DerefMut<Target=U>`
3. Từ `&mut T` sang `&U` khi `T: Deref<Target=U>`

Hai trường hợp đầu tiên là giống nhau ngoại trừ trường hợp thứ hai thực thi tính đột biến.
Trường hợp đầu tiên phát biểu rằng nếu bạn có một `&T`, và `T` thực thi `Deref` đến
một kiểu `U` nào đó, bạn có thể lấy một `&U` một cách minh bạch. Trường hợp thứ hai phát biểu rằng
việc ép kiểu deref tương tự cũng xảy ra cho các tham chiếu có thể thay đổi.

Trường hợp thứ ba lắt léo hơn: Rust cũng sẽ ép một tham chiếu có thể thay đổi thành một
tham chiếu bất biến. Nhưng điều ngược lại là _không_ thể: các tham chiếu bất biến sẽ
không bao giờ ép kiểu thành các tham chiếu có thể thay đổi. Bởi vì các quy tắc mượn, nếu bạn có
một tham chiếu có thể thay đổi, tham chiếu có thể thay đổi đó phải là tham chiếu duy nhất đến
dữ liệu đó (nếu không, chương trình sẽ không biên dịch được). Chuyển đổi một tham chiếu
có thể thay đổi thành một tham chiếu bất biến sẽ không bao giờ phá vỡ các quy tắc mượn.
Chuyển đổi một tham chiếu bất biến thành một tham chiếu có thể thay đổi sẽ yêu cầu rằng
tham chiếu bất biến ban đầu là tham chiếu bất biến duy nhất đến dữ liệu đó, nhưng
các quy tắc mượn không đảm bảo điều đó. Do đó, Rust không thể đưa ra
giả định rằng việc chuyển đổi một tham chiếu bất biến thành một tham chiếu có thể thay đổi là khả thi.

{{#quiz ../quizzes/ch15-02-deref.toml}}

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
