## Macros

Chúng ta đã sử dụng các macro như `println!` xuyên suốt cuốn sách này, nhưng chúng ta vẫn chưa khám phá đầy đủ
macro là gì và nó hoạt động như thế nào. Thuật ngữ _macro_ đề cập đến một nhóm
các tính năng trong Rust: macro _khai báo_ (declarative) với `macro_rules!` và ba loại
macro _thủ tục_ (procedural):

- Các macro `#[derive]` tùy chỉnh chỉ định mã được thêm vào với thuộc tính `derive`
  được sử dụng trên các struct và enum
- Các macro giống thuộc tính (Attribute-like) định nghĩa các thuộc tính tùy chỉnh có thể sử dụng trên bất kỳ mục nào
- Các macro giống hàm (Function-like) trông giống như các lời gọi hàm nhưng hoạt động trên các token
  được chỉ định làm đối số của chúng

Chúng ta sẽ lần lượt nói về từng loại này, nhưng trước tiên, hãy xem tại sao chúng ta thậm chí
cần các macro khi chúng ta đã có các hàm.

### Sự khác biệt giữa Macro và Hàm

Về cơ bản, macro là một cách viết mã nguồn để viết mã nguồn khác, cái mà
được gọi là _siêu lập trình_ (metaprogramming). Trong Phụ lục C, chúng ta thảo luận về thuộc tính `derive`,
thứ tạo ra một bản triển khai của các trait khác nhau cho bạn. Chúng ta
cũng đã sử dụng các macro `println!` và `vec!` xuyên suốt cuốn sách. Tất cả những
macro này _mở rộng_ (expand) để tạo ra nhiều mã nguồn hơn so với mã nguồn bạn đã viết thủ công.

Siêu lập trình hữu ích cho việc giảm lượng mã nguồn bạn phải viết và
bảo trì, vốn cũng là một trong những vai trò của các hàm. Tuy nhiên, các macro có
một số sức mạnh bổ sung mà các hàm không có.

Một chữ ký hàm phải khai báo số lượng và kiểu của các tham số mà
hàm có. Mặt khác, các macro có thể nhận một số lượng tham số thay đổi:
chúng ta có thể gọi `println!("hello")` với một đối số hoặc
`println!("hello {}", name)` với hai đối số. Ngoài ra, các macro được mở rộng
trước khi trình biên dịch diễn giải ý nghĩa của mã nguồn, vì vậy một macro có thể, ví dụ,
triển khai một trait cho một kiểu dữ liệu nhất định. Một hàm thì không thể, bởi vì nó được
gọi lúc runtime trong khi một trait cần được triển khai tại thời điểm biên dịch (compile time).

> [!NOTE]
> **Liên hệ với Python:**
> - Trong Python, bạn có thể dễ dàng viết một hàm nhận số lượng tham số tùy ý bằng cú pháp `*args, **kwargs`.
> - Trong Rust, hàm thông thường bắt buộc phải khai báo cố định số lượng và kiểu dữ liệu của từng tham số tại thời điểm biên dịch. Để tạo ra những cấu trúc nhận số lượng đối số linh hoạt (như `println!("a", "b", ...)` hay `vec![1, 2, 3]`), Rust bắt buộc phải sử dụng **Macro**.

Nhược điểm của việc triển khai một macro thay vì một hàm là các định nghĩa
macro phức tạp hơn các định nghĩa hàm bởi vì bạn đang viết mã Rust
để viết mã Rust. Do sự gián tiếp này, các định nghĩa macro nhìn chung
khó đọc, khó hiểu và khó bảo trì hơn các định nghĩa hàm.

Một sự khác biệt quan trọng khác giữa macro và hàm là bạn phải
định nghĩa các macro hoặc đưa chúng vào phạm vi (scope) _trước khi_ bạn gọi chúng trong một tệp,
trái ngược với các hàm mà bạn có thể định nghĩa ở bất cứ đâu và gọi ở bất cứ đâu.

### Macro khai báo với `macro_rules!` cho siêu lập trình tổng quát

Dạng macro được sử dụng rộng rãi nhất trong Rust là _macro khai báo_ (declarative macro). Những
macro này đôi khi cũng được gọi là “macros by example,” “`macro_rules!` macros,”
hoặc chỉ đơn giản là “macros.” Về cốt lõi, các macro khai báo cho phép bạn viết
một thứ gì đó tương tự như một biểu thức `match` của Rust. Như đã thảo luận trong Chương 6,
các biểu thức `match` là các cấu trúc điều khiển nhận một biểu thức, so sánh
giá trị kết quả của biểu thức với các mẫu (patterns), và sau đó chạy mã liên kết
với mẫu khớp. Các macro cũng so sánh một giá trị với các mẫu được liên kết
với mã cụ thể: trong tình huống này, giá trị là mã nguồn Rust theo nghĩa đen
được truyền cho macro; các mẫu được so sánh với cấu trúc của mã nguồn đó;
và mã liên kết với mỗi mẫu, khi khớp, sẽ thay thế mã được truyền cho macro.
Tất cả điều này xảy ra trong quá trình biên dịch.

Để định nghĩa một macro, bạn sử dụng cấu trúc `macro_rules!`. Hãy cùng khám phá cách
sử dụng `macro_rules!` bằng cách xem cách macro `vec!` được định nghĩa. Chương 8
đã đề cập đến cách chúng ta có thể sử dụng macro `vec!` để tạo một vector mới với các giá trị
cụ thể. Ví dụ, macro sau đây tạo một vector mới chứa ba
số nguyên:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Chúng ta cũng có thể sử dụng macro `vec!` để tạo một vector gồm hai số nguyên hoặc một vector
gồm năm string slice. Chúng ta sẽ không thể sử dụng một hàm để làm điều tương tự
bởi vì chúng ta sẽ không biết trước số lượng hoặc kiểu của các giá trị.

Danh sách 20-35 trình bày một định nghĩa hơi đơn giản hóa của macro `vec!`.

<Listing number="20-35" file-name="src/lib.rs" caption="Một phiên bản đơn giản hóa của định nghĩa macro `vec!` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> Lưu ý: Định nghĩa thực tế của macro `vec!` trong thư viện chuẩn
> bao gồm mã để phân bổ trước lượng bộ nhớ chính xác ngay từ đầu. Mã đó
> là một sự tối ưu hóa mà chúng ta không bao gồm ở đây, để làm cho ví dụ đơn giản hơn.

Chú thích `#[macro_export]` cho biết rằng macro này nên được cung cấp
bất cứ khi nào crate mà macro được định nghĩa được đưa vào
phạm vi. Nếu không có chú thích này, macro không thể được đưa vào phạm vi.

Sau đó, chúng ta bắt đầu định nghĩa macro với `macro_rules!` và tên của
macro chúng ta đang định nghĩa _không có_ dấu chấm than. Tên, trong trường hợp này
là `vec`, được theo sau bởi các dấu ngoặc nhọn biểu thị thân của định nghĩa macro.

Cấu trúc trong thân của `vec!` tương tự như cấu trúc của một biểu thức `match`.
Ở đây chúng ta có một nhánh với mẫu `( $( $x:expr ),* )`,
theo sau là `=>` và khối mã liên kết với mẫu này. Nếu mẫu
khớp, khối mã liên kết sẽ được phát ra. Vì đây là mẫu duy nhất
trong macro này, chỉ có một cách hợp lệ để khớp; bất kỳ mẫu nào khác
sẽ dẫn đến lỗi. Các macro phức tạp hơn sẽ có nhiều hơn một nhánh.

Cú pháp mẫu hợp lệ trong các định nghĩa macro khác với cú pháp mẫu
được đề cập trong Chương 19 bởi vì các mẫu macro được khớp với cấu trúc mã Rust
thay vì các giá trị. Hãy cùng xem qua ý nghĩa của các phần mẫu trong
Danh sách 20-35; để biết cú pháp mẫu macro đầy đủ, hãy xem [Tài liệu tham khảo
Rust][ref].

Đầu tiên chúng ta sử dụng một cặp dấu ngoặc đơn để bao quanh toàn bộ mẫu. Chúng ta sử dụng một
dấu đô la (`$`) để khai báo một biến trong hệ thống macro sẽ chứa
mã Rust khớp với mẫu. Dấu đô la làm rõ đây là một biến macro
trái ngược với một biến Rust thông thường. Tiếp theo là một cặp dấu ngoặc đơn
bắt giữ các giá trị khớp với mẫu bên trong dấu ngoặc đơn để sử dụng trong
mã thay thế. Bên trong `$()` là `$x:expr`, thứ khớp với bất kỳ biểu thức Rust nào
và đặt tên cho biểu thức đó là `$x`.

Dấu phẩy theo sau `$()` cho biết rằng một ký tự phân cách dấu phẩy theo nghĩa đen
phải xuất hiện giữa mỗi lần xuất hiện của đoạn mã khớp với mã bên trong
`$()`. Dấu `*` chỉ định rằng mẫu khớp với không hoặc nhiều hơn bất cứ thứ gì
đứng trước dấu `*`.

Khi chúng ta gọi macro này với `vec![1, 2, 3];`, mẫu `$x` khớp ba
lần với ba biểu thức `1`, `2`, và `3`.

Bây giờ hãy nhìn vào mẫu trong thân của mã liên kết với nhánh này:
`temp_vec.push()` bên trong `$()*` được tạo ra cho mỗi phần khớp với `$()`
trong mẫu không hoặc nhiều lần tùy thuộc vào số lần mẫu khớp.
`$x` được thay thế bằng mỗi biểu thức được khớp. Khi chúng ta gọi macro này
với `vec![1, 2, 3];`, mã được tạo ra thay thế cho lời gọi macro này
sẽ là như sau:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Chúng ta đã định nghĩa một macro có thể nhận bất kỳ số lượng đối số thuộc bất kỳ kiểu nào và có thể
tạo ra mã để tạo một vector chứa các phần tử được chỉ định.

Để tìm hiểu thêm về cách viết macro, hãy tham khảo tài liệu trực tuyến hoặc
các tài nguyên khác, chẳng hạn như [“The Little Book of Rust Macros”][tlborm] được bắt đầu bởi
Daniel Keep và được tiếp tục bởi Lukas Wirth.

### Macro thủ tục để tạo mã từ các thuộc tính

Dạng macro thứ hai là macro thủ tục (procedural macro), hoạt động giống như một
hàm (và là một loại thủ tục). _Macro thủ tục_ chấp nhận một số mã làm
đầu vào, hoạt động trên mã đó, và tạo ra một số mã làm đầu ra thay vì
khớp với các mẫu và thay thế mã bằng mã khác như các macro khai báo
vẫn làm. Ba loại macro thủ tục là `derive` tùy chỉnh,
giống thuộc tính (attribute-like), và giống hàm (function-like), và tất cả đều hoạt động theo cách tương tự.

Khi tạo các macro thủ tục, các định nghĩa phải nằm trong crate của riêng chúng
với một kiểu crate đặc biệt. Điều này là vì các lý do kỹ thuật phức tạp mà chúng tôi hy vọng
sẽ loại bỏ trong tương lai. Trong Danh sách 20-36, chúng tôi trình bày cách định nghĩa một
macro thủ tục, trong đó `some_attribute` là một trình giữ chỗ cho việc sử dụng một
loại macro cụ thể.

<Listing number="20-36" file-name="src/lib.rs" caption="Một ví dụ về việc định nghĩa một macro thủ tục">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

Hàm định nghĩa một macro thủ tục nhận một `TokenStream` làm đầu vào
và tạo ra một `TokenStream` làm đầu ra. Kiểu `TokenStream` được định nghĩa bởi
crate `proc_macro` được đi kèm với Rust và đại diện cho một chuỗi các
token. Đây là cốt lõi của macro: mã nguồn mà macro đang hoạt động
tạo nên đầu vào `TokenStream`, và mã mà macro tạo ra là
đầu ra `TokenStream`. Hàm này cũng có một thuộc tính đi kèm với nó
chỉ định loại macro thủ tục nào chúng ta đang tạo. Chúng ta có thể có
nhiều loại macro thủ tục trong cùng một crate.

Hãy xem xét các loại macro thủ tục khác nhau. Chúng ta sẽ bắt đầu với một
macro `derive` tùy chỉnh và sau đó giải thích các khác biệt nhỏ tạo nên
các dạng khác.

### Cách viết một macro `derive` tùy chỉnh

Hãy tạo một crate tên là `hello_macro` định nghĩa một trait tên là
`HelloMacro` với một hàm liên kết tên là `hello_macro`. Thay vì
bắt người dùng của chúng ta phải tự tay triển khai trait `HelloMacro` cho từng kiểu dữ liệu của họ,
chúng ta sẽ cung cấp một macro thủ tục để người dùng có thể chú thích kiểu của họ với
`#[derive(HelloMacro)]` nhằm có được một bản triển khai mặc định của hàm
`hello_macro`. Bản triển khai mặc định sẽ in ra `Hello, Macro! My name is
TypeName!` trong đó `TypeName` là tên của kiểu mà trait này đã
được triển khai cho kiểu đó. Nói cách khác, chúng ta sẽ viết một crate cho phép một
lập trình viên khác viết mã như Danh sách 20-37 bằng cách sử dụng crate của chúng ta.

<Listing number="20-37" file-name="src/main.rs" caption="Mã nguồn mà người dùng crate của chúng ta sẽ có thể viết khi sử dụng macro thủ tục của chúng ta">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

Mã này sẽ in ra `Hello, Macro! My name is Pancakes!` khi chúng ta hoàn tất.
Bước đầu tiên là tạo một library crate mới, như thế này:

```console
$ cargo new hello_macro --lib
```

Tiếp theo, chúng ta sẽ định nghĩa trait `HelloMacro` và hàm liên kết của nó:

<Listing file-name="src/lib.rs" number="20-38" caption="Một trait đơn giản mà chúng ta sẽ sử dụng với macro `derive` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

Chúng ta có một trait và hàm của nó. Tại thời điểm này, người dùng crate của chúng ta có thể triển khai
trait để đạt được chức năng mong muốn, như trong Danh sách 20-39.

<Listing number="20-39" file-name="src/main.rs" caption="Nó sẽ trông như thế nào nếu người dùng viết một triển khai thủ công cho trait `HelloMacro` ">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

Tuy nhiên, họ sẽ cần viết khối triển khai cho từng kiểu mà họ
muốn sử dụng với `hello_macro`; chúng ta muốn giúp họ không phải làm
công việc này.

Ngoài ra, chúng ta chưa thể cung cấp cho hàm `hello_macro` một bản triển khai mặc định
để in ra tên của kiểu mà trait được triển khai cho kiểu đó:
Rust không có cơ chế phản chiếu (reflection), vì vậy nó không thể tra cứu tên của kiểu
lúc runtime. Chúng ta cần một macro để tạo mã tại thời điểm biên dịch.

> [!NOTE]
> **Khác biệt với Python:** Python là ngôn ngữ thông dịch động, hỗ trợ Reflection mạnh mẽ (bạn có thể lấy tên class qua `obj.__class__.__name__` hoặc can thiệp lúc tạo class bằng Metaclass). Rust không có Reflection lúc runtime để đảm bảo hiệu năng tối đa và an toàn bộ nhớ. Thay vào đó, Rust giải quyết bài toán này bằng **Procedural Macro**: phân tích cây cú pháp (AST) của code và tự động sinh thêm mã nguồn mới ngay khi biên dịch.

Bước tiếp theo là định nghĩa macro thủ tục. Tại thời điểm viết bài này,
các macro thủ tục cần phải nằm trong crate của riêng chúng. Cuối cùng, hạn chế này
có thể được dỡ bỏ. Quy ước cấu trúc các crate và crate macro như
sau: đối với một crate tên là `foo`, một crate macro thủ tục `derive` tùy chỉnh
được gọi là `foo_derive`. Hãy bắt đầu một crate mới tên là `hello_macro_derive` bên trong
dự án `hello_macro` của chúng ta:

```console
$ cargo new hello_macro_derive --lib
```

Hai crate của chúng ta có liên quan chặt chẽ với nhau, vì vậy chúng ta tạo crate macro thủ tục
bên trong thư mục của crate `hello_macro`. Nếu chúng ta thay đổi định nghĩa
trait trong `hello_macro`, chúng ta sẽ phải thay đổi việc triển khai
macro thủ tục trong `hello_macro_derive`. Hai crate này sẽ cần
được xuất bản riêng biệt, và các lập trình viên sử dụng các crate này sẽ cần thêm
cả hai làm phụ thuộc (dependencies) và đưa cả hai vào phạm vi. Thay vào đó, chúng ta có thể cho
crate `hello_macro` sử dụng `hello_macro_derive` như một phụ thuộc và xuất lại (re-export)
mã macro thủ tục. Tuy nhiên, cách chúng ta cấu trúc dự án giúp
các lập trình viên có thể sử dụng `hello_macro` ngay cả khi họ không muốn chức năng
`derive`.

Chúng ta cần khai báo crate `hello_macro_derive` là một crate macro thủ tục.
Chúng ta cũng sẽ cần các chức năng từ các crate `syn` và `quote`, như bạn sẽ thấy
trong giây lát, vì vậy chúng ta cần thêm chúng làm phụ thuộc. Thêm đoạn sau vào
tệp _Cargo.toml_ cho `hello_macro_derive`:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Để bắt đầu định nghĩa macro thủ tục, hãy đặt mã trong Danh sách 20-40 vào
tệp _src/lib.rs_ cho crate `hello_macro_derive`. Lưu ý rằng mã này
sẽ không biên dịch cho đến khi chúng ta thêm định nghĩa cho hàm `impl_hello_macro`.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="Mã nguồn mà hầu hết các crate macro thủ tục sẽ yêu cầu để xử lý mã Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Lưu ý rằng chúng ta đã chia mã thành hàm `hello_macro_derive`, hàm này
chịu trách nhiệm phân tích cú pháp `TokenStream`, và hàm `impl_hello_macro`,
hàm này chịu trách nhiệm chuyển đổi cây cú pháp: điều này làm cho việc
viết một macro thủ tục thuận tiện hơn. Mã trong hàm bên ngoài
(`hello_macro_derive` trong trường hợp này) sẽ giống nhau đối với hầu hết mọi
crate macro thủ tục mà bạn thấy hoặc tạo ra. Mã bạn chỉ định trong thân của
hàm bên trong (`impl_hello_macro` trong trường hợp này) sẽ khác nhau
tùy thuộc vào mục đích của macro thủ tục của bạn.

Chúng ta đã giới thiệu ba crate mới: `proc_macro`, [`syn`], và [`quote`]. Crate
`proc_macro` đi kèm với Rust, vì vậy chúng ta không cần thêm nó vào các
phụ thuộc trong _Cargo.toml_. Crate `proc_macro` là API của trình biên dịch
cho phép chúng ta đọc và thao tác mã Rust từ mã của chúng ta.

Crate `syn` phân tích cú pháp mã Rust từ một chuỗi thành một cấu trúc dữ liệu mà chúng ta
có thể thực hiện các thao tác trên đó. Crate `quote` chuyển các cấu trúc dữ liệu `syn` quay trở lại
thành mã Rust. Các crate này giúp việc phân tích bất kỳ loại mã Rust nào
chúng ta muốn xử lý trở nên đơn giản hơn nhiều: viết một trình phân tích cú pháp đầy đủ cho mã Rust không phải là
một nhiệm vụ đơn giản.

Hàm `hello_macro_derive` sẽ được gọi khi một người dùng thư viện của chúng ta
chỉ định `#[derive(HelloMacro)]` trên một kiểu. Điều này khả thi vì chúng ta đã
chú thích hàm `hello_macro_derive` ở đây với `proc_macro_derive` và
chỉ định tên `HelloMacro`, khớp với tên trait của chúng ta; đây là
quy ước mà hầu hết các macro thủ tục tuân theo.

Hàm `hello_macro_derive` trước tiên chuyển đổi `input` từ một
`TokenStream` sang một cấu trúc dữ liệu mà sau đó chúng ta có thể diễn giải và thực hiện
các thao tác trên đó. Đây là nơi `syn` phát huy tác dụng. Hàm `parse` trong
`syn` nhận một `TokenStream` và trả về một struct `DeriveInput` đại diện cho
mã Rust đã được phân tích cú pháp. Danh sách 20-41 trình bày các phần liên quan của struct
`DeriveInput` mà chúng ta nhận được từ việc phân tích cú pháp chuỗi `struct Pancakes;`.

<Listing number="20-41" caption="Thể hiện `DeriveInput` chúng ta nhận được khi phân tích cú pháp mã có thuộc tính của macro trong Danh sách 20-37">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Các trường của struct này cho thấy mã Rust chúng ta đã phân tích cú pháp là một unit struct
bằng `ident` (_identifier_, nghĩa là tên) là `Pancakes`. Có nhiều
trường hơn trên struct này để mô tả tất cả các loại mã Rust; hãy kiểm tra [tài liệu
`syn` cho `DeriveInput`][syn-docs] để biết thêm thông tin.

Sớm thôi chúng ta sẽ định nghĩa hàm `impl_hello_macro`, đây là nơi chúng ta sẽ xây dựng
mã Rust mới mà chúng ta muốn bao gồm. Nhưng trước khi làm vậy, hãy lưu ý rằng đầu ra
cho macro `derive` của chúng ta cũng là một `TokenStream`. `TokenStream` được trả về
được thêm vào mã mà người dùng crate của chúng ta viết, vì vậy khi họ biên dịch crate của mình,
họ sẽ nhận được các chức năng bổ sung mà chúng ta cung cấp trong
`TokenStream` đã được sửa đổi.

Bạn có thể đã nhận thấy rằng chúng ta đang gọi `unwrap` để khiến hàm
`hello_macro_derive` panic nếu lời gọi hàm `syn::parse`
thất bại ở đây. Việc macro thủ tục của chúng ta panic khi có lỗi là cần thiết vì
các hàm `proc_macro_derive` phải trả về `TokenStream` thay vì `Result` để
tuân thủ API macro thủ tục. Chúng ta đã đơn giản hóa ví dụ này bằng cách sử dụng
`unwrap`; trong mã sản xuất (production), bạn nên cung cấp các thông báo lỗi cụ thể hơn
về những gì đã xảy ra bằng cách sử dụng `panic!` hoặc `expect`.

Bây giờ chúng ta đã có mã để chuyển mã Rust được chú thích từ một `TokenStream`
thành một thể hiện `DeriveInput`, hãy tạo mã triển khai
trait `HelloMacro` cho kiểu được chú thích, như được trình bày trong Danh sách 20-42.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="Triển khai trait `HelloMacro` bằng mã Rust đã được phân tích cú pháp">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

Chúng ta nhận được một thể hiện struct `Ident` chứa tên (định danh) của
kiểu được chú thích bằng cách sử dụng `ast.ident`. Struct trong Danh sách 20-41 cho thấy khi
chúng ta chạy hàm `impl_hello_macro` trên mã trong Danh sách 20-37,
`ident` chúng ta nhận được sẽ có trường `ident` với giá trị là `"Pancakes"`. Do đó,
biến `name` trong Danh sách 20-42 sẽ chứa một thể hiện struct `Ident`
mà khi in ra sẽ là chuỗi `"Pancakes"`, tên của struct trong
Danh sách 20-37.

Macro `quote!` cho phép chúng ta định nghĩa mã Rust mà chúng ta muốn trả về.
Trình biên dịch mong đợi một thứ gì đó khác với kết quả trực tiếp của việc thực thi macro
`quote!`, vì vậy chúng ta cần chuyển đổi nó thành một `TokenStream`. Chúng ta thực hiện việc này bằng cách
gọi phương thức `into`, phương thức này tiêu thụ biểu diễn trung gian này và
trả về một giá trị thuộc kiểu `TokenStream` được yêu cầu.

Macro `quote!` cũng cung cấp một số cơ chế tạo mẫu (templating) rất thú vị: chúng ta có thể
nhập `#name`, và `quote!` sẽ thay thế nó bằng giá trị trong biến
`name`. Bạn thậm chí có thể thực hiện một số lặp lại tương tự như cách các macro thông thường hoạt động.
Hãy xem [tài liệu của crate `quote`][quote-docs] để biết phần giới thiệu kỹ lưỡng.

Chúng ta muốn macro thủ tục của mình tạo ra một bản triển khai của trait `HelloMacro`
cho kiểu mà người dùng đã chú thích, cái mà chúng ta có thể lấy được bằng cách sử dụng `#name`.
Bản triển khai trait có một hàm `hello_macro`, thân hàm chứa
chức năng chúng ta muốn cung cấp: in ra `Hello, Macro! My name is` và sau đó là
tên của kiểu được chú thích.

Macro `stringify!` được sử dụng ở đây được tích hợp sẵn trong Rust. Nó nhận một biểu thức
Rust, chẳng hạn như `1 + 2`, và tại thời điểm biên dịch chuyển biểu thức đó thành một
chuỗi ký tự (string literal), chẳng hạn như `"1 + 2"`. Điều này khác với `format!` hoặc
`println!`, các macro đánh giá biểu thức và sau đó chuyển kết quả thành một
`String`. Có khả năng đầu vào `#name` có thể là một
biểu thức để in theo nghĩa đen, vì vậy chúng ta sử dụng `stringify!`. Việc sử dụng `stringify!` cũng
tiết kiệm một lần cấp phát bằng cách chuyển đổi `#name` thành một chuỗi ký tự tại thời điểm biên dịch.

Tại thời điểm này, `cargo build` sẽ hoàn tất thành công trong cả `hello_macro`
và `hello_macro_derive`. Hãy kết nối các crate này với mã trong Danh sách
20-37 để xem macro thủ tục hoạt động! Tạo một dự án binary mới trong
thư mục _projects_ của bạn bằng cách sử dụng `cargo new pancakes`. Chúng ta cần thêm
`hello_macro` và `hello_macro_derive` làm phụ thuộc trong tệp _Cargo.toml_ của crate
`pancakes`. Nếu bạn đang xuất bản các phiên bản `hello_macro` và
`hello_macro_derive` của mình lên [crates.io](https://crates.io/), chúng sẽ là các phụ thuộc
thông thường; nếu không, bạn có thể chỉ định chúng là các phụ thuộc `path` như sau:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:7:9}}
```

Đặt mã trong Danh sách 20-37 vào _src/main.rs_, và chạy `cargo run`: nó
sẽ in ra `Hello, Macro! My name is Pancakes!`. Bản triển khai trait
`HelloMacro` từ macro thủ tục đã được tự động thêm vào mà không cần crate
`pancakes` phải tự tay triển khai nó; `#[derive(HelloMacro)]` đã tự động bổ sung
bản triển khai của trait.

Tiếp theo, hãy khám phá xem các loại macro thủ tục khác khác gì so với các macro
`derive` tùy chỉnh.

### Macro giống thuộc tính (Attribute-Like macros)

Các macro giống thuộc tính tương tự như các macro `derive` tùy chỉnh, nhưng thay vì
tạo mã cho thuộc tính `derive`, chúng cho phép bạn tạo các
thuộc tính mới. Chúng cũng linh hoạt hơn: `derive` chỉ hoạt động cho các struct và
enum; các thuộc tính cũng có thể được áp dụng cho các mục khác, chẳng hạn như các hàm.
Đây là một ví dụ về việc sử dụng một macro giống thuộc tính. Giả sử bạn có một thuộc tính
tên là `route` chú thích các hàm khi sử dụng một khung ứng dụng web (web application framework):

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Thuộc tính `#[route]` này sẽ được định nghĩa bởi khung ứng dụng dưới dạng một macro thủ tục.
Chữ ký của hàm định nghĩa macro sẽ trông như thế này:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Ở đây, chúng ta có hai tham số kiểu `TokenStream`. Tham số thứ nhất dành cho
nội dung của thuộc tính: phần `GET, "/"`. Tham số thứ hai là thân của
mục mà thuộc tính được đính kèm: trong trường hợp này là `fn index() {}` và phần còn lại
của thân hàm.

Ngoài ra, các macro giống thuộc tính hoạt động theo cùng cách với các macro `derive`
tùy chỉnh: bạn tạo một crate với kiểu crate `proc-macro` và triển khai một
hàm tạo ra mã bạn muốn!

> [!NOTE]
> **Liên hệ với Python:** Cú pháp Attribute-like Macro trong Rust (`#[route(GET, "/")]`) trông rất giống với **Decorator** trong Python (`@app.route('/')`). Tuy nhiên có sự khác biệt bản chất:
> - **Python Decorator:** Thực thi lúc runtime, nhận vào một đối tượng hàm và bọc nó bằng một hàm khác.
> - **Rust Attribute Macro:** Thực thi lúc compile time, nhận vào toàn bộ chuỗi token mã nguồn của hàm, cho phép bạn phân tích cú pháp, sửa đổi hoặc sinh ra mã máy mới hoàn toàn trước khi chương trình chạy.

### Macro giống hàm (Function-Like macros)

Các macro giống hàm định nghĩa các macro trông giống như các lời gọi hàm. Tương tự như
các macro `macro_rules!`, chúng linh hoạt hơn các hàm; ví dụ, chúng
có thể nhận một số lượng đối số không xác định. Tuy nhiên, các macro `macro_rules!` chỉ có thể
được định nghĩa bằng cú pháp giống match mà chúng ta đã thảo luận trong [“Macro khai báo với
`macro_rules!` cho siêu lập trình tổng quát”][decl]<!-- ignore --> ở trên.
Các macro giống hàm nhận một tham số `TokenStream` và định nghĩa của chúng
thao tác trên `TokenStream` đó bằng mã Rust như hai loại macro thủ tục kia vẫn làm.
Một ví dụ về macro giống hàm là macro `sql!` có thể được
gọi như sau:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Macro này sẽ phân tích câu lệnh SQL bên trong nó và kiểm tra xem nó có
chính xác về mặt cú pháp hay không, đây là quá trình xử lý phức tạp hơn nhiều so với những gì một
macro `macro_rules!` có thể làm. Macro `sql!` sẽ được định nghĩa như thế này:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Định nghĩa này tương tự như chữ ký của macro `derive` tùy chỉnh: chúng ta nhận
các token nằm bên trong dấu ngoặc đơn và trả về mã chúng ta muốn tạo ra.

{{#quiz ../quizzes/ch19-06-macros.toml}}

## Tổng kết

Phù! Giờ đây bạn đã có một số tính năng Rust trong hộp công cụ của mình mà có lẽ bạn sẽ không sử dụng
thường xuyên, nhưng bạn sẽ biết chúng có sẵn trong những trường hợp rất cụ thể.
Chúng tôi đã giới thiệu một số chủ đề phức tạp để khi bạn bắt gặp chúng trong
các gợi ý thông báo lỗi hoặc trong mã của người khác, bạn sẽ có thể
nhận ra các khái niệm và cú pháp này. Hãy sử dụng chương này như một tài liệu tham khảo để hướng dẫn
bạn đến các giải pháp.

Tiếp theo, chúng ta sẽ đưa mọi thứ chúng ta đã thảo luận xuyên suốt cuốn sách vào thực tế
và thực hiện thêm một dự án nữa!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[`syn`]: https://crates.io/crates/syn
[`quote`]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
