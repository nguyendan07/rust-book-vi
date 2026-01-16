## Kiểu Slice

_Slice_ cho phép bạn tham chiếu tới một chuỗi liên tiếp các phần tử trong một [tập hợp](ch08-00-common-collections.md) thay vì toàn bộ tập hợp đó. Một slice là một loại tham chiếu, vì vậy nó là một con trỏ không có quyền sở hữu (non-owning pointer).

Để tạo động lực tại sao slice lại hữu ích, hãy cùng giải quyết một bài toán lập trình nhỏ: viết một hàm nhận vào một chuỗi các từ được phân tách bằng dấu cách và trả về từ đầu tiên mà nó tìm thấy trong chuỗi đó. Nếu hàm không tìm thấy dấu cách nào trong chuỗi, thì cả chuỗi đó phải là một từ, vì vậy toàn bộ chuỗi nên được trả về. Nếu không có slice, chúng ta có thể viết chữ ký của hàm như thế này:

```rust,ignore
fn first_word(s: &String) -> ?
```

Hàm `first_word` có tham số là một `&String`. Chúng ta không muốn quyền sở hữu chuỗi, vì vậy điều này là ổn. Nhưng chúng ta nên trả về cái gì? Chúng ta thực sự không có cách nào để nói về _một phần_ của chuỗi. Tuy nhiên, chúng ta có thể trả về chỉ mục (index) của phần cuối từ, được biểu thị bằng dấu cách. Hãy thử làm điều đó, như được hiển thị trong Liệt kê 4-7.

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

<span class="caption">Liệt kê 4-7: Hàm `first_word` trả về một giá trị chỉ mục byte vào trong tham số `String`</span>

Bởi vì chúng ta cần đi qua `String` từng phần tử một và kiểm tra xem một giá trị có phải là dấu cách không, chúng ta sẽ chuyển đổi `String` của mình thành một mảng các byte bằng cách sử dụng phương thức `as_bytes`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Tiếp theo, chúng ta tạo một iterator (trình lặp) trên mảng các byte bằng cách sử dụng phương thức `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Chúng tôi sẽ thảo luận chi tiết hơn về các iterator trong [Chương 13][ch13]<!-- ignore -->. Hiện tại, hãy biết rằng `iter` là một phương thức trả về từng phần tử trong một tập hợp và `enumerate` bao bọc kết quả của `iter` và trả về mỗi phần tử như một phần của một tuple (bộ). Phần tử đầu tiên của tuple được trả về từ `enumerate` là chỉ mục, và phần tử thứ hai là một tham chiếu tới phần tử đó. Điều này tiện lợi hơn một chút so với việc tự tính toán chỉ mục.

Bởi vì phương thức `enumerate` trả về một tuple, chúng ta có thể sử dụng các pattern (mẫu) để giải cấu trúc tuple đó. Chúng tôi sẽ thảo luận về các pattern nhiều hơn trong [Chương 6][ch6]<!-- ignore -->. Trong vòng lặp `for`, chúng ta chỉ định một pattern có `i` cho chỉ mục trong tuple và `&item` cho byte đơn lẻ trong tuple. Bởi vì chúng ta nhận được một tham chiếu tới phần tử từ `.iter().enumerate()`, chúng ta sử dụng `&` trong pattern.

Bên trong vòng lặp `for`, chúng ta tìm kiếm byte đại diện cho dấu cách bằng cách sử dụng cú pháp ký tự byte (byte literal). Nếu chúng ta tìm thấy một dấu cách, chúng ta trả về vị trí đó. Nếu không, chúng ta trả về độ dài của chuỗi bằng cách sử dụng `s.len()`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Bây giờ chúng ta có một cách để tìm ra chỉ mục kết thúc của từ đầu tiên trong chuỗi, nhưng có một vấn đề. Chúng ta đang trả về một `usize` đứng một mình, nhưng nó chỉ là một con số có ý nghĩa trong ngữ cảnh của `&String`. Nói cách khác, bởi vì nó là một giá trị tách biệt khỏi `String`, không có gì đảm bảo rằng nó vẫn sẽ hợp lệ trong tương lai. Hãy xem xét chương trình trong Liệt kê 4-8 sử dụng hàm `first_word` từ Liệt kê 4-7.

<span class="filename">Tên tệp: src/main.rs</span>

```aquascope,interpreter+permissions,boundaries,stepper,horizontal
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {
    let mut s = String::from("hello world");`(focus)`
    let word = first_word(&s);`[]`
    s.clear();`[]``{}`
}
```

<span class="caption">Liệt kê 4-8: Lưu trữ kết quả từ việc gọi hàm `first_word` và sau đó thay đổi nội dung của `String`</span>

Chương trình này biên dịch mà không có lỗi nào, vì `s` vẫn giữ quyền ghi sau khi gọi `first_word`. Bởi vì `word` hoàn toàn không kết nối với trạng thái của `s`, `word` vẫn chứa giá trị `5`. Chúng ta có thể sử dụng giá trị `5` đó với biến `s` để cố gắng trích xuất từ đầu tiên ra, nhưng điều này sẽ là một lỗi (bug) vì nội dung của `s` đã thay đổi kể từ khi chúng ta lưu `5` trong `word`.

Việc phải lo lắng về chỉ mục trong `word` bị mất đồng bộ với dữ liệu trong `s` thật tẻ nhạt và dễ gây lỗi! Việc quản lý các chỉ mục này thậm chí còn mong manh hơn nếu chúng ta viết một hàm `second_word`. Chữ ký của nó sẽ phải trông như thế này:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Bây giờ chúng ta đang theo dõi một chỉ mục bắt đầu _và_ một chỉ mục kết thúc, và chúng ta có nhiều giá trị hơn nữa được tính toán từ dữ liệu ở một trạng thái cụ thể nhưng hoàn toàn không gắn liền với trạng thái đó. Chúng ta có ba biến không liên quan trôi nổi xung quanh cần được giữ đồng bộ.

May mắn thay, Rust có một giải pháp cho vấn đề này: string slice (slice chuỗi).

### String Slice

Một _string slice_ là một tham chiếu tới một phần của `String`, và nó trông như thế này:

```aquascope,interpreter
#fn main() {
let s = String::from("hello world");

let hello: &str = &s[0..5];
let world: &str = &s[6..11];
let s2: &String = &s; `[]`
#}
```

Thay vì là một tham chiếu tới toàn bộ `String` (như `s2`), `hello` là một tham chiếu tới một phần của `String`, được chỉ định trong phần `[0..5]` thêm vào. Chúng ta tạo các slice bằng cách sử dụng một phạm vi (range) trong dấu ngoặc vuông bằng cách chỉ định `[starting_index..ending_index]`, trong đó `starting_index` là vị trí đầu tiên trong slice và `ending_index` lớn hơn vị trí cuối cùng trong slice một đơn vị.

Các slice là loại tham chiếu đặc biệt vì chúng là các con trỏ "béo" (fat pointers), hay các con trỏ đi kèm siêu dữ liệu (metadata). Ở đây, siêu dữ liệu là độ dài của slice. Chúng ta có thể thấy siêu dữ liệu này bằng cách thay đổi hình ảnh trực quan hóa để nhìn vào bên trong cấu trúc dữ liệu của Rust:

```aquascope,interpreter,concreteTypes,hideCode
fn main() {
    let s = String::from("hello world");

    let hello: &str = &s[0..5];
    let world: &str = &s[6..11];
    let s2: &String = &s; // not a slice, for comparison
    `[]`
}
```

Hãy quan sát rằng các biến `hello` và `world` đều có cả trường `ptr` và `len`, chúng cùng nhau xác định các vùng được gạch dưới của chuỗi trên heap. Bạn cũng có thể thấy ở đây một `String` thực sự trông như thế nào: một chuỗi là một vector các byte (`Vec<u8>`), chứa một độ dài `len` và một bộ đệm `buf` có một con trỏ `ptr` và một dung lượng `cap`.

Bởi vì các slice là các tham chiếu, chúng cũng thay đổi quyền hạn trên dữ liệu được tham chiếu. Ví dụ, hãy quan sát bên dưới rằng khi `hello` được tạo ra như một slice của `s`, thì `s` mất quyền ghi và quyền sở hữu:

```aquascope,permissions,stepper,boundaries
fn main() {
    let mut s = String::from("hello");
    let hello: &str = &s[0..5];
    println!("{hello}");
    s.push_str(" world");
}
```

#### Cú pháp Range

Với cú pháp range `..` của Rust, nếu bạn muốn bắt đầu ở chỉ mục 0, bạn có thể bỏ giá trị trước hai dấu chấm. Nói cách khác, những dòng này là tương đương:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Tương tự, nếu slice của bạn bao gồm byte cuối cùng của `String`, bạn có thể bỏ con số phía sau. Điều đó có nghĩa là những dòng này là tương đương:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Bạn cũng có thể bỏ cả hai giá trị để lấy một slice của toàn bộ chuỗi. Vì vậy, những dòng này là tương đương:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Lưu ý: Các chỉ mục phạm vi của string slice phải xảy ra tại các ranh giới ký tự UTF-8 hợp lệ. Nếu bạn cố gắng tạo một string slice ở giữa một ký tự nhiều byte, chương trình của bạn sẽ thoát với một lỗi. Vì mục đích giới thiệu string slice, chúng ta giả định chỉ dùng ASCII trong phần này; một cuộc thảo luận kỹ lưỡng hơn về xử lý UTF-8 nằm trong phần [“Lưu trữ văn bản được mã hóa UTF-8 với Strings”][strings]<!-- ignore --> của Chương 8.

#### Viết lại `first_word` với string slice

Với tất cả thông tin này trong đầu, hãy viết lại `first_word` để trả về một slice. Kiểu biểu thị “string slice” được viết là `&str`:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

Chúng ta lấy chỉ mục cho phần cuối của từ theo cách tương tự như chúng ta đã làm trong Liệt kê 4-7, bằng cách tìm kiếm lần xuất hiện đầu tiên của dấu cách. Khi chúng ta tìm thấy một dấu cách, chúng ta trả về một string slice sử dụng phần đầu của chuỗi và chỉ mục của dấu cách làm các chỉ mục bắt đầu và kết thúc.

Bây giờ khi chúng ta gọi `first_word`, chúng ta nhận lại một giá trị đơn lẻ được gắn liền với dữ liệu bên dưới. Giá trị này được tạo thành từ một tham chiếu đến điểm bắt đầu của slice và số lượng phần tử trong slice.

Việc trả về một slice cũng sẽ hoạt động cho một hàm `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Bây giờ chúng ta có một API đơn giản khó làm hỏng hơn nhiều, bởi vì trình biên dịch sẽ đảm bảo các tham chiếu vào `String` vẫn hợp lệ. Hãy nhớ lại lỗi trong chương trình ở Liệt kê 4-8, khi chúng ta lấy chỉ mục đến cuối từ đầu tiên nhưng sau đó xóa chuỗi đi nên chỉ mục của chúng ta không còn hợp lệ? Mã đó không chính xác về mặt logic nhưng không hiển thị bất kỳ lỗi ngay lập tức nào. Các vấn đề sẽ xuất hiện sau đó nếu chúng ta tiếp tục cố gắng sử dụng chỉ mục từ đầu tiên với một chuỗi đã bị làm trống. Các slice làm cho lỗi này không thể xảy ra và cho chúng ta biết chúng ta có vấn đề với mã của mình sớm hơn nhiều. Ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```aquascope,permissions,boundaries,stepper,shouldFail
#fn first_word(s: &String) -> &str {
#    let bytes = s.as_bytes();
#
#    for (i, &item) in bytes.iter().enumerate() {
#        if item == b' ' {
#            return &s[0..i];
#        }
#    }
#
#    &s[..]
#}
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);`(focus,paths:s)`
    s.clear();`{}`
    println!("the first word is: {}", word);
}
```

Bạn có thể thấy rằng việc gọi `first_word` bây giờ loại bỏ quyền ghi khỏi `s`, điều này ngăn chúng ta gọi `s.clear()`. Đây là lỗi trình biên dịch:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Hãy nhớ lại các quy tắc vay mượn (borrowing rules) rằng nếu chúng ta có một tham chiếu bất biến tới một cái gì đó, chúng ta không thể đồng thời lấy một tham chiếu khả biến. Bởi vì `clear` cần cắt bớt `String`, nó cần lấy một tham chiếu khả biến. Lệnh `println!` sau lời gọi tới `clear` sử dụng tham chiếu trong `word`, vì vậy tham chiếu bất biến vẫn phải đang hoạt động tại thời điểm đó. Rust không cho phép tham chiếu khả biến trong `clear` và tham chiếu bất biến trong `word` tồn tại cùng một lúc, và việc biên dịch thất bại. Rust không chỉ làm cho API của chúng ta dễ sử dụng hơn, mà nó còn loại bỏ toàn bộ một lớp lỗi tại thời gian biên dịch!

#### String Literal Là Các Slice

Hãy nhớ lại rằng chúng ta đã nói về các string literal được lưu trữ bên trong tệp nhị phân. Bây giờ khi chúng ta đã biết về slice, chúng ta có thể hiểu đúng về string literal:

```rust
let s = "Hello, world!";
```

Kiểu của `s` ở đây là `&str`: nó là một slice trỏ tới điểm cụ thể đó của tệp nhị phân. Đây cũng là lý do tại sao các string literal là bất biến; `&str` là một tham chiếu bất biến.

#### String Slice Dưới Dạng Tham Số

Việc biết rằng bạn có thể lấy slice của các literal và các giá trị `String` dẫn chúng ta đến một cải tiến nữa trên `first_word`, và đó là chữ ký của nó:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Một Rustacean có kinh nghiệm hơn sẽ viết chữ ký như được hiển thị trong Liệt kê 4-9 thay vào đó vì nó cho phép chúng ta sử dụng cùng một hàm trên cả giá trị `&String` và giá trị `&str`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

<span class="caption">Liệt kê 4-9: Cải thiện hàm `first_word` bằng cách sử dụng một string slice cho kiểu của tham số `s`</span>

Nếu chúng ta có một string slice, chúng ta có thể truyền nó trực tiếp. Nếu chúng ta có một `String`, chúng ta có thể truyền một slice của `String` hoặc một tham chiếu tới `String`. Sự linh hoạt này tận dụng _deref coercions_ (ép kiểu deref), một tính năng chúng tôi sẽ đề cập trong phần [“Deref Coercion Ngầm Định Với Hàm và Phương Thức”][deref-coercions]<!--ignore--> của Chương 15.

Việc định nghĩa một hàm nhận vào một string slice thay vì một tham chiếu tới `String` làm cho API của chúng ta tổng quát và hữu ích hơn mà không làm mất đi bất kỳ chức năng nào:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

### Các Slice Khác

String slice, như bạn có thể hình dung, là dành riêng cho các chuỗi. Nhưng cũng có một kiểu slice tổng quát hơn. Hãy xem xét mảng này:

```rust
let a = [1, 2, 3, 4, 5];
```

Cũng giống như chúng ta có thể muốn tham chiếu tới một phần của một chuỗi, chúng ta có thể muốn tham chiếu tới một phần của một mảng. Chúng ta sẽ làm như thế này:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Slice này có kiểu `&[i32]`. Nó hoạt động giống như cách string slice hoạt động, bằng cách lưu trữ một tham chiếu tới phần tử đầu tiên và một độ dài. Bạn sẽ sử dụng loại slice này cho tất cả các loại tập hợp khác. Chúng tôi sẽ thảo luận chi tiết về các tập hợp này khi chúng ta nói về vector trong Chương 8.

{{#quiz ../quizzes/ch04-04-slices.toml}}

## Tóm Tắt

Các slice là một loại tham chiếu đặc biệt tham chiếu đến các phạm vi con của một chuỗi, như một chuỗi (string) hoặc một vector. Tại thời gian chạy (runtime), một slice được biểu diễn dưới dạng một "con trỏ béo" (fat pointer) chứa một con trỏ tới phần đầu của phạm vi và một độ dài của phạm vi đó. Một lợi thế của slice so với các phạm vi dựa trên chỉ mục là slice không thể bị vô hiệu hóa trong khi nó đang được sử dụng.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods
