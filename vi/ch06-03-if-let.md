## Điều khiển luồng súc tích với `if let` và `let else`

Cú pháp `if let` cho phép bạn kết hợp `if` và `let` thành một cách ít rườm rà hơn để xử lý các giá trị khớp với một mẫu trong khi bỏ qua phần còn lại. Hãy xem xét chương trình trong Liệt kê 6-6 thực hiện khớp trên một giá trị `Option<u8>` trong biến `config_max` nhưng chỉ muốn thực thi mã nếu giá trị đó là biến thể `Some`.

<Listing number="6-6" caption="Một `match` chỉ quan tâm đến việc thực thi mã khi giá trị là `Some`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Nếu giá trị là `Some`, chúng ta in ra giá trị trong biến thể `Some` bằng cách liên kết (binding) giá trị đó với biến `max` trong mẫu. Chúng ta không muốn làm gì với giá trị `None`. Để thỏa mãn biểu thức `match`, chúng ta phải thêm `_ => ()` sau khi xử lý chỉ một biến thể, đây là đoạn mã mẫu (boilerplate code) gây phiền nhiễu khi phải thêm vào.

Thay vào đó, chúng ta có thể viết điều này theo cách ngắn gọn hơn bằng cách sử dụng `if let`. Đoạn mã sau đây hoạt động giống như `match` trong Liệt kê 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

Cú pháp `if let` nhận một mẫu và một biểu thức ngăn cách bởi một dấu bằng. Nó hoạt động giống như một `match`, trong đó biểu thức được đưa vào `match` và mẫu là nhánh đầu tiên của nó. Trong trường hợp này, mẫu là `Some(max)`, và `max` liên kết với giá trị bên trong `Some`. Sau đó, chúng ta có thể sử dụng `max` trong thân khối `if let` giống như cách chúng ta đã sử dụng `max` trong nhánh `match` tương ứng. Mã trong khối `if let` chỉ chạy nếu giá trị khớp với mẫu.

Sử dụng `if let` có nghĩa là gõ ít hơn, ít thụt lề hơn và ít mã mẫu hơn. Tuy nhiên, bạn sẽ mất đi khả năng kiểm tra tính toàn diện (exhaustive checking) mà `match` bắt buộc để đảm bảo bạn không quên xử lý bất kỳ trường hợp nào. Việc lựa chọn giữa `match` và `if let` phụ thuộc vào những gì bạn đang làm trong tình huống cụ thể của mình và liệu việc đạt được sự súc tích có phải là một sự đánh đổi phù hợp cho việc mất đi kiểm tra tính toàn diện hay không.

Nói cách khác, bạn có thể coi `if let` như là một cú pháp thay thế (syntax sugar) cho một `match` chạy mã khi giá trị khớp với một mẫu và sau đó bỏ qua tất cả các giá trị khác.

Chúng ta có thể bao gồm một `else` với một `if let`. Khối mã đi kèm với `else` tương đương với khối mã đi kèm với trường hợp `_` trong biểu thức `match` tương ứng với `if let` và `else`. Nhớ lại định nghĩa enum `Coin` trong Liệt kê 6-4, nơi biến thể `Quarter` cũng giữ một giá trị `UsState`. Nếu chúng ta muốn đếm tất cả các đồng xu không phải quarter mà chúng ta thấy, đồng thời thông báo tiểu bang của các đồng quarter, chúng ta có thể làm điều đó với một biểu thức `match`, như thế này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Hoặc chúng ta có thể sử dụng một biểu thức `if let` và `else`, như thế này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## Luôn ở trên “Con đường hạnh phúc” với `let...else`

Một mẫu phổ biến là thực hiện một số tính toán khi một giá trị hiện diện và trả về một giá trị mặc định nếu ngược lại. Tiếp tục với ví dụ về các đồng xu có giá trị `UsState`, nếu chúng ta muốn nói điều gì đó thú vị tùy thuộc vào độ lâu đời của tiểu bang trên đồng quarter, chúng ta có thể giới thiệu một phương thức trên `UsState` để kiểm tra tuổi của một tiểu bang, như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Sau đó, chúng ta có thể sử dụng `if let` để khớp với loại đồng xu, giới thiệu một biến `state` bên trong thân điều kiện, như trong Liệt kê 6-7.

<Listing number="6-7" caption="Kiểm tra xem một tiểu bang có tồn tại vào năm 1900 hay không bằng cách sử dụng các điều kiện lồng nhau bên trong một `if let`.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Cách đó giải quyết được công việc, nhưng nó đã đẩy công việc vào trong thân của câu lệnh `if let`, và nếu công việc cần làm phức tạp hơn, có thể khó theo dõi chính xác cách các nhánh cấp cao nhất liên quan đến nhau. Chúng ta cũng có thể tận dụng thực tế là các biểu thức tạo ra một giá trị để tạo ra `state` từ `if let` hoặc trả về sớm, như trong Liệt kê 6-8. (Bạn cũng có thể làm điều tương tự với một `match`.)

<Listing number="6-8" caption="Sử dụng `if let` để tạo ra một giá trị hoặc trả về sớm.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

Tuy nhiên, cách này cũng có điểm gây khó khăn khi theo dõi! Một nhánh của `if let` tạo ra một giá trị, và nhánh còn lại thoát khỏi hàm hoàn toàn.

Để làm cho mẫu phổ biến này dễ diễn đạt hơn, Rust có `let...else`. Cú pháp `let...else` nhận một mẫu ở phía bên trái và một biểu thức ở phía bên phải, rất giống với `if let`, nhưng nó không có nhánh `if`, chỉ có nhánh `else`. Nếu mẫu khớp, nó sẽ liên kết giá trị từ mẫu trong phạm vi bên ngoài. Nếu mẫu _không_ khớp, chương trình sẽ đi vào nhánh `else`, nhánh này phải trả về khỏi hàm.

Trong Liệt kê 6-9, bạn có thể thấy Liệt kê 6-8 trông như thế nào khi sử dụng `let...else` thay cho `if let`.

<Listing number="6-9" caption="Sử dụng `let...else` để làm rõ luồng xử lý trong hàm.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Lưu ý rằng bằng cách này, mã vẫn ở trên “con đường hạnh phúc” (happy path) trong thân chính của hàm, mà không có luồng điều khiển khác biệt đáng kể cho hai nhánh như cách `if let` đã làm.

Nếu bạn gặp tình huống trong đó chương trình của bạn có logic quá dài dòng để diễn đạt bằng `match`, hãy nhớ rằng `if let` và `let...else` cũng có sẵn trong hộp công cụ Rust của bạn.

{{#quiz ../quizzes/ch06-03-if-let.toml}}

## Tóm tắt

Đến đây chúng ta đã tìm hiểu cách sử dụng enum để tạo các kiểu tùy chỉnh có thể là một trong một tập hợp các giá trị được liệt kê. Chúng ta đã chỉ ra cách kiểu `Option<T>` của thư viện tiêu chuẩn giúp bạn sử dụng hệ thống kiểu để ngăn ngừa lỗi. Khi các giá trị enum có dữ liệu bên trong, bạn có thể sử dụng `match` hoặc `if let` để trích xuất và sử dụng các giá trị đó, tùy thuộc vào số lượng trường hợp bạn cần xử lý.

Giờ đây các chương trình Rust của bạn có thể diễn đạt các khái niệm trong lĩnh vực của mình bằng cách sử dụng struct và enum. Việc tạo các kiểu tùy chỉnh để sử dụng trong API của bạn đảm bảo tính an toàn kiểu (type safety): trình biên dịch sẽ chắc chắn rằng các hàm của bạn chỉ nhận được các giá trị thuộc kiểu mà mỗi hàm mong đợi.

Để cung cấp một API được tổ chức tốt cho người dùng, dễ sử dụng và chỉ để lộ chính xác những gì người dùng của bạn sẽ cần, bây giờ chúng ta hãy chuyển sang các mô-đun của Rust.
