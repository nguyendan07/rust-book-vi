## Tất cả những nơi mẫu có thể được sử dụng

Mẫu xuất hiện ở nhiều nơi trong Rust, và bạn đã sử dụng chúng rất nhiều
mà không nhận ra đấy! Phần này thảo luận về tất cả những nơi mà
mẫu là hợp lệ.

### Các nhánh (Arms) của `match`

Như đã thảo luận trong Chương 6, chúng ta sử dụng các mẫu trong các nhánh của biểu thức `match`.
Về mặt hình thức, các biểu thức `match` được định nghĩa là từ khóa `match`, một giá trị để
khớp, và một hoặc nhiều nhánh match bao gồm một mẫu và một
biểu thức để chạy nếu giá trị khớp với mẫu của nhánh đó, như thế này:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>GIÁ_TRỊ</em> {
    <em>MẪU</em> => <em>BIỂU_THỨC</em>,
    <em>MẪU</em> => <em>BIỂU_THỨC</em>,
    <em>MẪU</em> => <em>BIỂU_THỨC</em>,
}</code></pre>

Ví dụ, đây là biểu thức `match` từ Liệt kê 6-5 khớp trên một
giá trị `Option<i32>` trong biến `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Các mẫu trong biểu thức `match` này là `None` và `Some(i)` ở
bên trái của mỗi mũi tên.

Một yêu cầu đối với các biểu thức `match` là chúng cần phải _đầy đủ_ (exhaustive) theo
nghĩa là tất cả các khả năng cho giá trị trong biểu thức `match` phải được
tính đến. Một cách để đảm bảo bạn đã bao quát mọi khả năng là có
một mẫu bao quát tất cả (catchall) cho nhánh cuối cùng: ví dụ, một tên biến
khớp với bất kỳ giá trị nào sẽ không bao giờ thất bại và do đó bao quát mọi trường hợp còn lại.

Mẫu cụ thể `_` sẽ khớp với bất kỳ thứ gì, nhưng nó không bao giờ liên kết với một biến,
vì vậy nó thường được sử dụng trong nhánh match cuối cùng. Mẫu `_` có thể
hữu ích khi bạn muốn bỏ qua bất kỳ giá trị nào không được chỉ định, chẳng hạn. Chúng ta sẽ đề cập đến
mẫu `_` chi tiết hơn trong phần [“Bỏ qua các giá trị trong một mẫu”][ignoring-values-in-a-pattern]<!-- ignore -->
sau này trong chương này.

### Biểu thức điều kiện `if let`

Trong Chương 6, chúng ta đã thảo luận về cách sử dụng biểu thức `if let` chủ yếu như
một cách ngắn gọn hơn để viết tương đương với một `match` chỉ khớp với một trường hợp.
Tùy chọn, `if let` có thể có một `else` tương ứng chứa mã để chạy nếu
mẫu trong `if let` không khớp.

Liệt kê 19-1 cho thấy cũng có thể kết hợp `if let`, `else if`,
và các biểu thức `else if let`. Làm như vậy mang lại cho chúng ta sự linh hoạt hơn so với
biểu thức `match` mà trong đó chúng ta chỉ có thể biểu thị một giá trị để so sánh với
các mẫu. Ngoài ra, Rust không yêu cầu các điều kiện trong một chuỗi các nhánh
`if let`, `else if`, `else if let` phải liên quan đến nhau.

Mã trong Liệt kê 19-1 xác định màu sắc nào để làm nền của bạn dựa trên
một loạt các kiểm tra cho một vài điều kiện. Đối với ví dụ này, chúng tôi đã tạo
các biến với các giá trị được mã hóa cứng (hardcoded) mà một chương trình thực tế có thể nhận được
từ đầu vào của người dùng.

<Listing number="19-1" file-name="src/main.rs" caption="Kết hợp `if let`, `else if`, `else if let`, và `else` nội dung">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs}}
```

</Listing>

Nếu người dùng chỉ định một màu yêu thích, màu đó sẽ được sử dụng làm nền.
Nếu không có màu yêu thích nào được chỉ định và hôm nay là thứ Ba, màu nền là
xanh lá cây. Ngược lại, nếu người dùng chỉ định tuổi của họ dưới dạng một chuỗi và
chúng ta có thể phân tích cú pháp nó thành một số thành công, màu sắc sẽ là tím
hoặc cam tùy thuộc vào giá trị của con số. Nếu không có điều kiện nào trong số này
áp dụng, màu nền là xanh dương.

Cấu trúc điều kiện này cho phép chúng ta hỗ trợ các yêu cầu phức tạp. Với các
giá trị được mã hóa cứng mà chúng ta có ở đây, ví dụ này sẽ in `Using purple as the
background color`.

Bạn có thể thấy rằng `if let` cũng có thể giới thiệu các biến mới che bóng (shadow)
các biến hiện có theo cùng cách mà các nhánh `match` có thể: dòng `if let Ok(age) = age`
giới thiệu một biến `age` mới chứa giá trị bên trong biến thể `Ok`,
che bóng biến `age` hiện có. Điều này có nghĩa là chúng ta cần đặt
điều kiện `if age > 30` bên trong khối đó: chúng ta không thể kết hợp hai điều kiện này
thành `if let Ok(age) = age && age > 30`. Biến `age` mới mà chúng ta muốn so sánh
với 30 không hợp lệ cho đến khi phạm vi (scope) mới bắt đầu với dấu ngoặc nhọn.

Nhược điểm của việc sử dụng các biểu thức `if let` là trình biên dịch không kiểm tra
tính đầy đủ, trong khi với các biểu thức `match` thì có. Nếu chúng ta bỏ qua
khối `else` cuối cùng và do đó bỏ lỡ việc xử lý một số trường hợp, trình biên dịch
sẽ không cảnh báo chúng ta về lỗi logic có thể xảy ra.

### Vòng lặp điều kiện `while let`

Tương tự về cấu trúc với `if let`, vòng lặp điều kiện `while let` cho phép một
vòng lặp `while` chạy miễn là một mẫu tiếp tục khớp. Trong Liệt kê 19-2
chúng tôi hiển thị một vòng lặp `while let` chờ các tin nhắn được gửi giữa các luồng,
nhưng trong trường hợp này là kiểm tra một `Result` thay vì một `Option`.

<Listing number="19-2" caption="Sử dụng vòng lặp `while let` để in các giá trị miễn là `rx.recv()` trả về `Ok` nội dung">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Ví dụ này in `1`, `2`, và sau đó là `3`. Phương thức `recv` lấy
tin nhắn đầu tiên ra khỏi phía người nhận của kênh và trả về một `Ok(value)`. Khi
lần đầu tiên chúng ta thấy `recv` trong Chương 16, chúng ta đã unwrap lỗi trực tiếp, hoặc
tương tác với nó như một trình lặp (iterator) bằng cách sử dụng vòng lặp `for`. Tuy nhiên, như
Liệt kê 19-2 cho thấy, chúng ta cũng có thể sử dụng `while let`, bởi vì phương thức `recv`
trả về `Ok` mỗi khi có tin nhắn đến, miễn là phía gửi còn tồn tại, và sau đó
tạo ra một `Err` khi phía gửi ngắt kết nối.

### Vòng lặp `for`

Trong một vòng lặp `for`, giá trị đứng ngay sau từ khóa `for` là một
mẫu. Ví dụ, trong `for x in y`, `x` là mẫu. Liệt kê 19-3
minh họa cách sử dụng một mẫu trong vòng lặp `for` để phá cấu trúc, hoặc chia nhỏ,
một tuple như một phần của vòng lặp `for`.

<Listing number="19-3" caption="Sử dụng một mẫu trong vòng lặp `for` để phá cấu trúc một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs:here}}
```

</Listing>

Mã trong Liệt kê 19-3 sẽ in ra như sau:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-03/output.txt}}
```

Chúng ta điều chỉnh một trình lặp bằng phương thức `enumerate` để nó tạo ra một giá trị và
chỉ số cho giá trị đó, được đặt vào một tuple. Giá trị đầu tiên được tạo ra là
tuple `(0, 'a')`. Khi giá trị này được khớp với mẫu `(index, value)`,
`index` sẽ là `0` và `value` sẽ là `'a'`, in ra dòng đầu tiên của
đầu ra.

### Các câu lệnh `let`

Trước chương này, chúng ta mới chỉ thảo luận rõ ràng về việc sử dụng các mẫu với
`match` và `if let`, nhưng thực tế, chúng ta đã sử dụng các mẫu ở
những nơi khác nữa, bao gồm cả trong các câu lệnh `let`. Ví dụ, hãy xem xét việc
gán biến đơn giản này với `let`:

```rust
let x = 5;
```

Mỗi khi bạn sử dụng một câu lệnh `let` như thế này, bạn đã và đang sử dụng các mẫu,
mặc dù bạn có thể đã không nhận ra điều đó! Chính thức hơn, một câu lệnh `let` trông
như thế này:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>MẪU</em> = <em>BIỂU_THỨC</em>;</code>
</pre>

Trong các câu lệnh như `let x = 5;` với một tên biến trong vị trí _`MẪU`_,
tên biến chỉ là một dạng mẫu đặc biệt đơn giản. Rust so sánh
biểu thức với mẫu và gán bất kỳ tên nào nó tìm thấy. Vì vậy, trong ví dụ
`let x = 5;`, `x` là một mẫu có nghĩa là “liên kết những gì khớp ở đây
với biến `x`.” Bởi vì tên `x` là toàn bộ mẫu, mẫu này
có nghĩa hiệu quả là “liên kết mọi thứ với biến `x`, bất kể giá trị là gì.”

Để thấy khía cạnh khớp mẫu của `let` rõ ràng hơn, hãy xem xét Liệt kê
19-4, sử dụng một mẫu với `let` để phá cấu trúc một tuple.

<Listing number="19-4" caption="Sử dụng một mẫu để phá cấu trúc một tuple và tạo ba biến cùng một lúc">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta khớp một tuple với một mẫu. Rust so sánh giá trị `(1, 2, 3)`
với mẫu `(x, y, z)` và thấy rằng giá trị khớp với mẫu, ở chỗ
số lượng phần tử là giống nhau ở cả hai, vì vậy Rust liên kết `1` với `x`,
`2` với `y`, và `3` với `z`. Bạn có thể nghĩ về mẫu tuple này
như việc lồng ba mẫu biến riêng lẻ vào bên trong nó.

Nếu số lượng phần tử trong mẫu không khớp với số lượng phần tử
trong tuple, kiểu tổng thể sẽ không khớp và chúng ta sẽ nhận được lỗi biên dịch.
Ví dụ, Liệt kê 19-5 cho thấy một nỗ lực phá cấu trúc một tuple
có ba phần tử thành hai biến, điều này sẽ không hoạt động.

<Listing number="19-5" caption="Xây dựng sai một mẫu có các biến không khớp với số lượng phần tử trong tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

Việc cố gắng biên dịch mã này dẫn đến lỗi kiểu dữ liệu sau:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

Để khắc phục lỗi, chúng ta có thể bỏ qua một hoặc nhiều giá trị trong tuple bằng cách sử dụng
`_` hoặc `..`, như bạn sẽ thấy trong phần [“Bỏ qua các giá trị trong một mẫu”][ignoring-values-in-a-pattern]

<!-- ignore -->. Nếu vấn đề là chúng ta có quá nhiều biến trong mẫu,

giải pháp là làm cho các kiểu khớp nhau bằng cách loại bỏ các biến để số lượng
biến bằng với số lượng phần tử trong tuple.

### Các tham số hàm

Các tham số hàm cũng có thể là các mẫu. Mã trong Liệt kê 19-6,
khai báo một hàm tên là `foo` nhận một tham số tên là `x` kiểu `i32`,
giờ đây hẳn đã trông quen thuộc với bạn.

<Listing number="19-6" caption="Một chữ ký hàm sử dụng các mẫu trong các tham số">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

Phần `x` là một mẫu! Như chúng ta đã làm với `let`, chúng ta có thể khớp
một tuple trong các đối số của một hàm với mẫu. Liệt kê 19-7
chia nhỏ các giá trị trong một tuple khi chúng ta truyền nó vào một hàm.

<Listing number="19-7" file-name="src/main.rs" caption="Một hàm với các tham số phá cấu trúc một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Mã này in ra `Current location: (3, 5)`. Các giá trị `&(3, 5)`
khớp với mẫu `&(x, y)`, vì vậy `x` là giá trị `3` và `y` là giá trị
`5`.

Chúng ta cũng có thể sử dụng các mẫu trong danh sách tham số của closure theo cùng cách
như trong danh sách tham số hàm vì closure tương tự như hàm,
như đã thảo luận trong Chương 13.

Tại thời điểm này, bạn đã thấy một số cách để sử dụng mẫu, nhưng mẫu không
hoạt động giống nhau ở mọi nơi chúng ta có thể sử dụng chúng. Ở một số nơi, các mẫu
phải là irrefutable; trong các trường hợp khác, chúng có thể là refutable.
Chúng ta sẽ thảo luận về hai khái niệm này tiếp theo.

{{#quiz ../quizzes/ch18-01-all-the-places-for-patterns.toml}}

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
