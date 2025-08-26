## Data Types

Mọi giá trị trong Rust thuộc một _kiểu dữ liệu_ nhất định, điều này cho Rust biết kiểu
dữ liệu đang được chỉ định để nó biết cách xử lý dữ liệu đó. Chúng ta sẽ xem
hai tập con kiểu dữ liệu: scalar và compound.

Hãy nhớ rằng Rust là một ngôn ngữ _kiểu tĩnh_ (statically typed), nghĩa là nó
phải biết kiểu của tất cả các biến tại thời điểm biên dịch. Trình biên dịch
thường có thể suy ra kiểu mà chúng ta muốn dùng dựa trên giá trị và cách chúng
ta sử dụng nó. Trong những trường hợp khi nhiều kiểu có thể đúng, chẳng hạn
khi chúng ta chuyển một `String` sang kiểu số bằng `parse` trong phần
[“Comparing the Guess to the Secret Number”][comparing-the-guess-to-the-secret-number]<!-- ignore --> của
Chương 2, chúng ta phải thêm chú thích kiểu, như sau:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Nếu chúng ta không thêm chú thích kiểu `: u32` như trong ví dụ trên, Rust sẽ
hiển thị lỗi sau, điều này có nghĩa là trình biên dịch cần thêm thông tin từ
chúng ta để biết kiểu mà chúng ta muốn sử dụng:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Bạn sẽ thấy các chú thích kiểu khác cho các kiểu dữ liệu khác.

### Scalar Types

Một kiểu _scalar_ biểu diễn một giá trị đơn. Rust có bốn kiểu scalar chính:
integers, floating-point numbers, Booleans, và characters. Bạn có thể nhận
ra những kiểu này từ các ngôn ngữ lập trình khác. Hãy bắt đầu với cách chúng
hoạt động trong Rust.

#### Integer Types

Một _số nguyên_ (integer) là một số không có phần thập phân. Chúng ta đã dùng
một kiểu số nguyên trong Chương 2, kiểu `u32`. Khai báo kiểu này chỉ ra rằng
giá trị liên kết với nó nên là một số nguyên không dấu (các kiểu số nguyên
có dấu bắt đầu bằng `i` thay vì `u`) chiếm 32 bit. Bảng 3-1 hiển thị các
kiểu số nguyên tích hợp trong Rust. Chúng ta có thể dùng bất kỳ biến thể nào
này để khai báo kiểu của giá trị số nguyên.

<span class="caption">Bảng 3-1: Các kiểu số nguyên trong Rust</span>

| Độ dài | Có dấu (Signed) | Không dấu (Unsigned) |
| ------- | --------------- | --------------------- |
| 8-bit   | `i8`            | `u8`                  |
| 16-bit  | `i16`           | `u16`                 |
| 32-bit  | `i32`           | `u32`                 |
| 64-bit  | `i64`           | `u64`                 |
| 128-bit | `i128`          | `u128`                |
| phụ thuộc kiến trúc | `isize` | `usize`         |

Mỗi biến thể có thể là có dấu hoặc không dấu và có kích thước cố định.
_Có dấu_ và _không dấu_ đề cập đến việc liệu số đó có thể là số âm hay không —
nói cách khác, liệu số có cần có dấu hay không (có dấu) hoặc liệu nó sẽ luôn
là dương và do đó có thể biểu diễn mà không cần dấu (không dấu). Nó giống như
việc viết số trên giấy: khi dấu quan trọng, một số được hiển thị với dấu cộng
hoặc dấu trừ; tuy nhiên, khi an toàn để giả sử số là dương, nó được hiển thị mà
không có dấu. Các số có dấu được lưu trữ bằng cách sử dụng biểu diễn [two’s complement][twos-complement]<!-- ignore -->.

Mỗi biến thể có dấu có thể lưu các số từ −(2<sup>n − 1</sup>) đến 2<sup>n −
1</sup> − 1 bao gồm, trong đó _n_ là số bit mà biến thể đó sử dụng. Vì vậy một
`i8` có thể lưu các số từ −(2<sup>7</sup>) đến 2<sup>7</sup> − 1, tương đương
−128 đến 127. Các biến thể không dấu có thể lưu các số từ 0 đến 2<sup>n</sup> − 1,
vì vậy một `u8` có thể lưu các số từ 0 đến 2<sup>8</sup> − 1, tương đương 0 đến 255.

Ngoài ra, các kiểu `isize` và `usize` phụ thuộc vào kiến trúc của máy tính
mà chương trình của bạn chạy: 64 bit nếu bạn đang trên kiến trúc 64 bit và 32
bit nếu bạn đang trên kiến trúc 32 bit.

Bạn có thể viết các literal số nguyên theo bất kỳ dạng nào được chỉ ra trong
Bảng 3-2. Lưu ý rằng các literal số có thể thuộc nhiều kiểu khác nhau cho
phép hậu tố kiểu, chẳng hạn như `57u8`, để chỉ định kiểu. Các literal số cũng
có thể sử dụng `_` như một dấu phân cách trực quan để làm cho số dễ đọc
hơn, chẳng hạn `1_000`, sẽ có cùng giá trị như khi bạn viết `1000`.

<span class="caption">Bảng 3-2: Literal số nguyên trong Rust</span>

| Literal số  | Ví dụ         |
| ------------ | ------------- |
| Thập phân    | `98_222`      |
| Thập lục phân| `0xff`        |
| Bát phân     | `0o77`        |
| Nhị phân     | `0b1111_0000` |
| Byte (chỉ `u8`)| `b'A'`     |

Vậy làm sao bạn biết nên dùng loại số nguyên nào? Nếu bạn không chắc, các
mặc định của Rust thường là nơi tốt để bắt đầu: các kiểu số nguyên mặc định
là `i32`. Tình huống chính mà bạn dùng `isize` hoặc `usize` là khi lập chỉ mục
(một số loại collection).

> ##### Integer Overflow
>
> Giả sử bạn có một biến kiểu `u8` có thể giữ giá trị từ 0 đến 255. Nếu bạn cố
> gắng thay đổi biến sang một giá trị ngoài khoảng đó, chẳng hạn 256, sẽ
> xảy ra _tràn số nguyên_ (integer overflow), điều này có thể dẫn đến hai hành
> vi khác nhau. Khi biên dịch ở chế độ debug, Rust bao gồm các kiểm tra tràn
> số nguyên khiến chương trình của bạn _panic_ khi chạy nếu hành vi này xảy
> ra. Rust dùng thuật ngữ _panicking_ khi một chương trình thoát với lỗi; chúng
> ta sẽ thảo luận kỹ hơn về panic trong phần [“Unrecoverable Errors with
> `panic!`”][unrecoverable-errors-with-panic]<!-- ignore --> ở Chương 9.
>
> Khi bạn biên dịch ở chế độ release với cờ `--release`, Rust _không_ bao gồm
> các kiểm tra tràn số nguyên gây panic. Thay vào đó, nếu xảy ra tràn, Rust
> thực hiện _quay vòng theo two’s complement_ (two’s complement wrapping). Tóm
> lại, các giá trị lớn hơn giá trị lớn nhất mà kiểu có thể giữ sẽ “quay vòng”
> về giá trị nhỏ nhất mà kiểu có thể giữ. Trong trường hợp `u8`, giá trị 256
> trở thành 0, giá trị 257 trở thành 1, v.v. Chương trình sẽ không panic, nhưng
> biến sẽ có một giá trị có lẽ không phải là điều bạn mong đợi. Dựa vào
> hành vi quay vòng của tràn số là một lỗi.
>
> Để xử lý rõ ràng khả năng xảy ra tràn, bạn có thể sử dụng các họ phương
> thức sau được cung cấp bởi thư viện chuẩn cho các kiểu số nguyên nguyên thủy:
>
> - Bao bọc trong mọi chế độ với các phương thức `wrapping_*`, chẳng hạn `wrapping_add`.
> - Trả về giá trị `None` nếu có tràn với các phương thức `checked_*`.
> - Trả về giá trị và một Boolean chỉ ra liệu có tràn hay không với các phương thức `overflowing_*`.
> - Bão hòa ở giá trị tối thiểu hoặc tối đa của kiểu với các phương thức `saturating_*`.

#### Floating-Point Types

Rust cũng có hai kiểu nguyên thủy cho _số thực_ (floating-point numbers), là
những số có dấu thập phân. Các kiểu floating-point của Rust là `f32` và `f64`,
lần lượt chiếm 32 bit và 64 bit. Kiểu mặc định là `f64` vì trên CPU hiện
đại, nó có tốc độ tương đương `f32` nhưng có thể biểu diễn chính xác hơn.
Tất cả các kiểu floating-point đều có dấu.

Dưới đây là một ví dụ cho thấy các số thực trong hành động:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Các số thực được biểu diễn theo tiêu chuẩn IEEE-754.

#### Numeric Operations

Rust hỗ trợ các phép toán toán học cơ bản mà bạn mong đợi cho tất cả các
kiểu số: cộng, trừ, nhân, chia, và phép dư. Phép chia số nguyên làm tròn
về phía 0 đến số nguyên gần nhất. Đoạn mã sau cho thấy cách bạn dùng mỗi phép
toán số trong một câu lệnh `let`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Mỗi biểu thức trong các câu lệnh này sử dụng một toán tử toán học và đánh
giá thành một giá trị đơn, sau đó được gán vào một biến. [Phụ lục
B][appendix_b]<!-- ignore --> chứa danh sách tất cả các toán tử mà Rust cung cấp.

#### The Boolean Type

Như hầu hết ngôn ngữ lập trình khác, kiểu Boolean trong Rust có hai giá trị
có thể: `true` và `false`. Boolean chiếm một byte. Kiểu Boolean trong Rust
được chỉ định bằng `bool`. Ví dụ:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Cách chính để sử dụng các giá trị Boolean là qua các biểu thức điều kiện,
chẳng hạn như một biểu thức `if`. Chúng ta sẽ trình bày cách hoạt động của
các biểu thức `if` trong Rust trong phần [“Control
Flow”][control-flow]<!-- ignore -->.

#### The Character Type

Kiểu `char` của Rust là kiểu chữ cái nguyên thủy cơ bản nhất của ngôn ngữ.
Dưới đây là một vài ví dụ khai báo các giá trị `char`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Lưu ý rằng ta khai báo literal `char` bằng dấu nháy đơn, trái ngược với literal
chuỗi (string) dùng dấu nháy kép. Kiểu `char` trong Rust chiếm bốn byte và
biểu diễn một Unicode scalar value, nghĩa là nó có thể biểu diễn nhiều hơn
chỉ ASCII. Các chữ cái có dấu; các ký tự Trung, Nhật, Hàn; emoji; và ký tự
không rộng (zero-width) đều là các giá trị `char` hợp lệ trong Rust. Các giá
trị Unicode scalar nằm trong khoảng từ `U+0000` đến `U+D7FF` và `U+E000` đến
`U+10FFFF` bao gồm. Tuy nhiên, “ký tự” không thực sự là một khái niệm trong
Unicode, vì vậy cảm nhận của con người về “ký tự” có thể không khớp với
những gì `char` biểu diễn trong Rust. Chúng ta sẽ thảo luận chi tiết chủ
đề này trong phần [“Storing UTF-8 Encoded Text with
Strings”][strings]<!-- ignore --> ở Chương 8.

### Compound Types

_Các kiểu compound_ có thể nhóm nhiều giá trị thành một kiểu. Rust có hai
kiểu compound nguyên thủy: tuples và arrays.

#### The Tuple Type

Một _tuple_ là cách tổng quát để nhóm một số giá trị với nhiều kiểu khác nhau
vào một kiểu compound. Tuple có độ dài cố định: một khi khai báo, chúng không
thể lớn hơn hoặc nhỏ hơn.

Chúng ta tạo một tuple bằng cách viết một danh sách các giá trị ngăn cách bởi dấu phẩy
bên trong ngoặc đơn. Mỗi vị trí trong tuple có một kiểu, và kiểu của các
giá trị khác nhau trong tuple không cần phải giống nhau. Chúng ta đã thêm
các chú thích kiểu tùy chọn trong ví dụ này:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

Biến `tup` gán cho toàn bộ tuple vì một tuple được xem là một phần tử compound
đơn. Để lấy các giá trị riêng lẻ từ một tuple, ta có thể sử dụng pattern
matching để destructure một giá trị tuple, như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Chương trình này trước tiên tạo một tuple và gán nó cho biến `tup`. Sau đó
nó dùng một pattern với `let` để lấy `tup` và tách nó thành ba biến riêng
biệt, `x`, `y`, và `z`. Điều này gọi là _destructuring_ vì nó phá vỡ tuple
đơn thành ba phần. Cuối cùng, chương trình in giá trị của `y`, là `6.4`.

Chúng ta cũng có thể truy cập phần tử tuple trực tiếp bằng cách dùng dấu chấm
(`.`) theo sau bởi chỉ số của giá trị muốn truy cập. Ví dụ:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Chương trình này tạo tuple `x` và sau đó truy cập mỗi phần tử của tuple bằng
các chỉ số tương ứng. Như hầu hết các ngôn ngữ lập trình, chỉ số đầu tiên trong
tuple là 0.

Tuple không có giá trị nào gọi là một tên đặc biệt, _unit_. Giá trị này và
kiểu tương ứng được viết `()` và biểu thị một giá trị rỗng hoặc kiểu trả về
rỗng. Các biểu thức ngầm định trả về giá trị unit nếu chúng không trả về giá trị khác.

#### The Array Type

Một cách khác để có một bộ nhiều giá trị là dùng _mảng_ (array). Khác với
tuple, mỗi phần tử của một array phải cùng kiểu. Khác với một số ngôn ngữ
khác, các mảng trong Rust có độ dài cố định.

Chúng ta viết các giá trị trong một mảng như một danh sách ngăn cách bởi dấu
phẩy bên trong ngoặc vuông:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Mảng hữu ích khi bạn muốn dữ liệu được cấp phát trên stack, giống như các
kiểu khác ta đã thấy, thay vì heap (chúng ta sẽ thảo luận về stack và heap
chi tiết hơn trong [Chương 4][stack-and-heap]<!-- ignore -->) hoặc khi bạn
muốn đảm bảo luôn có một số lượng phần tử cố định. Một mảng không linh hoạt
như kiểu vector. Một _vector_ là một kiểu collection tương tự được cung cấp
bởi thư viện chuẩn cho phép _tăng_ hoặc _giảm_ kích thước vì nội dung của
nó sống trên heap. Nếu bạn không chắc nên dùng mảng hay vector, có khả năng
bạn nên dùng vector. [Chương 8][vectors]<!-- ignore --> bàn luận vector chi tiết hơn.

Tuy nhiên, mảng hữu ích hơn khi bạn biết số lượng phần tử sẽ không cần thay đổi.
Ví dụ, nếu bạn dùng tên các tháng trong một chương trình, bạn có thể sẽ dùng
một mảng thay vì vector vì bạn biết nó sẽ luôn chứa 12 phần tử:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Bạn viết kiểu của một mảng bằng cách dùng ngoặc vuông với kiểu của mỗi phần tử,
một dấu chấm phẩy, rồi số lượng phần tử trong mảng, như sau:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Ở đây, `i32` là kiểu của mỗi phần tử. Sau dấu chấm phẩy, số `5` cho biết
mảng chứa năm phần tử.

Bạn cũng có thể khởi tạo một mảng để chứa cùng một giá trị cho mỗi phần tử
bằng cách chỉ định giá trị khởi tạo, theo sau bởi một dấu chấm phẩy, rồi độ dài
mảng trong ngoặc vuông, như ví dụ sau:

```rust
let a = [3; 5];
```

Mảng có tên `a` sẽ chứa 5 phần tử, tất cả đều được gán giá trị `3` ban đầu.
Điều này tương đương với việc viết `let a = [3, 3, 3, 3, 3];` nhưng ngắn gọn hơn.

##### Accessing Array Elements

Một mảng là một khối bộ nhớ đơn có kích thước cố định biết trước có thể được
cấp phát trên stack. Bạn có thể truy cập các phần tử của mảng bằng cách lập
chỉ mục, như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Trong ví dụ này, biến có tên `first` sẽ nhận giá trị `1` vì đó là giá trị ở
chỉ số `[0]` trong mảng. Biến có tên `second` sẽ nhận giá trị `2` từ chỉ số `[1]`
trong mảng.

##### Invalid Array Element Access

Hãy xem chuyện gì xảy ra nếu bạn cố gắng truy cập một phần tử của mảng vượt
quá cuối mảng. Giả sử bạn chạy mã này, tương tự trò chơi đoán số trong Chương 2,
để lấy chỉ số mảng từ người dùng:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Mã này biên dịch thành công. Nếu bạn chạy mã này bằng `cargo run` và nhập
`0`, `1`, `2`, `3`, hoặc `4`, chương trình sẽ in ra giá trị tương ứng tại chỉ
số đó trong mảng. Nếu thay vào đó bạn nhập một số vượt quá cuối mảng, chẳng
hạn `10`, bạn sẽ thấy đầu ra như sau:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Chương trình dẫn tới lỗi _runtime_ tại thời điểm sử dụng một giá trị không hợp lệ
trong phép lập chỉ mục. Chương trình thoát với thông báo lỗi và không thực
hiện câu lệnh `println!` cuối cùng. Khi bạn cố truy cập một phần tử bằng lập
chỉ mục, Rust sẽ kiểm tra rằng chỉ số bạn chỉ định nhỏ hơn độ dài mảng. Nếu chỉ
số lớn hơn hoặc bằng độ dài, Rust sẽ panic. Kiểm tra này phải xảy ra ở runtime,
đặc biệt trong trường hợp này, vì trình biên dịch không thể biết trước giá trị
mà người dùng sẽ nhập khi họ chạy mã sau này.

Đây là một ví dụ về nguyên tắc an toàn bộ nhớ của Rust. Trong nhiều ngôn ngữ
cấp thấp, loại kiểm tra này không được thực hiện, và khi bạn cung cấp một chỉ số
không đúng, bộ nhớ không hợp lệ có thể bị truy cập. Rust bảo vệ bạn khỏi loại lỗi
này bằng cách thoát ngay lập tức thay vì cho phép truy cập bộ nhớ và tiếp tục.
Chương 9 thảo luận thêm về xử lý lỗi trong Rust và cách bạn có thể viết mã
đọc được, an toàn sao cho không panic cũng không cho phép truy cập bộ nhớ không hợp lệ.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
