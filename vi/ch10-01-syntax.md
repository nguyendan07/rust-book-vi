## Các kiểu dữ liệu Generic

Chúng ta sử dụng generics để tạo ra các định nghĩa cho các mục như chữ ký hàm hoặc
struct, những thứ mà sau đó chúng ta có thể sử dụng với nhiều kiểu dữ liệu cụ thể khác nhau. Trước
tiên hãy xem cách định nghĩa các hàm, struct, enum và phương thức bằng cách
sử dụng generics. Sau đó, chúng ta sẽ thảo luận về cách generics ảnh hưởng đến hiệu suất mã nguồn.

### Trong các định nghĩa hàm

Khi định nghĩa một hàm sử dụng generics, chúng ta đặt các generics vào
chữ ký của hàm tại nơi mà chúng ta thường chỉ định kiểu dữ liệu của các
tham số và giá trị trả về. Làm như vậy giúp mã nguồn của chúng ta linh hoạt hơn và cung cấp
nhiều chức năng hơn cho những người gọi hàm của chúng ta trong khi ngăn chặn việc trùng lặp mã nguồn.

Tiếp tục với hàm `largest` của chúng ta, Liệt kê 10-4 hiển thị hai hàm mà
cả hai đều tìm giá trị lớn nhất trong một slice. Sau đó, chúng ta sẽ kết hợp chúng thành một
hàm duy nhất sử dụng generics.

<Listing number="10-4" file-name="src/main.rs" caption="Hai hàm chỉ khác nhau về tên và kiểu dữ liệu trong chữ ký của chúng">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

Hàm `largest_i32` là hàm chúng ta đã trích xuất trong Liệt kê 10-3 để tìm
số `i32` lớn nhất trong một slice. Hàm `largest_char` tìm
`char` lớn nhất trong một slice. Thân các hàm có cùng mã nguồn, vì vậy hãy loại bỏ
sự trùng lặp bằng cách giới thiệu một tham số kiểu generic trong một hàm duy nhất.

Để tham số hóa các kiểu trong một hàm duy nhất mới, chúng ta cần đặt tên cho
tham số kiểu, giống như cách chúng ta làm với các tham số giá trị cho một hàm. Bạn có thể sử dụng
bất kỳ tên định danh nào làm tên tham số kiểu. Nhưng chúng ta sẽ sử dụng `T` vì, theo
quy ước, tên tham số kiểu trong Rust thường ngắn, thường chỉ một chữ cái, và
quy ước đặt tên kiểu của Rust là CamelCase. Viết tắt của _type_, `T` là lựa chọn mặc định
của hầu hết các lập trình viên Rust.

Khi chúng ta sử dụng một tham số trong thân hàm, chúng ta phải khai báo
tên tham số đó trong chữ ký để trình biên dịch biết tên đó có ý nghĩa gì.
Tương tự, khi chúng ta sử dụng một tên tham số kiểu trong một chữ ký hàm, chúng ta
phải khai báo tên tham số kiểu đó trước khi sử dụng nó. Để định nghĩa hàm
`largest` generic, chúng ta đặt các khai báo tên kiểu bên trong dấu ngoặc nhọn,
`<>`, giữa tên của hàm và danh sách tham số, như thế này:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Chúng ta đọc định nghĩa này là: hàm `largest` là generic trên một kiểu
`T` nào đó. Hàm này có một tham số tên là `list`, đó là một slice của các giá trị
thuộc kiểu `T`. Hàm `largest` sẽ trả về một tham chiếu đến một giá trị có
cùng kiểu `T` đó.

Liệt kê 10-5 hiển thị định nghĩa hàm `largest` kết hợp sử dụng kiểu
dữ liệu generic trong chữ ký của nó. Liệt kê này cũng cho thấy cách chúng ta có thể gọi hàm
với một slice gồm các giá trị `i32` hoặc các giá trị `char`. Lưu ý rằng mã nguồn này sẽ
chưa thể biên dịch được.

<Listing number="10-5" file-name="src/main.rs" caption="Hàm `largest` sử dụng các tham số kiểu generic; mã này chưa thể biên dịch">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

Nếu chúng ta biên dịch mã này ngay bây giờ, chúng ta sẽ nhận được lỗi này:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

<!-- BEGIN INTERVENTION: 0aad53ff-89d7-4d14-8e3d-c17809220252 -->

Vấn đề ở trên là khi `largest` nhận một slice `&[T]` làm đầu vào, hàm này không thể giả định _bất cứ điều gì_ về kiểu `T`. Nó có thể là `i32`, nó có thể là `String`, nó có thể là [`File`](https://doc.rust-lang.org/std/fs/struct.File.html). Tuy nhiên, `largest` yêu cầu `T` phải là thứ mà bạn có thể so sánh bằng `>` (tức là `T` triển khai `PartialOrd`, một trait mà chúng ta sẽ thảo luận trong phần tiếp theo). Một số kiểu như `i32` và `String` có thể so sánh được, nhưng các kiểu khác như `File` thì không thể so sánh được.

Trong một ngôn ngữ như C++ với [templates](https://en.cppreference.com/w/cpp/language/templates), trình biên dịch sẽ không phàn nàn về việc triển khai `largest`, thay vào đó nó sẽ phàn nàn khi cố gắng gọi `largest` trên ví dụ như một slice file `&[File]`. Thay vào đó, Rust yêu cầu bạn tuyên bố các khả năng mong đợi của các kiểu generic ngay từ đầu. Nếu `T` cần phải so sánh được, thì `largest` phải nói rõ như vậy. Do đó, lỗi trình biên dịch này cho biết `largest` sẽ không được biên dịch cho đến khi `T` bị hạn chế.

Ngoài ra, không giống như các ngôn ngữ như Java nơi tất cả các đối tượng đều có một tập hợp các phương thức cốt lõi như [`Object.toString()`](<https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#toString()>), không có phương thức cốt lõi nào trong Rust. Không có các hạn chế, một kiểu generic `T` không có khả năng nào: nó không thể được in ra, nhân bản (clone), hoặc thay đổi (mutated) (mặc dù nó có thể bị hủy - dropped).

<!-- END INTERVENTION -->

### Trong các định nghĩa Struct

Chúng ta cũng có thể định nghĩa các struct để sử dụng một tham số kiểu generic trong một hoặc
nhiều trường bằng cách sử dụng cú pháp `<>`. Liệt kê 10-6 định nghĩa một struct `Point<T>` để giữ
các giá trị tọa độ `x` và `y` thuộc bất kỳ kiểu nào.

<Listing number="10-6" file-name="src/main.rs" caption="Một struct `Point<T>` giữ các giá trị `x` và `y` thuộc kiểu `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

Cú pháp để sử dụng generics trong các định nghĩa struct tương tự như cú pháp được sử dụng trong
các định nghĩa hàm. Đầu tiên chúng ta khai báo tên của tham số kiểu bên trong
dấu ngoặc nhọn ngay sau tên của struct. Sau đó, chúng ta sử dụng kiểu
generic trong định nghĩa struct tại nơi mà lẽ ra chúng ta sẽ chỉ định các kiểu dữ liệu
cụ thể.

Lưu ý rằng vì chúng ta chỉ sử dụng một kiểu generic để định nghĩa `Point<T>`,
định nghĩa này nói rằng struct `Point<T>` là generic trên một kiểu `T` nào đó, và
các trường `x` và `y` _cả hai_ đều thuộc cùng kiểu đó, bất kể kiểu đó là gì. Nếu
chúng ta tạo một phiên bản của một `Point<T>` có các giá trị thuộc các kiểu khác nhau, như trong
Liệt kê 10-7, mã nguồn của chúng ta sẽ không biên dịch được.

<Listing number="10-7" file-name="src/main.rs" caption="Các trường `x` và `y` phải cùng kiểu vì cả hai đều có cùng kiểu dữ liệu generic `T`.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

Trong ví dụ này, khi chúng ta gán giá trị số nguyên `5` cho `x`, chúng ta cho
trình biên dịch biết rằng kiểu generic `T` sẽ là một số nguyên cho instance này của
`Point<T>`. Sau đó, khi chúng ta chỉ định `4.0` cho `y`, thứ mà chúng ta đã định nghĩa là có
cùng kiểu với `x`, chúng ta sẽ nhận được một lỗi không khớp kiểu như thế này:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Để định nghĩa một struct `Point` mà `x` và `y` đều là generics nhưng có thể có
các kiểu khác nhau, chúng ta có thể sử dụng nhiều tham số kiểu generic. Ví dụ, trong
Liệt kê 10-8, chúng ta thay đổi định nghĩa của `Point` thành generic trên các kiểu `T`
và `U` trong đó `x` thuộc kiểu `T` và `y` thuộc kiểu `U`.

<Listing number="10-8" file-name="src/main.rs" caption="Một struct `Point<T, U>` generic trên hai kiểu để `x` và `y` có thể là các giá trị thuộc các kiểu khác nhau">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

Bây giờ tất cả các instance của `Point` được hiển thị đều được cho phép! Bạn có thể sử dụng bao nhiêu tham số
kiểu generic trong một định nghĩa tùy thích, nhưng sử dụng nhiều hơn một vài tham số sẽ làm cho
mã nguồn của bạn khó đọc. Nếu bạn thấy mình cần rất nhiều kiểu generic trong
mã nguồn của mình, điều đó có thể chỉ ra rằng mã nguồn của bạn cần được cấu trúc lại thành các phần
nhỏ hơn.

### Trong các định nghĩa Enum

Giống như chúng ta đã làm với các struct, chúng ta có thể định nghĩa các enum để giữ các kiểu dữ liệu generic trong các
biến thể của chúng. Hãy xem lại enum `Option<T>` mà thư viện
chuẩn cung cấp, thứ mà chúng ta đã sử dụng trong Chương 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Định nghĩa này bây giờ sẽ có ý nghĩa hơn đối với bạn. Như bạn có thể thấy,
enum `Option<T>` là generic trên kiểu `T` và có hai biến thể: `Some`,
giữ một giá trị thuộc kiểu `T`, và một biến thể `None` không giữ bất kỳ giá trị nào.
Bằng cách sử dụng enum `Option<T>`, chúng ta có thể diễn đạt khái niệm trừu tượng về một
giá trị tùy chọn, và vì `Option<T>` là generic, chúng ta có thể sử dụng sự trừu tượng này
bất kể kiểu của giá trị tùy chọn đó là gì.

Các enum cũng có thể sử dụng nhiều kiểu generic. Định nghĩa của enum `Result`
mà chúng ta đã sử dụng trong Chương 9 là một ví dụ:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Enum `Result` là generic trên hai kiểu, `T` và `E`, và có hai biến thể:
`Ok`, giữ một giá trị thuộc kiểu `T`, và `Err`, giữ một giá trị thuộc kiểu
`E`. Định nghĩa này giúp thuận tiện khi sử dụng enum `Result` ở bất cứ đâu chúng ta
có một thao tác có thể thành công (trả về một giá trị thuộc kiểu `T` nào đó) hoặc thất bại
(trả về một lỗi thuộc kiểu `E` nào đó). Thực tế, đây là những gì chúng ta đã sử dụng để mở một
tệp trong Liệt kê 9-3, trong đó `T` được điền vào với kiểu `std::fs::File` khi
tệp được mở thành công và `E` được điền vào với kiểu
`std::io::Error` khi có vấn đề khi mở tệp.

Khi bạn nhận thấy các tình huống trong mã nguồn của mình với nhiều định nghĩa struct hoặc enum
chỉ khác nhau ở kiểu của các giá trị mà chúng nắm giữ, bạn có thể
tránh sự trùng lặp bằng cách sử dụng các kiểu generic thay thế.

### Trong các định nghĩa phương thức

Chúng ta có thể triển khai các phương thức trên struct và enum (như chúng ta đã làm trong Chương 5) và sử dụng
các kiểu generic trong các định nghĩa của chúng nữa. Liệt kê 10-9 hiển thị struct `Point<T>`
chúng ta đã định nghĩa trong Liệt kê 10-6 với một phương thức tên là `x` được triển khai trên đó.

<Listing number="10-9" file-name="src/main.rs" caption="Triển khai một phương thức tên là `x` trên struct `Point<T>` sẽ trả về một tham chiếu đến trường `x` thuộc kiểu `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã định nghĩa một phương thức tên là `x` trên `Point<T>` để trả về một tham chiếu
đến dữ liệu trong trường `x`.

Lưu ý rằng chúng ta phải khai báo `T` ngay sau `impl` để chúng ta có thể sử dụng `T` nhằm chỉ định
rằng chúng ta đang triển khai các phương thức trên kiểu `Point<T>`. Bằng cách khai báo `T` như một
kiểu generic sau `impl`, Rust có thể xác định rằng kiểu trong dấu ngoặc
nhọn trong `Point` là một kiểu generic thay vì một kiểu cụ thể. Chúng ta
có thể đã chọn một tên khác cho tham số generic này so với tham số generic
đã khai báo trong định nghĩa struct, nhưng sử dụng cùng một tên là theo quy ước.
Nếu bạn viết một phương thức trong một `impl` có khai báo một kiểu generic,
phương thức đó sẽ được định nghĩa trên bất kỳ instance nào của kiểu đó, bất kể kiểu
cụ thể nào cuối cùng được thay thế cho kiểu generic.

Chúng ta cũng có thể chỉ định các ràng buộc trên các kiểu generic khi định nghĩa các phương thức trên
kiểu đó. Ví dụ, chúng ta có thể chỉ triển khai các phương thức trên các instance `Point<f32>`
thay vì trên các instance `Point<T>` với bất kỳ kiểu generic nào. Trong Liệt kê 10-10, chúng ta
sử dụng kiểu cụ thể `f32`, nghĩa là chúng ta không khai báo bất kỳ kiểu nào sau `impl`.

<Listing number="10-10" file-name="src/main.rs" caption="Một khối `impl` chỉ áp dụng cho một struct với một kiểu cụ thể cho tham số kiểu generic `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

Mã nguồn này có nghĩa là kiểu `Point<f32>` sẽ có một phương thức `distance_from_origin`;
các instance khác của `Point<T>` trong đó `T` không thuộc kiểu `f32` sẽ không
có phương thức này được định nghĩa. Phương thức này đo khoảng cách từ điểm của chúng ta đến
điểm tại tọa độ (0.0, 0.0) và sử dụng các phép toán toán học chỉ có
sẵn cho các kiểu số thực dấu phẩy động.

<!-- BEGIN INTERVENTION: 694bb2d0-f2e6-4b0b-a3e7-2d9f9e8b3d09 -->

Bạn không thể triển khai đồng thời các phương thức cụ thể _và_ generic có cùng tên theo cách này. Ví dụ, nếu bạn triển khai một `distance_from_origin` chung cho tất cả các kiểu `T` và một `distance_from_origin` cụ thể cho `f32`, thì trình biên dịch sẽ từ chối chương trình của bạn: Rust không biết nên sử dụng bản triển khai nào khi bạn gọi `Point<f32>::distance_from_origin`. Tổng quát hơn, Rust không có các cơ chế giống như kế thừa (inheritance) để chuyên biệt hóa các phương thức như bạn có thể thấy trong một ngôn ngữ hướng đối tượng, ngoại trừ một trường hợp (các phương thức trait mặc định) sẽ được thảo luận trong phần tiếp theo.

<!-- END INTERVENTION -->

Các tham số kiểu generic trong một định nghĩa struct không phải lúc nào cũng giống với những tham số
bạn sử dụng trong các chữ ký phương thức của cùng struct đó. Liệt kê 10-11 sử dụng các kiểu generic
`X1` và `Y1` cho struct `Point` và `X2` `Y2` cho chữ ký phương thức `mixup`
để làm cho ví dụ rõ ràng hơn. Phương thức này tạo ra một instance `Point` mới
với giá trị `x` từ `Point` `self` (thuộc kiểu `X1`) và giá trị `y` từ
`Point` được truyền vào (thuộc kiểu `Y2`).

<Listing number="10-11" file-name="src/main.rs" caption="Một phương thức sử dụng các kiểu generic khác với định nghĩa struct của nó">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

Trong `main`, chúng ta đã định nghĩa một `Point` có một `i32` cho `x` (với giá trị `5`)
và một `f64` for `y` (với giá trị `10.4`). Biến `p2` là một struct `Point`
có một string slice cho `x` (với giá trị `"Hello"`) và một `char` cho `y`
(với giá trị `c`). Việc gọi `mixup` trên `p1` với đối số `p2` cho chúng ta `p3`,
thứ sẽ có một `i32` cho `x` vì `x` đến từ `p1`. Biến `p3`
sẽ có một `char` cho `y` vì `y` đến từ `p2`. Lời gọi macro `println!`
sẽ in ra `p3.x = 5, p3.y = c`.

Mục đích của ví dụ này là để chứng minh một tình huống trong đó một số tham số generic
được khai báo với `impl` và một số được khai báo với định nghĩa
phương thức. Ở đây, các tham số generic `X1` và `Y1` được khai báo sau
`impl` vì chúng đi cùng với định nghĩa struct. Các tham số generic `X2`
và `Y2` được khai báo sau `fn mixup` vì chúng chỉ liên quan đến
phương thức.

### Hiệu suất của mã nguồn sử dụng Generics

Bạn có thể thắc mắc liệu có chi phí thời gian chạy (runtime cost) nào khi sử dụng các tham số kiểu
generic hay không. Tin tốt là việc sử dụng các kiểu generic sẽ không làm chương trình của bạn
chạy chậm hơn so với khi sử dụng các kiểu cụ thể.

Rust đạt được điều này bằng cách thực hiện monomorphization (đơn hình hóa) mã nguồn sử dụng
generics tại thời điểm biên dịch. _Monomorphization_ là quá trình chuyển đổi mã nguồn
generic thành mã nguồn cụ thể bằng cách điền vào các kiểu cụ thể được sử dụng khi
biên dịch. Trong quá trình này, trình biên dịch thực hiện ngược lại các bước chúng ta đã sử dụng
để tạo hàm generic trong Liệt kê 10-5: trình biên dịch xem xét tất cả các nơi
mà mã nguồn generic được gọi và tạo ra mã nguồn cho các kiểu cụ thể mà
mã nguồn generic được gọi với chúng.

Hãy xem cách điều này hoạt động bằng cách sử dụng enum generic `Option<T>`
của thư viện chuẩn:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Khi Rust biên dịch mã nguồn này, nó thực hiện monomorphization. Trong suốt quá trình
đó, trình biên dịch đọc các giá trị đã được sử dụng trong các instance `Option<T>`
và xác định hai loại `Option<T>`: một là `i32` và loại còn lại
là `f64`. Như vậy, nó mở rộng định nghĩa generic của `Option<T>` thành hai
định nghĩa được chuyên biệt hóa cho `i32` và `f64`, từ đó thay thế định nghĩa
generic bằng các định nghĩa cụ thể.

Phiên bản monomorphized của mã nguồn trông tương tự như sau (trình biên dịch
sử dụng các tên khác với những gì chúng ta đang sử dụng ở đây để minh họa):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

Enum generic `Option<T>` được thay thế bằng các định nghĩa cụ thể được tạo ra bởi
trình biên dịch. Bởi vì Rust biên dịch mã nguồn generic thành mã nguồn chỉ định
kiểu trong mỗi instance, chúng ta không phải trả chi phí thời gian chạy cho việc sử dụng generics. Khi mã nguồn
chạy, nó hoạt động giống như khi chúng ta sao chép từng định nghĩa bằng
tay. Quá trình monomorphization làm cho generics của Rust cực kỳ hiệu quả
tại thời điểm thực thi.

{{#quiz ../quizzes/ch10-01-generics.toml}}
