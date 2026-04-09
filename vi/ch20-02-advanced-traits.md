## Các Trait nâng cao

Chúng ta đã lần đầu tiên đề cập đến các trait trong [“Trait: Định nghĩa hành vi
chung”][traits-defining-shared-behavior]<!-- ignore --> ở Chương 10, nhưng chúng ta
đã không thảo luận về các chi tiết nâng cao hơn. Bây giờ khi bạn đã biết nhiều hơn về Rust, chúng ta
có thể đi vào những chi tiết cụ thể.

<!-- Old link, do not remove -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>

### Các kiểu liên kết (Associated Types)

_Các kiểu liên kết_ kết nối một kiểu giữ chỗ (placeholder type) với một trait sao cho các
định nghĩa phương thức của trait có thể sử dụng các kiểu giữ chỗ này trong chữ ký của chúng. Người
thực thi một trait sẽ chỉ định kiểu cụ thể được sử dụng thay cho
kiểu giữ chỗ cho việc thực thi cụ thể đó. Bằng cách đó, chúng ta có thể định nghĩa một
trait sử dụng một số kiểu mà không cần biết chính xác những kiểu đó là gì
cho đến khi trait được thực thi.

Chúng ta đã mô tả hầu hết các tính năng nâng cao trong chương này là hiếm khi
cần thiết. Các kiểu liên kết nằm ở đâu đó ở giữa: chúng được sử dụng hiếm hơn
các tính năng được giải thích trong phần còn lại của cuốn sách nhưng phổ biến hơn nhiều
tính năng khác được thảo luận trong chương này.

Một ví dụ về một trait có kiểu liên kết là trait `Iterator` mà
thư viện chuẩn cung cấp. Kiểu liên kết được đặt tên là `Item` và đại diện
cho kiểu của các giá trị mà kiểu đang thực thi trait `Iterator` đang
lặp qua. Định nghĩa của trait `Iterator` như được trình bày trong Danh sách
20-13.

<Listing number="20-13" caption="Định nghĩa của trait `Iterator` có một kiểu liên kết `Item` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

Kiểu `Item` là một kiểu giữ chỗ, và định nghĩa của phương thức `next` cho thấy rằng
nó sẽ trả về các giá trị có kiểu `Option<Self::Item>`. Những người thực thi
trait `Iterator` sẽ chỉ định kiểu cụ thể cho `Item`, và phương thức `next`
sẽ trả về một `Option` chứa một giá trị của kiểu cụ thể đó.

Các kiểu liên kết có vẻ giống như một khái niệm tương tự như generic, ở chỗ
generic cho phép chúng ta định nghĩa một hàm mà không cần chỉ định kiểu nào nó có thể
xử lý. Để xem xét sự khác biệt giữa hai khái niệm này, chúng ta sẽ xem xét một
triển khai của trait `Iterator` trên một kiểu tên là `Counter` chỉ định
kiểu `Item` là `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Cú pháp này có vẻ tương đương với cú pháp của generic. Vậy tại sao không định nghĩa
trait `Iterator` với generic, như được trình bày trong Danh sách 20-14?

<Listing number="20-14" caption="Một định nghĩa giả định của trait `Iterator` sử dụng generic">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

Sự khác biệt là khi sử dụng generic, như trong Danh sách 20-14, chúng ta phải
chú thích các kiểu trong mỗi lần thực thi; bởi vì chúng ta cũng có thể thực thi
`Iterator<String> for Counter` hoặc bất kỳ kiểu nào khác, chúng ta có thể có nhiều
triển khai của `Iterator` cho `Counter`. Nói cách khác, khi một trait có một
tham số generic, nó có thể được thực thi cho một kiểu nhiều lần, thay đổi
các kiểu cụ thể của các tham số kiểu generic mỗi lần. Khi chúng ta sử dụng
phương thức `next` trên `Counter`, chúng ta sẽ phải cung cấp các chú thích kiểu để
chỉ ra triển khai nào của `Iterator` mà chúng ta muốn sử dụng.

Với các kiểu liên kết, chúng ta không cần chú thích các kiểu bởi vì chúng ta không thể
thực thi một trait trên một kiểu nhiều lần. Trong Danh sách 20-13 với định nghĩa
sử dụng các kiểu liên kết, chúng ta có thể chọn kiểu của `Item` sẽ là gì chỉ
một lần, bởi vì chỉ có thể có một `impl Iterator for Counter`. Chúng ta không
phải chỉ định rằng chúng ta muốn một iterator của các giá trị `u32` ở mọi nơi mà chúng ta gọi
`next` trên `Counter`.

Các kiểu liên kết cũng trở thành một phần của hợp đồng của trait: những người thực thi
trait phải cung cấp một kiểu để thay thế cho kiểu liên kết giữ chỗ.
Các kiểu liên kết thường có một cái tên mô tả cách kiểu đó sẽ được sử dụng,
và việc ghi lại kiểu liên kết trong tài liệu API là một thói quen tốt.

### Các tham số kiểu Generic mặc định và nạp chồng toán tử

Khi chúng ta sử dụng các tham số kiểu generic, chúng ta có thể chỉ định một kiểu cụ thể mặc định cho
kiểu generic đó. Điều này loại bỏ nhu cầu cho những người thực thi trait phải
chỉ định một kiểu cụ thể nếu kiểu mặc định đã hoạt động tốt. Bạn chỉ định một kiểu mặc định
khi khai báo một kiểu generic với cú pháp `<PlaceholderType=ConcreteType>`.

Một ví dụ tuyệt vời về tình huống mà kỹ thuật này hữu ích là với _nạp chồng toán tử_ (operator
overloading), trong đó bạn tùy chỉnh hành vi của một toán tử (chẳng hạn như `+`)
trong các tình huống cụ thể.

Rust không cho phép bạn tạo các toán tử của riêng mình hoặc nạp chồng các toán tử
tùy ý. Nhưng bạn có thể nạp chồng các thao tác và các trait tương ứng được liệt kê
trong `std::ops` bằng cách thực thi các trait liên quan đến toán tử đó. Ví dụ,
trong Danh sách 20-15, chúng ta nạp chồng toán tử `+` để cộng hai thực thể
`Point` lại với nhau. Chúng ta thực hiện việc này bằng cách thực thi trait `Add` trên một
struct `Point`.

<Listing number="20-15" file-name="src/main.rs" caption="Thực thi trait `Add` để nạp chồng toán tử `+` cho các thực thể `Point` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

Phương thức `add` cộng các giá trị `x` của hai thực thể `Point` và các giá trị `y`
của hai thực thể `Point` để tạo ra một `Point` mới. Trait `Add` có một
kiểu liên kết tên là `Output` xác định kiểu được trả về từ phương thức `add`.

Kiểu generic mặc định trong mã này nằm trong trait `Add`. Đây là định nghĩa
của nó:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Mã này nhìn chung có vẻ quen thuộc: một trait với một phương thức và một
kiểu liên kết. Phần mới là `Rhs=Self`: cú pháp này được gọi là _các tham số kiểu mặc định_
(default type parameters). Tham số kiểu generic `Rhs` (viết tắt của “right-hand
side” - vế phải) định nghĩa kiểu của tham số `rhs` trong phương thức `add`. Nếu chúng ta không
chỉ định một kiểu cụ thể cho `Rhs` khi chúng ta thực thi trait `Add`, kiểu
của `Rhs` sẽ mặc định là `Self`, chính là kiểu mà chúng ta đang thực thi
`Add` trên đó.

Khi chúng ta thực thi `Add` cho `Point`, chúng ta đã sử dụng giá trị mặc định cho `Rhs` vì chúng ta
muốn cộng hai thực thể `Point`. Hãy xem một ví dụ về việc thực thi
trait `Add` nơi chúng ta muốn tùy chỉnh kiểu `Rhs` thay vì sử dụng
mặc định.

Chúng ta có hai struct, `Millimeters` và `Meters`, giữ các giá trị trong các đơn vị
khác nhau. Việc bao bọc mỏng một kiểu hiện có trong một struct khác này được gọi là
_mẫu newtype_ (newtype pattern), điều mà chúng ta mô tả chi tiết hơn trong phần [“Sử dụng mẫu Newtype
để thực thi các Trait bên ngoài trên các kiểu bên ngoài”][newtype]<!-- ignore
-->. Chúng ta muốn cộng các giá trị tính bằng milimet với các giá trị tính bằng mét và để
việc thực thi `Add` thực hiện việc chuyển đổi một cách chính xác. Chúng ta có thể thực thi `Add`
cho `Millimeters` với `Meters` là `Rhs`, như được trình bày trong Danh sách 20-16.

<Listing number="20-16" file-name="src/lib.rs" caption="Thực thi trait `Add` trên `Millimeters` để cộng `Millimeters` với `Meters` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

Để cộng `Millimeters` và `Meters`, chúng ta chỉ định `impl Add<Meters>` để thiết lập
giá trị của tham số kiểu `Rhs` thay vì sử dụng mặc định là `Self`.

Bạn sẽ sử dụng các tham số kiểu mặc định theo hai cách chính:

1. Để mở rộng một kiểu mà không làm hỏng mã nguồn hiện có
2. Để cho phép tùy chỉnh trong các trường hợp cụ thể mà hầu hết người dùng sẽ không cần

Trait `Add` của thư viện chuẩn là một ví dụ cho mục đích thứ hai:
thông thường, bạn sẽ cộng hai kiểu giống nhau, nhưng trait `Add` cung cấp khả năng
tùy chỉnh vượt ra ngoài điều đó. Việc sử dụng một tham số kiểu mặc định trong định nghĩa trait
`Add` có nghĩa là bạn không phải chỉ định tham số bổ sung trong hầu hết
thời gian. Nói cách khác, một chút mã mẫu (boilerplate) triển khai là không cần thiết, giúp
sử dụng trait dễ dàng hơn.

Mục đích thứ nhất tương tự như mục đích thứ hai nhưng theo chiều ngược lại: nếu bạn muốn thêm một
tham số kiểu vào một trait hiện có, bạn có thể cung cấp cho nó một giá trị mặc định để cho phép
mở rộng chức năng của trait mà không làm hỏng mã nguồn triển khai
hiện có.

<!-- Old link, do not remove -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>

### Phân biệt giữa các phương thức có cùng tên

Không có gì trong Rust ngăn cản một trait có một phương thức trùng tên với
phương thức của một trait khác, Rust cũng không ngăn cản bạn thực thi cả hai trait
trên cùng một kiểu. Bạn cũng có thể thực thi một phương thức trực tiếp trên kiểu trùng
tên với các phương thức từ các trait.

Khi gọi các phương thức trùng tên, bạn sẽ cần cho Rust biết bạn
muốn sử dụng phương thức nào. Hãy xem xét mã trong Danh sách 20-17 nơi chúng ta đã định nghĩa hai trait,
`Pilot` và `Wizard`, cả hai đều có một phương thức tên là `fly`. Sau đó chúng ta thực thi
cả hai trait trên một kiểu `Human` mà bản thân nó đã có một phương thức tên là `fly` được thực thi
trên đó. Mỗi phương thức `fly` thực hiện một điều gì đó khác nhau.

<Listing number="20-17" file-name="src/main.rs" caption="Hai trait được định nghĩa có phương thức `fly` và được thực thi trên kiểu `Human`, và một phương thức `fly` được thực thi trực tiếp trên `Human`.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Khi chúng ta gọi `fly` trên một thực thể của `Human`, trình biên dịch mặc định gọi
phương thức được thực thi trực tiếp trên kiểu đó, như được trình bày trong Danh sách 20-18.

<Listing number="20-18" file-name="src/main.rs" caption="Gọi `fly` trên một thực thể của `Human` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Chạy mã này sẽ in ra `*waving arms furiously*`, cho thấy Rust
đã gọi phương thức `fly` được thực thi trực tiếp trên `Human`.

Để gọi các phương thức `fly` từ trait `Pilot` hoặc trait `Wizard`,
chúng ta cần sử dụng cú pháp rõ ràng hơn để chỉ định phương thức `fly` nào chúng ta muốn nói đến.
Danh sách 20-19 minh họa cú pháp này.

<Listing number="20-19" file-name="src/main.rs" caption="Chỉ định phương thức `fly` của trait nào mà chúng ta muốn gọi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Việc chỉ định tên trait trước tên phương thức làm rõ cho Rust biết
triển khai nào của `fly` mà chúng ta muốn gọi. Chúng ta cũng có thể viết
`Human::fly(&person)`, tương đương với `person.fly()` mà chúng ta đã sử dụng
trong Danh sách 20-19, nhưng cách này dài hơn một chút nếu chúng ta không cần
phân biệt.

Chạy mã này sẽ in ra kết quả sau:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Bởi vì phương thức `fly` nhận một tham số `self`, nếu chúng ta có hai _kiểu_
cùng thực thi một _trait_, Rust có thể tìm ra triển khai nào của một
trait để sử dụng dựa trên kiểu của `self`.

Tuy nhiên, các hàm liên kết (associated functions) không phải là phương thức thì không có tham số `self`.
Khi có nhiều kiểu hoặc trait định nghĩa các hàm không phải phương thức
có cùng tên hàm, Rust không phải lúc nào cũng biết bạn đang nói đến kiểu nào
trừ khi bạn sử dụng _cú pháp định danh đầy đủ_ (fully qualified syntax). Ví dụ, trong Danh sách 20-20 chúng ta
tạo một trait cho một trạm cứu hộ động vật muốn đặt tên cho tất cả các con chó con là _Spot_.
Chúng ta tạo một trait `Animal` với một hàm liên kết không phải phương thức `baby_name`.
Trait `Animal` được thực thi cho struct `Dog`, trên đó chúng ta cũng
cung cấp một hàm liên kết không phải phương thức `baby_name` trực tiếp.

<Listing number="20-20" file-name="src/main.rs" caption="Một trait với một hàm liên kết và một kiểu với một hàm liên kết trùng tên cũng thực thi trait đó">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Chúng ta thực thi mã để đặt tên cho tất cả các con chó con là Spot trong hàm liên kết `baby_name`
được định nghĩa trên `Dog`. Kiểu `Dog` cũng thực thi trait
`Animal`, mô tả các đặc điểm mà tất cả các loài động vật đều có. Chó con được
gọi là puppy, và điều đó được thể hiện trong việc thực thi trait `Animal`
trên `Dog` trong hàm `baby_name` liên kết với trait `Animal`.

Trong `main`, chúng ta gọi hàm `Dog::baby_name`, hàm này gọi hàm liên kết
được định nghĩa trực tiếp trên `Dog`. Mã này in ra kết quả sau:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Đầu ra này không phải là những gì chúng ta muốn. Chúng ta muốn gọi hàm `baby_name` là
một phần của trait `Animal` mà chúng ta đã thực thi trên `Dog` để mã in ra
`A baby dog is called a puppy`. Kỹ thuật chỉ định tên trait mà
chúng ta đã sử dụng trong Danh sách 20-19 không giúp ích gì ở đây; nếu chúng ta thay đổi `main` thành mã trong
Danh sách 20-21, chúng ta sẽ gặp lỗi biên dịch.

<Listing number="20-21" file-name="src/main.rs" caption="Cố gắng gọi hàm `baby_name` từ trait `Animal`, nhưng Rust không biết triển khai nào để sử dụng">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Bởi vì `Animal::baby_name` không có tham số `self`, và có thể có
các kiểu khác thực thi trait `Animal`, Rust không thể tìm ra
triển khai nào của `Animal::baby_name` mà chúng ta muốn. Chúng ta sẽ nhận được lỗi trình biên dịch này:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Để phân biệt và nói với Rust rằng chúng ta muốn sử dụng triển khai của
`Animal` cho `Dog` trái ngược với triển khai của `Animal` cho một số kiểu
khác, chúng ta cần sử dụng cú pháp định danh đầy đủ. Danh sách 20-22 minh họa cách
sử dụng cú pháp định danh đầy đủ.

<Listing number="20-22" file-name="src/main.rs" caption="Sử dụng cú pháp định danh đầy đủ để chỉ định rằng chúng ta muốn gọi hàm `baby_name` từ trait `Animal` như được thực thi trên `Dog` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Chúng ta đang cung cấp cho Rust một chú thích kiểu bên trong các dấu ngoặc nhọn, điều này
cho biết chúng ta muốn gọi phương thức `baby_name` từ trait `Animal` như
được thực thi trên `Dog` bằng cách nói rằng chúng ta muốn coi kiểu `Dog` như một
`Animal` cho lời gọi hàm này. Mã này bây giờ sẽ in ra những gì chúng ta muốn:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Nói chung, cú pháp định danh đầy đủ được định nghĩa như sau:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Đối với các hàm liên kết không phải là phương thức, sẽ không có một `receiver`:
chỉ có danh sách các đối số khác. Bạn có thể sử dụng cú pháp định danh đầy đủ
ở mọi nơi mà bạn gọi các hàm hoặc phương thức. Tuy nhiên, bạn được phép
bỏ qua bất kỳ phần nào của cú pháp này mà Rust có thể tìm ra từ các thông tin khác
trong chương trình. Bạn chỉ cần sử dụng cú pháp dài dòng này trong các trường hợp mà
có nhiều triển khai sử dụng cùng một tên và Rust cần sự trợ giúp
để xác định triển khai nào bạn muốn gọi.

<!-- Old link, do not remove -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Sử dụng Supertraits

Đôi khi bạn có thể viết một định nghĩa trait phụ thuộc vào một trait khác: để
một kiểu thực thi trait thứ nhất, bạn muốn yêu cầu kiểu đó cũng phải
thực thi trait thứ hai. Bạn làm điều này để định nghĩa trait của bạn có thể
sử dụng các mục liên kết (associated items) của trait thứ hai. Trait mà định nghĩa trait của bạn
đang dựa vào được gọi là một _supertrait_ của trait của bạn.

Ví dụ, giả sử chúng ta muốn tạo một trait `OutlinePrint` với một
phương thức `outline_print` sẽ in ra một giá trị được định dạng sao cho nó được
đóng khung trong các dấu hoa thị. Nghĩa là, cho một struct `Point` thực thi trait
`Display` của thư viện chuẩn để cho kết quả `(x, y)`, khi chúng ta gọi
`outline_print` trên một thực thể `Point` có `1` cho `x` và `3` cho `y`, nó
sẽ in ra như sau:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Trong phần thực thi phương thức `outline_print`, chúng ta muốn sử dụng
chức năng của trait `Display`. Do đó, chúng ta cần chỉ định rằng trait
`OutlinePrint` sẽ chỉ hoạt động cho các kiểu cũng thực thi `Display` và
cung cấp chức năng mà `OutlinePrint` cần. Chúng ta có thể làm điều đó trong
định nghĩa trait bằng cách chỉ định `OutlinePrint: Display`. Kỹ thuật này
tương tự như việc thêm một trait bound vào trait. Danh sách 20-23 trình bày một
triển khai của trait `OutlinePrint`.

<Listing number="20-23" file-name="src/main.rs" caption="Thực thi trait `OutlinePrint` yêu cầu chức năng từ `Display` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

Bởi vì chúng ta đã chỉ định rằng `OutlinePrint` yêu cầu trait `Display`, chúng ta
có thể sử dụng hàm `to_string` được tự động thực thi cho bất kỳ kiểu nào
thực thi `Display`. Nếu chúng ta cố gắng sử dụng `to_string` mà không thêm
dấu hai chấm và chỉ định trait `Display` sau tên trait, chúng ta sẽ gặp một
lỗi nói rằng không tìm thấy phương thức nào tên là `to_string` cho kiểu `&Self` trong
phạm vi hiện tại.

Hãy xem điều gì xảy ra khi chúng ta cố gắng thực thi `OutlinePrint` trên một kiểu
không thực thi `Display`, chẳng hạn như struct `Point`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Chúng ta nhận được một lỗi nói rằng `Display` là bắt buộc nhưng chưa được thực thi:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Để khắc phục điều này, chúng ta thực thi `Display` trên `Point` và đáp ứng ràng buộc mà
`OutlinePrint` yêu cầu, như sau:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Sau đó, việc thực thi trait `OutlinePrint` trên `Point` sẽ được biên dịch
thành công, và chúng ta có thể gọi `outline_print` trên một thực thể `Point` để hiển thị
nó bên trong một khung bằng các dấu hoa thị.

### Sử dụng mẫu Newtype để thực thi các Trait bên ngoài trên các kiểu bên ngoài

Trong [“Thực thi một Trait trên một Kiểu”][implementing-a-trait-on-a-type]<!-- ignore
--> ở Chương 10, chúng ta đã đề cập đến quy tắc mồ côi (orphan rule) quy định rằng chúng ta chỉ được phép
thực thi một trait trên một kiểu nếu trait đó hoặc kiểu đó, hoặc cả hai, là
cục bộ (local) đối với crate của chúng ta. Có thể lách qua hạn chế này bằng cách sử dụng
_mẫu newtype_ (newtype pattern), liên quan đến việc tạo một kiểu mới trong một tuple struct. (Chúng ta
đã đề cập đến tuple struct trong [“Sử dụng Tuple Struct không có các trường được đặt tên để tạo
các kiểu khác nhau”][tuple-structs]<!-- ignore --> ở Chương 5.) Tuple struct
sẽ có một trường và là một lớp bao bọc mỏng (thin wrapper) xung quanh kiểu mà chúng ta muốn
thực thi một trait cho nó. Khi đó kiểu bao bọc là cục bộ đối với crate của chúng ta, và chúng ta có thể
thực thi trait trên lớp bao bọc đó. _Newtype_ là một thuật ngữ bắt nguồn từ
ngôn ngữ lập trình Haskell. Không có hình phạt nào về hiệu suất lúc thực thi (runtime) khi sử dụng
mẫu này, và kiểu bao bọc sẽ được loại bỏ (elided) tại thời điểm biên dịch.

Ví dụ, giả sử chúng ta muốn thực thi `Display` trên `Vec<T>`, điều mà
quy tắc mồ côi ngăn cản chúng ta thực hiện trực tiếp vì trait `Display` và kiểu
`Vec<T>` được định nghĩa bên ngoài crate của chúng ta. Chúng ta có thể tạo một struct `Wrapper`
giữ một thực thể của `Vec<T>`; sau đó chúng ta có thể thực thi `Display` trên
`Wrapper` và sử dụng giá trị `Vec<T>`, như được trình bày trong Danh sách 20-24.

<Listing number="20-24" file-name="src/main.rs" caption="Tạo một kiểu `Wrapper` xung quanh `Vec<String>` để thực thi `Display` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

Việc thực thi `Display` sử dụng `self.0` để truy cập `Vec<T>` bên trong,
bởi vì `Wrapper` là một tuple struct và `Vec<T>` là phần tử tại chỉ số 0 trong
tuple. Sau đó chúng ta có thể sử dụng chức năng của trait `Display` trên `Wrapper`.

Nhược điểm của việc sử dụng kỹ thuật này là `Wrapper` là một kiểu mới, vì vậy nó
không có các phương thức của giá trị mà nó đang giữ. Chúng ta sẽ phải thực thi
tất cả các phương thức của `Vec<T>` trực tiếp trên `Wrapper` sao cho các phương thức đó ủy quyền
cho `self.0`, điều này sẽ cho phép chúng ta coi `Wrapper` hoàn toàn giống như một `Vec<T>`. Nếu
chúng ta muốn kiểu mới có mọi phương thức mà kiểu bên trong có, việc thực thi
trait `Deref` trên `Wrapper` để trả về kiểu bên trong sẽ là một giải pháp (chúng ta
đã thảo luận về việc thực thi trait `Deref` trong [“Coi các con trỏ thông minh như
các tham chiếu thông thường với trait `Deref`”][smart-pointer-deref]<!-- ignore -->
trong Chương 15). Nếu chúng ta không muốn kiểu `Wrapper` có tất cả các phương thức của
kiểu bên trong—ví dụ, để hạn chế hành vi của kiểu `Wrapper`—chúng ta sẽ
phải thực thi thủ công chỉ những phương thức mà chúng ta thực sự muốn.

Mẫu newtype này cũng hữu ích ngay cả khi không liên quan đến các trait. Hãy
chuyển trọng tâm và xem xét một số cách nâng cao để tương tác với hệ thống kiểu của Rust.

{{#quiz ../quizzes/ch19-03-advanced-traits.toml}}

[newtype]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]: ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
