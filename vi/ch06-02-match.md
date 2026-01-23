<!-- Old heading. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## Cấu trúc điều khiển luồng `match`

Rust có một cấu trúc điều khiển luồng cực kỳ mạnh mẽ gọi là `match`, cho phép bạn so sánh một giá trị với một loạt các mẫu (patterns) và sau đó thực thi mã dựa trên mẫu nào khớp. Các mẫu có thể được tạo thành từ các giá trị hằng (literal values), tên biến, ký tự đại diện (wildcards) và nhiều thứ khác; [Chương 19][ch19-00-patterns]<!-- ignore --> sẽ đề cập đến tất cả các loại mẫu khác nhau và chức năng của chúng. Sức mạnh của `match` đến từ tính biểu đạt của các mẫu và thực tế là trình biên dịch xác nhận rằng tất cả các trường hợp có thể xảy ra đều đã được xử lý.

Hãy nghĩ về một biểu thức `match` giống như một máy phân loại tiền xu: các đồng xu trượt xuống một đường ray với các lỗ có kích thước khác nhau dọc theo nó, và mỗi đồng xu sẽ rơi qua cái lỗ đầu tiên mà nó vừa vặn. Theo cách tương tự, các giá trị đi qua từng mẫu trong một `match`, và tại mẫu đầu tiên mà giá trị "khớp", giá trị đó sẽ rơi vào khối mã liên kết để được sử dụng trong quá trình thực thi.

Nói về tiền xu, hãy sử dụng chúng làm ví dụ cho việc sử dụng `match`! Chúng ta có thể viết một hàm nhận một đồng xu Hoa Kỳ chưa biết và, theo cách tương tự như máy đếm tiền, xác định đó là đồng xu nào và trả về giá trị của nó tính bằng cent (xu), như được hiển thị trong Liệt kê 6-3.

<Listing number="6-3" caption="Một enum và một biểu thức `match` có các biến thể của enum làm các mẫu của nó">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

Hãy phân tích lệnh `match` trong hàm `value_in_cents`. Đầu tiên, chúng ta liệt kê từ khóa `match` theo sau là một biểu thức, trong trường hợp này là giá trị `coin`. Điều này có vẻ rất giống với một biểu thức điều kiện được sử dụng với `if`, nhưng có một sự khác biệt lớn: với `if`, điều kiện cần phải trả về một giá trị Boolean, nhưng ở đây nó có thể là bất kỳ kiểu dữ liệu nào. Kiểu của `coin` trong ví dụ này là enum `Coin` mà chúng ta đã định nghĩa ở dòng đầu tiên.

Tiếp theo là các nhánh (arms) của `match`. Một nhánh có hai phần: một mẫu và một đoạn mã. Nhánh đầu tiên ở đây có một mẫu là giá trị `Coin::Penny` và sau đó là toán tử `=>` ngăn cách mẫu và đoạn mã sẽ chạy. Đoạn mã trong trường hợp này chỉ là giá trị `1`. Mỗi nhánh được ngăn cách với nhánh tiếp theo bằng một dấu phẩy.

Khi biểu thức `match` thực thi, nó so sánh giá trị kết quả với mẫu của từng nhánh, theo thứ tự. Nếu một mẫu khớp với giá trị, đoạn mã liên kết với mẫu đó sẽ được thực thi. Nếu mẫu đó không khớp với giá trị, quá trình thực thi sẽ tiếp tục đến nhánh tiếp theo, giống như trong máy phân loại tiền xu. Chúng ta có thể có bao nhiêu nhánh tùy thích: trong Liệt kê 6-3, biểu thức `match` của chúng ta có bốn nhánh.

Đoạn mã liên kết với mỗi nhánh là một biểu thức, và giá trị kết quả của biểu thức trong nhánh khớp là giá trị được trả về cho toàn bộ biểu thức `match`.

Chúng ta thường không sử dụng dấu ngoặc nhọn nếu mã của nhánh match ngắn, như trong Liệt kê 6-3, nơi mỗi nhánh chỉ trả về một giá trị. Nếu bạn muốn chạy nhiều dòng mã trong một nhánh match, bạn phải sử dụng dấu ngoặc nhọn, và dấu phẩy sau nhánh đó khi đó là tùy chọn. Ví dụ, đoạn mã sau sẽ in "Lucky penny!" mỗi khi phương thức được gọi với `Coin::Penny`, nhưng vẫn trả về giá trị cuối cùng của khối mã là `1`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Các mẫu liên kết với giá trị

Một tính năng hữu ích khác của các nhánh match là chúng có thể liên kết (bind) với các phần của giá trị khớp với mẫu. Đây là cách chúng ta có thể trích xuất các giá trị ra khỏi các biến thể của enum.

Ví dụ, hãy thay đổi một trong các biến thể enum của chúng ta để giữ dữ liệu bên trong nó. Từ năm 1999 đến năm 2008, Hoa Kỳ đã đúc các đồng quarter (25 xu) với các thiết kế khác nhau cho mỗi trong số 50 tiểu bang ở một mặt. Không có đồng xu nào khác có thiết kế tiểu bang, vì vậy chỉ có đồng quarter mới có giá trị bổ sung này. Chúng ta có thể thêm thông tin này vào `enum` của mình bằng cách thay đổi biến thể `Quarter` để bao gồm một giá trị `UsState` được lưu trữ bên trong nó, như chúng ta đã làm trong Liệt kê 6-4.

<Listing number="6-4" caption="Một enum `Coin` trong đó biến thể `Quarter` cũng giữ một giá trị `UsState` ">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

Hãy tưởng tượng rằng một người bạn đang cố gắng thu thập tất cả 50 đồng quarter của các tiểu bang. Trong khi chúng ta phân loại tiền lẻ theo loại đồng xu, chúng ta cũng sẽ gọi tên tiểu bang liên kết với mỗi đồng quarter để nếu đó là đồng xu mà bạn của chúng ta chưa có, họ có thể thêm nó vào bộ sưu tập của mình.

Trong biểu thức match cho đoạn mã này, chúng ta thêm một biến gọi là `state` vào mẫu khớp với các giá trị của biến thể `Coin::Quarter`. Khi một `Coin::Quarter` khớp, biến `state` sẽ liên kết với giá trị tiểu bang của đồng quarter đó. Sau đó, chúng ta có thể sử dụng `state` trong mã cho nhánh đó, như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Nếu chúng ta gọi `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` sẽ là `Coin::Quarter(UsState::Alaska)`. Khi chúng ta so sánh giá trị đó với từng nhánh match, không có nhánh nào khớp cho đến khi chúng ta đến `Coin::Quarter(state)`. Tại thời điểm đó, liên kết cho `state` sẽ là giá trị `UsState::Alaska`. Sau đó, chúng ta có thể sử dụng liên kết đó trong biểu thức `println!`, từ đó lấy được giá trị tiểu bang bên trong từ biến thể enum `Coin` cho `Quarter`.

### Khớp với `Option<T>`

Trong phần trước, chúng ta muốn lấy giá trị `T` bên trong ra khỏi trường hợp `Some` khi sử dụng `Option<T>`; chúng ta cũng có thể xử lý `Option<T>` bằng cách sử dụng `match`, giống như chúng ta đã làm với enum `Coin`! Thay vì so sánh các đồng xu, chúng ta sẽ so sánh các biến thể của `Option<T>`, nhưng cách biểu thức `match` hoạt động vẫn giữ nguyên.

Giả sử chúng ta muốn viết một hàm nhận một `Option<i32>` và nếu có một giá trị bên trong, nó sẽ cộng thêm 1 vào giá trị đó. Nếu không có giá trị bên trong, hàm sẽ trả về giá trị `None` và không cố gắng thực hiện bất kỳ thao tác nào.

Hàm này rất dễ viết nhờ có `match`, và sẽ trông giống như Liệt kê 6-5.

<Listing number="6-5" caption="Một hàm sử dụng biểu thức `match` trên một `Option<i32>`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

Hãy xem xét lần thực thi đầu tiên của `plus_one` chi tiết hơn. Khi chúng ta gọi `plus_one(five)`, biến `x` trong thân của `plus_one` sẽ có giá trị `Some(5)`. Sau đó, chúng ta so sánh giá trị đó với từng nhánh match:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Giá trị `Some(5)` không khớp với mẫu `None`, vì vậy chúng ta tiếp tục đến nhánh tiếp theo:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` có khớp với `Some(i)` không? Có! Chúng ta có cùng một biến thể. Biến `i` liên kết với giá trị chứa trong `Some`, vì vậy `i` nhận giá trị `5`. Mã trong nhánh match sau đó được thực thi, vì vậy chúng ta cộng thêm 1 vào giá trị của `i` và tạo ra một giá trị `Some` mới với tổng `6` của chúng ta bên trong.

Bây giờ hãy xem xét lần gọi thứ hai của `plus_one` trong Liệt kê 6-5, nơi `x` là `None`. Chúng ta đi vào `match` và so sánh với nhánh đầu tiên:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Nó khớp! Không có giá trị nào để cộng vào, vì vậy chương trình dừng lại và trả về giá trị `None` ở phía bên phải của `=>`. Vì nhánh đầu tiên đã khớp, không có nhánh nào khác được so sánh.

Kết hợp `match` và enum là hữu ích trong nhiều tình huống. Bạn sẽ thấy mẫu này rất nhiều trong mã Rust: `match` với một enum, liên kết một biến với dữ liệu bên trong, và sau đó thực thi mã dựa trên nó. Lúc đầu có thể hơi khó khăn, nhưng khi bạn đã quen với nó, bạn sẽ ước mình có nó trong tất cả các ngôn ngữ. Nó luôn là một trong những tính năng yêu thích của người dùng.

### Các phép khớp mẫu phải mang tính toàn diện

Có một khía cạnh khác của `match` mà chúng ta cần thảo luận: các mẫu của các nhánh phải bao phủ tất cả các khả năng. Hãy xem xét phiên bản này của hàm `plus_one` của chúng ta, nó có lỗi và sẽ không biên dịch được:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Chúng ta đã không xử lý trường hợp `None`, vì vậy đoạn mã này sẽ gây ra lỗi. May mắn thay, đó là một lỗi mà Rust biết cách bắt được. Nếu chúng ta cố gắng biên dịch đoạn mã này, chúng ta sẽ nhận được lỗi này:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust biết rằng chúng ta đã không bao quát mọi trường hợp có thể xảy ra, và thậm chí còn biết mẫu nào chúng ta đã quên! Các phép khớp mẫu (matches) trong Rust là _toàn diện (exhaustive)_: chúng ta phải vét cạn mọi khả năng cuối cùng để mã hợp lệ. Đặc biệt trong trường hợp của `Option<T>`, khi Rust ngăn chúng ta quên xử lý rõ ràng trường hợp `None`, nó bảo vệ chúng ta khỏi việc giả định rằng chúng ta có một giá trị khi chúng ta thực sự có thể có null, do đó làm cho sai lầm tỷ đô đã thảo luận trước đó trở nên bất khả thi.

### Các mẫu bắt-tất-cả (Catch-All) và trình giữ chỗ `_`

Sử dụng enum, chúng ta cũng có thể thực hiện các hành động đặc biệt cho một vài giá trị cụ thể, nhưng đối với tất cả các giá trị khác, hãy thực hiện một hành động mặc định. Hãy tưởng tượng chúng ta đang triển khai một trò chơi mà ở đó, nếu bạn tung xúc xắc được 3, người chơi của bạn sẽ không di chuyển mà thay vào đó nhận được một chiếc mũ mới lạ mắt. Nếu bạn tung được 7, người chơi của bạn sẽ mất một chiếc mũ lạ mắt. Đối với tất cả các giá trị khác, người chơi của bạn sẽ di chuyển đúng số ô đó trên bàn cờ. Đây là một `match` triển khai logic đó, với kết quả tung xúc xắc được ghi cứng (hardcoded) thay vì là một giá trị ngẫu nhiên, và tất cả các logic khác được đại diện bằng các hàm không có thân vì việc triển khai thực tế chúng nằm ngoài phạm vi của ví dụ này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

Đối với hai nhánh đầu tiên, các mẫu là các giá trị hằng `3` và `7`. Đối với nhánh cuối cùng bao quát mọi giá trị khả thi khác, mẫu là biến mà chúng ta đã chọn đặt tên là `other`. Mã chạy cho nhánh `other` sử dụng biến đó bằng cách truyền nó vào hàm `move_player`.

Đoạn mã này biên dịch được, mặc dù chúng ta chưa liệt kê tất cả các giá trị có thể có của một `u8`, bởi vì mẫu cuối cùng sẽ khớp với tất cả các giá trị không được liệt kê cụ thể. Mẫu bắt-tất-cả (catch-all) này đáp ứng yêu cầu rằng `match` phải mang tính toàn diện. Lưu ý rằng chúng ta phải đặt nhánh bắt-tất-cả ở cuối cùng vì các mẫu được đánh giá theo thứ tự. Nếu chúng ta đặt nhánh bắt-tất-cả lên trước, các nhánh khác sẽ không bao giờ được chạy, vì vậy Rust sẽ cảnh báo chúng ta nếu chúng ta thêm các nhánh sau một nhánh bắt-tất-cả!

Rust cũng có một mẫu mà chúng ta có thể sử dụng khi chúng ta muốn một mẫu bắt-tất-cả nhưng không muốn _sử dụng_ giá trị trong mẫu đó: `_` là một mẫu đặc biệt khớp với bất kỳ giá trị nào và không liên kết với giá trị đó. Điều này nói với Rust rằng chúng ta sẽ không sử dụng giá trị đó, vì vậy Rust sẽ không cảnh báo chúng ta về một biến không được sử dụng.

Hãy thay đổi luật chơi: bây giờ, nếu bạn tung bất cứ thứ gì khác ngoài 3 hoặc 7, bạn phải tung lại. Chúng ta không còn cần sử dụng giá trị bắt-tất-cả nữa, vì vậy chúng ta có thể thay đổi mã của mình để sử dụng `_` thay vì biến có tên `other`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Ví dụ này cũng đáp ứng yêu cầu về tính toàn diện vì chúng ta đang bỏ qua một cách rõ ràng tất cả các giá trị khác trong nhánh cuối cùng; chúng ta đã không bỏ quên bất cứ thứ gì.

Cuối cùng, chúng ta sẽ thay đổi luật chơi một lần nữa để không có gì khác xảy ra trong lượt của bạn nếu bạn tung được bất cứ thứ gì khác ngoài 3 hoặc 7. Chúng ta có thể diễn đạt điều đó bằng cách sử dụng giá trị đơn vị (unit value - kiểu tuple trống mà chúng ta đã đề cập trong phần [“Kiểu Tuple”][tuples]<!-- ignore -->) làm mã đi kèm với nhánh `_`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Ở đây, chúng ta đang nói với Rust một cách rõ ràng rằng chúng ta sẽ không sử dụng bất kỳ giá trị nào khác không khớp với một mẫu trong các nhánh trước đó, và chúng ta không muốn chạy bất kỳ đoạn mã nào trong trường hợp này.

Còn nhiều điều về các mẫu và phép khớp mẫu mà chúng ta sẽ đề cập trong [Chương 19][ch19-00-patterns]<!-- ignore -->.

<!-- BEGIN INTERVENTION: 1e4f082c-ffa4-4d33-8726-2dbcd72e1aa2 -->

### Cách các phép khớp mẫu tương tác với Quyền sở hữu

Nếu một enum chứa dữ liệu không thể sao chép (non-copyable) như một String, thì bạn nên cẩn thận với việc liệu một phép match sẽ di chuyển (move) hay mượn (borrow) dữ liệu đó. Ví dụ, chương trình sử dụng `Option<String>` này sẽ biên dịch được:

```aquascope,permissions,stepper,boundaries
# fn main() {
let opt: Option<String> =
    Some(String::from("Hello world"));

match opt {
    Some(_) => println!("Some!"),
    None => println!("None!")
};

println!("{:?}", opt);
# }
```

Nhưng nếu chúng ta thay thế trình giữ chỗ trong `Some(_)` bằng một tên biến, như `Some(s)`, thì chương trình sẽ KHÔNG biên dịch được:

```aquascope,permissions,stepper,boundaries,shouldFail
#fn main() {
let opt: Option<String> =
    Some(String::from("Hello world"));

match opt {
    // _ trở thành s
    Some(s) => println!("Some: {}", s),
    None => println!("None!")
};

println!("{:?}", opt);`{}`
#}
```

`opt` là một enum thông thường — kiểu của nó là `Option<String>` chứ không phải là một tham chiếu như `&Option<String>`. Do đó, một phép match trên `opt` sẽ di chuyển các trường không bị bỏ qua như `s`. Hãy chú ý cách `opt` mất quyền đọc và quyền sở hữu sớm hơn trong chương trình thứ hai so với chương trình thứ nhất. Sau biểu thức match, dữ liệu bên trong `opt` đã bị di chuyển, vì vậy việc đọc `opt` trong lệnh `println` là không hợp lệ.

Nếu chúng ta muốn xem nội dung bên trong `opt` mà không di chuyển nội dung của nó, giải pháp theo quy ước (idiomatic) là thực hiện match trên một tham chiếu:

```aquascope,permissions,stepper,boundaries
#fn main() {
let opt: Option<String> =
    Some(String::from("Hello world"));

// opt trở thành &opt
match &opt {
    Some(s) => println!("Some: {}", s),
    None => println!("None!")
};

println!("{:?}", opt);
#}
```

Rust sẽ “đẩy xuống” (push down) tham chiếu từ enum bên ngoài, `&Option<String>`, đến trường bên trong, `&String`. Do đó `s` có kiểu `&String`, và `opt` có thể được sử dụng sau phép match. Để hiểu rõ hơn về cơ chế “đẩy xuống” này, hãy xem phần về [chế độ liên kết (binding modes)](https://doc.rust-lang.org/reference/patterns.html#binding-modes) trong Tài liệu tham khảo Rust (Rust Reference).

<!-- END INTERVENTION -->

{{#quiz ../quizzes/ch06-02-match.toml}}

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
