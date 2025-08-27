## Định nghĩa và Khởi tạo Structs

Structs tương tự như tuples, được thảo luận trong phần [“The Tuple Type”][tuples]<!--
ignore --> , vì cả hai đều chứa nhiều giá trị liên quan. Giống như tuples, các
phần của một struct có thể có các kiểu khác nhau. Không giống như tuples, trong
một struct, bạn sẽ đặt tên cho từng phần dữ liệu để rõ ràng ý nghĩa của các
giá trị. Việc thêm các tên này khiến structs linh hoạt hơn tuples: bạn không
cần dựa vào thứ tự của dữ liệu để chỉ định hoặc truy cập các giá trị của một
instance.

Để định nghĩa một struct, chúng ta nhập từ khóa `struct` và đặt tên cho toàn
bộ struct. Tên của struct nên mô tả ý nghĩa của các phần dữ liệu được nhóm
lại. Sau đó, bên trong dấu ngoặc nhọn, chúng ta định nghĩa tên và kiểu của
các phần dữ liệu, mà chúng ta gọi là _fields_. Ví dụ, Listing 5-1 hiển thị một
struct lưu trữ thông tin về tài khoản người dùng.

<Listing number="5-1" file-name="src/main.rs" caption="A `User` struct definition">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Để sử dụng một struct sau khi chúng ta đã định nghĩa nó, chúng ta tạo một
_instance_ của struct đó bằng cách chỉ định các giá trị cụ thể cho từng field.
Chúng ta tạo một instance bằng cách nêu tên của struct và sau đó thêm dấu
ngoặc nhọn chứa các cặp _`key: value`_, trong đó keys là tên của các fields và
values là dữ liệu chúng ta muốn lưu trữ trong các fields đó. Chúng ta không
cần chỉ định các fields theo cùng thứ tự mà chúng ta đã khai báo trong struct.
Nói cách khác, định nghĩa struct giống như một mẫu chung cho kiểu, và các
instances điền vào mẫu đó với dữ liệu cụ thể để tạo ra các giá trị của kiểu.
Ví dụ, chúng ta có thể khai báo một người dùng cụ thể như hiển thị trong
Listing 5-2.

<Listing number="5-2" file-name="src/main.rs" caption="Creating an instance of the `User` struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Để lấy một giá trị cụ thể từ một struct, chúng ta sử dụng ký hiệu chấm. Ví
dụ, để truy cập địa chỉ email của người dùng này, chúng ta sử dụng
`user1.email`. Nếu instance là mutable, chúng ta có thể thay đổi một giá trị
bằng cách sử dụng ký hiệu chấm và gán vào một field cụ thể. Listing 5-3 hiển
thị cách thay đổi giá trị trong field `email` của một instance `User` mutable.

<Listing number="5-3" file-name="src/main.rs" caption="Changing the value in the `email` field of a `User` instance">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Lưu ý rằng toàn bộ instance phải là mutable; Rust không cho phép chúng ta đánh
dấu chỉ một số fields nhất định là mutable. Giống như bất kỳ biểu thức nào,
chúng ta có thể xây dựng một instance mới của struct như biểu thức cuối cùng
trong thân hàm để ngầm trả về instance mới đó.

Listing 5-4 hiển thị một hàm `build_user` trả về một instance `User` với email
và username đã cho. Field `active` nhận giá trị `true`, và `sign_in_count`
nhận giá trị `1`.

<Listing number="5-4" file-name="src/main.rs" caption="A `build_user` function that takes an email and username and returns a `User` instance">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Có ý nghĩa khi đặt tên cho các tham số hàm giống với tên các fields của
struct, nhưng việc lặp lại tên fields `email` và `username` và các biến là
một chút nhàm chán. Nếu struct có nhiều fields hơn, việc lặp lại mỗi tên sẽ
càng nhàm chán hơn. May mắn thay, có một cách viết tắt tiện lợi!

<!-- Old heading. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Sử dụng Cách Viết Tắt Khởi tạo Field

Vì tên tham số và tên fields của struct giống hệt nhau trong Listing 5-4,
chúng ta có thể sử dụng cú pháp _field init shorthand_ để viết lại `build_user`
để nó hoạt động giống hệt nhưng không có sự lặp lại của `username` và
`email`, như hiển thị trong Listing 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="A `build_user` function that uses field init shorthand because the `username` and `email` parameters have the same name as struct fields">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta đang tạo một instance mới của struct `User`, có field tên
`email`. Chúng ta muốn đặt giá trị của field `email` thành giá trị trong tham
số `email` của hàm `build_user`. Vì field `email` và tham số `email` có cùng
tên, chúng ta chỉ cần viết `email` thay vì `email: email`.

### Tạo Instances từ Các Instances Khác với Cú Pháp Cập Nhật Struct

Thường hữu ích khi tạo một instance mới của struct bao gồm hầu hết các giá
trị từ một instance khác của cùng kiểu, nhưng thay đổi một số. Bạn có thể làm
điều này bằng cách sử dụng _struct update syntax_.

Đầu tiên, trong Listing 5-6, chúng ta hiển thị cách tạo một instance `User`
mới trong `user2` thường xuyên, mà không có cú pháp cập nhật. Chúng ta đặt
một giá trị mới cho `email` nhưng nếu không thì sử dụng cùng giá trị từ
`user1` mà chúng ta đã tạo trong Listing 5-2.

<Listing number="5-6" file-name="src/main.rs" caption="Creating a new `User` instance using all but one of the values from `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Sử dụng cú pháp cập nhật struct, chúng ta có thể đạt được hiệu ứng tương tự
với ít mã hơn, như hiển thị trong Listing 5-7. Cú pháp `..` chỉ định rằng các
fields còn lại không được đặt rõ ràng nên có cùng giá trị như các fields trong
instance đã cho.

<Listing number="5-7" file-name="src/main.rs" caption="Using struct update syntax to set a new `email` value for a `User` instance but to use the rest of the values from `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Mã trong Listing 5-7 cũng tạo một instance trong `user2` có giá trị khác cho
`email` nhưng có cùng giá trị cho các fields `username`, `active`, và
`sign_in_count` từ `user1`. `..user1` phải đến cuối cùng để chỉ định rằng bất
kỳ fields còn lại nào nên nhận giá trị từ các fields tương ứng trong `user1`,
nhưng chúng ta có thể chọn chỉ định giá trị cho bao nhiêu fields tùy ý theo
bất kỳ thứ tự nào, bất kể thứ tự của các fields trong định nghĩa struct.

Lưu ý rằng cú pháp cập nhật struct sử dụng `=` như một phép gán; điều này là
vì nó di chuyển dữ liệu, giống như chúng ta đã thấy trong phần [“Variables and
Data Interacting with Move”][move]<!-- ignore --> . Trong ví dụ này, chúng ta
không thể sử dụng `user1` nữa sau khi tạo `user2` vì `String` trong field
`username` của `user1` đã được di chuyển vào `user2`. Nếu chúng ta đã cho
`user2` các giá trị `String` mới cho cả `email` và `username`, và do đó chỉ
sử dụng các giá trị `active` và `sign_in_count` từ `user1`, thì `user1` vẫn
hợp lệ sau khi tạo `user2`. Cả `active` và `sign_in_count` đều là các kiểu
triển khai trait `Copy`, vì vậy hành vi chúng ta đã thảo luận trong phần
[“Stack-Only Data: Copy”][copy]<!-- ignore --> sẽ áp dụng. Chúng ta cũng vẫn
có thể sử dụng `user1.email` trong ví dụ này, vì giá trị của nó không được di
chuyển ra khỏi `user1`.

### Sử dụng Tuple Structs Không Có Fields Đặt Tên để Tạo Các Kiểu Khác Nhau

Rust cũng hỗ trợ structs trông tương tự như tuples, được gọi là _tuple
structs_. Tuple structs có ý nghĩa bổ sung mà tên struct cung cấp nhưng không
có tên liên quan với các fields của chúng; thay vào đó, chúng chỉ có kiểu của
các fields. Tuple structs hữu ích khi bạn muốn đặt tên cho toàn bộ tuple và
làm cho tuple trở thành một kiểu khác với các tuples khác, và khi đặt tên cho
mỗi field như trong một struct thông thường sẽ dài dòng hoặc thừa thãi.

Để định nghĩa một tuple struct, bắt đầu với từ khóa `struct` và tên struct
theo sau bởi các kiểu trong tuple. Ví dụ, ở đây chúng ta định nghĩa và sử
dụng hai tuple structs tên `Color` và `Point`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

Lưu ý rằng các giá trị `black` và `origin` là các kiểu khác nhau vì chúng là
instances của các tuple structs khác nhau. Mỗi struct bạn định nghĩa là kiểu
riêng của nó, ngay cả khi các fields trong struct có thể có cùng kiểu. Ví dụ,
một hàm nhận tham số kiểu `Color` không thể nhận `Point` làm đối số, ngay cả
khi cả hai kiểu đều được tạo từ ba giá trị `i32`. Nếu không, các instances
tuple struct tương tự như tuples ở chỗ bạn có thể phá hủy chúng thành các
phần riêng lẻ, và bạn có thể sử dụng `.` theo sau bởi chỉ số để truy cập một
giá trị riêng lẻ. Không giống như tuples, tuple structs yêu cầu bạn đặt tên
kiểu của struct khi phá hủy chúng. Ví dụ, chúng ta sẽ viết `let Point(x, y,
z) = origin;` để phá hủy các giá trị trong điểm `origin` thành các biến tên
`x`, `y`, và `z`.

### Unit-Like Structs Không Có Bất Kỳ Fields Nào

Bạn cũng có thể định nghĩa structs không có bất kỳ fields nào! Những cái này
được gọi là _unit-like structs_ vì chúng hoạt động tương tự như `()`, kiểu
unit mà chúng ta đã đề cập trong phần [“The Tuple Type”][tuples]<!-- ignore
--> . Unit-like structs có thể hữu ích khi bạn cần triển khai một trait trên
một số kiểu nhưng không có bất kỳ dữ liệu nào bạn muốn lưu trữ trong kiểu đó
bản thân. Chúng ta sẽ thảo luận traits trong Chương 10. Đây là ví dụ về việc
khai báo và khởi tạo một unit struct tên `AlwaysEqual`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

Để định nghĩa `AlwaysEqual`, chúng ta sử dụng từ khóa `struct`, tên chúng ta
muốn, và sau đó một dấu chấm phẩy. Không cần dấu ngoặc nhọn hoặc ngoặc đơn!
Sau đó, chúng ta có thể nhận một instance của `AlwaysEqual` trong biến
`subject` theo cách tương tự: sử dụng tên chúng ta đã định nghĩa, mà không có
bất kỳ dấu ngoặc nhọn hoặc ngoặc đơn nào. Hãy tưởng tượng rằng sau này chúng
ta sẽ triển khai hành vi cho kiểu này sao cho mọi instance của `AlwaysEqual`
luôn bằng mọi instance của bất kỳ kiểu nào khác, có lẽ để có kết quả đã biết
cho mục đích kiểm tra. Chúng ta sẽ không cần bất kỳ dữ liệu nào để triển
khai hành vi đó! Bạn sẽ thấy trong Chương 10 cách định nghĩa traits và triển
khai chúng trên bất kỳ kiểu nào, bao gồm unit-like structs.

> ### Sở hữu Dữ liệu Struct
>
> Trong định nghĩa struct `User` trong Listing 5-1, chúng ta đã sử dụng kiểu
> `String` sở hữu thay vì kiểu slice chuỗi `&str`. Đây là một lựa chọn có chủ
> đích vì chúng ta muốn mỗi instance của struct này sở hữu tất cả dữ liệu của
> nó và dữ liệu đó hợp lệ miễn là toàn bộ struct hợp lệ.
>
> Cũng có thể structs lưu trữ tham chiếu đến dữ liệu được sở hữu bởi thứ gì
> đó khác, nhưng để làm vậy yêu cầu sử dụng _lifetimes_, một tính năng Rust
> mà chúng ta sẽ thảo luận trong Chương 10. Lifetimes đảm bảo rằng dữ liệu
> được tham chiếu bởi một struct hợp lệ miễn là struct hợp lệ. Hãy nói bạn
> thử lưu trữ một tham chiếu trong một struct mà không chỉ định lifetimes,
> như sau; điều này sẽ không hoạt động:
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
> Trình biên dịch sẽ phàn nàn rằng nó cần các chỉ định lifetime:
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
> Trong Chương 10, chúng ta sẽ thảo luận cách sửa các lỗi này để bạn có thể
> lưu trữ tham chiếu trong structs, nhưng bây giờ, chúng ta sẽ sửa các lỗi
> như thế này bằng cách sử dụng các kiểu sở hữu như `String` thay vì tham
> chiếu như `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
