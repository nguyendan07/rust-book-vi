## Các kiểu nâng cao

Hệ thống kiểu của Rust có một số tính năng mà cho đến nay chúng ta đã đề cập nhưng vẫn chưa
thảo luận. Chúng ta sẽ bắt đầu bằng cách thảo luận về các newtype nói chung khi chúng ta xem xét lý do tại sao
newtype lại hữu ích dưới dạng các kiểu dữ liệu. Sau đó chúng ta sẽ chuyển sang bí danh kiểu (type aliases), một tính năng
tương tự như newtype nhưng có ngữ nghĩa hơi khác một chút. Chúng ta cũng sẽ thảo luận về
kiểu `!` và các kiểu có kích thước động (dynamically sized types).

### Sử dụng mẫu Newtype cho tính an toàn kiểu và tính trừu tượng

Phần này giả định rằng bạn đã đọc phần trước [“Sử dụng mẫu Newtype
để triển khai các Trait bên ngoài trên các kiểu bên ngoài”][using-the-newtype-pattern]<!--
ignore -->. Mẫu newtype cũng hữu ích cho các tác vụ ngoài những tác vụ mà chúng ta đã
thảo luận cho đến nay, bao gồm việc đảm bảo an toàn kiểu ở cấp độ tĩnh (ngay tại thời điểm biên dịch) để các giá trị không bao giờ bị nhầm lẫn
và chỉ ra các đơn vị của một giá trị. Bạn đã thấy một ví dụ về việc sử dụng newtype để
chỉ ra các đơn vị trong Danh sách 20-16: hãy nhớ lại rằng các struct `Millimeters` và `Meters`
đã bao bọc các giá trị `u32` trong một newtype. Nếu chúng ta viết một hàm với một
tham số kiểu `Millimeters`, chúng ta sẽ không thể biên dịch một chương trình mà
vô tình cố gắng gọi hàm đó với một giá trị có kiểu `Meters` hoặc một
kiểu `u32` thuần túy.

Chúng ta cũng có thể sử dụng mẫu newtype để trừu tượng hóa một số chi tiết triển khai
của một kiểu: kiểu mới có thể để lộ một API công khai (public API) khác với
API của kiểu bên trong riêng tư (private inner type).

Các newtype cũng có thể che giấu việc triển khai nội bộ. Ví dụ, chúng ta có thể cung cấp một
kiểu `People` để bao bọc một `HashMap<i32, String>` lưu trữ ID của một người
liên kết với tên của họ. Mã sử dụng `People` sẽ chỉ tương tác với API công khai mà
chúng ta cung cấp, chẳng hạn như một phương thức để thêm một chuỗi tên vào bộ sưu tập `People`;
mã đó sẽ không cần biết rằng chúng ta gán một ID `i32` cho các tên
ở nội bộ. Mẫu newtype là một cách nhẹ nhàng để đạt được sự đóng gói (encapsulation) nhằm
che giấu các chi tiết triển khai, điều mà chúng ta đã thảo luận trong [“Sự đóng gói che giấu
các chi tiết triển khai”][encapsulation-that-hides-implementation-details]<!--
ignore --> ở Chương 18.

### Từ đồng nghĩa kiểu và Bí danh kiểu (Type Aliases)

Rust cung cấp khả năng khai báo một _bí danh kiểu_ (type alias) để đặt cho một kiểu hiện có
một cái tên khác. Để làm việc này, chúng ta sử dụng từ khóa `type`. Ví dụ, chúng ta có thể tạo
bí danh `Kilometers` cho `i32` như sau:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Bây giờ, bí danh `Kilometers` là một _từ đồng nghĩa_ (synonym) cho `i32`; không giống như các kiểu
`Millimeters` và `Meters` mà chúng ta đã tạo trong Danh sách 20-16, `Kilometers` không phải là một
kiểu mới, riêng biệt. Các giá trị có kiểu `Kilometers` sẽ được trình biên dịch xử lý hoàn toàn giống như
các giá trị có kiểu `i32`:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Bởi vì `Kilometers` và `i32` là cùng một kiểu, chúng ta có thể cộng các giá trị của cả hai
kiểu và chúng ta có thể truyền các giá trị `Kilometers` vào các hàm nhận các tham số
`i32`. Tuy nhiên, khi sử dụng phương pháp này, chúng ta không nhận được các lợi ích về kiểm tra kiểu
mà chúng ta nhận được từ mẫu newtype đã thảo luận trước đó. Nói cách khác, nếu chúng ta
trộn lẫn các giá trị `Kilometers` và `i32` ở đâu đó, trình biên dịch sẽ không đưa ra
lỗi cho chúng ta.

> [!NOTE]
> **Liên hệ với Python:**
> - **Type Alias (`type Kilometers = i32;`):** Tương tự như gán biến kiểu trong Python (`Kilometers = int` hoặc cú pháp `type Kilometers = int` ở Python 3.12+). Nó chỉ là tên gọi khác (bí danh) giúp code ngắn hơn, bạn vẫn có thể truyền nhầm `i32` vào hàm nhận `Kilometers` mà không gặp lỗi biên dịch.
> - **Newtype Pattern (`struct Millimeters(u32);`):** Tương tự `typing.NewType('Millimeters', int)` trong Python, tạo ra một kiểu dữ liệu mới hoàn toàn độc lập. Trình biên dịch Rust sẽ báo lỗi ngay lập tức nếu bạn vô tình truyền nhầm kiểu dữ liệu khác vào.

Trường hợp sử dụng chính cho các từ đồng nghĩa kiểu là để giảm bớt sự lặp lại. Ví dụ, chúng ta
có thể có một kiểu dài như thế này:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Việc viết kiểu dài này trong các chữ ký hàm và dưới dạng các chú thích kiểu ở khắp mọi
nơi trong mã nguồn có thể gây mệt mỏi và dễ xảy ra sai sót. Hãy tưởng tượng việc có một dự án đầy
mã nguồn như trong Danh sách 20-25.

<Listing number="20-25" caption="Sử dụng một kiểu dài ở nhiều nơi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Một bí danh kiểu làm cho mã này dễ quản lý hơn bằng cách giảm bớt sự lặp lại. Trong
Danh sách 20-26, chúng ta đã giới thiệu một bí danh tên là `Thunk` cho kiểu dài dòng đó và
có thể thay thế tất cả các lần sử dụng kiểu đó bằng bí danh ngắn hơn `Thunk`.

<Listing number="20-26" caption="Giới thiệu một bí danh kiểu `Thunk` để giảm bớt sự lặp lại">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

Mã này dễ đọc và dễ viết hơn nhiều! Việc chọn một cái tên có ý nghĩa cho một
bí danh kiểu cũng có thể giúp truyền đạt ý định của bạn (_thunk_ là một từ dùng để chỉ mã nguồn
sẽ được đánh giá vào một thời điểm sau đó, vì vậy nó là một cái tên thích hợp cho một closure
được lưu trữ).

Các bí danh kiểu cũng thường được sử dụng với kiểu `Result<T, E>` để giảm bớt
sự lặp lại. Hãy xem xét module `std::io` trong thư viện chuẩn. Các thao tác
I/O thường trả về một `Result<T, E>` để xử lý các tình huống khi các thao tác
thất bại. Thư viện này có một struct `std::io::Error` đại diện cho tất cả
các lỗi I/O có thể xảy ra. Nhiều hàm trong `std::io` sẽ trả về
`Result<T, E>` trong đó `E` là `std::io::Error`, chẳng hạn như các hàm này trong
trait `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

Cụm `Result<..., Error>` được lặp lại rất nhiều. Do đó, `std::io` có khai báo
bí danh kiểu này:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Bởi vì khai báo này nằm trong module `std::io`, chúng ta có thể sử dụng bí danh
định danh đầy đủ `std::io::Result<T>`; nghĩa là một kiểu `Result<T, E>` trong đó kiểu lỗi `E` đã được cố định
sẵn là `std::io::Error`. Các chữ ký hàm của trait `Write` cuối cùng
trông như thế này:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Bí danh kiểu giúp ích theo hai cách: nó làm cho mã dễ viết hơn _và_ nó mang lại cho
chúng ta một giao diện nhất quán trong toàn bộ `std::io`. Bởi vì nó là một bí danh, nó
chỉ là một `Result<T, E>` khác, có nghĩa là chúng ta có thể sử dụng bất kỳ phương thức nào hoạt động trên
`Result<T, E>` với nó, cũng như cú pháp đặc biệt như toán tử `?`.

### Kiểu Never (Never Type) – Kiểu dữ liệu không bao giờ trả về

Rust có một kiểu đặc biệt tên là `!` được gọi trong thuật ngữ lý thuyết kiểu là
_kiểu trống_ (empty type) vì nó không có giá trị nào. Chúng ta thích gọi nó là _kiểu never_
bởi vì nó đứng ở vị trí của kiểu trả về khi một hàm sẽ không bao giờ
trả về. Đây là một ví dụ:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Mã này được đọc là “hàm `bar` không bao giờ trả về.” Các hàm không bao giờ trả về
được gọi là _các hàm phân kỳ_ (diverging functions). Chúng ta không thể tạo ra các giá trị thuộc kiểu `!`,
do đó hàm `bar` không bao giờ có thể kết thúc và trả về một giá trị nào.

> [!NOTE]
> **Liên hệ với Python:** Kiểu `!` trong Rust tương đương với kiểu trả về `typing.Never` (hoặc `typing.NoReturn`) trong Python type hints — dùng cho các hàm chắc chắn không bao giờ return bình thường (ví dụ: hàm chỉ toàn `raise Exception` hoặc chứa vòng lặp vô tận `while True`). Trong Rust, kiểu `!` có thể tự động ép kiểu về bất kỳ kiểu dữ liệu nào khác, giúp các biểu thức như `panic!`, `continue`, `break` có thể đặt vừa vặn trong mọi nhánh của `match`.

Nhưng một kiểu mà bạn không bao giờ có thể tạo ra các giá trị cho nó thì có ích gì? Hãy nhớ lại mã từ
Danh sách 2-5, một phần của trò chơi đoán số; chúng ta đã tái hiện một phần của nó
ở đây trong Danh sách 20-27.

<Listing number="20-27" caption="Một `match` với một nhánh (arm) kết thúc bằng `continue` ">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

Vào lúc đó, chúng ta đã bỏ qua một số chi tiết trong mã này. Trong [“Toán tử điều khiển
luồng `match`”][the-match-control-flow-operator]<!-- ignore --> ở Chương 6, chúng ta
đã thảo luận rằng các nhánh của `match` đều phải trả về cùng một kiểu. Vì vậy, ví dụ,
mã sau đây không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Kiểu của `guess` trong mã này sẽ phải vừa là một số nguyên _vừa_ là một chuỗi,
và Rust yêu cầu `guess` chỉ có một kiểu. Vậy `continue` trả về
gì? Làm thế nào chúng ta được phép trả về một `u32` từ một nhánh và có một nhánh khác
kết thúc bằng `continue` trong Danh sách 20-27?

Như bạn có thể đã đoán, `continue` có giá trị `!`. Nghĩa là, khi Rust
suy luận kiểu dữ liệu cho biến `guess`, nó kiểm tra cả hai nhánh match, nhánh trước có một
giá trị kiểu `u32` và nhánh sau có một giá trị kiểu `!`. Bởi vì `!` không bao giờ có thể có
một giá trị, Rust quyết định rằng kiểu của `guess` là `u32`.

Cách mô tả chính thức về hành vi này là các biểu thức kiểu `!` có thể
được ép kiểu (coerced) thành bất kỳ kiểu nào khác. Chúng ta được phép kết thúc nhánh `match` này bằng
`continue` bởi vì `continue` không trả về một giá trị; thay vào đó, nó di chuyển quyền điều khiển
trở lại đầu vòng lặp, vì vậy trong trường hợp `Err`, chúng ta không bao giờ gán một giá trị cho
`guess`.

Kiểu never cũng hữu ích với macro `panic!`. Hãy nhớ lại hàm `unwrap`
mà chúng ta gọi trên các giá trị `Option<T>` để tạo ra một giá trị hoặc panic với
định nghĩa này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Trong mã này, điều tương tự cũng xảy ra như trong `match` ở Danh sách 20-27: Rust
thấy rằng `val` có kiểu `T` và `panic!` có kiểu `!`, vì vậy kết quả
của biểu thức `match` tổng thể là `T`. Mã này hoạt động vì `panic!`
không tạo ra một giá trị; nó kết thúc chương trình. Trong trường hợp `None`, chúng ta sẽ không
trả về một giá trị từ `unwrap`, vì vậy mã này là hợp lệ.

Một biểu thức cuối cùng có kiểu `!` là một `loop`:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Ở đây, vòng lặp không bao giờ kết thúc, vì vậy `!` là giá trị của biểu thức. Tuy nhiên, điều này
sẽ không đúng nếu chúng ta bao gồm một `break`, bởi vì vòng lặp sẽ kết thúc
khi nó gặp `break`.

### Các kiểu có kích thước động và Trait `Sized`

Rust cần biết một số chi tiết nhất định về các kiểu của nó, chẳng hạn như cần bao nhiêu không gian để
cấp phát cho một giá trị của một kiểu cụ thể. Điều này dẫn đến một khía cạnh trong hệ thống kiểu của Rust
có thể gây bỡ ngỡ lúc đầu: khái niệm về _các kiểu có kích thước động_ (dynamically sized types - DST).
Đôi khi được gọi là _DSTs_ hoặc _các kiểu không có kích thước_ (unsized types), các kiểu này cho phép chúng ta viết
mã sử dụng các giá trị mà kích thước của chúng chúng ta chỉ có thể biết được lúc runtime (khi chương trình chạy).

Hãy đi sâu vào chi tiết của một kiểu có kích thước động tên là `str`, cái mà
chúng ta đã sử dụng xuyên suốt cuốn sách. Đúng vậy, không phải `&str`, mà là bản thân `str`,
là một DST. Trong nhiều trường hợp, chẳng hạn như khi lưu trữ văn bản do người dùng nhập vào, chúng ta không thể biết chuỗi dài bao nhiêu byte cho đến khi chương trình chạy (runtime). Điều đó có
nghĩa là chúng ta không thể tạo một biến kiểu `str`, chúng ta cũng không thể nhận một đối số kiểu
`str`. Hãy xem xét mã sau đây, cái mà không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust cần biết cần cấp phát bao nhiêu bộ nhớ cho bất kỳ giá trị nào của một kiểu
cụ thể, và tất cả các giá trị của một kiểu phải sử dụng cùng một lượng bộ nhớ. Nếu Rust
cho phép chúng ta viết mã này, hai giá trị `str` này sẽ cần chiếm cùng một
lượng không gian. Nhưng chúng có độ dài khác nhau: `s1` cần 12 byte để lưu trữ
và `s2` cần 15 byte. Đây là lý do tại sao không thể tạo một biến
giữ một kiểu có kích thước động.

Vậy chúng ta phải làm gì? Trong trường hợp này, bạn đã biết câu trả lời: chúng ta tạo kiểu
của `s1` và `s2` là string slice (`&str`) thay vì một `str`. Hãy nhớ lại từ [“String
Slices”][string-slices]<!-- ignore --> ở Chương 4 rằng cấu trúc dữ liệu slice
chỉ lưu trữ vị trí bắt đầu và độ dài của slice. Vì vậy
mặc dù một `&T` là một giá trị đơn lẻ lưu trữ địa chỉ bộ nhớ nơi
`T` được lưu trữ, một string slice `&str` là _hai_ giá trị (fat pointer): địa chỉ của `str` và độ dài của nó.
Như vậy, chúng ta luôn có thể biết kích thước của một giá trị `&str` tại thời điểm biên dịch: nó bằng
hai lần kích thước của một `usize`. Nghĩa là, chúng ta luôn biết kích thước của một `&str`, bất kể
chuỗi mà nó tham chiếu đến dài bao nhiêu. Nói chung, đây là cách mà
các kiểu có kích thước động được sử dụng trong Rust: chúng có thêm một chút siêu dữ liệu (metadata)
để lưu trữ kích thước của thông tin động. Quy tắc vàng của các kiểu có kích thước động
là chúng ta phải luôn đặt các giá trị của các kiểu có kích thước động đằng sau
một con trỏ thuộc loại nào đó.

> [!NOTE]
> **Khác biệt với Python:** Trong Python, mọi biến thực chất đều là con trỏ trỏ đến đối tượng trên Heap (`PyObject*`), trong đó đối tượng luôn mang sẵn metadata (loại kiểu, độ dài, số lượng tham chiếu). Trong Rust, vì hiệu năng tối đa và không có garbage collector, các giá trị trên Stack bắt buộc phải có kích thước cố định tại thời điểm biên dịch. Với các kiểu có kích thước thay đổi linh hoạt lúc chạy như `str` hay slice mảng `[T]`, Rust không thể lưu trực tiếp trên Stack mà buộc phải đặt sau một con trỏ (như `&str`, `Box<str>`), con trỏ này kèm theo độ dài dữ liệu (gọi là *fat pointer*).

Chúng ta có thể kết hợp `str` với tất cả các loại con trỏ: ví dụ, `Box<str>` hoặc
`Rc<str>`. Trên thực tế, bạn đã thấy điều này trước đây nhưng với một kiểu có kích thước động
khác: các trait. Mọi trait là một kiểu có kích thước động mà chúng ta có thể tham chiếu đến bằng cách
sử dụng tên của trait đó. Trong [“Sử dụng các đối tượng Trait cho phép các giá trị thuộc
các kiểu khác nhau”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore
--> ở Chương 18, chúng ta đã đề cập rằng để sử dụng các trait như các đối tượng trait, chúng ta phải đặt
chúng đằng sau một con trỏ, chẳng hạn như `&dyn Trait` hoặc `Box<dyn Trait>` (`Rc<dyn Trait>`
cũng sẽ hoạt động).

Để làm việc với các DST, Rust cung cấp trait `Sized` để xác định xem kích thước của
một kiểu có được biết tại thời điểm biên dịch hay không. Trait này được tự động triển khai (implement)
cho mọi kiểu dữ liệu có kích thước đã biết tại thời điểm biên dịch. Ngoài ra, Rust
ngầm định thêm một ràng buộc (bound) trên `Sized` cho mọi hàm generic. Nghĩa là, một
định nghĩa hàm generic như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

thực sự được đối xử như thể chúng ta đã viết thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Theo mặc định, các hàm generic sẽ chỉ hoạt động trên các kiểu có kích thước đã biết tại
thời điểm biên dịch. Tuy nhiên, bạn có thể sử dụng cú pháp đặc biệt sau đây để nới lỏng
hạn chế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Một trait bound dạng `?Sized` có nghĩa là “`T` có thể có kích thước cố định (`Sized`) hoặc không (unsized/DST)”, và ký hiệu
này ghi đè lên quy tắc mặc định rằng các kiểu generic phải có kích thước đã biết tại
thời điểm biên dịch. Cú pháp `?Trait` với ý nghĩa này chỉ khả dụng cho
`Sized`, không phải bất kỳ trait nào khác.

Cũng lưu ý rằng chúng ta đã chuyển kiểu của tham số `t` từ `T` sang `&T`.
Bởi vì kiểu đó có thể không phải là `Sized`, chúng ta cần sử dụng nó đằng sau một loại
con trỏ nào đó. Trong trường hợp này, chúng ta đã chọn một tham chiếu.

> [!NOTE]
> **Ghi chú mở rộng:** Theo mặc định, mọi hàm generic `fn foo<T>(x: T)` trong Rust đều ngầm hiểu là `fn foo<T: Sized>(x: T)` (chỉ chấp nhận các kiểu có kích thước biết trước lúc biên dịch). Cú pháp đặc biệt `T: ?Sized` là cách duy nhất để bạn nói với Rust rằng: *"Hàm này chấp nhận cả những kiểu có kích thước động (như `str`, `dyn Trait`)"*, và khi đó tham số bắt buộc phải được truyền qua con trỏ tham chiếu `&T`.

Tiếp theo, chúng ta sẽ nói về các hàm và closure!

{{#quiz ../quizzes/ch19-04-advanced-types.toml}}

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-04-slices.html#string-slices
[the-match-control-flow-operator]: ch06-02-match.html#the-match-control-flow-operator
[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[using-the-newtype-pattern]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
