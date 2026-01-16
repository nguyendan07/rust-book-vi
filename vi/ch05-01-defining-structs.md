## Định nghĩa và Khởi tạo Struct

Struct tương tự như tuple, đã được thảo luận trong phần ["Kiểu Tuple"][tuples]<!--
ignore -->, ở chỗ cả hai đều chứa nhiều giá trị liên quan. Giống như tuple, các
thành phần của một struct có thể là các kiểu khác nhau. Không giống như tuple, trong một struct
bạn sẽ đặt tên cho từng phần dữ liệu để ý nghĩa của các giá trị trở nên rõ ràng. Việc thêm các
tên này có nghĩa là struct linh hoạt hơn tuple: bạn không cần phải dựa
vào thứ tự của dữ liệu để chỉ định hoặc truy cập các giá trị của một thể hiện.

Để định nghĩa một struct, chúng ta nhập từ khóa `struct` và đặt tên cho toàn bộ struct.
Tên của một struct nên mô tả ý nghĩa của các phần dữ liệu đang được
nhóm lại với nhau. Sau đó, bên trong dấu ngoặc nhọn, chúng ta định nghĩa tên và kiểu
của các phần dữ liệu, mà chúng ta gọi là _trường_ (field). Ví dụ, Listing 5-1 hiển thị một
struct lưu trữ thông tin về một tài khoản người dùng.

<Listing number="5-1" file-name="src/main.rs" caption="Định nghĩa struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Để sử dụng một struct sau khi chúng ta đã định nghĩa nó, chúng ta tạo một _thể hiện_ (instance) của struct đó
bằng cách chỉ định các giá trị cụ thể cho từng trường. Chúng ta tạo một thể hiện bằng cách
nêu tên của struct và sau đó thêm các dấu ngoặc nhọn chứa các cặp _`khóa:
giá trị`_, trong đó khóa là tên của các trường và giá trị là
dữ liệu chúng ta muốn lưu trữ trong các trường đó. Chúng ta không cần phải chỉ định các trường
theo cùng thứ tự mà chúng ta đã khai báo chúng trong struct. Nói cách khác,
định nghĩa struct giống như một khuôn mẫu chung cho kiểu dữ liệu, và các thể hiện điền
vào khuôn mẫu đó với dữ liệu cụ thể để tạo ra các giá trị của kiểu.
Ví dụ, chúng ta có thể khai báo một người dùng cụ thể như được hiển thị trong Listing 5-2.

```aquascope,interpreter
#struct User {
#    active: bool,
#    username: String,
#    email: String,
#    sign_in_count: u64,
#}
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };`[]`
}
```

Để lấy một giá trị cụ thể từ một struct, chúng ta sử dụng ký pháp dấu chấm. Ví dụ, để
truy cập địa chỉ email của người dùng này, chúng ta sử dụng `user1.email`. Nếu thể hiện là
có thể thay đổi (mutable), chúng ta có thể thay đổi một giá trị bằng cách sử dụng ký pháp dấu chấm và gán vào
một trường cụ thể. Listing 5-3 chỉ ra cách thay đổi giá trị trong trường `email`
của một thể hiện `User` có thể thay đổi.

```aquascope,interpreter
#struct User {
#    active: bool,
#    username: String,
#    email: String,
#    sign_in_count: u64,
#}
fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };`[]`

    user1.email = String::from("anotheremail@example.com");`[]`
}
```

Lưu ý rằng toàn bộ thể hiện phải có thể thay đổi; Rust không cho phép chúng ta đánh dấu
chỉ một số trường nhất định là có thể thay đổi. Như với bất kỳ biểu thức nào, chúng ta có thể xây dựng một
thể hiện mới của struct như là biểu thức cuối cùng trong thân hàm để
trả về ngầm định thể hiện mới đó.

Listing 5-4 hiển thị một hàm `build_user` trả về một thể hiện `User` với
email và username đã cho. Trường `active` nhận giá trị là `true`, và
`sign_in_count` nhận giá trị là `1`.

<Listing number="5-4" file-name="src/main.rs" caption="Một hàm `build_user` nhận vào email và username và trả về một thể hiện `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Việc đặt tên các tham số hàm trùng với tên các trường của struct là rất hợp lý,
nhưng việc phải lặp lại các tên trường và biến `email` và `username`
hơi tẻ nhạt. Nếu struct có nhiều trường hơn, việc lặp lại từng tên
sẽ càng trở nên phiền toái hơn. May mắn thay, có một cách viết tắt tiện lợi!

<!-- Old heading. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Sử dụng Cú pháp Khởi tạo Trường Rút gọn

Bởi vì tên tham số và tên trường struct hoàn toàn giống nhau
trong Listing 5-4, chúng ta có thể sử dụng cú pháp _khởi tạo trường rút gọn_ (field init shorthand)
để viết lại `build_user` sao cho nó hoạt động chính xác như cũ nhưng không có sự lặp lại
của `username` và `email`, như được hiển thị trong Listing 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="Một hàm `build_user` sử dụng cú pháp khởi tạo trường rút gọn vì tham số `username` và `email` có cùng tên với các trường của struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta đang tạo một thể hiện mới của struct `User`, vốn có một trường
tên là `email`. Chúng ta muốn đặt giá trị của trường `email` thành giá trị trong
tham số `email` của hàm `build_user`. Bởi vì trường `email` và
tham số `email` có cùng tên, chúng ta chỉ cần viết `email` thay vì
`email: email`.

### Tạo Thể hiện từ Thể hiện Khác bằng Cú pháp Cập nhật Struct

Thường rất hữu ích khi tạo một thể hiện mới của một struct bao gồm hầu hết
các giá trị từ một thể hiện khác cùng kiểu, nhưng thay đổi một số giá trị. Bạn có thể làm
điều này bằng cách sử dụng _cú pháp cập nhật struct_ (struct update syntax).

Đầu tiên, trong Listing 5-6 chúng ta chỉ ra cách tạo một thể hiện `User` mới trong `user2`
theo cách thông thường, không dùng cú pháp cập nhật. Chúng ta đặt một giá trị mới cho `email` nhưng
giữ nguyên các giá trị khác từ `user1` mà chúng ta đã tạo trong Listing 5-2.

```aquascope,interpreter
#struct User {
#    active: bool,
#    username: String,
#    email: String,
#    sign_in_count: u64,
#}
fn main() {
#   let user1 = User {
#      email: String::from("someone@example.com"),
#      username: String::from("someusername123"),
#      active: true,
#      sign_in_count: 1,
#   };
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };`[]`
}
```

<span class="caption">Listing 5-6: Tạo một thể hiện `User` mới sử dụng tất cả ngoại trừ một trong
các giá trị từ `user1`</span>

Sử dụng cú pháp cập nhật struct, chúng ta có thể đạt được hiệu quả tương tự với ít mã hơn, như
được hiển thị trong Listing 5-7. Cú pháp `..` chỉ định rằng các trường còn lại không
được đặt giá trị rõ ràng sẽ có cùng giá trị với các trường trong thể hiện đã cho.

<Listing number="5-7" file-name="src/main.rs" caption="Sử dụng cú pháp cập nhật struct để đặt giá trị `email` mới cho một thể hiện `User` nhưng sử dụng phần còn lại của các giá trị từ `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Mã trong Listing 5-7 cũng tạo ra một thể hiện trong `user2` có
giá trị khác cho `email` nhưng có cùng giá trị cho các trường `username`,
`active`, và `sign_in_count` từ `user1`. `..user1` phải đặt ở cuối cùng
để chỉ định rằng bất kỳ trường nào còn lại sẽ nhận giá trị của chúng từ các
trường tương ứng trong `user1`, nhưng chúng ta có thể chọn chỉ định giá trị cho
bao nhiêu trường tùy ý theo bất kỳ thứ tự nào, bất kể thứ tự của các trường trong
định nghĩa của struct.

Lưu ý rằng cú pháp cập nhật struct sử dụng `=` giống như một phép gán; điều này
là do nó di chuyển dữ liệu, giống như chúng ta đã thấy trong phần ["Quyền sở hữu là gì?"][move]<!-- ignore -->. Trong ví dụ này, sau khi tạo `user2`, `user1` bị vô hiệu một phần vì `String` trong
trường `username` của `user1` đã được di chuyển vào `user2`. Nếu chúng ta đã cung cấp cho `user2` các giá trị
`String` mới cho cả `email` và `username`, và do đó chỉ sử dụng các giá trị
`active` và `sign_in_count` từ `user1`, thì `user1` vẫn sẽ
hoàn toàn hợp lệ sau khi tạo `user2`. Các kiểu của `active` và `sign_in_count` là các
kiểu có triển khai trait `Copy`, nên hành vi mà chúng ta đã thảo luận trong
phần ["Sao chép và Di chuyển ra khỏi một Bộ sưu tập"][copy]<!-- ignore --> sẽ được áp dụng.

### Sử dụng Tuple Struct không có Trường Định danh để Tạo các Kiểu Khác nhau

Rust cũng hỗ trợ các struct trông tương tự như tuple, được gọi là _tuple struct_.
Tuple struct có ý nghĩa bổ sung mà tên struct cung cấp nhưng không có
tên gắn liền với các trường của chúng; thay vào đó, chúng chỉ có các kiểu của
các trường. Tuple struct rất hữu ích khi bạn muốn đặt tên cho toàn bộ tuple
và làm cho tuple đó trở thành một kiểu khác biệt so với các tuple khác, và khi việc đặt tên cho từng
trường như trong một struct thông thường sẽ dài dòng hoặc dư thừa.

Để định nghĩa một tuple struct, bắt đầu bằng từ khóa `struct` và tên struct
theo sau là các kiểu trong tuple. Ví dụ, ở đây chúng ta định nghĩa và sử dụng hai
tuple struct có tên là `Color` và `Point`:

<Listing file-name="src/main.rs">

```aquascope,interpreter
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);`[]`
}
```

</Listing>

Lưu ý rằng các giá trị `black` và `origin` là các kiểu khác nhau vì chúng là
các thể hiện của các tuple struct khác nhau. Mỗi struct bạn định nghĩa là một kiểu riêng của nó,
ngay cả khi các trường bên trong struct có thể có cùng kiểu.
Ví dụ, một hàm nhận tham số kiểu `Color` không thể nhận
`Point` làm đối số, mặc dù cả hai kiểu đều được tạo thành từ ba giá trị
`i32`. Mặt khác, các thể hiện tuple struct tương tự như tuple ở chỗ bạn có thể
phân rã chúng thành các phần riêng lẻ, và bạn có thể sử dụng dấu `.` theo sau
là chỉ số để truy cập một giá trị riêng lẻ. Không giống như tuple, tuple struct
yêu cầu bạn phải gọi tên kiểu của struct khi bạn phân rã chúng.
Ví dụ, chúng ta sẽ viết `let Point(x, y, z) = origin;` để phân rã các
giá trị trong điểm `origin` vào các biến tên là `x`, `y`, và `z`.

### Struct Giống Unit Không có Bất kỳ Trường nào

Bạn cũng có thể định nghĩa các struct không có bất kỳ trường nào! Những struct này được gọi là
_struct giống unit_ (unit-like structs) vì chúng hoạt động tương tự như `()`, kiểu unit
mà chúng ta đã đề cập trong phần ["Kiểu Tuple"][tuples]<!-- ignore -->. Các struct
giống unit có thể hữu ích khi bạn cần triển khai một trait trên một kiểu nào đó nhưng
không có bất kỳ dữ liệu nào bạn muốn lưu trữ trong chính kiểu đó. Chúng ta sẽ thảo luận về trait
trong Chương 10. Dưới đây là ví dụ về khai báo và khởi tạo một struct unit
có tên là `AlwaysEqual`:

```aquascope,interpreter
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;`[]`
}
```

Để định nghĩa `AlwaysEqual`, chúng ta sử dụng từ khóa `struct`, tên chúng ta muốn, và
sau đó là dấu chấm phẩy. Không cần dấu ngoặc nhọn hay ngoặc đơn! Sau đó chúng ta có thể lấy
một thể hiện của `AlwaysEqual` trong biến `subject` theo cách tương tự: sử dụng
tên chúng ta đã định nghĩa, mà không cần bất kỳ dấu ngoặc nhọn hay ngoặc đơn nào. Hãy tưởng tượng rằng sau này
chúng ta sẽ triển khai hành vi cho kiểu này sao cho mọi thể hiện của
`AlwaysEqual` luôn bằng với mọi thể hiện của bất kỳ kiểu nào khác, có thể là để
có kết quả đã biết cho mục đích kiểm thử. Chúng ta sẽ không cần bất kỳ dữ liệu nào để
triển khai hành vi đó! Bạn sẽ thấy trong Chương 10 cách định nghĩa các trait và
triển khai chúng trên bất kỳ kiểu nào, bao gồm cả các struct giống unit.

> ### Quyền sở hữu Dữ liệu Struct
>
> Trong định nghĩa struct `User` ở Listing 5-1, chúng ta đã sử dụng kiểu `String` được sở hữu
> thay vì kiểu lát cắt chuỗi `&str`. Đây là một lựa chọn có chủ ý
> vì chúng ta muốn mỗi thể hiện của struct này sở hữu tất cả dữ liệu của nó và để
> dữ liệu đó hợp lệ chừng nào toàn bộ struct còn hợp lệ.
>
> Cũng có thể để các struct lưu trữ tham chiếu đến dữ liệu được sở hữu bởi thứ
> khác, nhưng để làm như vậy đòi hỏi việc sử dụng _vòng đời_ (lifetimes), một tính năng của Rust mà chúng ta sẽ
> thảo luận trong Chương 10. Vòng đời đảm bảo rằng dữ liệu được tham chiếu bởi một struct
> là hợp lệ chừng nào struct đó còn hợp lệ. Giả sử bạn cố gắng lưu trữ một tham chiếu
> trong một struct mà không chỉ định vòng đời, như sau; điều này sẽ không hoạt động:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> Trình biên dịch sẽ phàn nàn rằng nó cần các định danh vòng đời:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> Trong Chương 10, chúng ta sẽ thảo luận cách sửa các lỗi này để bạn có thể lưu trữ
> tham chiếu trong struct, nhưng hiện tại, chúng ta sẽ sửa các lỗi như thế này bằng cách sử dụng các kiểu
> được sở hữu như `String` thay vì tham chiếu như `&str`.

### Vay mượn các Trường của một Struct

Tương tự như thảo luận của chúng ta trong phần ["Các Trường Tuple Khác nhau"][differentfields], bộ kiểm tra vay mượn của Rust sẽ theo dõi quyền sở hữu
ở cả cấp độ struct và cấp độ trường. Ví dụ, nếu chúng ta vay mượn một trường `x` của một cấu trúc `Point`, thì cả `p` và `p.x` tạm thời mất quyền hạn của chúng (nhưng `p.y` thì không):

```aquascope,permissions,stepper,boundaries
#fn main() {
struct Point { x: i32, y: i32 }

let mut p = Point { x: 0, y: 0 };`(focus,paths:p)`
let x = &mut p.x;`(focus,paths:p)`
*x += 1;`(focus,paths:p)`
println!("{}, {}", p.x, p.y);
#}
```

Kết quả là, nếu chúng ta thử và sử dụng `p` trong khi `p.x` đang được vay mượn dưới dạng có thể thay đổi như thế này:

```aquascope,permissions,stepper,boundaries,shouldFail
struct Point { x: i32, y: i32 }

fn print_point(p: &Point) {
    println!("{}, {}", p.x, p.y);
}

fn main() {
    let mut p = Point { x: 0, y: 0 };`(focus,paths:p)`
    let x = &mut p.x;`(focus,paths:p)`
    print_point(&p);`{}`
    *x += 1;`(focus,paths:p)`
}
```

Thì trình biên dịch sẽ từ chối chương trình của chúng ta với lỗi sau:

```text
error[E0502]: cannot borrow `p` as immutable because it is also borrowed as mutable
  --> test.rs:10:17
   |
9  |     let x = &mut p.x;
   |             -------- mutable borrow occurs here
10 |     print_point(&p);
   |                 ^^ immutable borrow occurs here
11 |     *x += 1;
   |     ------- mutable borrow later used here
```

Tổng quát hơn, nếu bạn gặp lỗi quyền sở hữu liên quan đến một struct, bạn nên xem xét trường nào của cấu trúc
được cho là sẽ bị vay mượn với những quyền hạn nào. Nhưng hãy nhận thức về các giới hạn của bộ kiểm tra vay mượn, vì Rust đôi khi có thể
giả định nhiều trường bị vay mượn hơn thực tế.

{{#quiz ../quizzes/ch05-01-structs.toml}}

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html
[copy]: ch04-03-fixing-ownership-errors.html#fixing-an-unsafe-program-copying-vs-moving-out-of-a-collection
[differentfields]: ch04-03-fixing-ownership-errors.html#fixing-a-safe-program-mutating-different-tuple-fields
