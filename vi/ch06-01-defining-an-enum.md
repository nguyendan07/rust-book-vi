## Định nghĩa một Enum

Trong khi struct cung cấp cho bạn một cách để nhóm các trường và dữ liệu liên quan lại với nhau, như
`Rectangle` với `width` và `height` của nó, thì enum cung cấp cho bạn một cách để nói rằng một giá trị
là một trong số các giá trị có thể có. Ví dụ, chúng ta có thể muốn nói rằng `Rectangle` là một trong một
tập hợp các hình dạng có thể bao gồm cả `Circle` và `Triangle`. Để làm điều này, Rust cho phép chúng ta
mã hóa các khả năng này dưới dạng một enum.

Hãy cùng xem xét một tình huống mà chúng ta muốn diễn đạt bằng mã và xem tại sao enum lại
hữu ích và phù hợp hơn struct trong trường hợp này. Giả sử chúng ta cần làm việc với các địa chỉ IP.
Hiện tại, có hai tiêu chuẩn chính được sử dụng cho địa chỉ IP: phiên bản bốn và phiên bản sáu. Bởi vì
đây là những khả năng duy nhất cho một địa chỉ IP mà chương trình của chúng ta sẽ gặp phải, chúng ta
có thể _liệt kê (enumerate)_ tất cả các biến thể có thể có, đó là lý do tại sao phép liệt kê có tên gọi như vậy.

Bất kỳ địa chỉ IP nào cũng có thể là phiên bản bốn hoặc phiên bản sáu, nhưng không thể là cả hai cùng
một lúc. Đặc tính đó của địa chỉ IP làm cho cấu trúc dữ liệu enum trở nên phù hợp vì một giá trị enum
chỉ có thể là một trong các biến thể của nó. Cả địa chỉ phiên bản bốn và phiên bản sáu về cơ bản vẫn
là địa chỉ IP, vì vậy chúng nên được coi là cùng một kiểu dữ liệu khi mã xử lý các tình huống áp dụng
cho bất kỳ loại địa chỉ IP nào.

Chúng ta có thể diễn đạt khái niệm này trong mã bằng cách định nghĩa một enum `IpAddrKind` và liệt kê
các loại địa chỉ IP có thể có, `V4` và `V6`. Đây là các biến thể của enum:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` hiện là một kiểu dữ liệu tùy chỉnh mà chúng ta có thể sử dụng ở những nơi khác trong mã của mình.

### Các giá trị Enum

Chúng ta có thể tạo các thực thể (instances) của mỗi loại trong hai biến thể của `IpAddrKind` như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Lưu ý rằng các biến thể của enum được đặt trong không gian tên (namespaced) dưới tên định danh của nó,
và chúng ta sử dụng dấu hai chấm kép để ngăn cách hai phần. Điều này hữu ích vì giờ đây cả hai giá trị
`IpAddrKind::V4` và `IpAddrKind::V6` đều thuộc cùng một kiểu: `IpAddrKind`. Sau đó, chẳng hạn, chúng ta
có thể định nghĩa một hàm nhận bất kỳ `IpAddrKind` nào:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Và chúng ta có thể gọi hàm này với bất kỳ biến thể nào:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Sử dụng enum thậm chí còn có nhiều ưu điểm hơn. Suy nghĩ kỹ hơn về kiểu địa chỉ IP của chúng ta, tại
thời điểm này chúng ta chưa có cách nào để lưu trữ _dữ liệu_ địa chỉ IP thực tế; chúng ta chỉ biết nó
thuộc _loại_ nào. Với những gì bạn vừa học về struct trong Chương 5, bạn có thể muốn giải quyết vấn đề
này bằng struct như trong Liệt kê 6-1.

```aquascope,interpreter
#fn main() {
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};`[]`
#}
```

Ở đây, chúng ta đã định nghĩa một struct `IpAddr` có hai trường: một trường `kind` thuộc kiểu
`IpAddrKind` (enum chúng ta đã định nghĩa trước đó) và một trường `address` thuộc kiểu `String`.
Chúng ta có hai thực thể của struct này. Thực thể đầu tiên là `home`, và nó có giá trị `IpAddrKind::V4`
cho trường `kind` với dữ liệu địa chỉ liên kết là `127.0.0.1`. Thực thể thứ hai là `loopback`. Nó có
biến thể khác của `IpAddrKind` cho giá trị `kind` là `V6`, và có địa chỉ `::1` liên kết với nó. Chúng
ta đã sử dụng một struct để gộp các giá trị `kind` và `address` lại với nhau, vì vậy bây giờ biến thể
đã được liên kết với giá trị.

Tuy nhiên, việc biểu thị cùng một khái niệm bằng cách chỉ sử dụng một enum sẽ súc tích hơn: thay vì một
enum bên trong một struct, chúng ta có thể đưa dữ liệu trực tiếp vào mỗi biến thể enum. Định nghĩa mới
này của enum `IpAddr` nói rằng cả hai biến thể `V4` và `V6` sẽ có các giá trị `String` liên kết:

```aquascope,interpreter
#fn main() {
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));`[]`
#}
```

Chúng ta đính kèm dữ liệu vào từng biến thể của enum một cách trực tiếp, vì vậy không cần thêm một
struct phụ nào nữa. Ở đây, cũng dễ dàng thấy một chi tiết khác về cách enum hoạt động: tên của mỗi
biến thể enum mà chúng ta định nghĩa cũng trở thành một hàm khởi tạo một thực thể của enum. Nghĩa là,
`IpAddr::V4()` là một lời gọi hàm nhận một đối số `String` và trả về một thực thể của kiểu `IpAddr`.
Chúng ta tự động có hàm khởi tạo này được định nghĩa như một kết quả của việc định nghĩa enum.

Có một ưu điểm khác khi sử dụng enum thay vì struct: mỗi biến thể có thể có các kiểu và lượng dữ liệu
liên kết khác nhau. Địa chỉ IP phiên bản bốn sẽ luôn có bốn thành phần số có giá trị từ 0 đến 255. Nếu
chúng ta muốn lưu trữ địa chỉ `V4` dưới dạng bốn giá trị `u8` nhưng vẫn biểu thị địa chỉ `V6` dưới dạng
một giá trị `String`, chúng ta sẽ không thể làm điều đó với một struct. Enum xử lý trường hợp này một cách dễ dàng:

```aquascope,interpreter
#fn main() {
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));`[]`
#}

```

Chúng ta đã chỉ ra một vài cách khác nhau để định nghĩa các cấu trúc dữ liệu nhằm lưu trữ địa chỉ IP
phiên bản bốn và phiên bản sáu. Tuy nhiên, hóa ra việc muốn lưu trữ địa chỉ IP và mã hóa loại địa chỉ
là rất phổ biến nên [thư viện tiêu chuẩn đã có một định nghĩa mà chúng ta có thể sử dụng!][IpAddr] Hãy
xem cách thư viện tiêu chuẩn định nghĩa `IpAddr`: nó có chính xác enum và các biến thể mà chúng ta đã
định nghĩa và sử dụng, nhưng nó nhúng dữ liệu địa chỉ bên trong các biến thể dưới dạng hai struct khác
nhau, được định nghĩa khác nhau cho mỗi biến thể:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Mã này minh họa rằng bạn có thể đặt bất kỳ loại dữ liệu nào bên trong một biến thể enum: ví dụ như chuỗi,
kiểu số hoặc struct. Bạn thậm chí có thể bao gồm một enum khác! Ngoài ra, các kiểu trong thư viện tiêu
chuẩn thường không phức tạp hơn nhiều so với những gì bạn có thể nghĩ ra.

Lưu ý rằng mặc dù thư viện tiêu chuẩn chứa một định nghĩa cho `IpAddr`, chúng ta vẫn có thể tạo và sử
dụng định nghĩa của riêng mình mà không có xung đột vì chúng ta chưa đưa định nghĩa của thư viện tiêu
chuẩn vào phạm vi của mình. Chúng ta sẽ nói nhiều hơn về việc đưa các kiểu vào phạm vi trong Chương 7.

Hãy xem một ví dụ khác về enum trong Liệt kê 6-2: ví dụ này có nhiều kiểu khác nhau được nhúng trong
các biến thể của nó.

<Listing number="6-2" caption="Một enum `Message` có các biến thể lưu trữ lượng và kiểu giá trị khác nhau">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Enum này có bốn biến thể với các kiểu khác nhau:

- `Quit`: Không có dữ liệu nào liên kết với nó
- `Move`: Có các trường được đặt tên, giống như một struct
- `Write`: Bao gồm một `String` duy nhất
- `ChangeColor`: Bao gồm ba giá trị `i32`

Định nghĩa một enum với các biến thể như trong Liệt kê 6-2 tương tự như việc định nghĩa các loại định
nghĩa struct khác nhau, ngoại trừ việc enum không sử dụng từ khóa `struct` và tất cả các biến thể được
nhóm lại với nhau dưới kiểu `Message`. Các struct sau đây có thể giữ cùng một dữ liệu mà các biến thể
enum trước đó nắm giữ:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Nhưng nếu chúng ta sử dụng các struct khác nhau, mỗi struct có kiểu riêng của nó, chúng ta không thể dễ
dàng định nghĩa một hàm để nhận bất kỳ loại thông điệp nào trong số này như chúng ta có thể làm với enum
`Message` được định nghĩa trong Liệt kê 6-2, vốn là một kiểu duy nhất.

Có thêm một điểm tương đồng nữa giữa enum và struct: giống như việc chúng ta có thể định nghĩa các phương
thức trên struct bằng cách sử dụng `impl`, chúng ta cũng có thể định nghĩa các phương thức trên enum.
Đây là một phương thức tên là `call` mà chúng ta có thể định nghĩa trên enum `Message` của mình:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

Thân của phương thức sẽ sử dụng `self` để lấy giá trị mà chúng ta đã gọi phương thức trên đó. Trong ví
dụ này, chúng ta đã tạo một biến `m` có giá trị `Message::Write(String::from("hello"))`, và đó là những
gì `self` sẽ là trong thân của phương thức `call` khi `m.call()` chạy.

Hãy cùng xem một enum khác trong thư viện tiêu chuẩn rất phổ biến và hữu ích: `Option`.

### Enum `Option` và những ưu điểm của nó so với giá trị Null

Phần này khám phá một nghiên cứu điển hình về `Option`, đây là một enum khác được định nghĩa bởi thư viện
tiêu chuẩn. Kiểu `Option` mã hóa kịch bản rất phổ biến trong đó một giá trị có thể là một thứ gì đó hoặc
nó có thể không là gì cả.

Ví dụ, nếu bạn yêu cầu mục đầu tiên trong một danh sách không trống, bạn sẽ nhận được một giá trị. Nếu bạn
yêu cầu mục đầu tiên trong một danh sách trống, bạn sẽ không nhận được gì. Việc diễn đạt khái niệm này dưới
dạng hệ thống kiểu dữ liệu có nghĩa là trình biên dịch có thể kiểm tra xem bạn đã xử lý tất cả các trường
hợp mà bạn nên xử lý chưa; chức năng này có thể ngăn chặn các lỗi cực kỳ phổ biến trong các ngôn ngữ lập
trình khác.

Thiết kế ngôn ngữ lập trình thường được nghĩ đến dưới góc độ những tính năng nào bạn đưa vào, nhưng những
tính năng bạn loại bỏ cũng quan trọng không kém. Rust không có tính năng null như nhiều ngôn ngữ khác.
_Null_ là một giá trị có nghĩa là không có giá trị nào ở đó. Trong các ngôn ngữ có null, các biến luôn có
thể ở một trong hai trạng thái: null hoặc không-null (not-null).

Trong bài thuyết trình năm 2009 của mình "Null References: The Billion Dollar Mistake" (Tham chiếu Null: Sai lầm tỷ đô),
Tony Hoare, người phát minh ra null, đã nói thế này:

> Tôi gọi đó là sai lầm tỷ đô của mình. Vào thời điểm đó, tôi đang thiết kế hệ thống kiểu toàn diện đầu
> tiên cho các tham chiếu trong một ngôn ngữ hướng đối tượng. Mục tiêu của tôi là đảm bảo rằng tất cả các
> cách sử dụng tham chiếu phải tuyệt đối an toàn, với việc kiểm tra được thực hiện tự động bởi trình biên
> dịch. Nhưng tôi đã không thể cưỡng lại cám dỗ đưa vào một tham chiếu null, chỉ đơn giản vì nó quá dễ
> thực hiện. Điều này đã dẫn đến vô số lỗi, lỗ hổng bảo mật và sự cố hệ thống, có lẽ đã gây ra một tỷ đô
> la đau đớn và thiệt hại trong bốn mươi năm qua.

Vấn đề với các giá trị null là nếu bạn cố gắng sử dụng một giá trị null như một giá trị không-null, bạn sẽ
gặp một lỗi nào đó. Bởi vì thuộc tính null hoặc không-null này rất phổ biến, nên cực kỳ dễ mắc phải loại
lỗi này.

Tuy nhiên, khái niệm mà null đang cố gắng diễn đạt vẫn là một khái niệm hữu ích: null là một giá trị hiện
đang không hợp lệ hoặc vắng mặt vì một lý do nào đó.

Vấn đề thực sự không nằm ở khái niệm mà nằm ở việc triển khai cụ thể. Vì vậy, Rust không có null, nhưng
nó có một enum có thể mã hóa khái niệm về một giá trị hiện diện hoặc vắng mặt. Enum này là `Option<T>`,
và nó được [định nghĩa bởi thư viện tiêu chuẩn][option] như sau:

```rust
enum Option<T> {
    None,
    Some(T),
    }
```

Enum `Option<T>` hữu ích đến mức nó thậm chí còn được đưa vào prelude; bạn không cần phải đưa nó vào phạm
vi một cách rõ ràng. Các biến thể của nó cũng được đưa vào prelude: bạn có thể sử dụng trực tiếp `Some` và
`None` mà không cần tiền tố `Option::`. Enum `Option<T>` vẫn chỉ là một enum bình thường, và `Some(T)` và
`None` vẫn là các biến thể của kiểu `Option<T>`.

Cú pháp `<T>` là một tính năng của Rust mà chúng ta chưa nói tới. Đó là một tham số kiểu generic, và chúng
ta sẽ tìm hiểu kỹ hơn về generic trong Chương 10. Hiện tại, tất cả những gì bạn cần biết là `<T>` có nghĩa
là biến thể `Some` của enum `Option` có thể giữ một phần dữ liệu của bất kỳ kiểu nào, và mỗi kiểu cụ thể
được sử dụng thay cho `T` sẽ làm cho kiểu `Option<T>` tổng thể trở thành một kiểu khác nhau. Dưới đây là
một số ví dụ về việc sử dụng các giá trị `Option` để giữ các kiểu số và kiểu ký tự:

```aquascope,interpreter
#fn main() {
let some_number = Some(5);
let some_char = Some('e');

let absent_number: Option<i32> = None;`[]`
#}
```

Kiểu của `some_number` là `Option<i32>`. Kiểu của `some_char` là `Option<char>`, đây là một kiểu khác.
Rust có thể suy luận các kiểu này vì chúng ta đã chỉ định một giá trị bên trong biến thể `Some`. Đối với
`absent_number`, Rust yêu cầu chúng ta chú thích kiểu `Option` tổng thể: trình biên dịch không thể suy
luận kiểu mà biến thể `Some` tương ứng sẽ giữ chỉ bằng cách nhìn vào một giá trị `None`. Ở đây, chúng ta
nói với Rust rằng chúng ta muốn `absent_number` có kiểu `Option<i32>`.

Khi chúng ta có một giá trị `Some`, chúng ta biết rằng một giá trị hiện diện và giá trị đó được giữ trong
`Some`. Khi chúng ta có một giá trị `None`, theo một nghĩa nào đó, nó có nghĩa tương tự như null: chúng ta
không có một giá trị hợp lệ. Vậy tại sao việc có `Option<T>` lại tốt hơn việc có null?

Nói ngắn gọn, vì `Option<T>` và `T` (trong đó `T` có thể là bất kỳ kiểu nào) là các kiểu khác nhau, trình
biên dịch sẽ không cho phép chúng ta sử dụng một giá trị `Option<T>` như thể nó chắc chắn là một giá trị
hợp lệ. Ví dụ, mã này sẽ không biên dịch được, vì nó đang cố gắng cộng một `i8` với một `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Nếu chúng ta chạy mã này, chúng ta sẽ nhận được một thông báo lỗi như thế này:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Thật dữ dội! Thực tế, thông báo lỗi này có nghĩa là Rust không hiểu cách cộng một `i8` và một `Option<i8>`,
bởi vì chúng là các kiểu khác nhau. Khi chúng ta có một giá trị thuộc một kiểu như `i8` trong Rust, trình
biên dịch sẽ đảm bảo rằng chúng ta luôn có một giá trị hợp lệ. Chúng ta có thể tự tin tiếp tục mà không
cần phải kiểm tra null trước khi sử dụng giá trị đó. Chỉ khi chúng ta có một `Option<i8>` (hoặc bất kỳ
kiểu giá trị nào mà chúng ta đang làm việc cùng), chúng ta mới phải lo lắng về việc có thể không có giá
trị, và trình biên dịch sẽ đảm bảo rằng chúng ta xử lý trường hợp đó trước khi sử dụng giá trị.

Nói cách khác, bạn phải chuyển đổi một `Option<T>` thành một `T` trước khi bạn có thể thực hiện các thao
tác `T` với nó. Nhìn chung, điều này giúp bắt được một trong những vấn đề phổ biến nhất với null: giả định
rằng một thứ gì đó không phải là null trong khi thực tế nó là null.

Việc loại bỏ rủi ro khi giả định sai về một giá trị không-null giúp bạn tự tin hơn vào mã của mình. Để có
một giá trị có thể là null, bạn phải tham gia một cách rõ ràng bằng cách đặt kiểu của giá trị đó là
`Option<T>`. Sau đó, khi bạn sử dụng giá trị đó, bạn được yêu cầu xử lý rõ ràng trường hợp giá trị là null.
Ở bất kỳ nơi nào mà một giá trị có kiểu không phải là `Option<T>`, bạn _có thể_ yên tâm giả định rằng giá
trị đó không phải là null. Đây là một quyết định thiết kế có chủ ý của Rust nhằm hạn chế sự phổ biến của
null và tăng tính an toàn của mã nguồn Rust.

Vậy làm thế nào để bạn lấy giá trị `T` ra khỏi biến thể `Some` khi bạn có một giá trị kiểu `Option<T>` để
bạn có thể sử dụng giá trị đó? Enum `Option<T>` có một số lượng lớn các phương thức hữu ích trong nhiều
tình huống khác nhau; bạn có thể xem chúng trong [tài liệu của nó][docs]. Làm quen với các phương thức
trên `Option<T>` sẽ cực kỳ hữu ích trong hành trình của bạn với Rust.

Nhìn chung, để sử dụng một giá trị `Option<T>`, bạn muốn có mã sẽ xử lý từng biến thể. Bạn muốn một số mã
chỉ chạy khi bạn có giá trị `Some(T)`, và mã này được phép sử dụng `T` bên trong. Bạn muốn một số mã khác
chỉ chạy nếu bạn có giá trị `None`, và mã đó không có sẵn giá trị `T`. Biểu thức `match` là một cấu trúc
điều khiển luồng thực hiện chính xác điều này khi được sử dụng với enum: nó sẽ chạy các mã khác nhau tùy
thuộc vào biến thể nào của enum mà nó có, và mã đó có thể sử dụng dữ liệu bên trong giá trị khớp.

{{#quiz ../quizzes/ch06-01-defining-an-enum.toml}}

[IpAddr]: https://doc.rust-lang.org/std/net/enum.IpAddr.html
[option]: https://doc.rust-lang.org/std/option/enum.Option.html
[docs]: https://doc.rust-lang.org/std/option/enum.Option.html
