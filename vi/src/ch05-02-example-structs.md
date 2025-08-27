## Một chương trình ví dụ sử dụng struct

Để hiểu khi nào chúng ta nên dùng struct, hãy viết một chương trình tính diện tích hình chữ nhật. Ta sẽ bắt đầu bằng các biến riêng lẻ, rồi refactor chương trình cho đến khi dùng struct.

Hãy tạo một dự án nhị phân mới với Cargo tên là _rectangles_ để nhận chiều rộng và chiều cao của một hình chữ nhật (đơn vị pixel) và tính diện tích của nó. Liệt kê 5-8 cho thấy một chương trình ngắn làm đúng điều đó trong _src/main.rs_ của dự án.

<Listing number="5-8" file-name="src/main.rs" caption="Tính diện tích hình chữ nhật khi truyền riêng lẻ chiều rộng và chiều cao">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Bây giờ, chạy chương trình bằng `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Đoạn mã này đã tính được diện tích hình chữ nhật bằng cách gọi hàm `area` với từng kích thước, nhưng ta có thể làm cho mã rõ ràng và dễ đọc hơn nữa.

Vấn đề của đoạn mã thể hiện trong chữ ký của `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Hàm `area` được kỳ vọng tính diện tích của một hình chữ nhật, nhưng hàm ta viết có hai tham số, và không có chỗ nào trong chương trình thể hiện rõ rằng hai tham số này có liên quan đến nhau. Sẽ dễ đọc và dễ quản lý hơn nếu nhóm chiều rộng và chiều cao lại với nhau. Ta đã bàn về một cách làm điều đó trong phần [“Kiểu tuple”][the-tuple-type]<!-- ignore --> của Chương 3: dùng tuple.

### Refactor với tuple

Liệt kê 5-9 cho thấy một phiên bản khác của chương trình dùng tuple.

<Listing number="5-9" file-name="src/main.rs" caption="Chỉ định chiều rộng và chiều cao của hình chữ nhật bằng một tuple">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

Theo một khía cạnh, chương trình này tốt hơn. Tuple cho phép ta thêm một chút cấu trúc, và giờ ta chỉ truyền một đối số. Nhưng ở khía cạnh khác, phiên bản này kém rõ ràng hơn: tuple không đặt tên cho các phần tử, nên ta phải đánh chỉ số vào các phần của tuple, làm cho phép tính kém hiển nhiên.

Nhầm lẫn giữa chiều rộng và chiều cao không ảnh hưởng đến phép tính diện tích, nhưng nếu muốn vẽ hình chữ nhật lên màn hình thì lại quan trọng! Ta sẽ phải nhớ rằng `width` là chỉ số `0` của tuple và `height` là chỉ số `1`. Điều này còn khó hơn cho người khác khi họ dùng mã của ta. Bởi vì ta chưa truyền đạt ý nghĩa của dữ liệu trong mã, nên giờ dễ phát sinh lỗi hơn.

### Refactor với struct: Thêm ý nghĩa

Ta dùng struct để thêm ý nghĩa bằng cách gắn nhãn cho dữ liệu. Ta có thể chuyển tuple đang dùng thành một struct có tên cho toàn bộ lẫn tên cho từng phần, như trong Liệt kê 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Định nghĩa struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Ở đây, ta đã định nghĩa một struct tên là `Rectangle`. Bên trong dấu ngoặc nhọn, ta định nghĩa các trường `width` và `height`, cả hai đều có kiểu `u32`. Sau đó, trong `main`, ta tạo một instance cụ thể của `Rectangle` có chiều rộng `30` và chiều cao `50`.

Hàm `area` giờ được định nghĩa với một tham số, ta đặt tên là `rectangle`, có kiểu là một tham chiếu mượn bất biến đến một instance của struct `Rectangle`. Như đã nói trong Chương 4, ta muốn mượn struct thay vì chuyển quyền sở hữu của nó. Cách này giúp `main` giữ quyền sở hữu và tiếp tục dùng `rect1`, đó là lý do ta dùng `&` trong chữ ký hàm và khi gọi hàm.

Hàm `area` truy cập các trường `width` và `height` của instance `Rectangle` (lưu ý rằng truy cập các trường của một struct đang được mượn không di chuyển giá trị của trường, đó là lý do bạn thường thấy mượn struct). Chữ ký hàm `area` giờ nói đúng điều ta muốn: tính diện tích của `Rectangle` bằng các trường `width` và `height` của nó. Điều này thể hiện rằng chiều rộng và chiều cao có liên hệ với nhau, và đặt tên mô tả cho các giá trị thay vì dùng chỉ số tuple `0` và `1`. Rất rõ ràng!

### Thêm chức năng hữu ích với các trait dẫn xuất (derived)

Sẽ hữu ích nếu ta có thể in một instance của `Rectangle` khi debug chương trình và xem giá trị của toàn bộ các trường. Liệt kê 5-11 thử dùng [macro `println!`][println]<!-- ignore --> như ta đã dùng trong các chương trước. Tuy nhiên, cách này sẽ không hoạt động.

<Listing number="5-11" file-name="src/main.rs" caption="Thử in một instance `Rectangle`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Khi biên dịch mã này, ta nhận được lỗi với thông điệp chính:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Macro `println!` có thể làm nhiều kiểu định dạng, và theo mặc định, dấu ngoặc nhọn báo cho `println!` dùng định dạng gọi là `Display`: đầu ra dành cho người dùng cuối. Các kiểu nguyên thủy ta đã thấy mặc định triển khai `Display` vì chỉ có một cách bạn muốn hiển thị `1` hay bất kỳ kiểu nguyên thủy nào khác cho người dùng. Nhưng với struct, cách `println!` nên định dạng đầu ra kém rõ ràng vì có nhiều khả năng hiển thị: Có cần dấu phẩy không? Có in dấu ngoặc nhọn không? Có nên hiển thị tất cả các trường không? Do sự mơ hồ này, Rust không cố đoán ý ta, và struct không có sẵn phần triển khai `Display` để dùng với `println!` và placeholder `{}`.

Nếu tiếp tục đọc lỗi, ta sẽ thấy ghi chú hữu ích này:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Hãy thử xem! Lời gọi macro `println!` giờ sẽ trông như `println!("rect1 is {rect1:?}");`. Đặt bộ chỉ định `:?` vào trong ngoặc nhọn báo cho `println!` rằng ta muốn định dạng đầu ra kiểu `Debug`. Trait `Debug` cho phép ta in struct theo cách hữu ích cho lập trình viên để thấy giá trị khi ta đang debug mã.

Biên dịch mã với thay đổi này. Chà! Ta vẫn gặp lỗi:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Nhưng một lần nữa, trình biên dịch đưa ra ghi chú hữu ích:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust có hỗ trợ in thông tin debug, nhưng ta phải chủ động bật nó để khả dụng cho struct của mình. Để làm vậy, thêm thuộc tính bên ngoài `#[derive(Debug)]` ngay trước định nghĩa struct, như trong Liệt kê 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Thêm thuộc tính để dẫn xuất trait `Debug` và in instance `Rectangle` bằng định dạng debug">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Giờ khi chạy chương trình, ta sẽ không còn lỗi và sẽ thấy đầu ra sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Tuyệt! Đây không phải là đầu ra đẹp nhất, nhưng nó hiển thị giá trị của tất cả các trường cho instance này, điều rất hữu ích khi debug. Khi có các struct lớn hơn, có đầu ra dễ đọc hơn sẽ hữu ích; trong các trường hợp đó, ta có thể dùng `{:#?}` thay cho `{:?}` trong chuỗi `println!`. Trong ví dụ này, dùng kiểu `{:#?}` sẽ cho đầu ra sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Một cách khác để in giá trị với định dạng `Debug` là dùng [macro `dbg!`][dbg]<!-- ignore -->, macro này nhận quyền sở hữu của một biểu thức (trái với `println!`, vốn nhận tham chiếu), in ra tệp và số dòng nơi lệnh gọi `dbg!` xuất hiện trong mã cùng với giá trị thu được của biểu thức đó, và trả lại quyền sở hữu của giá trị.

> Lưu ý: Gọi macro `dbg!` sẽ in ra luồng lỗi chuẩn (`stderr`), trái với `println!` in ra luồng đầu ra chuẩn (`stdout`). Ta sẽ nói thêm về `stderr` và `stdout` trong [phần “Ghi thông báo lỗi ra luồng lỗi chuẩn thay vì luồng đầu ra chuẩn” ở Chương 12][err]<!-- ignore -->.

Đây là ví dụ khi ta quan tâm đến giá trị gán cho trường `width`, cũng như giá trị của cả struct trong `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Ta có thể đặt `dbg!` quanh biểu thức `30 * scale` và, vì `dbg!` trả lại quyền sở hữu giá trị của biểu thức, trường `width` sẽ nhận cùng giá trị như khi không có lời gọi `dbg!` ở đó. Ta không muốn `dbg!` lấy quyền sở hữu của `rect1`, nên ta dùng tham chiếu đến `rect1` trong lần gọi tiếp theo. Đầu ra của ví dụ này trông như sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Ta có thể thấy phần đầu của đầu ra đến từ dòng 10 của _src/main.rs_ nơi ta đang debug biểu thức `30 * scale`, và giá trị kết quả là `60` (định dạng `Debug` cho số nguyên chỉ in giá trị của chúng). Lời gọi `dbg!` ở dòng 14 của _src/main.rs_ in giá trị của `&rect1`, tức struct `Rectangle`. Đầu ra này dùng định dạng `Debug` kiểu “đẹp” cho kiểu `Rectangle`. Macro `dbg!` có thể rất hữu ích khi bạn cố tìm hiểu mã của mình đang làm gì!

Ngoài trait `Debug`, Rust còn cung cấp một số trait để ta dùng với thuộc tính `derive` nhằm bổ sung hành vi hữu ích cho các kiểu tùy biến. Các trait đó và hành vi của chúng được liệt kê trong [Phụ lục C][app-c]<!-- ignore -->. Ta sẽ học cách tự triển khai các trait này với hành vi tùy biến cũng như cách tạo trait của riêng bạn trong Chương 10. Cũng có nhiều thuộc tính khác ngoài `derive`; để biết thêm, xem [phần “Attributes” trong Rust Reference][attributes].

Hàm `area` của ta hiện rất đặc thù: nó chỉ tính diện tích hình chữ nhật. Sẽ hữu ích nếu gắn hành vi này chặt chẽ hơn với struct `Rectangle` vì nó không áp dụng cho kiểu khác. Hãy xem ta có thể tiếp tục refactor mã bằng cách biến hàm `area` thành một phương thức `area` được định nghĩa trên kiểu `Rectangle` của chúng ta như thế nào.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
