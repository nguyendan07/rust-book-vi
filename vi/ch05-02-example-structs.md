## Chương trình Sử dụng Struct

Để hiểu khi nào chúng ta có thể muốn sử dụng struct, hãy viết một chương trình
tính diện tích của một hình chữ nhật. Chúng ta sẽ bắt đầu bằng cách sử dụng các biến đơn lẻ, và
sau đó cấu trúc lại (refactor) chương trình cho đến khi chúng ta sử dụng struct thay thế.

Hãy tạo một dự án binary mới với Cargo tên là _rectangles_ để nhận vào
chiều rộng và chiều cao của một hình chữ nhật được chỉ định bằng pixel và tính diện tích
của hình chữ nhật đó. Listing 5-8 hiển thị một chương trình ngắn với một cách để thực hiện
chính xác điều đó trong file _src/main.rs_ của dự án.

<Listing number="5-8" file-name="src/main.rs" caption="Tính diện tích của một hình chữ nhật được chỉ định bởi các biến chiều rộng và chiều cao riêng biệt">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Bây giờ, hãy chạy chương trình này bằng lệnh `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Đoạn mã này đã thành công trong việc tính toán diện tích của hình chữ nhật bằng cách gọi
hàm `area` với từng kích thước, nhưng chúng ta có thể làm nhiều hơn nữa để mã này trở nên rõ ràng
và dễ đọc hơn.

Vấn đề với mã này được thể hiện rõ trong chữ ký (signature) của hàm `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Hàm `area` được cho là để tính diện tích của một hình chữ nhật, nhưng hàm
chúng ta viết lại có hai tham số, và không có chỗ nào trong chương trình thể hiện rõ
rằng các tham số đó có liên quan với nhau. Sẽ dễ đọc hơn và dễ quản lý hơn nếu
nhóm chiều rộng và chiều cao lại với nhau. Chúng ta đã thảo luận về một cách
chúng ta có thể làm điều đó trong phần [“Kiểu Tuple”][the-tuple-type]<!-- ignore -->
của Chương 3: bằng cách sử dụng các tuple.

### Cấu trúc lại với Tuple

Listing 5-9 cho thấy một phiên bản khác của chương trình sử dụng tuple.

<Listing number="5-9" file-name="src/main.rs" caption="Chỉ định chiều rộng và chiều cao của hình chữ nhật bằng một tuple">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

Theo một cách nào đó, chương trình này tốt hơn. Tuple cho phép chúng ta thêm một chút cấu trúc, và
bây giờ chúng ta chỉ truyền một đối số duy nhất. Nhưng theo một cách khác, phiên bản này lại ít
rõ ràng hơn: các tuple không đặt tên cho các thành phần của chúng, vì vậy chúng ta phải truy cập bằng chỉ số
vào các phần của tuple, làm cho việc tính toán của chúng ta trở nên kém rõ ràng hơn.

Việc nhầm lẫn giữa chiều rộng và chiều cao sẽ không quan trọng đối với việc tính toán diện tích, nhưng nếu
chúng ta muốn vẽ hình chữ nhật lên màn hình, nó sẽ rất quan trọng! Chúng ta sẽ phải
luôn ghi nhớ rằng `width` là chỉ số tuple `0` và `height` là chỉ số
tuple `1`. Điều này thậm chí còn khó khăn hơn cho người khác để tìm hiểu và ghi nhớ
nếu họ sử dụng mã của chúng ta. Bởi vì chúng ta đã không truyền đạt được ý nghĩa của
dữ liệu trong mã nguồn, giờ đây việc dẫn đến sai sót sẽ dễ dàng hơn.

### Cấu trúc lại với Struct: Thêm nhiều ý nghĩa hơn

Chúng ta sử dụng struct để thêm ý nghĩa bằng cách dán nhãn cho dữ liệu. Chúng ta có thể chuyển đổi tuple
chúng ta đang dùng thành một struct với tên cho toàn bộ cũng như tên cho các
thành phần, như được hiển thị trong Listing 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Định nghĩa một struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã định nghĩa một struct và đặt tên cho nó là `Rectangle`. Bên trong dấu ngoặc nhọn,
chúng ta đã định nghĩa các trường (fields) là `width` và `height`, cả hai đều có
kiểu `u32`. Sau đó, trong hàm `main`, chúng ta đã tạo một thể hiện (instance) cụ thể của `Rectangle`
có chiều rộng là `30` và chiều cao là `50`.

Hàm `area` của chúng ta hiện được định nghĩa với một tham số, mà chúng ta đã đặt tên là
`rectangle`, kiểu của nó là một phép vay mượn bất biến (immutable borrow) của một thể hiện struct `Rectangle`.
Như đã đề cập trong Chương 4, chúng ta muốn vay mượn struct thay vì
nắm quyền sở hữu nó. Bằng cách này, `main` vẫn giữ được quyền sở hữu của mình và có thể tiếp tục
sử dụng `rect1`, đó là lý do chúng ta sử dụng dấu `&` trong chữ ký hàm và
nơi chúng ta gọi hàm.

Hàm `area` truy cập các trường `width` và `height` của thể hiện `Rectangle`
(lưu ý rằng việc truy cập các trường của một thể hiện struct được vay mượn không
di chuyển các giá trị của trường, đó là lý do tại sao bạn thường thấy các phép vay mượn struct). Chữ ký
hàm của chúng ta cho `area` hiện giờ đã nói chính xác những gì chúng ta muốn: tính diện tích
của `Rectangle`, sử dụng các trường `width` và `height` của nó. Điều này truyền đạt rằng
chiều rộng và chiều cao có liên quan đến nhau, và nó cung cấp những cái tên mô tả cho
các giá trị thay vì sử dụng các giá trị chỉ số tuple là `0` và `1`. Đây là một điểm cộng cho sự rõ ràng.

### Thêm các chức năng hữu ích với Derived Traits

Sẽ rất hữu ích nếu có thể in ra một thể hiện của `Rectangle` trong khi chúng ta đang
gỡ lỗi chương trình và xem giá trị của tất cả các trường của nó. Listing 5-11 cố gắng
sử dụng macro [`println!`][println]<!-- ignore --> như chúng ta đã sử dụng trong các
chương trước. Tuy nhiên, việc này sẽ không hoạt động.

<Listing number="5-11" file-name="src/main.rs" caption="Cố gắng in ra một thể hiện `Rectangle` thực tế">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Khi chúng ta biên dịch mã này, chúng ta nhận được một lỗi với thông điệp cốt lõi sau:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Macro `println!` có thể thực hiện nhiều kiểu định dạng, và theo mặc định, các dấu ngoặc nhọn
nói với `println!` sử dụng định dạng được gọi là `Display`: đầu ra dành cho việc tiêu thụ trực tiếp
của người dùng cuối. Các kiểu dữ liệu nguyên thủy mà chúng ta đã thấy cho đến nay
đều triển khai `Display` theo mặc định vì chỉ có một cách duy nhất mà bạn muốn hiển thị
số `1` hoặc bất kỳ kiểu nguyên thủy nào khác cho người dùng. Nhưng với struct, cách thức mà
`println!` nên định dạng đầu ra lại kém rõ ràng hơn vì có nhiều khả năng hiển thị hơn:
Bạn có muốn dấu phẩy hay không? Bạn có muốn in các dấu ngoặc nhọn không? Có nên hiển thị tất cả
các trường không? Do sự mơ hồ này, Rust không cố gắng đoán những gì chúng ta muốn, và các struct
không có sẵn một triển khai của `Display` để sử dụng với `println!` và trình giữ chỗ `{}`.

Nếu chúng ta tiếp tục đọc các lỗi, chúng ta sẽ tìm thấy ghi chú hữu ích này:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Hãy thử xem! Lời gọi macro `println!` giờ sẽ trông giống như `println!("rect1 is
{rect1:?}");`. Việc đặt ký hiệu `:?` vào bên trong dấu ngoặc nhọn nói với
`println!` rằng chúng ta muốn sử dụng một định dạng đầu ra được gọi là `Debug`. Trait `Debug`
cho phép chúng ta in struct của mình theo cách hữu ích cho các nhà phát triển để chúng ta có thể
thấy giá trị của nó trong khi đang gỡ lỗi mã nguồn.

Biên dịch mã với thay đổi này. Chán thật! Chúng ta vẫn nhận được lỗi:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Nhưng một lần nữa, trình biên dịch lại cho chúng ta một ghi chú hữu ích:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _có_ bao gồm chức năng để in ra thông tin gỡ lỗi, nhưng chúng ta
phải chọn tham gia một cách rõ ràng để làm cho chức năng đó có sẵn cho struct của chúng ta.
Để làm điều đó, chúng ta thêm thuộc tính bên ngoài (outer attribute) `#[derive(Debug)]` ngay trước
định nghĩa struct, như được hiển thị trong Listing 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Thêm thuộc tính để dẫn xuất (derive) trait `Debug` và in thể hiện `Rectangle` bằng định dạng gỡ lỗi">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Bây giờ khi chúng ta chạy chương trình, chúng ta sẽ không gặp bất kỳ lỗi nào, và chúng ta sẽ thấy
kết quả đầu ra như sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Tuyệt! Nó không phải là đầu ra đẹp nhất, nhưng nó hiển thị giá trị của tất cả các trường
cho thể hiện này, điều này chắc chắn sẽ giúp ích trong quá trình gỡ lỗi. Khi chúng ta có
các struct lớn hơn, việc có đầu ra dễ đọc hơn một chút sẽ hữu ích; trong
những trường hợp đó, chúng ta có thể sử dụng `{:#?}` thay vì `{:?}` trong chuỗi `println!`. Trong
ví dụ này, việc sử dụng kiểu `{:#?}` sẽ cho kết quả đầu ra như sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Một cách khác để in ra một giá trị bằng định dạng `Debug` là sử dụng macro [`dbg!`][dbg]<!-- ignore -->,
vốn nắm quyền sở hữu của một biểu thức (ngược lại với `println!`, vốn nhận một tham chiếu),
in ra tên file và số dòng nơi lời gọi macro `dbg!` đó xảy ra trong mã của bạn cùng với giá trị
kết quả của biểu thức đó, và trả lại quyền sở hữu của giá trị đó.

> Lưu ý: Gọi macro `dbg!` in ra luồng console lỗi tiêu chuẩn
> (`stderr`), ngược lại với `println!`, vốn in ra luồng console đầu ra tiêu chuẩn
> (`stdout`). Chúng ta sẽ nói nhiều hơn về `stderr` và `stdout` trong phần
> [“Ghi Thông điệp Lỗi ra Standard Error thay vì Standard Output”
> trong Chương 12][err]<!-- ignore -->.

Dưới đây là một ví dụ nơi chúng ta quan tâm đến giá trị được gán cho
trường `width`, cũng như giá trị của toàn bộ struct trong `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Chúng ta có thể đặt `dbg!` bao quanh biểu thức `30 * scale` và, bởi vì `dbg!`
trả lại quyền sở hữu giá trị của biểu thức, trường `width` sẽ nhận được
cùng một giá trị giống như thể chúng ta không có lời gọi `dbg!` ở đó. Chúng ta không muốn `dbg!`
nắm quyền sở hữu của `rect1`, vì vậy chúng ta sử dụng một tham chiếu đến `rect1` trong lời gọi tiếp theo.
Dưới đây là kết quả đầu ra của ví dụ này trông như thế nào:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Chúng ta có thể thấy phần đầu tiên của đầu ra đến từ file _src/main.rs_ dòng 10 nơi chúng ta đang
gỡ lỗi biểu thức `30 * scale`, và giá trị kết quả của nó là `60` (định dạng `Debug`
được triển khai cho các số nguyên là chỉ in ra giá trị của chúng). Lời gọi
`dbg!` trên dòng 14 của _src/main.rs_ in ra giá trị của `&rect1`, chính là
struct `Rectangle`. Đầu ra này sử dụng định dạng `Debug` đẹp của
kiểu `Rectangle`. Macro `dbg!` có thể thực sự hữu ích khi bạn đang cố gắng
tìm hiểu xem mã của mình đang làm gì!

Ngoài trait `Debug`, Rust đã cung cấp một số trait để chúng ta sử dụng với thuộc tính
`derive` nhằm thêm các hành vi hữu ích vào các kiểu tùy chỉnh của mình. Những trait đó và hành vi của chúng
được liệt kê trong [Phụ lục C][app-c]<!-- ignore -->. Chúng ta sẽ tìm hiểu cách triển khai các trait này với các hành vi
tùy chỉnh cũng như cách tạo các trait của riêng bạn trong Chương 10. Ngoài `derive`,
cũng còn nhiều thuộc tính khác; để biết thêm thông tin, hãy xem [phần “Attributes” của Rust Reference][attributes].

Hàm `area` của chúng ta rất cụ thể: nó chỉ tính diện tích của các hình chữ nhật.
Sẽ hữu ích nếu gắn kết hành vi này chặt chẽ hơn với struct `Rectangle` của chúng ta
vì nó sẽ không hoạt động với bất kỳ kiểu nào khác. Hãy xem cách chúng ta có thể tiếp tục
cấu trúc lại mã này bằng cách biến hàm `area` thành một _phương thức_ (method) `area`
được định nghĩa trên kiểu `Rectangle` của chúng ta.

{{#quiz ../quizzes/ch05-02-example-structs.toml}}

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: https://doc.rust-lang.org/std/macro.println.html
[dbg]: https://doc.rust-lang.org/std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: https://doc.rust-lang.org/reference/attributes.html
