## Lưu trữ Văn bản được Mã hóa UTF-8 với String

Chúng ta đã nói về chuỗi (string) trong Chương 4, nhưng bây giờ chúng ta sẽ tìm hiểu chúng sâu hơn.
Những người mới học Rust (New Rustaceans) thường gặp khó khăn với chuỗi vì sự kết hợp của ba
lý do: xu hướng của Rust trong việc để lộ các lỗi có thể xảy ra, chuỗi là một cấu trúc dữ liệu
phức tạp hơn nhiều so với những gì các lập trình viên thường nghĩ, và UTF-8. Những yếu tố này
kết hợp lại theo cách có vẻ khó khăn khi bạn chuyển từ các ngôn ngữ lập trình khác sang.

Chúng ta thảo luận về chuỗi trong ngữ cảnh của các bộ sưu tập (collections) vì chuỗi được
triển khai như một bộ sưu tập các byte, cộng với một số phương thức để cung cấp chức năng
hữu ích khi các byte đó được thông dịch dưới dạng văn bản. Trong phần này, chúng ta sẽ
nói về các thao tác trên `String` mà mọi kiểu bộ sưu tập đều có, chẳng hạn như
tạo, cập nhật và đọc. Chúng ta cũng sẽ thảo luận về những điểm mà `String`
khác với các bộ sưu tập khác, cụ thể là việc đánh chỉ số (indexing) vào một `String`
trở nên phức tạp như thế nào do sự khác biệt giữa cách con người và máy tính thông dịch
dữ liệu `String`.

### String là gì?

Trước tiên, chúng ta sẽ định nghĩa ý nghĩa của thuật ngữ _string_. Rust chỉ có một kiểu chuỗi
duy nhất trong ngôn ngữ cốt lõi, đó là lát cắt chuỗi (string slice) `str`, thường được thấy
dưới dạng mượn là `&str`. Trong Chương 4, chúng ta đã nói về _lát cắt chuỗi_,
là các tham chiếu đến một số dữ liệu chuỗi được mã hóa UTF-8 được lưu trữ ở nơi khác. Ví dụ,
các chuỗi văn bản (string literals) được lưu trữ trong tệp nhị phân của chương trình và do đó
là các lát cắt chuỗi.

Kiểu `String`, được cung cấp bởi thư viện tiêu chuẩn của Rust thay vì được lập trình sẵn
vào ngôn ngữ cốt lõi, là một kiểu chuỗi có thể mở rộng (growable), có thể thay đổi (mutable),
có quyền sở hữu (owned) và được mã hóa UTF-8. Khi những người dùng Rust nói đến “string” trong Rust, họ có thể
đang ám chỉ kiểu `String` hoặc kiểu lát cắt chuỗi `&str`, chứ không chỉ một trong
những kiểu đó. Mặc dù phần này chủ yếu nói về `String`, cả hai kiểu đều được sử dụng
nhiều trong thư viện tiêu chuẩn của Rust, và cả `String` lẫn lát cắt chuỗi đều được mã hóa UTF-8.

### Tạo một String Mới

Nhiều thao tác tương tự có sẵn với `Vec<T>` cũng có sẵn với `String`
vì thực tế `String` được triển khai như một lớp bao bọc (wrapper) quanh một vector các byte
với một số đảm bảo, hạn chế và khả năng bổ sung. Một ví dụ về một hàm hoạt động
tương tự với `Vec<T>` và `String` là hàm `new` để tạo một thể hiện, được hiển thị trong Listing 8-11.

<Listing number="8-11" caption="Tạo một `String` trống mới">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

Dòng này tạo ra một chuỗi trống mới tên là `s`, sau đó chúng ta có thể nạp dữ liệu
vào đó. Thông thường, chúng ta sẽ có một số dữ liệu ban đầu mà chúng ta muốn bắt đầu cho
chuỗi. Để làm điều đó, chúng ta sử dụng phương thức `to_string`, phương thức này có sẵn trên bất kỳ kiểu nào
triển khai trait `Display`, giống như các chuỗi văn bản. Listing 8-12 hiển thị
hai ví dụ.

<Listing number="8-12" caption="Sử dụng phương thức `to_string` để tạo một `String` từ một chuỗi văn bản">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

Đoạn mã này tạo ra một chuỗi chứa `initial contents`.

Chúng ta cũng có thể sử dụng hàm `String::from` để tạo một `String` từ một chuỗi
văn bản. Đoạn mã trong Listing 8-13 tương đương với đoạn mã trong Listing 8-12
có sử dụng `to_string`.

<Listing number="8-13" caption="Sử dụng hàm `String::from` để tạo một `String` từ một chuỗi văn bản">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

Bởi vì chuỗi được sử dụng cho rất nhiều thứ, chúng ta có thể sử dụng nhiều API generic khác nhau cho chuỗi, cung cấp cho chúng ta rất nhiều lựa chọn. Một số trong số đó có vẻ
dư thừa, nhưng tất cả chúng đều có vị trí của mình! Trong trường hợp này, `String::from` và
`to_string` thực hiện cùng một việc, vì vậy việc bạn chọn cái nào là vấn đề về phong cách và
khả năng dễ đọc.

Hãy nhớ rằng chuỗi được mã hóa UTF-8, vì vậy chúng ta có thể bao gồm bất kỳ dữ liệu nào được mã hóa
đúng cách vào chúng, như được hiển thị trong Listing 8-14.

<Listing number="8-14" caption="Lưu trữ các lời chào bằng các ngôn ngữ khác nhau trong các chuỗi">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

Tất cả những giá trị này đều là các giá trị `String` hợp lệ.

### Cập nhật một String

Một `String` có thể tăng kích thước và nội dung của nó có thể thay đổi, giống như nội dung
của một `Vec<T>`, nếu bạn đẩy thêm dữ liệu vào đó. Ngoài ra, bạn có thể sử dụng một cách tiện lợi
toán tử `+` hoặc macro `format!` để nối (concatenate) các giá trị `String`.

#### Thêm vào một String với `push_str` và `push`

Chúng ta có thể mở rộng một `String` bằng cách sử dụng phương thức `push_str` để thêm một lát cắt chuỗi,
như được hiển thị trong Listing 8-15.

<Listing number="8-15" caption="Thêm một lát cắt chuỗi vào một `String` bằng phương thức `push_str`平衡">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

Sau hai dòng này, `s` sẽ chứa `foobar`. Phương thức `push_str` nhận vào một
lát cắt chuỗi vì chúng ta không nhất thiết muốn lấy quyền sở hữu của
tham số đó. Ví dụ, trong đoạn mã ở Listing 8-16, chúng ta muốn có thể sử dụng
`s2` sau khi thêm nội dung của nó vào `s1`.

<Listing number="8-16" caption="Sử dụng một lát cắt chuỗi sau khi thêm nội dung của nó vào một `String`平衡">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

Nếu phương thức `push_str` lấy quyền sở hữu của `s2`, chúng ta sẽ không thể in
giá trị của nó ở dòng cuối cùng. Tuy nhiên, đoạn mã này hoạt động như chúng ta mong đợi!

Phương thức `push` nhận một ký tự duy nhất làm tham số và thêm nó vào
`String`. Listing 8-17 thêm chữ cái _l_ vào một `String` bằng phương thức `push`.

<Listing number="8-17" caption="Thêm một ký tự vào một giá trị `String` bằng cách sử dụng `push`平衡">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

Kết quả là, `s` sẽ chứa `lol`.

#### Nối chuỗi với Toán tử `+` hoặc Macro `format!`

Thông thường, bạn sẽ muốn kết hợp hai chuỗi hiện có. Một cách để làm điều đó là sử dụng
toán tử `+`, như được hiển thị trong Listing 8-18.

<Listing number="8-18" caption="Sử dụng toán tử `+` để kết hợp hai giá trị `String` thành một giá trị `String` mới">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

Chuỗi `s3` sẽ chứa `Hello, world!`. Lý do `s1` không còn
hợp lệ sau khi cộng, và lý do chúng ta sử dụng một tham chiếu đến `s2`, có liên quan
đến chữ ký (signature) của phương thức được gọi khi chúng ta sử dụng toán tử `+`.
Toán tử `+` sử dụng phương thức `add`, có chữ ký trông giống như
thế này:

```rust,ignore
fn add(self, s: &str) -> String {
```

Trong thư viện tiêu chuẩn, bạn sẽ thấy `add` được định nghĩa bằng cách sử dụng generic và các kiểu liên kết (associated types). Ở đây, chúng ta đã thay thế bằng các kiểu cụ thể, đó là những gì xảy ra khi chúng ta
gọi phương thức này với các giá trị `String`. Chúng ta sẽ thảo luận về generic trong Chương 10.
Chữ ký này cung cấp cho chúng ta những manh mối cần thiết để hiểu những phần lắt léo của toán tử `+`.

Đầu tiên, `s2` có một dấu `&`, nghĩa là chúng ta đang cộng một _tham chiếu_ của chuỗi thứ hai vào chuỗi thứ nhất. Điều này là do tham số `s` trong hàm `add`: chúng ta chỉ có thể cộng một `&str` vào một `String`; chúng ta không thể cộng hai giá trị `String`
với nhau. Nhưng chờ đã—kiểu của `&s2` là `&String`, chứ không phải `&str`, như
được chỉ định trong tham số thứ hai của `add`. Vậy tại sao Listing 8-18 lại biên dịch được?

Lý do chúng ta có thể sử dụng `&s2` trong lời gọi `add` là vì trình biên dịch
có thể _ép kiểu_ (coerce) đối số `&String` thành `&str`. Khi chúng ta gọi phương thức `add`, Rust sử dụng một _ép kiểu giải tham chiếu_ (deref coercion), ở đây nó chuyển `&s2` thành `&s2[..]`.
Chúng ta sẽ thảo luận về ép kiểu giải tham chiếu sâu hơn trong Chương 15. Bởi vì `add`
không lấy quyền sở hữu của tham số `s`, `s2` vẫn sẽ là một `String` hợp lệ
sau thao tác này.

<!-- BEGIN INTERVENTION: f1ab2171-96f0-4380-b16d-9055a9a00415 -->

Thứ hai, chúng ta có thể thấy trong chữ ký rằng `add` lấy quyền sở hữu của `self`,
bởi vì `self` _không_ có dấu `&`. Điều này có nghĩa là `s1` trong Listing 8-18 sẽ được
chuyển vào lời gọi `add` và sẽ không còn hợp lệ sau đó. Vì vậy, mặc dù
`let s3 = s1 + &s2;` trông giống như nó sẽ sao chép cả hai chuỗi và tạo ra một chuỗi mới,
câu lệnh này thay vào đó thực hiện các việc sau:

1. `add` lấy quyền sở hữu của `s1`,
2. nó thêm một bản sao nội dung của `s2` vào `s1`,
3. và sau đó nó trả lại quyền sở hữu của `s1`.

Nếu `s1` có đủ dung lượng cho `s2`, thì không có việc cấp phát bộ nhớ nào xảy ra. Tuy nhiên, nếu `s1` không có đủ dung lượng cho `s2`, thì `s1` sẽ thực hiện việc cấp phát một vùng bộ nhớ lớn hơn ở bên trong để chứa cả hai chuỗi.

<!-- END INTERVENTION -->

Nếu chúng ta cần nối nhiều chuỗi, hành vi của toán tử `+`
trở nên cồng kềnh:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Tại thời điểm này, `s` sẽ là `tic-tac-toe`. Với tất cả các ký tự `+` và `"`
thì thật khó để thấy những gì đang diễn ra. Để kết hợp chuỗi theo
những cách phức tạp hơn, chúng ta có thể sử dụng macro `format!` thay thế:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Đoạn mã này cũng đặt `s` thành `tic-tac-toe`. Macro `format!` hoạt động giống như
`println!`, nhưng thay vì in kết quả ra màn hình, nó trả về một
`String` với nội dung đó. Phiên bản của đoạn mã sử dụng `format!`
dễ đọc hơn nhiều, và mã được tạo bởi macro `format!` sử dụng các tham chiếu
để lời gọi này không lấy quyền sở hữu của bất kỳ tham số nào của nó.

{{#quiz ../quizzes/ch08-02-string-sec1.toml}}

### Đánh chỉ số vào String

Trong nhiều ngôn ngữ lập trình khác, việc truy cập các ký tự riêng lẻ trong một
chuỗi bằng cách tham chiếu chúng qua chỉ số (index) là một thao tác hợp lệ và phổ biến. Tuy nhiên,
nếu bạn cố gắng truy cập các phần của một `String` bằng cú pháp đánh chỉ số trong Rust, bạn sẽ
gặp lỗi. Hãy xem xét đoạn mã không hợp lệ trong Listing 8-19.

<Listing number="8-19" caption="Cố gắng sử dụng cú pháp đánh chỉ số với một String">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

Đoạn mã này sẽ dẫn đến lỗi sau:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

Lỗi và phần ghi chú đã giải thích lý do: chuỗi trong Rust không hỗ trợ đánh chỉ số. Nhưng
tại sao không? Để trả lời câu hỏi đó, chúng ta cần thảo luận về cách Rust lưu trữ chuỗi trong
bộ nhớ.

#### Biểu diễn Nội bộ

Một `String` là một lớp bao bọc bên ngoài một `Vec<u8>`. Hãy xem xét một số chuỗi ví dụ được mã hóa UTF-8 đúng cách của chúng ta từ Listing 8-14. Đầu tiên là chuỗi này:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Trong trường hợp này, `len` sẽ là `4`, nghĩa là vector lưu trữ chuỗi
`"Hola"` dài 4 byte. Mỗi chữ cái này chiếm một byte khi được mã hóa trong
UTF-8. Tuy nhiên, dòng sau đây có thể làm bạn ngạc nhiên (lưu ý rằng chuỗi này
bắt đầu bằng chữ cái Cyrillic viết hoa _Ze_, không phải là số 3):

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

Nếu bạn được hỏi chuỗi đó dài bao nhiêu, bạn có thể nói là 12. Thực tế, câu trả lời của Rust
là 24: đó là số byte cần thiết để mã hóa “Здравствуйте” trong
UTF-8, bởi vì mỗi giá trị vô hướng Unicode (Unicode scalar value) trong chuỗi đó chiếm 2 byte
dung lượng lưu trữ. Do đó, một chỉ số dẫn vào các byte của chuỗi sẽ không luôn luôn tương ứng
với một giá trị vô hướng Unicode hợp lệ. Để chứng minh, hãy xem xét đoạn mã Rust không hợp lệ này:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

Bạn đã biết rằng `answer` sẽ không phải là `З`, chữ cái đầu tiên. Khi được mã hóa
trong UTF-8, byte đầu tiên của `З` là `208` và byte thứ hai là `151`, vì vậy có vẻ
như `answer` thực tế phải là `208`, nhưng `208` tự thân nó không phải là một ký tự hợp lệ. Trả về `208` có lẽ không phải là điều người dùng mong muốn nếu họ yêu cầu
chữ cái đầu tiên của chuỗi này; tuy nhiên, đó là dữ liệu duy nhất mà Rust
có ở chỉ số byte 0. Người dùng thường không muốn giá trị byte được trả về, ngay cả
nếu chuỗi chỉ chứa các chữ cái Latinh: nếu `&"hi"[0]` là mã hợp lệ
trả về giá trị byte, nó sẽ trả về `104`, chứ không phải `h`.

Vì vậy, câu trả lời là để tránh trả về một giá trị không mong muốn và gây ra
các lỗi có thể không được phát hiện ngay lập tức, Rust hoàn toàn không biên dịch đoạn mã này
và ngăn chặn sự hiểu lầm ngay từ đầu quá trình phát triển.

#### Byte và Giá trị vô hướng và Cụm chữ cái! Ôi trời!

Một điểm khác về UTF-8 là thực sự có ba cách phù hợp để
nhìn vào các chuỗi theo góc nhìn của Rust: dưới dạng byte, giá trị vô hướng (scalar values), và cụm chữ cái (grapheme clusters - thứ gần giống nhất với những gì chúng ta gọi là _chữ cái_).

Nếu chúng ta nhìn vào từ tiếng Hindi “नमस्ते” được viết bằng chữ Devanagari, nó được
lưu trữ dưới dạng một vector các giá trị `u8` trông giống như thế này:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Đó là 18 byte và là cách máy tính lưu trữ dữ liệu này cuối cùng. Nếu chúng ta nhìn vào
chúng dưới dạng các giá trị vô hướng Unicode, vốn là kiểu `char` của Rust, những
byte đó trông như thế này:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Có sáu giá trị `char` ở đây, nhưng giá trị thứ tư và thứ sáu không phải là chữ cái:
chúng là các dấu phụ (diacritics) không có ý nghĩa khi đứng một mình. Cuối cùng, nếu chúng ta nhìn vào
chúng dưới dạng cụm chữ cái, chúng ta sẽ nhận được những gì một người bình thường gọi là bốn chữ cái
tạo nên từ tiếng Hindi:

```text
["न", "म", "स्", "ते"]
```

Rust cung cấp các cách khác nhau để thông dịch dữ liệu chuỗi thô mà máy tính
lưu trữ để mỗi chương trình có thể chọn cách thông dịch mà nó cần, bất kể dữ liệu đó thuộc ngôn ngữ nào của con người.

Lý do cuối cùng khiến Rust không cho phép chúng ta đánh chỉ số vào một `String` để lấy một
ký tự là các thao tác đánh chỉ số được kỳ vọng luôn tốn thời gian không đổi
(O(1)). Nhưng không thể đảm bảo hiệu suất đó với một `String`,
bởi vì Rust sẽ phải duyệt qua nội dung từ đầu cho đến chỉ số đó để xác định xem có bao nhiêu ký tự hợp lệ.

### Cắt chuỗi

Đánh chỉ số vào một chuỗi thường là một ý tưởng tồi vì không rõ kiểu trả về
của thao tác đánh chỉ số chuỗi nên là gì: một giá trị byte, một ký tự, một cụm chữ cái, hay một lát cắt chuỗi. Do đó, nếu bạn thực sự cần sử dụng
các chỉ số để tạo các lát cắt chuỗi, Rust yêu cầu bạn phải cụ thể hơn.

Thay vì đánh chỉ số bằng `[]` với một con số duy nhất, bạn có thể sử dụng `[]` với một
phạm vi (range) để tạo một lát cắt chuỗi chứa các byte cụ thể:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Ở đây, `s` sẽ là một `&str` chứa bốn byte đầu tiên của chuỗi.
Trước đó, chúng ta đã đề cập rằng mỗi ký tự này chiếm hai byte, điều đó có nghĩa là
`s` sẽ là `Зд`.

Nếu chúng ta cố gắng cắt chỉ một phần byte của một ký tự với thứ gì đó như
`&hello[0..1]`, Rust sẽ bị hoảng loạn (panic) khi chạy tương tự như khi một
chỉ số không hợp lệ được truy cập trong một vector:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Bạn nên thận trọng khi tạo các lát cắt chuỗi bằng các phạm vi, vì làm
như vậy có thể khiến chương trình của bạn bị dừng đột ngột.

### Các phương thức để Duyệt qua Chuỗi

Cách tốt nhất để thao tác trên các phần của chuỗi là chỉ rõ xem
bạn muốn ký tự hay byte. Đối với các giá trị vô hướng Unicode riêng lẻ, hãy sử dụng
phương thức `chars`. Gọi `chars` trên chuỗi “Зд” sẽ tách ra và trả về hai giá trị của
kiểu `char`, và bạn có thể duyệt qua kết quả để truy cập từng phần tử:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Đoạn mã này sẽ in ra những nội dung sau:

```text
З
д
```

Ngoài ra, phương thức `bytes` trả về từng byte thô, điều này có thể
phù hợp với lĩnh vực của bạn:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Đoạn mã này sẽ in ra bốn byte tạo nên chuỗi này:

```text
208
151
208
180
```

Nhưng hãy nhớ rằng các giá trị vô hướng Unicode hợp lệ có thể được tạo thành từ nhiều
hơn một byte.

Việc lấy các cụm chữ cái (grapheme clusters) từ chuỗi, như với chữ Devanagari, rất
phức tạp, vì vậy chức năng này không được cung cấp bởi thư viện tiêu chuẩn. Các crate
có sẵn trên [crates.io](https://crates.io/)<!-- ignore --> nếu đây là
chức năng bạn cần.

### Chuỗi không hề đơn giản

Tóm lại, chuỗi rất phức tạp. Các ngôn ngữ lập trình khác nhau đưa ra
những lựa chọn khác nhau về cách trình bày sự phức tạp này cho lập trình viên. Rust
đã chọn cách xử lý chính xác dữ liệu `String` làm hành vi mặc định
cho tất cả các chương trình Rust, điều đó có nghĩa là các lập trình viên phải suy nghĩ nhiều hơn về việc
xử lý dữ liệu UTF-8 ngay từ đầu. Sự đánh đổi này để lộ nhiều hơn sự phức tạp của
chuỗi so với những gì thấy được trong các ngôn ngữ lập trình khác, nhưng nó ngăn bạn
khỏi việc phải xử lý các lỗi liên quan đến các ký tự không phải ASCII sau này trong
vòng đời phát triển của mình.

Tin tốt là thư viện tiêu chuẩn cung cấp rất nhiều chức năng được xây dựng
từ các kiểu `String` và `&str` để giúp xử lý các tình huống phức tạp này
một cách chính xác. Hãy nhớ xem tài liệu để biết các phương thức hữu ích như
`contains` để tìm kiếm trong một chuỗi và `replace` để thay thế các phần của
một chuỗi bằng một chuỗi khác.

Hãy chuyển sang một thứ ít phức tạp hơn một chút: bảng băm (hash map)!

{{#quiz ../quizzes/ch08-02-string-sec2.toml}}
