## Tính có thể bác bỏ (Refutability): Liệu một mẫu có thể không khớp

Mẫu có hai dạng: refutable (có thể bác bỏ) và irrefutable (không thể bác bỏ).
Các mẫu sẽ khớp với bất kỳ giá trị nào có thể được truyền vào là
_irrefutable_. Một ví dụ sẽ là `x` trong câu lệnh
`let x = 5;` bởi vì `x` khớp với bất kỳ thứ gì và do đó không thể
thất bại khi khớp. Các mẫu có thể không khớp với một số giá trị
có thể có là _refutable_. Dưới đây là một số ví dụ:

<!-- BEGIN INTERVENTION: 3c29eb2d-cbe9-4a2c-99b8-aa5c6467c8b4 -->

- Trong biểu thức `if let Some(x) = a_value`, thì `Some(x)` là refutable. Nếu giá trị trong biến `a_value` là `None` thay vì
  `Some`, mẫu `Some(x)` sẽ không khớp.
- Trong biểu thức `if let &[x, ..] = a_slice`, thì `&[x, ..]` là refutable. Nếu giá trị trong biến `a_slice` không có
  phần tử nào, mẫu `&[x, ..]` sẽ không khớp.
<!-- END INTERVENTION: 3c29eb2d-cbe9-4a2c-99b8-aa5c6467c8b4 -->

Các tham số hàm, câu lệnh `let`, và vòng lặp `for` chỉ có thể
chấp nhận các mẫu irrefutable bởi vì chương trình không thể làm bất cứ điều gì
có ý nghĩa khi các giá trị không khớp. Các biểu thức `if let`
và `while let` và câu lệnh `let...else` chấp nhận cả mẫu
refutable và irrefutable, nhưng trình biên dịch cảnh báo đối với các mẫu
irrefutable bởi vì, theo định nghĩa, chúng được dùng để xử lý khả năng
thất bại: chức năng của một câu lệnh điều kiện nằm ở khả năng
thực hiện khác nhau tùy thuộc vào sự thành công hay thất bại.
tùy thuộc vào thành công hay thất bại.

Nói chung, bạn không cần phải lo lắng về sự phân biệt giữa mẫu
refutable và irrefutable; tuy nhiên, bạn cần phải làm quen với
khái niệm về tính có thể bác bỏ (refutability) để bạn có thể phản hồi khi
thấy nó trong một thông báo lỗi. Trong những trường hợp đó,
bạn sẽ cần thay đổi mẫu hoặc cấu trúc mà bạn đang sử dụng
với mẫu đó, tùy thuộc vào hành vi dự định của mã.
hành vi dự định của mã.

Hãy xem một ví dụ về điều gì xảy ra khi chúng ta cố gắng sử dụng
một mẫu refutable ở nơi mà Rust yêu cầu một mẫu irrefutable
và ngược lại. Liệt kê 19-8 hiển thị một câu lệnh `let`, nhưng đối với
mẫu, chúng ta đã chỉ định `Some(x)`, một mẫu refutable. Như bạn có thể
mong đợi, mã này sẽ không biên dịch.

<Listing number="19-8" caption="Cố gắng sử dụng một mẫu refutable với `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Nếu `some_option_value` là một giá trị `None`, nó sẽ thất bại khi
khớp với mẫu `Some(x)`, nghĩa là mẫu đó là refutable. Tuy nhiên,
câu lệnh `let` chỉ có thể chấp nhận một mẫu irrefutable bởi vì không có
gì hợp lệ mà mã có thể làm với giá trị `None`. Tại thời điểm biên dịch,
Rust sẽ phàn nàn rằng chúng ta đã cố gắng sử dụng một mẫu refutable
ở nơi yêu cầu một mẫu irrefutable:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Bởi vì chúng ta đã không bao quát (và không thể bao quát!) mọi giá trị hợp lệ
với mẫu `Some(x)`, Rust tạo ra một lỗi biên dịch một cách chính đáng.
lỗi biên dịch.

Nếu chúng ta có một mẫu refutable ở nơi cần một mẫu irrefutable,
chúng ta có thể khắc phục bằng cách thay đổi mã sử dụng mẫu đó:
thay vì sử dụng `let`, chúng ta có thể sử dụng `if let`. Khi đó nếu
mẫu không khớp, mã sẽ chỉ bỏ qua đoạn mã trong dấu ngoặc nhọn,
cung cấp cho nó một cách để tiếp tục một cách hợp lệ. Liệt kê 19-9
cho thấy cách sửa mã trong Liệt kê 19-8.

<Listing number="19-9" caption="Sử dụng `let...else` và một khối với các mẫu refutable thay vì `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Chúng ta đã cho mã một lối thoát! Mã này hiện đã hoàn toàn hợp lệ.
Tuy nhiên, nếu chúng ta đưa cho `if let` một mẫu irrefutable (một mẫu
sẽ luôn khớp), chẳng hạn như `x`, như được hiển thị trong Liệt kê 19-10,
trình biên dịch sẽ đưa ra một cảnh báo.

<Listing number="19-10" caption="Cố gắng sử dụng một mẫu irrefutable với `if let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust phàn nàn rằng việc sử dụng `if let` với một mẫu irrefutable là
không có ý nghĩa:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Vì lý do này, các nhánh match phải sử dụng các mẫu refutable, ngoại trừ
nhánh cuối cùng, nhánh này nên khớp với bất kỳ giá trị còn lại nào
bằng một mẫu irrefutable. Rust cho phép chúng ta sử dụng một mẫu
irrefutable trong một `match` chỉ có một nhánh, nhưng cú pháp này
không đặc biệt hữu ích và có thể được thay thế bằng một câu lệnh `let`
đơn giản hơn.

Bây giờ bạn đã biết nơi để sử dụng các mẫu và sự khác biệt giữa
mẫu refutable và irrefutable, hãy cùng tìm hiểu tất cả cú pháp
chúng ta có thể sử dụng để tạo ra các mẫu.

{{#quiz ../quizzes/ch18-02-refutability.toml}}
