## Các hàm và Closure nâng cao

Phần này khám phá một số tính năng nâng cao liên quan đến các hàm và closure,
bao gồm các con trỏ hàm và việc trả về các closure.

### Con trỏ hàm

Chúng ta đã nói về cách truyền các closure vào các hàm; bạn cũng có thể truyền các hàm
thông thường vào các hàm! Kỹ thuật này hữu ích khi bạn muốn truyền một
hàm mà bạn đã định nghĩa thay vì định nghĩa một closure mới. Các hàm
ép kiểu (coerce) sang kiểu `fn` (với chữ _f_ viết thường), không được nhầm lẫn với trait
closure `Fn`. Kiểu `fn` được gọi là một _con trỏ hàm_ (function pointer). Việc truyền các hàm
bằng con trỏ hàm sẽ cho phép bạn sử dụng các hàm như là các đối số cho các hàm
khác.

Cú pháp để chỉ định rằng một tham số là một con trỏ hàm tương tự như
cú pháp của các closure, như được trình bày trong Danh sách 20-28, nơi chúng ta đã định nghĩa một hàm
`add_one` cộng thêm 1 vào tham số của nó. Hàm `do_twice` nhận hai
tham số: một con trỏ hàm trỏ đến bất kỳ hàm nào nhận một tham số `i32`
và trả về một `i32`, và một giá trị `i32`. Hàm `do_twice` gọi
hàm `f` hai lần, truyền cho nó giá trị `arg`, sau đó cộng kết quả của hai lần gọi hàm
lại với nhau. Hàm `main` gọi `do_twice` với các đối số là
`add_one` và `5`.

<Listing number="20-28" file-name="src/main.rs" caption="Sử dụng kiểu `fn` để chấp nhận một con trỏ hàm làm đối số">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

Mã này in ra `The answer is: 12`. Chúng ta chỉ định rằng tham số `f` trong
`do_twice` là một `fn` nhận một tham số kiểu `i32` và trả về một
`i32`. Sau đó chúng ta có thể gọi `f` trong thân của `do_twice`. Trong `main`, chúng ta có thể truyền
tên hàm `add_one` làm đối số đầu tiên cho `do_twice`.

Không giống như các closure, `fn` là một kiểu thay vì là một trait, vì vậy chúng ta chỉ định `fn` làm
kiểu tham số trực tiếp thay vì khai báo một tham số kiểu generic với một
trong các trait `Fn` làm trait bound.

Con trỏ hàm tự động triển khai cả 3 trait closure (`Fn`, `FnMut`, và
`FnOnce`), nghĩa là bạn luôn có thể truyền một con trỏ hàm như một đối số cho một
hàm mong đợi một closure. Tốt nhất là nên viết các hàm sử dụng một kiểu generic
và một trong các trait closure để các hàm của bạn có thể chấp nhận linh hoạt cả
hàm thông thường lẫn closure.

Tuy nhiên, một ví dụ về nơi mà bạn chỉ muốn chấp nhận `fn` mà không phải
closure là khi giao tiếp với mã bên ngoài không có closure: các
hàm C có thể chấp nhận các hàm làm đối số, nhưng C không có closure.

> [!NOTE]
> **Liên hệ với Python:**
> - Trong Python, mọi hàm (`def foo():`) hay `lambda` đều là First-class Object và có thể truyền qua lại tự do.
> - Trong Rust có sự phân biệt rõ ràng:
>   - `fn` (chữ *f* thường): là **con trỏ hàm (function pointer)** trỏ trực tiếp đến địa chỉ mã máy của hàm, kích thước cố định bằng 1 con trỏ bộ nhớ, và **không thể nắm giữ (capture)** các biến từ môi trường bên ngoài.
>   - `Closure` (các trait `Fn`, `FnMut`, `FnOnce`): có thể **capture** các biến xung quanh (giống `lambda` hay hàm lồng nhau trong Python). Dưới nền, Rust sẽ tự động tạo một struct ẩn để lưu các biến được capture này.

Ví dụ về nơi bạn có thể sử dụng một closure được định nghĩa inline hoặc một hàm
được đặt tên, hãy xem xét việc sử dụng phương thức `map` được cung cấp bởi trait `Iterator`
trong thư viện chuẩn. Để sử dụng phương thức `map` nhằm chuyển một vector các
số thành một vector các chuỗi, chúng ta có thể sử dụng một closure, như trong Danh sách 20-29.

<Listing number="20-29" caption="Sử dụng một closure với phương thức `map` để chuyển đổi các số thành chuỗi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

Hoặc chúng ta có thể đặt tên cho một hàm làm đối số cho map thay vì closure.
Danh sách 20-30 cho thấy điều này sẽ trông như thế nào.

<Listing number="20-30" caption="Sử dụng hàm `String::to_string` với phương thức `map` để chuyển đổi các số thành chuỗi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta phải sử dụng cú pháp định danh đầy đủ (fully qualified syntax) mà chúng ta đã nói đến trong
[“Các Trait nâng cao”][advanced-traits]<!-- ignore --> bởi vì có nhiều
hàm có sẵn tên là `to_string`.

Ở đây, chúng ta đang sử dụng hàm `to_string` được định nghĩa trong trait `ToString`,
mà thư viện chuẩn đã triển khai sẵn cho bất kỳ kiểu nào triển khai
`Display`.

Hãy nhớ lại từ [“Các giá trị Enum”][enum-values]<!-- ignore --> ở Chương 6 rằng
tên của mỗi biến thể enum mà chúng ta định nghĩa cũng trở thành một hàm khởi tạo.
Chúng ta có thể sử dụng các hàm khởi tạo này như các con trỏ hàm triển khai các
trait closure, điều đó có nghĩa là chúng ta có thể chỉ định các hàm khởi tạo làm
đối số cho các phương thức nhận closure, như thấy trong Danh sách 20-31.

<Listing number="20-31" caption="Sử dụng các hàm khởi tạo enum với phương thức `map` để tạo một thể hiện `Status` từ các số">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

Ở đây chúng ta tạo các thể hiện `Status::Value` bằng cách sử dụng mỗi giá trị `u32` trong phạm vi
mà `map` được gọi bằng cách sử dụng hàm khởi tạo của `Status::Value`.
Một số người thích phong cách này và một số người thích sử dụng closure. Chúng
được biên dịch thành cùng một mã nguồn, vì vậy hãy sử dụng bất kỳ phong cách nào rõ ràng hơn đối với bạn.

> [!NOTE]
> **Liên hệ với Python:** Trong Python, tên của một class chính là một callable khởi tạo instance, ví dụ `map(int, ["1", "2"])` hoặc `map(MyClass, items)`. Trong Rust, mỗi enum variant có chứa dữ liệu dạng tuple (như `Status::Value(u32)`) cũng tự động đóng vai trò như một hàm khởi tạo nhận tham số, do đó bạn có thể truyền thẳng `Status::Value` vào `map(Status::Value)` mà không cần viết closure bọc ngoài như `|v| Status::Value(v)`.

### Trả về Closure

Các closure được đại diện bởi các trait, điều đó có nghĩa là bạn không thể trả về các closure
trực tiếp. Trong hầu hết các trường hợp bạn muốn trả về một trait, thay vào đó bạn có thể
sử dụng một kiểu dữ liệu cụ thể có triển khai trait đó làm giá trị trả về của
hàm. Tuy nhiên, bạn thường không thể làm điều đó với các closure vì chúng không
có một kiểu cụ thể có thể trả về được. Ví dụ, bạn không được phép sử dụng
con trỏ hàm `fn` làm kiểu trả về nếu closure nắm giữ (capture) bất kỳ giá trị nào từ phạm vi của nó.

Thay vào đó, thông thường bạn sẽ sử dụng cú pháp `impl Trait` mà chúng ta đã học ở
Chương 10. Bạn có thể trả về bất kỳ kiểu hàm nào, bằng cách sử dụng `Fn`, `FnOnce` và `FnMut`.
Ví dụ, mã trong Danh sách 20-32 sẽ hoạt động tốt.

<Listing number="20-32" caption="Trả về một closure từ một hàm bằng cách sử dụng cú pháp `impl Trait` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

Tuy nhiên, như chúng ta đã lưu ý trong [“Suy luận kiểu và chú thích closure”][closure-types]<!-- ignore --> ở Chương 13, mỗi closure cũng là
một kiểu riêng biệt của chính nó. Nếu bạn cần làm việc với nhiều hàm có
cùng chữ ký nhưng các cách triển khai khác nhau, bạn sẽ cần sử dụng một đối tượng trait
cho chúng. Hãy xem xét điều gì xảy ra nếu bạn viết mã như trong
Danh sách 20-33.

<Listing file-name="src/main.rs" number="20-33" caption="Tạo một `Vec<T>` gồm các closure được định nghĩa bởi các hàm trả về kiểu `impl Fn` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

Ở đây chúng ta có hai hàm, `returns_closure` và `returns_initialized_closure`,
cả hai đều trả về `impl Fn(i32) -> i32`. Lưu ý rằng các closure mà chúng
trả về là khác nhau, mặc dù chúng triển khai cùng một trait. Nếu chúng ta cố gắng
biên dịch mã này, Rust cho chúng ta biết rằng nó sẽ không hoạt động:

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

Thông báo lỗi cho chúng ta biết rằng bất cứ khi nào chúng ta trả về một `impl Trait`, Rust sẽ tạo ra một
_kiểu ẩn/kiểu mờ_ (opaque type) duy nhất — một kiểu dữ liệu nội bộ mà chúng ta không thể nhìn thấy chi tiết cấu trúc bên trong, cũng không thể đoán trước tên kiểu do Rust tự sinh ra để tự khai báo bằng tay. Vì vậy, mặc dù cả hai hàm này đều trả về các closure
triển khai cùng một trait, `Fn(i32) -> i32`, các kiểu mờ mà Rust tạo ra cho
mỗi cái là khác biệt. (Điều này tương tự như cách Rust tạo ra các kiểu cụ thể khác nhau
cho các khối async riêng biệt ngay cả khi chúng có cùng kiểu đầu ra, như chúng ta
đã thấy trong [“The `Pin` Type and the `Unpin` Trait”][future-types]<!-- ignore --> ở Chương 17.) Chúng ta đã thấy một giải pháp cho vấn đề này một vài lần: chúng ta có thể sử dụng một đối tượng
trait, như trong Danh sách 20-34.

<Listing number="20-34" caption="Tạo một `Vec<T>` gồm các closure được định nghĩa bởi các hàm trả về `Box<dyn Fn>` để chúng có cùng kiểu">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

Mã này sẽ biên dịch hoàn toàn ổn. Để biết thêm về các đối tượng trait, hãy tham khảo
phần [“Sử dụng các đối tượng Trait cho phép các giá trị thuộc các kiểu khác nhau”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore
--> ở Chương 18.

> [!NOTE]
> **Khác biệt với Python:** Trong Python, bạn có thể tạo một danh sách chứa nhiều hàm/closure khác nhau một cách tự do: `funcs = [lambda x: x+1, lambda x: x*2]`. Trong Rust, do mỗi closure được gán một kiểu dữ liệu ẩn (opaque type) riêng biệt tại thời điểm biên dịch, bạn không thể gom chúng vào cùng một `Vec<impl Fn(i32) -> i32>`. Để giải quyết, bạn phải bọc chúng trong con trỏ Trait Object: `Vec<Box<dyn Fn(i32) -> i32>>` (lưu trữ trên Heap và phân phối lời gọi động lúc chạy).

Tiếp theo, hãy cùng xem xét các macro!

{{#quiz ../quizzes/ch19-05-advanced-functions-and-closures.toml}}

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
