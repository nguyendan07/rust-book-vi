## The Slice Type

_Slice_ cho phép bạn tham chiếu đến một chuỗi các phần tử liền kề trong một
[collection](ch08-00-common-collections.md)<!-- ignore -->. Slice là một dạng
tham chiếu, vì vậy nó không có quyền sở hữu.

Đây là một bài toán lập trình nhỏ: viết một hàm nhận vào một chuỗi các từ được phân tách bằng dấu cách và trả về từ đầu tiên mà nó tìm thấy trong chuỗi đó.
Nếu hàm không tìm thấy dấu cách trong chuỗi, toàn bộ chuỗi được coi là một từ, vì vậy toàn bộ chuỗi sẽ được trả về.

> Lưu ý: Để giới thiệu về string slice, chúng ta giả định chỉ sử dụng ASCII
> trong phần này; một cuộc thảo luận kỹ lưỡng hơn về việc xử lý UTF-8 sẽ có trong
> phần [“Lưu trữ văn bản được mã hóa UTF-8 với String”][strings]<!-- ignore -->
> của Chương 8.

Hãy cùng xem cách chúng ta sẽ viết định nghĩa của hàm này mà không sử dụng
slice, để hiểu vấn đề mà slice sẽ giải quyết:

```rust,ignore
fn first_word(s: &String) -> ?
```

Hàm `first_word` có một tham số kiểu `&String`. Chúng ta không cần quyền sở hữu,
nên điều này ổn. (Trong Rust, theo cách viết thông thường, các hàm không nhận quyền sở hữu
đối số trừ khi cần thiết, và lý do cho điều đó sẽ trở nên rõ ràng hơn khi chúng ta tiếp tục.)
Nhưng chúng ta nên trả về cái gì? Chúng ta không thực sự có cách nào để nói về *một phần* của một chuỗi.
Tuy nhiên, chúng ta có thể trả về chỉ số của cuối từ, được biểu thị bằng một dấu cách.
Hãy thử cách đó, như trong Listing 4-7.

<Listing number="4-7" file-name="src/main.rs" caption="Hàm `first_word` trả về một giá trị chỉ số byte vào tham số `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Bởi vì chúng ta cần duyệt qua `String` từng phần tử một và kiểm tra xem một
giá trị có phải là dấu cách hay không, chúng ta sẽ chuyển đổi `String` của mình thành một mảng byte bằng
phương thức `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Tiếp theo, chúng ta tạo một iterator trên mảng byte bằng phương thức `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Chúng ta sẽ thảo luận chi tiết hơn về iterator trong [Chương 13][ch13]<!-- ignore -->.
Hiện tại, chỉ cần biết rằng `iter` là một phương thức trả về từng phần tử trong một collection
và `enumerate` bao bọc kết quả của `iter` và trả về mỗi phần tử như một phần của một tuple.
Phần tử đầu tiên của tuple được trả về từ `enumerate` là chỉ số, và phần tử thứ hai là một tham chiếu đến phần tử đó.
Điều này tiện lợi hơn một chút so với việc tự tính toán chỉ số.

Bởi vì phương thức `enumerate` trả về một tuple, chúng ta có thể sử dụng pattern để
phân rã tuple đó. Chúng ta sẽ thảo luận nhiều hơn về pattern trong [Chương
6][ch6]<!-- ignore -->. Trong vòng lặp `for`, chúng ta chỉ định một pattern có `i`
cho chỉ số trong tuple và `&item` cho byte đơn trong tuple.
Bởi vì chúng ta nhận được một tham chiếu đến phần tử từ `.iter().enumerate()`, chúng ta sử dụng
`&` trong pattern.

Bên trong vòng lặp `for`, chúng ta tìm kiếm byte đại diện cho dấu cách bằng
cách sử dụng cú pháp byte literal. Nếu tìm thấy một dấu cách, chúng ta trả về vị trí.
Nếu không, chúng ta trả về độ dài của chuỗi bằng cách sử dụng `s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Bây giờ chúng ta đã có cách để tìm ra chỉ số cuối của từ đầu tiên trong
chuỗi, nhưng có một vấn đề. Chúng ta đang trả về một `usize` riêng lẻ, nhưng nó
chỉ là một con số có ý nghĩa trong ngữ cảnh của `&String`. Nói cách khác,
bởi vì nó là một giá trị tách biệt với `String`, không có gì đảm bảo rằng nó
vẫn sẽ hợp lệ trong tương lai. Hãy xem xét chương trình trong Listing 4-8 sử dụng
hàm `first_word` từ Listing 4-7.

<Listing number="4-8" file-name="src/main.rs" caption="Lưu trữ kết quả từ việc gọi hàm `first_word` và sau đó thay đổi nội dung `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Chương trình này biên dịch mà không có lỗi nào và cũng sẽ như vậy nếu chúng ta sử dụng `word`
sau khi gọi `s.clear()`. Bởi vì `word` không hề được kết nối với trạng thái của `s`,
`word` vẫn chứa giá trị `5`. Chúng ta có thể sử dụng giá trị `5` đó với
biến `s` để cố gắng trích xuất từ đầu tiên, nhưng đây sẽ là một lỗi
vì nội dung của `s` đã thay đổi kể từ khi chúng ta lưu `5` vào `word`.

Việc phải lo lắng về chỉ số trong `word` bị lệch pha với dữ liệu trong
`s` là tẻ nhạt và dễ gây lỗi! Việc quản lý các chỉ số này thậm chí còn mong manh hơn
nếu chúng ta viết một hàm `second_word`. Định nghĩa của nó sẽ phải trông như thế này:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Bây giờ chúng ta đang theo dõi cả chỉ số bắt đầu _và_ kết thúc, và chúng ta có nhiều
giá trị hơn được tính toán từ dữ liệu ở một trạng thái cụ thể nhưng không hề gắn liền với
trạng thái đó. Chúng ta có ba biến không liên quan trôi nổi cần được giữ đồng bộ.

May mắn thay, Rust có một giải pháp cho vấn đề này: string slice.

### String Slice

Một _string slice_ là một tham chiếu đến một chuỗi các phần tử liền kề của một
`String`, và nó trông như thế này:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Thay vì một tham chiếu đến toàn bộ `String`, `hello` là một tham chiếu đến một
phần của `String`, được chỉ định trong phần `[0..5]` thêm vào. Chúng ta tạo slice
sử dụng một khoảng trong ngoặc vuông bằng cách chỉ định `[starting_index..ending_index]`,
trong đó _`starting_index`_ là vị trí đầu tiên trong slice và _`ending_index`_
là vị trí cuối cùng trong slice cộng một. Bên trong, cấu trúc dữ liệu slice
lưu trữ vị trí bắt đầu và độ dài của slice, tương ứng với
_`ending_index`_ trừ _`starting_index`_. Vì vậy, trong trường hợp của `let
world = &s[6..11];`, `world` sẽ là một slice chứa một con trỏ đến
byte tại chỉ số 6 của `s` với giá trị độ dài là `5`.

Hình 4-7 minh họa điều này trong một sơ đồ.

<img alt="Ba bảng: một bảng đại diện cho dữ liệu stack của s, trỏ đến
byte tại chỉ số 0 trong một bảng dữ liệu chuỗi &quot;hello world&quot; trên heap.
Bảng thứ ba đại diện cho dữ liệu stack của slice world, có giá trị độ dài là 5
và trỏ đến byte 6 của bảng dữ liệu heap."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-7: String slice tham chiếu đến một phần của một
`String`</span>

Với cú pháp khoảng `..` của Rust, nếu bạn muốn bắt đầu từ chỉ số 0, bạn có thể bỏ
giá trị trước hai dấu chấm. Nói cách khác, những cách sau là tương đương:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Tương tự, nếu slice của bạn bao gồm byte cuối cùng của `String`, bạn
có thể bỏ số ở cuối. Điều đó có nghĩa là những cách sau là tương đương:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Bạn cũng có thể bỏ cả hai giá trị để lấy một slice của toàn bộ chuỗi. Vì vậy, những cách sau là tương đương:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Lưu ý: Các chỉ số khoảng của string slice phải nằm ở các ranh giới ký tự UTF-8 hợp lệ.
> Nếu bạn cố gắng tạo một string slice ở giữa một ký tự đa byte,
> chương trình của bạn sẽ thoát với một lỗi.

Với tất cả thông tin này, hãy viết lại `first_word` để trả về một
slice. Kiểu biểu thị “string slice” được viết là `&str`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Chúng ta lấy chỉ số cho cuối từ theo cách tương tự như trong Listing 4-7, bằng
cách tìm lần xuất hiện đầu tiên của một dấu cách. Khi tìm thấy một dấu cách, chúng ta trả về một
string slice sử dụng điểm bắt đầu của chuỗi và chỉ số của dấu cách làm
chỉ số bắt đầu và kết thúc.

Bây giờ khi chúng ta gọi `first_word`, chúng ta nhận lại một giá trị duy nhất được gắn với
dữ liệu cơ bản. Giá trị này bao gồm một tham chiếu đến điểm bắt đầu của
slice và số lượng phần tử trong slice.

Việc trả về một slice cũng sẽ hoạt động cho một hàm `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Bây giờ chúng ta có một API đơn giản, khó gây lỗi hơn nhiều vì
trình biên dịch sẽ đảm bảo các tham chiếu vào `String` vẫn hợp lệ. Hãy nhớ lại
lỗi trong chương trình ở Listing 4-8, khi chúng ta lấy chỉ số đến cuối
từ đầu tiên nhưng sau đó xóa chuỗi đi khiến chỉ số của chúng ta không hợp lệ? Đoạn mã đó
về mặt logic là không chính xác nhưng không hiển thị bất kỳ lỗi tức thời nào. Các vấn đề sẽ
xuất hiện sau này nếu chúng ta tiếp tục cố gắng sử dụng chỉ số từ đầu tiên với một chuỗi đã
được làm trống. Slice làm cho lỗi này không thể xảy ra và cho chúng ta biết chúng ta có vấn đề với
mã của mình sớm hơn nhiều. Sử dụng phiên bản slice của `first_word` sẽ gây ra lỗi
biên dịch:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

Đây là lỗi của trình biên dịch:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Hãy nhớ lại từ các quy tắc vay mượn rằng nếu chúng ta có một tham chiếu bất biến đến
một thứ gì đó, chúng ta không thể đồng thời lấy một tham chiếu khả biến. Bởi vì `clear` cần
cắt ngắn `String`, nó cần lấy một tham chiếu khả biến. Lệnh `println!`
sau lệnh gọi `clear` sử dụng tham chiếu trong `word`, vì vậy tham chiếu bất biến
phải vẫn còn hoạt động tại thời điểm đó. Rust không cho phép tham chiếu khả biến
trong `clear` và tham chiếu bất biến trong `word` tồn tại cùng một lúc,
và quá trình biên dịch thất bại. Rust không chỉ làm cho API của chúng ta dễ sử dụng hơn,
mà còn loại bỏ cả một lớp lỗi tại thời điểm biên dịch!

<!-- Old heading. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### String Literal là Slice

Hãy nhớ lại rằng chúng ta đã nói về các string literal được lưu trữ bên trong tệp nhị phân. Bây giờ
chúng ta đã biết về slice, chúng ta có thể hiểu đúng về string literal:

```rust
let s = "Hello, world!";
```

Kiểu của `s` ở đây là `&str`: nó là một slice trỏ đến điểm cụ thể đó của
tệp nhị phân. Đây cũng là lý do tại sao các string literal là bất biến; `&str` là một
tham chiếu bất biến.

#### String Slice làm Tham số

Biết rằng bạn có thể lấy slice của literal và giá trị `String` dẫn chúng ta đến
một cải tiến nữa cho `first_word`, và đó là định nghĩa của nó:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Một Rustacean có kinh nghiệm hơn sẽ viết định nghĩa như trong Listing 4-9
thay vào đó vì nó cho phép chúng ta sử dụng cùng một hàm trên cả giá trị `&String`
và `&str`.

<Listing number="4-9" caption="Cải thiện hàm `first_word` bằng cách sử dụng một string slice cho kiểu của tham số `s`">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Nếu chúng ta có một string slice, chúng ta có thể truyền nó trực tiếp. Nếu chúng ta có một `String`, chúng ta
có thể truyền một slice của `String` hoặc một tham chiếu đến `String`. Sự
linh hoạt này tận dụng _deref coercions_, một tính năng chúng ta sẽ đề cập trong
phần [“Implicit Deref Coercions with Functions and
Methods”][deref-coercions]<!--ignore--> của Chương 15.

Việc định nghĩa một hàm nhận một string slice thay vì một tham chiếu đến một `String`
làm cho API của chúng ta tổng quát và hữu ích hơn mà không làm mất đi bất kỳ chức năng nào:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Các Slice Khác

String slice, như bạn có thể tưởng tượng, là đặc trưng cho chuỗi. Nhưng cũng có một
kiểu slice tổng quát hơn. Hãy xem xét mảng này:

```rust
let a = [1, 2, 3, 4, 5];
```

Giống như chúng ta có thể muốn tham chiếu đến một phần của một chuỗi, chúng ta có thể muốn tham chiếu đến
một phần của một mảng. Chúng ta sẽ làm điều đó như thế này:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Slice này có kiểu `&[i32]`. Nó hoạt động theo cách tương tự như string slice, bằng
cách lưu trữ một tham chiếu đến phần tử đầu tiên và một độ dài. Bạn sẽ sử dụng loại
slice này cho tất cả các loại collection khác. Chúng ta sẽ thảo luận chi tiết về các collection này
khi nói về vector trong Chương 8.

## Tóm tắt

Các khái niệm về quyền sở hữu, vay mượn và slice đảm bảo an toàn bộ nhớ trong các
chương trình Rust tại thời điểm biên dịch. Ngôn ngữ Rust cho phép bạn kiểm soát
việc sử dụng bộ nhớ của mình giống như các ngôn ngữ lập trình hệ thống khác, nhưng việc
chủ sở hữu dữ liệu tự động dọn dẹp dữ liệu đó khi chủ sở hữu ra khỏi phạm vi
có nghĩa là bạn không cần phải viết và gỡ lỗi thêm mã để có được sự kiểm soát này.

Quyền sở hữu ảnh hưởng đến cách hoạt động của nhiều phần khác của Rust, vì vậy chúng ta sẽ nói về
những khái niệm này nhiều hơn trong phần còn lại của cuốn sách. Hãy chuyển sang
Chương 5 và xem xét việc nhóm các mẩu dữ liệu lại với nhau trong một `struct`.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods
