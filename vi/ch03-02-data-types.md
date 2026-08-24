## Kiểu Dữ Liệu (Data Types)

Mọi giá trị trong Rust đều thuộc về một _kiểu dữ liệu_ (data type) nhất định, cho Rust biết loại dữ liệu nào đang được chỉ định để nó biết cách xử lý dữ liệu đó. Chúng ta sẽ xem xét hai tập hợp con của kiểu dữ liệu: kiểu vô hướng (scalar) và kiểu kết hợp (compound).

Hãy nhớ rằng Rust là một ngôn ngữ _định kiểu tĩnh_ (statically typed), có nghĩa là nó phải biết kiểu của tất cả các biến tại thời điểm biên dịch. Trình biên dịch thường có thể tự suy luận kiểu chúng ta muốn sử dụng dựa trên giá trị và cách chúng ta sử dụng nó. Trong các trường hợp có thể có nhiều kiểu dữ liệu, chẳng hạn như khi chúng ta chuyển đổi một `String` thành kiểu số bằng `parse` trong phần [“So Sánh Số Dự Đoán với Số Bí Mật”][comparing-the-guess-to-the-secret-number]<!-- ignore --> ở Chương 2, chúng ta phải thêm một chú thích kiểu (type annotation), như sau:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Nếu chúng ta không thêm chú thích kiểu `: u32` như trong đoạn mã trên, Rust sẽ hiển thị lỗi sau, có nghĩa là trình biên dịch cần thêm thông tin từ chúng ta để biết chúng ta muốn sử dụng kiểu dữ liệu nào:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Bạn sẽ thấy các chú thích kiểu khác nhau cho các kiểu dữ liệu khác.

### Kiểu Vô Hướng (Scalar Types)

Một kiểu _vô hướng_ (scalar) đại diện cho một giá trị đơn lẻ. Rust có bốn kiểu vô hướng chính: số nguyên (integers), số thực dấu phẩy động (floating-point numbers), Boolean, và ký tự (characters). Bạn có thể nhận ra những kiểu này từ các ngôn ngữ lập trình khác. Hãy cùng tìm hiểu cách chúng hoạt động trong Rust.

#### Các Kiểu Số Nguyên (Integer Types)

Một _số nguyên_ (integer) là một con số không có phần phân số. Chúng ta đã sử dụng một kiểu số nguyên trong Chương 2, kiểu `u32`. Khai báo kiểu này chỉ ra rằng giá trị liên kết với nó phải là một số nguyên không dấu (kiểu số nguyên có dấu bắt đầu bằng `i` thay vì `u`) chiếm 32 bit dung lượng. Bảng 3-1 hiển thị các kiểu số nguyên tích hợp sẵn trong Rust. Chúng ta có thể sử dụng bất kỳ kiểu nào trong số này để khai báo kiểu của một giá trị số nguyên.

<span class="caption">Bảng 3-1: Các kiểu số nguyên trong Rust</span>

| Độ dài | Có dấu (Signed) | Không dấu (Unsigned) |
| ------- | ------- | -------- |
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| Phụ thuộc kiến trúc CPU | `isize` | `usize`  |

Mỗi biến thể có thể có dấu hoặc không dấu và có một kích thước rõ ràng. _Có dấu_ (signed) và _không dấu_ (unsigned) đề cập đến việc con số đó có khả năng âm hay không — nói cách khác, liệu con số đó có cần mang theo dấu (có dấu) hay nó sẽ luôn luôn là số dương và do đó có thể biểu diễn mà không cần dấu (không dấu). Nó giống như viết số trên giấy: khi dấu là quan trọng, một con số được hiển thị với dấu cộng hoặc dấu trừ; tuy nhiên, khi an toàn để giả định con số là dương, nó được viết mà không có dấu. Các số có dấu được lưu trữ bằng cách sử dụng biểu diễn [bù 2 (two’s complement)][twos-complement]<!-- ignore -->.

Mỗi kiểu có dấu có thể lưu trữ các số từ −(2<sup>n − 1</sup>) đến 2<sup>n − 1</sup> − 1 (bao gồm cả hai đầu), trong đó _n_ là số bit mà kiểu đó sử dụng. Do đó, một `i8` có thể lưu trữ các số từ −(2<sup>7</sup>) đến 2<sup>7</sup> − 1, tương đương với −128 đến 127. Các kiểu không dấu có thể lưu trữ các số từ 0 đến 2<sup>n</sup> − 1, do đó một `u8` có thể lưu trữ các số từ 0 đến 2<sup>8</sup> − 1, tương đương với 0 đến 255.

Ngoài ra, các kiểu `isize` và `usize` phụ thuộc vào kiến trúc của máy tính mà chương trình của bạn đang chạy: 64 bit nếu bạn đang ở trên kiến trúc 64-bit và 32 bit nếu bạn đang ở trên kiến trúc 32-bit.

Bạn có thể viết các hằng số nguyên (integer literals) theo bất kỳ dạng nào được hiển thị trong Bảng 3-2. Lưu ý rằng các hằng số có thể thuộc nhiều kiểu số khác nhau cho phép dùng hậu tố kiểu, chẳng hạn như `57u8`, để chỉ định kiểu dữ liệu. Hằng số cũng có thể sử dụng `_` như một dấu phân cách trực quan để giúp số dễ đọc hơn, chẳng hạn như `1_000`, có giá trị giống hệt như khi bạn viết `1000`.

<span class="caption">Bảng 3-2: Các dạng hằng số nguyên trong Rust</span>

| Hằng số nguyên | Ví dụ |
| ---------------- | ------------- |
| Thập phân (Decimal) | `98_222` |
| Thập lục phân (Hex) | `0xff` |
| Bát phân (Octal) | `0o77` |
| Nhị phân (Binary) | `0b1111_0000` |
| Byte (chỉ dành cho `u8`) | `b'A'` |

Làm thế nào bạn biết nên sử dụng kiểu số nguyên nào? Nếu bạn không chắc chắn, các giá trị mặc định của Rust thường là điểm khởi đầu tốt: các kiểu số nguyên mặc định là `i32`. Trường hợp chính mà bạn sẽ sử dụng `isize` hoặc `usize` là khi đánh chỉ mục cho một tập hợp dữ liệu nào đó.

> ##### Tràn Số Nguyên (Integer Overflow)
>
> Giả sử bạn có một biến kiểu `u8` có thể chứa các giá trị từ 0 đến 255. Nếu bạn cố gắng thay đổi biến thành một giá trị nằm ngoài phạm vi đó, chẳng hạn như 256, hiện tượng _tràn số nguyên_ (integer overflow) sẽ xảy ra, dẫn đến một trong hai hành vi sau.
> Khi bạn biên dịch ở chế độ debug, Rust bao gồm các kiểm tra tràn số nguyên khiến chương trình của bạn _panic_ (dừng khẩn cấp) tại thời gian chạy nếu hành vi này xảy ra. Rust sử dụng thuật ngữ _panicking_ khi một chương trình thoát với một lỗi; chúng ta sẽ thảo luận về panic sâu hơn trong phần [“Lỗi Không Thể Phục Hồi với `panic!`”][unrecoverable-errors-with-panic]<!-- ignore --> ở Chương 9.
>
> Khi bạn biên dịch ở chế độ release với cờ `--release`, Rust _không_ bao gồm các kiểm tra tràn số nguyên gây ra panic. Thay vào đó, nếu xảy ra tràn số, Rust thực hiện _xoay vòng bù 2 (two’s complement wrapping)_. Nói ngắn gọn, các giá trị lớn hơn giá trị tối đa mà kiểu có thể chứa sẽ "quay vòng" về giá trị nhỏ nhất mà kiểu có thể chứa. Trong trường hợp của `u8`, giá trị 256 trở thành 0, giá trị 257 trở thành 1, v.v. Chương trình sẽ không panic, nhưng biến sẽ có một giá trị có thể không phải là những gì bạn mong đợi. Việc dựa vào hành vi xoay vòng của tràn số nguyên được coi là một lỗi logic.
>
> Để xử lý rõ ràng khả năng tràn số, bạn có thể sử dụng các nhóm phương thức được thư viện chuẩn cung cấp cho các kiểu số nguyên thủy:
>
> - Xoay vòng trong mọi chế độ với các phương thức `wrapping_*`, chẳng hạn như `wrapping_add`.
> - Trả về giá trị `None` nếu có tràn số với các phương thức `checked_*`.
> - Trả về giá trị và một giá trị Boolean cho biết liệu có tràn số hay không với các phương thức `overflowing_*`.
> - Giới hạn ở giá trị tối thiểu hoặc tối đa với các phương thức `saturating_*`.

#### Các Kiểu Số Thực Dấu Phẩy Động (Floating-Point Types)

Rust cũng có hai kiểu nguyên thủy cho _số thực dấu phẩy động_ (floating-point numbers), là các số có phần thập phân. Các kiểu số thực dấu phẩy động của Rust là `f32` và `f64`, có kích thước tương ứng là 32 bit và 64 bit. Kiểu mặc định là `f64` vì trên các CPU hiện đại, tốc độ xử lý của nó gần tương đương với `f32` nhưng mang lại độ chính xác cao hơn. Tất cả các kiểu số thực dấu phẩy động đều là số có dấu.

Dưới đây là một ví dụ minh họa cách sử dụng số thực dấu phẩy động:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Các số thực dấu phẩy động được biểu diễn theo tiêu chuẩn IEEE-754.

#### Các Phép Toán Số Học (Numeric Operations)

Rust hỗ trợ các phép toán số học cơ bản cho tất cả các kiểu số: cộng, trừ, nhân, chia, và chia lấy dư. Phép chia số nguyên sẽ làm tròn về phía số 0 đến số nguyên gần nhất (cắt bỏ phần thập phân). Đoạn mã sau đây minh họa cách sử dụng từng phép toán số học trong câu lệnh `let`:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Mỗi biểu thức trong các câu lệnh này sử dụng một toán tử toán học và cho ra một giá trị đơn lẻ, sau đó được liên kết với một biến. [Phụ lục B][appendix_b]<!-- ignore --> chứa danh sách đầy đủ tất cả các toán tử mà Rust cung cấp.

#### Kiểu Boolean

Giống như trong hầu hết các ngôn ngữ lập trình khác, kiểu Boolean trong Rust có hai giá trị có thể có: `true` và `false`. Kiểu Boolean có kích thước là một byte. Kiểu Boolean trong Rust được chỉ định bằng `bool`. Ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Cách chính để sử dụng các giá trị Boolean là thông qua các câu lệnh điều kiện, chẳng hạn như biểu thức `if`. Chúng ta sẽ tìm hiểu cách biểu thức `if` hoạt động trong Rust trong phần [“Luồng Điều Khiển”][control-flow]<!-- ignore -->.

#### Kiểu Ký Tự (Character Type)

Kiểu `char` của Rust là kiểu dữ liệu ký tự nguyên thủy nhất của ngôn ngữ. Dưới đây là một số ví dụ về việc khai báo các giá trị `char`:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Lưu ý rằng chúng ta chỉ định các hằng ký tự `char` bằng dấu nháy đơn (`'`), trái ngược với các chuỗi ký tự (string literals) sử dụng dấu nháy kép (`"`). Kiểu `char` của Rust có kích thước bốn byte và đại diện cho một Giá trị Vô hướng Unicode (Unicode Scalar Value), điều đó có nghĩa là nó có thể đại diện cho nhiều thứ hơn là chỉ các ký tự ASCII. Các chữ cái có dấu; ký tự tiếng Trung, tiếng Nhật, tiếng Hàn; biểu tượng cảm xúc (emoji); và các khoảng trắng có độ rộng bằng 0 đều là các giá trị `char` hợp lệ trong Rust. Các giá trị vô hướng Unicode nằm trong phạm vi từ `U+0000` đến `U+D7FF` và `U+E000` đến `U+10FFFF` (bao gồm cả hai đầu). Chúng ta sẽ thảo luận chi tiết về chủ đề này trong phần [“Lưu Trữ Văn Bản Được Mã Hóa UTF-8 Với String”][strings]<!-- ignore --> ở Chương 8.

{{#quiz ../quizzes/ch03-02-data-types-sec1-scalar.toml}}

### Các Kiểu Kết Hợp (Compound Types)

_Các kiểu kết hợp_ (compound types) có thể nhóm nhiều giá trị thành một kiểu duy nhất. Rust có hai kiểu kết hợp nguyên thủy: tuple và mảng (array).

#### Kiểu Tuple

Một _tuple_ là một cách tổng quát để nhóm một số giá trị với các kiểu dữ liệu khác nhau thành một kiểu kết hợp duy nhất. Tuple có độ dài cố định: một khi đã được khai báo, chúng không thể tăng hoặc giảm kích thước.

Chúng ta tạo một tuple bằng cách viết một danh sách các giá trị cách nhau bằng dấu phẩy bên trong dấu ngoặc đơn. Mỗi vị trí trong tuple có một kiểu dữ liệu riêng, và kiểu của các giá trị khác nhau trong tuple không bắt buộc phải giống nhau. Chúng ta đã thêm các chú thích kiểu tùy chọn trong ví dụ này:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

Biến `tup` liên kết với toàn bộ tuple vì tuple được coi là một phần tử kết hợp đơn lẻ. Để lấy các giá trị riêng lẻ ra khỏi tuple, chúng ta có thể sử dụng khớp mẫu (pattern matching) để giải cấu trúc (destructure) một giá trị tuple, như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Chương trình này đầu tiên tạo một tuple và liên kết nó với biến `tup`. Sau đó, nó sử dụng một mẫu với `let` để lấy `tup` và biến nó thành ba biến riêng biệt: `x`, `y`, và `z`. Thao tác này được gọi là _giải cấu trúc_ (destructuring) vì nó chia tách tuple đơn lẻ thành ba phần. Cuối cùng, chương trình in giá trị của `y` là `6.4`.

Chúng ta cũng có thể truy cập trực tiếp một phần tử của tuple bằng cách sử dụng dấu chấm (`.`) theo sau là chỉ số (index) của giá trị chúng ta muốn truy cập. Ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Chương trình này tạo tuple `x` và sau đó truy cập từng phần tử của tuple bằng cách sử dụng các chỉ số tương ứng của chúng. Giống như hầu hết các ngôn ngữ lập trình, chỉ số đầu tiên trong một tuple là 0.

Tuple không chứa bất kỳ giá trị nào có một tên gọi đặc biệt là _unit_. Giá trị này và kiểu tương ứng của nó đều được viết là `()` và đại diện cho một giá trị rỗng hoặc một kiểu trả về rỗng. Các biểu thức trả về giá trị unit một cách ngầm định nếu chúng không trả về bất kỳ giá trị nào khác.

Ngoài ra, chúng ta có thể sửa đổi các phần tử riêng lẻ của một tuple khả biến (`mut`). Ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
    let mut x: (i32, i32) = (1, 2);
    x.0 = 0;
    x.1 += 5;
}
```

Chương trình này đặt phần tử đầu tiên thành 0 và cộng 5 vào phần tử thứ hai. Giá trị cuối cùng của `x` là `(0, 7)`.

#### Kiểu Mảng (Array Type)

Một cách khác để có một tập hợp nhiều giá trị là sử dụng một _mảng_ (array). Không giống như tuple, mọi phần tử của một mảng **bắt buộc phải có cùng một kiểu dữ liệu**. Không giống như mảng trong một số ngôn ngữ khác, mảng trong Rust có **độ dài cố định**.

Chúng ta viết các giá trị trong một mảng dưới dạng danh sách cách nhau bằng dấu phẩy bên trong dấu ngoặc vuông:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Mảng rất hữu ích khi bạn muốn dữ liệu của mình được cấp phát trên stack giống như các kiểu dữ liệu khác mà chúng ta đã thấy cho đến nay, thay vì trên heap (chúng ta sẽ thảo luận kỹ hơn về stack và heap trong [Chương 4][stack-and-heap]<!-- ignore -->) hoặc khi bạn muốn đảm bảo rằng bạn luôn có một số lượng phần tử cố định. Tuy nhiên, mảng không linh hoạt bằng kiểu vector. Một _vector_ là một kiểu tập hợp tương tự do thư viện chuẩn cung cấp, _cho phép_ tăng hoặc giảm kích thước vì nội dung của nó nằm trên heap. Nếu bạn không chắc chắn nên sử dụng mảng hay vector, rất có thể bạn nên sử dụng vector. [Chương 8][vectors]<!-- ignore --> sẽ thảo luận chi tiết hơn về vector.

Tuy nhiên, mảng hữu ích hơn khi bạn biết chắc chắn số lượng phần tử sẽ không bao giờ cần thay đổi. Ví dụ: nếu bạn đang sử dụng tên các tháng trong một chương trình, bạn có thể sẽ sử dụng một mảng thay vì một vector vì bạn biết nó sẽ luôn chứa đúng 12 phần tử:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Bạn viết kiểu của một mảng bằng cách sử dụng dấu ngoặc vuông chứa kiểu của từng phần tử, một dấu chấm phẩy, và sau đó là số lượng phần tử trong mảng, như sau:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Ở đây, `i32` là kiểu của mỗi phần tử. Sau dấu chấm phẩy, số `5` cho biết mảng chứa năm phần tử.

Bạn cũng có thể khởi tạo một mảng chứa cùng một giá trị cho mỗi phần tử bằng cách chỉ định giá trị ban đầu, theo sau là dấu chấm phẩy, và sau đó là độ dài của mảng trong dấu ngoặc vuông:

```rust
let a = [3; 5];
```

Mảng có tên là `a` sẽ chứa `5` phần tử và tất cả ban đầu sẽ được đặt thành giá trị `3`. Cách viết này tương đương với `let a = [3, 3, 3, 3, 3];` nhưng ngắn gọn hơn nhiều.

##### Truy Cập Các Phần Tử Mảng

Một mảng là một khối bộ nhớ đơn lẻ có kích thước cố định và đã biết trước, có thể được cấp phát trên stack. Bạn có thể truy cập các phần tử của một mảng bằng cách đánh chỉ mục (indexing), như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Trong ví dụ này, biến có tên `first` sẽ nhận giá trị `1` vì đó là giá trị tại chỉ số `[0]` trong mảng. Biến có tên `second` sẽ nhận giá trị `2` từ chỉ số `[1]` trong mảng.

##### Truy Cập Phần Tử Mảng Không Hợp Lệ

Hãy xem điều gì xảy ra nếu bạn cố gắng truy cập một phần tử của mảng vượt quá phần cuối của mảng. Giả sử bạn chạy đoạn mã này để lấy chỉ số mảng từ người dùng:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Đoạn mã này biên dịch thành công. Nếu bạn chạy đoạn mã này bằng `cargo run` và nhập `0`, `1`, `2`, `3`, hoặc `4`, chương trình sẽ in ra giá trị tương ứng tại chỉ số đó trong mảng. Nếu thay vào đó bạn nhập một số vượt quá giới hạn của mảng, chẳng hạn như `10`, bạn sẽ thấy kết quả như sau:

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Chương trình đã tạo ra một lỗi _thời gian chạy_ (runtime error) tại thời điểm sử dụng một giá trị không hợp lệ trong thao tác đánh chỉ mục. Chương trình đã thoát với một thông báo lỗi và không thực thi câu lệnh `println!` cuối cùng. Khi bạn cố gắng truy cập một phần tử bằng chỉ mục, Rust sẽ kiểm tra xem chỉ mục bạn đã chỉ định có nhỏ hơn độ dài của mảng hay không. Nếu chỉ mục lớn hơn hoặc bằng độ dài, Rust sẽ panic. Việc kiểm tra này bắt buộc phải diễn ra tại thời gian chạy, đặc biệt là trong trường hợp này, vì trình biên dịch không thể biết trước người dùng sẽ nhập giá trị nào khi chạy mã.

Đây là một ví dụ về các nguyên tắc an toàn bộ nhớ của Rust trong thực tế. Trong nhiều ngôn ngữ cấp thấp khác, việc kiểm tra này không được thực hiện, và khi bạn cung cấp một chỉ mục không chính xác, vùng nhớ không hợp lệ có thể bị truy cập (*Buffer Overflow / Out-of-bounds Read*). Rust bảo vệ bạn khỏi loại lỗi này bằng cách thoát ngay lập tức thay vì cho phép truy cập bộ nhớ trái phép và tiếp tục thực thi. Chương 9 sẽ thảo luận nhiều hơn về cách xử lý lỗi trong Rust.

{{#quiz ../quizzes/ch03-02-data-types-sec2-compound.toml}}

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[wrapping]: https://doc.rust-lang.org/std/num/struct.Wrapping.html
[appendix_b]: appendix-02-operators.md
