## Sử dụng Trait Object cho phép các giá trị thuộc các kiểu khác nhau

Trong Chương 8, chúng ta đã đề cập rằng một hạn chế của vector là chúng chỉ có thể
lưu trữ các phần tử của một kiểu duy nhất. Chúng ta đã tạo ra một giải pháp thay thế trong Liệt kê 8-9, nơi
chúng ta định nghĩa một enum `SpreadsheetCell` có các biến thể để chứa số nguyên, số thực,
và văn bản. Điều này có nghĩa là chúng ta có thể lưu trữ các kiểu dữ liệu khác nhau trong mỗi ô và
vẫn có một vector đại diện cho một hàng các ô. Đây là một giải pháp hoàn hảo khi
các mục có thể thay đổi của chúng ta là một tập hợp cố định các kiểu mà chúng ta biết khi
mã được biên dịch.

Tuy nhiên, đôi khi chúng ta muốn người dùng thư viện của mình có thể mở rộng tập hợp
các kiểu hợp lệ trong một tình huống cụ thể. Để cho thấy cách chúng ta có thể đạt được
điều này, chúng ta sẽ tạo một ví dụ về công cụ giao diện người dùng đồ họa (GUI) duyệt qua
một danh sách các mục, gọi một phương thức `draw` trên mỗi mục để vẽ nó lên
màn hình—một kỹ thuật phổ biến cho các công cụ GUI. Chúng ta sẽ tạo một thư viện crate có tên
`gui` chứa cấu trúc của một thư viện GUI. Crate này có thể bao gồm
một số kiểu để mọi người sử dụng, chẳng hạn như `Button` hoặc `TextField`. Ngoài ra,
người dùng `gui` sẽ muốn tạo các kiểu của riêng họ có thể vẽ được: ví dụ,
một lập trình viên có thể thêm một `Image` và một người khác có thể thêm một
`SelectBox`.

Chúng ta sẽ không triển khai một thư viện GUI đầy đủ cho ví dụ này nhưng sẽ chỉ ra
cách các mảnh ghép khớp với nhau. Tại thời điểm viết thư viện, chúng ta không thể
biết và định nghĩa tất cả các kiểu mà các lập trình viên khác có thể muốn tạo. Nhưng chúng ta
biết rằng `gui` cần theo dõi nhiều giá trị thuộc các kiểu khác nhau và nó
cần gọi một phương thức `draw` trên mỗi giá trị có kiểu khác nhau này. Nó
không cần biết chính xác điều gì sẽ xảy ra khi chúng ta gọi phương thức `draw`,
chỉ cần biết rằng giá trị đó sẽ có phương thức đó sẵn sàng để chúng ta gọi.

Để làm điều này trong một ngôn ngữ có tính kế thừa, chúng ta có thể định nghĩa một lớp tên là
`Component` có một phương thức tên là `draw` trên đó. Các lớp khác, chẳng hạn như
`Button`, `Image`, và `SelectBox`, sẽ kế thừa từ `Component` và do đó
kế thừa phương thức `draw`. Mỗi lớp có thể ghi đè phương thức `draw` để định nghĩa
hành vi tùy chỉnh của chúng, nhưng khung làm việc (framework) có thể coi tất cả các kiểu như thể
chúng là các thực thể `Component` và gọi `draw` trên chúng. Nhưng vì Rust
không có tính kế thừa, chúng ta cần một cách khác để cấu trúc thư viện `gui` nhằm
cho phép người dùng mở rộng nó với các kiểu mới.

### Định nghĩa một Trait cho hành vi chung

Để triển khai hành vi mà chúng ta muốn `gui` có, chúng ta sẽ định nghĩa một trait tên là
`Draw` sẽ có một phương thức tên là `draw`. Sau đó, chúng ta có thể định nghĩa một vector
nhận một trait object. Một _trait object_ trỏ đến cả một thực thể của một kiểu
triển khai trait được chỉ định của chúng ta và một bảng được sử dụng để tra cứu các phương thức trait trên
kiểu đó tại thời điểm chạy. Chúng ta tạo một trait object bằng cách chỉ định một loại
con trỏ nào đó, chẳng hạn như một tham chiếu `&` hoặc một con trỏ thông minh `Box<T>`, sau đó là
từ khóa `dyn`, và sau đó chỉ định trait có liên quan. (Chúng ta sẽ nói về lý do
trait object phải sử dụng một con trỏ trong [“Các kiểu có kích thước động và Trait `Sized`”][dynamically-sized]<!-- ignore --> trong Chương 20.) Chúng ta có thể sử dụng trait object thay cho một kiểu generic hoặc cụ thể. Bất cứ nơi nào chúng ta sử dụng một trait object,
hệ thống kiểu của Rust sẽ đảm bảo tại thời điểm biên dịch rằng bất kỳ giá trị nào được sử dụng trong
ngữ cảnh đó sẽ triển khai trait của trait object đó. Do đó, chúng ta không cần
biết tất cả các kiểu có thể có tại thời điểm biên dịch.

Chúng ta đã đề cập rằng, trong Rust, chúng ta hạn chế gọi các struct và enum là
“đối tượng” để phân biệt chúng với các đối tượng của các ngôn ngữ khác. Trong một struct hoặc
enum, dữ liệu trong các trường struct và hành vi trong các khối `impl` được
tách biệt, trong khi ở các ngôn ngữ khác, dữ liệu và hành vi kết hợp thành một
khái niệm thường được dán nhãn là một đối tượng. Tuy nhiên, các trait object _thì_ giống
với các đối tượng trong các ngôn ngữ khác hơn theo nghĩa là chúng kết hợp dữ liệu và hành vi.
Nhưng các trait object khác với các đối tượng truyền thống ở chỗ chúng ta không thể
thêm dữ liệu vào một trait object. Các trait object không hữu ích một cách tổng quát như các đối tượng
trong các ngôn ngữ khác: mục đích cụ thể của chúng là cho phép trừu tượng hóa trên
các hành vi chung.

Liệt kê 18-3 cho thấy cách định nghĩa một trait tên là `Draw` với một phương thức tên là
`draw`.

<Listing number="18-3" file-name="src/lib.rs" caption="Định nghĩa trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Cú pháp này hẳn đã quen thuộc từ các cuộc thảo luận của chúng ta về cách định nghĩa các trait
trong Chương 10. Tiếp theo là một số cú pháp mới: Liệt kê 18-4 định nghĩa một struct tên là
`Screen` chứa một vector tên là `components`. Vector này có kiểu
`Box<dyn Draw>`, là một trait object; nó là một vật thay thế cho bất kỳ kiểu nào bên trong
một `Box` mà triển khai trait `Draw`.

<Listing number="18-4" file-name="src/lib.rs" caption="Định nghĩa struct `Screen` với một trường `components` chứa một vector các trait object triển khai trait `Draw` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

Trên struct `Screen`, chúng ta sẽ định nghĩa một phương thức tên là `run` sẽ gọi
phương thức `draw` trên mỗi `components` của nó, như được hiển thị trong Liệt kê 18-5.

<Listing number="18-5" file-name="src/lib.rs" caption="Một phương thức `run` trên `Screen` gọi phương thức `draw` trên mỗi component">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Điều này hoạt động khác với việc định nghĩa một struct sử dụng một tham số kiểu generic
với trait bounds. Một tham số kiểu generic chỉ có thể được thay thế bằng
một kiểu cụ thể tại một thời điểm, trong khi các trait object cho phép nhiều
kiểu cụ thể lấp đầy cho trait object tại thời điểm chạy. Ví dụ, chúng ta
có thể đã định nghĩa struct `Screen` bằng cách sử dụng một kiểu generic và một trait bound
như trong Liệt kê 18-6:

<Listing number="18-6" file-name="src/lib.rs" caption="Một triển khai thay thế của struct `Screen` và phương thức `run` của nó bằng cách sử dụng generics và trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Điều này giới hạn chúng ta trong một thực thể `Screen` có một danh sách các thành phần tất cả đều
thuộc kiểu `Button` hoặc tất cả đều thuộc kiểu `TextField`. Nếu bạn chỉ bao giờ có các
bộ sưu tập đồng nhất, việc sử dụng generics và trait bounds sẽ thích hợp hơn vì các
định nghĩa sẽ được đơn hình hóa (monomorphized) tại thời điểm biên dịch để sử dụng các kiểu cụ thể.

Mặt khác, với phương thức sử dụng các trait object, một thực thể `Screen`
có thể chứa một `Vec<T>` chứa một `Box<Button>` cũng như một
`Box<TextField>`. Hãy xem cách này hoạt động, và sau đó chúng ta sẽ nói về
các tác động đến hiệu năng tại thời điểm chạy.

### Triển khai Trait

Bây giờ chúng ta sẽ thêm một số kiểu triển khai trait `Draw`. Chúng ta sẽ cung cấp
kiểu `Button`. Một lần nữa, việc thực sự triển khai một thư viện GUI nằm ngoài phạm vi
của cuốn sách này, vì vậy phương thức `draw` sẽ không có bất kỳ triển khai hữu ích nào trong
thân của nó. Để tưởng tượng việc triển khai có thể trông như thế nào, một struct `Button`
có thể có các trường cho `width`, `height`, và `label`, như được hiển thị trong Liệt kê 18-7:

<Listing number="18-7" file-name="src/lib.rs" caption="Một struct `Button` triển khai trait `Draw` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

Các trường `width`, `height`, và `label` trên `Button` sẽ khác với các
trường trên các thành phần khác; ví dụ, một kiểu `TextField` có thể có những
trường tương tự cộng với một trường `placeholder`. Mỗi kiểu chúng ta muốn vẽ trên
màn hình sẽ triển khai trait `Draw` nhưng sẽ sử dụng mã khác nhau trong
phương thức `draw` để định nghĩa cách vẽ kiểu cụ thể đó, như `Button` đã làm
ở đây (không có mã GUI thực tế, như đã đề cập). Kiểu `Button`, chẳng hạn,
có thể có một khối `impl` bổ sung chứa các phương thức liên quan đến những gì
xảy ra khi người dùng nhấp vào nút. Những loại phương thức này sẽ không áp dụng cho
các kiểu như `TextField`.

Nếu ai đó sử dụng thư viện của chúng ta quyết định triển khai một struct `SelectBox` có
các trường `width`, `height`, và `options`, họ cũng sẽ triển khai trait `Draw`
trên kiểu `SelectBox`, như được hiển thị trong Liệt kê 18-8.

<Listing number="18-8" file-name="src/main.rs" caption="Một crate khác sử dụng `gui` và triển khai trait `Draw` trên một struct `SelectBox` ">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

### Sử dụng Trait

Giờ đây, người dùng thư viện của chúng ta có thể viết hàm `main` của họ để tạo một thực thể `Screen`.
Vào thực thể `Screen`, họ có thể thêm một `SelectBox` và một `Button`
bằng cách đặt mỗi cái vào một `Box<T>` để trở thành một trait object. Sau đó, họ có thể gọi
phương thức `run` trên thực thể `Screen`, phương thức này sẽ gọi `draw` trên mỗi
thành phần. Liệt kê 18-9 cho thấy triển khai này:

<Listing number="18-9" file-name="src/main.rs" caption="Sử dụng các trait object để lưu trữ các giá trị thuộc các kiểu khác nhau triển khai cùng một trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Khi chúng ta viết thư viện, chúng ta không biết rằng ai đó có thể thêm
kiểu `SelectBox`, nhưng triển khai `Screen` của chúng ta đã có thể hoạt động trên
kiểu mới và vẽ nó vì `SelectBox` triển khai trait `Draw`, điều đó
có nghĩa là nó triển khai phương thức `draw`.

Khái niệm này—về việc chỉ quan tâm đến các thông điệp mà một giá trị phản hồi
thay vì kiểu cụ thể của giá trị đó—tương tự như khái niệm _duck typing_ (kiểu con vịt)
trong các ngôn ngữ định kiểu động: nếu nó đi như một con vịt và kêu
như một con vịt, thì nó chắc chắn là một con vịt! Trong triển khai của `run` trên `Screen`
trong Liệt kê 18-5, `run` không cần biết kiểu cụ thể của mỗi
thành phần là gì. Nó không kiểm tra xem một thành phần là một thực thể của một `Button`
hay một `SelectBox`, nó chỉ gọi phương thức `draw` trên thành phần đó. Bằng cách
chỉ định `Box<dyn Draw>` làm kiểu của các giá trị trong vector `components`,
chúng ta đã định nghĩa `Screen` cần các giá trị mà chúng ta có thể gọi phương thức `draw` trên đó.

Ưu điểm của việc sử dụng các trait object và hệ thống kiểu của Rust để viết mã
tương tự như mã sử dụng duck typing là chúng ta không bao giờ phải kiểm tra xem một
giá trị có triển khai một phương thức cụ thể hay không tại thời điểm chạy hoặc lo lắng về việc gặp lỗi
nếu một giá trị không triển khai một phương thức nhưng chúng ta vẫn gọi nó. Rust sẽ không biên dịch
mã của chúng ta nếu các giá trị không triển khai các trait mà các trait object cần.

Ví dụ, Liệt kê 18-10 cho thấy điều gì xảy ra nếu chúng ta cố gắng tạo một `Screen`
với một `String` như một thành phần.

<Listing number="18-10" file-name="src/main.rs" caption="Cố gắng sử dụng một kiểu không triển khai trait của trait object">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

Chúng ta sẽ nhận được lỗi này vì `String` không triển khai trait `Draw`:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Lỗi này cho chúng ta biết rằng hoặc chúng ta đang truyền một thứ gì đó cho `Screen` mà chúng ta
không có ý định truyền và vì vậy nên truyền một kiểu khác, hoặc chúng ta nên triển khai
`Draw` trên `String` để `Screen` có thể gọi `draw` trên nó.

<!-- BEGIN INTERVENTION: cce62358-5291-4eb3-84d6-fbc570873ee3 -->

### Trait Object và Suy luận kiểu

Một nhược điểm của việc sử dụng các trait object là cách chúng tương tác với suy luận kiểu.
Ví dụ, hãy xem xét suy luận kiểu cho `Vec<T>`. Khi `T` không phải là một trait object,
Rust chỉ cần biết kiểu của một phần tử duy nhất trong vector để suy luận ra `T`. Vì vậy
một vector trống sẽ gây ra lỗi suy luận kiểu:

```rust,ignore,does_not_compile
# fn main() {
let v = vec![];
// error[E0282]: type annotations needed for `Vec<T>`
# }
```

Nhưng việc thêm một phần tử cho phép Rust suy luận ra kiểu của vector:

```rust,ignore
# fn main() {
let v = vec!["Hello world"];
// ok, v : Vec<&str>
# }
```

Suy luận kiểu khó khăn hơn đối với các trait object. Ví dụ, giả sử chúng ta cố gắng tách
mảng `components` trong Liệt kê 17-9 thành một biến riêng biệt, như thế này:

```rust,ignore,does_not_compile
fn main() {
    let components = vec![
        Box::new(SelectBox { /* .. */ }),
        Box::new(Button { /* .. */ }),
    ];
    let screen = Screen { components };
    screen.run();
}
```

<span class="caption">Liệt kê 17-11: Việc tách mảng components gây ra lỗi kiểu</span>

Việc cấu trúc lại này khiến chương trình không còn biên dịch được nữa! Trình biên dịch từ chối chương trình này với
lỗi sau:

```text
error[E0308]: mismatched types
   --> test.rs:55:14
    |
55  |       Box::new(Button {
    |  _____--------_^
    | |     |
    | |     arguments to this function are incorrect
56  | |       width: 50,
57  | |       height: 10,
58  | |       label: String::from("OK"),
59  | |     }),
    | |_____^ expected `SelectBox`, found `Button`
```

Trong Liệt kê 17-09, trình biên dịch hiểu rằng vector `components` phải có kiểu
`Vec<Box<dyn Draw>>` vì điều đó được chỉ định trong định nghĩa struct `Screen`. Nhưng trong Liệt kê 17-11,
trình biên dịch mất thông tin đó tại thời điểm `components` được định nghĩa. Để khắc phục vấn đề, bạn
phải đưa ra một gợi ý cho thuật toán suy luận kiểu. Điều đó có thể thông qua một phép ép kiểu (cast) rõ ràng trên
bất kỳ phần tử nào của vector, như thế này:

```rust,ignore
  let components = vec![
        Box::new(SelectBox { /* .. */ }) as Box<dyn Draw>,
        Box::new(Button { /* .. */ }),
  ];
```

Hoặc nó có thể thông qua một chú thích kiểu (type annotation) trên let-binding, như thế này:

```rust,ignore
  let components: Vec<Box<dyn Draw>> = vec![
        Box::new(SelectBox { /* .. */ }),
        Box::new(Button { /* .. */ }),
  ];
```

Nhìn chung, tốt hơn hết là bạn nên biết rằng việc sử dụng các trait object có thể gây ra trải nghiệm nhà phát triển kém hơn cho
các khách hàng sử dụng API trong trường hợp suy luận kiểu.

<!-- END INTERVENTION: cce62358-5291-4eb3-84d6-fbc570873ee3 -->

### Trait Object thực hiện Điều phối động

Hãy nhớ lại trong [“Hiệu năng của mã sử dụng Generics”][performance-of-code-using-generics]<!-- ignore --> ở Chương 10 cuộc thảo luận của chúng ta về quá trình đơn hình hóa (monomorphization) được thực hiện trên generics bởi
trình biên dịch: trình biên dịch tạo ra các triển khai không generic của các hàm và
phương thức cho mỗi kiểu cụ thể mà chúng ta sử dụng thay cho một tham số kiểu generic.
Mã kết quả từ quá trình đơn hình hóa đang thực hiện _điều phối tĩnh_ (static dispatch), đó là
khi trình biên dịch biết phương thức nào bạn đang gọi tại thời điểm biên dịch. Điều này
ngược lại với _điều phối động_ (dynamic dispatch), đó là khi trình biên dịch không thể biết tại
thời điểm biên dịch phương thức nào bạn đang gọi. Trong các trường hợp điều phối động, trình biên dịch phát ra
mã mà tại thời điểm chạy sẽ tìm ra phương thức nào cần gọi.

Khi chúng ta sử dụng các trait object, Rust phải sử dụng điều phối động. Trình biên dịch không
biết tất cả các kiểu có thể được sử dụng với mã đang sử dụng các trait object,
vì vậy nó không biết phương thức nào được triển khai trên kiểu nào để gọi. Thay vào đó, tại
thời điểm chạy, Rust sử dụng các con trỏ bên trong trait object để biết phương thức nào
cần gọi. Việc tra cứu này làm phát sinh chi phí thời gian chạy không xảy ra với điều phối tĩnh.
Điều phối động cũng ngăn cản trình biên dịch chọn nội tuyến (inline) mã của một phương thức,
điều này lần lượt ngăn cản một số tối ưu hóa, và Rust có một số quy tắc, được gọi là
_tương thích dyn_ (dyn compatibility), về nơi bạn có thể và không thể sử dụng điều phối động. Những
quy tắc đó nằm ngoài phạm vi của cuộc thảo luận này, nhưng bạn có thể đọc thêm về chúng
[trong tài liệu tham khảo][dyn-compatibility]. Tuy nhiên, chúng ta đã có thêm sự linh hoạt trong
mã mà chúng ta đã viết trong Liệt kê 18-5 và có thể hỗ trợ trong Liệt kê 18-9,
vì vậy đó là một sự đánh đổi cần cân nhắc.

{{#quiz ../quizzes/ch17-02-trait-objects.toml}}

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
