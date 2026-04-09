## Sử dụng `Box<T>` để trỏ đến dữ liệu trên Heap

Con trỏ thông minh đơn giản nhất là một _box_, có kiểu được viết là
`Box<T>`. Box cho phép bạn lưu trữ dữ liệu trên heap thay vì trên stack. Những gì
còn lại trên stack là con trỏ đến dữ liệu trên heap. Tham khảo Chương 4 để
xem lại sự khác biệt giữa stack và heap.

Box không có chi phí hiệu năng bổ sung, ngoại trừ việc lưu trữ dữ liệu của chúng trên
heap thay vì trên stack. Nhưng chúng cũng không có nhiều khả năng bổ sung.
Bạn sẽ sử dụng chúng thường xuyên nhất trong các tình huống sau:

- Khi bạn có một kiểu mà kích thước của nó không thể biết được tại thời điểm biên dịch và bạn muốn
  sử dụng một giá trị của kiểu đó trong một ngữ cảnh yêu cầu kích thước chính xác
- Khi bạn có một lượng lớn dữ liệu và bạn muốn chuyển giao quyền sở hữu nhưng
  đảm bảo dữ liệu sẽ không bị sao chép khi bạn làm như vậy
- Khi bạn muốn sở hữu một giá trị và bạn chỉ quan tâm rằng đó là một kiểu thực thi
  một trait cụ thể thay vì là một kiểu cụ thể

Chúng ta sẽ chứng minh tình huống đầu tiên trong [“Cho phép các kiểu đệ quy với Box”](#enabling-recursive-types-with-boxes)<!-- ignore -->. Trong trường hợp thứ hai,
việc chuyển giao quyền sở hữu một lượng lớn dữ liệu có thể mất nhiều thời gian
vì dữ liệu bị sao chép xung quanh trên stack. Để cải thiện hiệu năng trong tình huống này,
chúng ta có thể lưu trữ lượng lớn dữ liệu trên heap trong một box. Sau đó,
chỉ một lượng nhỏ dữ liệu con trỏ được sao chép xung quanh trên stack, trong khi
dữ liệu mà nó tham chiếu vẫn ở một chỗ trên heap. Trường hợp thứ ba được gọi là một
_trait object_, và phần [“Sử dụng Trait Objects cho phép các giá trị thuộc các kiểu khác nhau,”][trait-objects]<!-- ignore --> trong Chương 18 dành riêng cho chủ đề đó.
Vì vậy, những gì bạn học ở đây bạn sẽ áp dụng lại trong phần đó!

### Sử dụng `Box<T>` để lưu trữ dữ liệu trên Heap

Trước khi chúng ta thảo luận về trường hợp sử dụng lưu trữ trên heap cho `Box<T>`, chúng ta sẽ đề cập đến
cú pháp và cách tương tác với các giá trị được lưu trữ bên trong một `Box<T>`.

Liệt kê 15-1 cho thấy cách sử dụng một box để lưu trữ một giá trị `i32` trên heap.

<Listing number="15-1" file-name="src/main.rs" caption="Lưu trữ một giá trị `i32` trên heap bằng cách sử dụng một box">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

Chúng ta định nghĩa biến `b` có giá trị của một `Box` trỏ đến
giá trị `5`, giá trị này được cấp phát trên heap. Chương trình này sẽ in ra `b = 5`; trong
trường hợp này, chúng ta có thể truy cập dữ liệu trong box tương tự như cách chúng ta làm nếu dữ liệu này
ở trên stack. Giống như bất kỳ giá trị sở hữu nào, khi một box ra khỏi
phạm vi, như `b` làm ở cuối `main`, nó sẽ được giải phóng. Việc
giải phóng xảy ra cho cả box (được lưu trên stack) và dữ liệu mà nó trỏ tới (được lưu trên heap).

Việc đặt một giá trị đơn lẻ trên heap không hữu ích lắm, vì vậy bạn sẽ không thường xuyên sử dụng box
theo cách này. Việc có các giá trị như một `i32` đơn lẻ trên
stack, nơi chúng được lưu trữ theo mặc định, là phù hợp hơn trong phần lớn các tình huống. Hãy cùng xem
một trường hợp mà box cho phép chúng ta định nghĩa các kiểu mà chúng ta
không được phép nếu không có box.

### Cho phép các kiểu đệ quy với Box

Một giá trị của một _kiểu đệ quy_ (recursive type) có thể có một giá trị khác cùng kiểu như một phần của
chính nó. Các kiểu đệ quy gây ra một vấn đề vì Rust cần biết tại thời điểm biên dịch
một kiểu chiếm bao nhiêu không gian. Tuy nhiên, việc lồng nhau của các giá trị của các kiểu đệ quy
về mặt lý thuyết có thể tiếp tục vô hạn, vì vậy Rust không thể biết giá trị đó cần bao nhiêu không gian.
Bởi vì các box có kích thước đã biết, chúng ta có thể cho phép các kiểu đệ quy
bằng cách chèn một box vào định nghĩa kiểu đệ quy.

Ví dụ về một kiểu đệ quy, hãy khám phá _cons list_. Đây là một kiểu dữ liệu
thường thấy trong các ngôn ngữ lập trình hàm. Kiểu cons list
chúng ta sẽ định nghĩa là đơn giản ngoại trừ phần đệ quy; do đó, các
khái niệm trong ví dụ chúng ta sẽ làm việc sẽ hữu ích bất cứ khi nào bạn gặp phải
những tình huống phức tạp hơn liên quan đến các kiểu đệ quy.

#### Thông tin thêm về Cons List

Một _cons list_ là một cấu trúc dữ liệu bắt nguồn từ ngôn ngữ lập trình Lisp
và các phương ngữ của nó, được tạo thành từ các cặp lồng nhau, và là phiên bản Lisp của một
danh sách liên kết (linked list). Tên của nó bắt nguồn từ hàm `cons` (viết tắt của _construct function_) trong Lisp dùng để xây dựng một cặp mới từ hai đối số của nó. Bằng cách
gọi `cons` trên một cặp bao gồm một giá trị và một cặp khác, chúng ta có thể
xây dựng các cons list được tạo thành từ các cặp đệ quy.

Ví dụ, đây là một biểu diễn giả mã của một cons list chứa
danh sách `1, 2, 3` với mỗi cặp nằm trong dấu ngoặc đơn:

```text
(1, (2, (3, Nil)))
```

Mỗi mục trong một cons list chứa hai thành phần: giá trị của mục hiện tại
và mục tiếp theo. Mục cuối cùng trong danh sách chỉ chứa một giá trị được gọi là `Nil`
mà không có mục tiếp theo. Một cons list được tạo ra bằng cách gọi đệ quy
hàm `cons`. Tên chuẩn để biểu thị trường hợp cơ sở của đệ quy là `Nil`.
Lưu ý rằng điều này không giống với khái niệm “null” hoặc “nil” đã thảo luận trong
Chương 6, vốn là một giá trị không hợp lệ hoặc vắng mặt.

Cons list không phải là một cấu trúc dữ liệu được sử dụng phổ biến trong Rust. Hầu hết các trường hợp
khi bạn có một danh sách các mục trong Rust, `Vec<T>` là một lựa chọn tốt hơn để sử dụng.
Các kiểu dữ liệu đệ quy khác phức tạp hơn _lại_ hữu ích trong nhiều tình huống khác nhau,
nhưng bằng cách bắt đầu với cons list trong chương này, chúng ta có thể khám phá cách các box
cho phép chúng ta định nghĩa một kiểu dữ liệu đệ quy mà không có nhiều sự xao nhãng.

Liệt kê 15-2 chứa một định nghĩa enum cho một cons list. Lưu ý rằng mã này
sẽ chưa biên dịch được vì kiểu `List` không có kích thước đã biết, điều mà
chúng ta sẽ chứng minh.

<Listing number="15-2" file-name="src/main.rs" caption="Nỗ lực đầu tiên trong việc định nghĩa một enum để biểu diễn cấu trúc dữ liệu cons list của các giá trị `i32` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> Ghi chú: Chúng ta đang thực thi một cons list chỉ giữ các giá trị `i32` cho
> mục đích của ví dụ này. Chúng ta có thể đã thực thi nó bằng cách sử dụng generics, như chúng ta
> đã thảo luận trong Chương 10, để định nghĩa một kiểu cons list có thể lưu trữ các giá trị của
> bất kỳ kiểu nào.

Việc sử dụng kiểu `List` để lưu trữ danh sách `1, 2, 3` sẽ trông giống như mã trong
Liệt kê 15-3.

<Listing number="15-3" file-name="src/main.rs" caption="Sử dụng enum `List` để lưu trữ danh sách `1, 2, 3` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

Giá trị `Cons` đầu tiên giữ `1` và một giá trị `List` khác. Giá trị `List` này
là một giá trị `Cons` khác giữ `2` và một giá trị `List` khác. Giá trị `List` này
là một giá trị `Cons` nữa giữ `3` và một giá trị `List`, cuối cùng là
`Nil`, biến thể không đệ quy báo hiệu kết thúc danh sách.

Nếu chúng ta cố gắng biên dịch mã trong Liệt kê 15-3, chúng ta sẽ nhận được lỗi hiển thị trong
Liệt kê 15-4.

<Listing number="15-4" file-name="output.txt" caption="Lỗi chúng ta nhận được khi cố gắng định nghĩa một enum đệ quy">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

Lỗi cho thấy kiểu này “có kích thước vô hạn.” Lý do là chúng ta đã định nghĩa
`List` với một biến thể đệ quy: nó trực tiếp giữ một giá trị khác của chính nó.
Kết quả là, Rust không thể tính toán được nó cần bao nhiêu không gian để lưu trữ một
giá trị `List`. Hãy cùng phân tích tại sao chúng ta gặp lỗi này. Trước tiên, chúng ta sẽ xem cách
Rust quyết định cần bao nhiêu không gian để lưu trữ một giá trị của một kiểu không đệ quy.

#### Tính toán kích thước của một kiểu không đệ quy

Nhớ lại enum `Message` chúng ta đã định nghĩa trong Liệt kê 6-2 khi chúng ta thảo luận về các định nghĩa
enum trong Chương 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Để xác định cần cấp phát bao nhiêu không gian cho một giá trị `Message`, Rust đi
qua từng biến thể để xem biến thể nào cần nhiều không gian nhất. Rust
thấy rằng `Message::Quit` không cần bất kỳ không gian nào, `Message::Move` cần đủ
không gian để lưu trữ hai giá trị `i32`, và cứ thế. Bởi vì chỉ có một biến thể sẽ được
sử dụng, không gian tối đa mà một giá trị `Message` sẽ cần là không gian mà nó sẽ chiếm để
lưu trữ biến thể lớn nhất của nó.

Tương phản điều này với những gì xảy ra khi Rust cố gắng xác định cần bao nhiêu không gian cho một
kiểu đệ quy như enum `List` trong Liệt kê 15-2. Trình biên dịch bắt đầu
bằng cách xem xét biến thể `Cons`, biến thể này giữ một giá trị kiểu `i32` và một giá trị
kiểu `List`. Do đó, `Cons` cần một lượng không gian bằng kích thước của
một `i32` cộng với kích thước của một `List`. Để tìm ra kiểu `List` cần bao nhiêu bộ nhớ,
trình biên dịch xem xét các biến thể, bắt đầu với biến thể `Cons`.
Biến thể `Cons` giữ một giá trị kiểu `i32` và một giá trị kiểu `List`, và quá trình này
tiếp tục vô hạn, như được hiển thị trong Hình 15-1.

<img alt="An infinite Cons list" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 15-1: Một `List` vô hạn bao gồm các biến thể `Cons` vô hạn</span>

#### Sử dụng `Box<T>` để có một kiểu đệ quy với kích thước đã biết

Bởi vì Rust không thể tính toán được cần cấp phát bao nhiêu không gian cho các kiểu được định nghĩa
đệ quy, trình biên dịch đưa ra một lỗi với gợi ý hữu ích này:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

Trong gợi ý này, _sự gián tiếp_ (indirection) có nghĩa là thay vì lưu trữ một giá trị
trực tiếp, chúng ta nên thay đổi cấu trúc dữ liệu để lưu trữ giá trị một cách gián tiếp bằng cách
lưu trữ một con trỏ đến giá trị đó.

Bởi vì một `Box<T>` là một con trỏ, Rust luôn biết một `Box<T>` cần bao nhiêu
không gian: kích thước của một con trỏ không thay đổi dựa trên lượng dữ liệu mà nó
đang trỏ tới. Điều này có nghĩa là chúng ta có thể đặt một `Box<T>` vào bên trong biến thể `Cons`
thay vì một giá trị `List` khác trực tiếp. `Box<T>` sẽ trỏ đến giá trị `List`
tiếp theo sẽ nằm trên heap thay vì bên trong biến thể `Cons`.
Về mặt khái niệm, chúng ta vẫn có một danh sách, được tạo ra với các danh sách giữ các danh sách khác, nhưng
việc thực thi này bây giờ giống như việc đặt các mục cạnh nhau
thay vì lồng bên trong nhau.

Chúng ta có thể thay đổi định nghĩa của enum `List` trong Liệt kê 15-2 và cách sử dụng
của `List` trong Liệt kê 15-3 thành mã trong Liệt kê 15-5, mã này sẽ biên dịch được.

<Listing number="15-5" file-name="src/main.rs" caption="Định nghĩa của `List` sử dụng `Box<T>` để có kích thước đã biết">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

Biến thể `Cons` cần kích thước của một `i32` cộng với không gian để lưu trữ
dữ liệu con trỏ của box. Biến thể `Nil` không lưu trữ giá trị nào, vì vậy nó cần ít không gian hơn
biến thể `Cons`. Bây giờ chúng ta biết rằng bất kỳ giá trị `List` nào cũng sẽ chiếm
kích thước của một `i32` cộng với kích thước dữ liệu con trỏ của một box. Bằng cách sử dụng một box, chúng ta
đã phá vỡ chuỗi đệ quy vô hạn, vì vậy trình biên dịch có thể tính toán kích thước nó cần để
lưu trữ một giá trị `List`. Hình 15-2 cho thấy biến thể `Cons` trông như thế nào bây giờ.

<img alt="A finite Cons list" src="img/trpl15-02.svg" class="center" />

<span class="caption">Hình 15-2: Một `List` không có kích thước vô hạn
bởi vì `Cons` giữ một `Box`</span>

Box chỉ cung cấp sự gián tiếp và cấp phát trên heap; chúng không có bất kỳ
khả năng đặc biệt nào khác, giống như những gì chúng ta sẽ thấy với các kiểu con trỏ thông minh khác.
Chúng cũng không có chi phí hiệu năng mà các khả năng đặc biệt này gây ra, vì vậy
chúng có thể hữu ích trong các trường hợp như cons list nơi sự gián tiếp là tính năng duy nhất
chúng ta cần. Chúng ta sẽ xem xét thêm các trường hợp sử dụng cho box trong Chương 18.

Kiểu `Box<T>` là một con trỏ thông minh vì nó thực thi trait `Deref`,
cho phép các giá trị `Box<T>` được đối xử như các tham chiếu. Khi một giá trị `Box<T>`
ra khỏi phạm vi, dữ liệu heap mà box đang trỏ tới cũng được dọn dẹp
nhờ vào việc thực thi trait `Drop`. Hai trait này thậm chí sẽ
quan trọng hơn đối với chức năng được cung cấp bởi các kiểu con trỏ thông minh khác mà
chúng ta sẽ thảo luận trong phần còn lại của chương này. Hãy cùng khám phá hai trait này
chi tiết hơn.

{{#quiz ../quizzes/ch15-01-box.toml}}

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
