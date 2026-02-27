## Lỗi có thể phục hồi với `Result`

Hầu hết các lỗi không đủ nghiêm trọng để yêu cầu chương trình phải dừng hoàn toàn.
Đôi khi khi một hàm thất bại, đó là vì một lý do mà bạn có thể dễ dàng giải thích
và phản hồi lại. Ví dụ, nếu bạn cố gắng mở một file và thao tác đó thất bại
vì file không tồn tại, bạn có thể muốn tạo file đó thay vì
chấm dứt tiến trình.

Hãy nhớ lại từ [“Xử lý Thất bại Tiềm ẩn với `Result`”][handle_failure]<!--
ignore --> trong Chương 2 rằng enum `Result` được định nghĩa là có hai
biến thể, `Ok` và `Err`, như sau:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` và `E` là các tham số kiểu generic (generic type parameters): chúng ta sẽ thảo luận về generic chi tiết
hơn trong Chương 10. Những gì bạn cần biết ngay bây giờ là `T` đại diện
cho kiểu của giá trị sẽ được trả về trong trường hợp thành công bên trong biến thể `Ok`,
và `E` đại diện cho kiểu của lỗi sẽ được trả về trong
trường hợp thất bại bên trong biến thể `Err`. Bởi vì `Result` có các tham số kiểu generic
này, chúng ta có thể sử dụng kiểu `Result` và các hàm được định nghĩa trên nó trong
nhiều tình huống khác nhau khi mà giá trị thành công và giá trị lỗi chúng ta muốn
trả về có thể khác nhau.

Hãy gọi một hàm trả về một giá trị `Result` vì hàm đó có thể
thất bại. Trong Listing 9-3 chúng ta cố gắng mở một file.

<Listing number="9-3" file-name="src/main.rs" caption="Mở một file">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

Kiểu trả về của `File::open` là một `Result<T, E>`. Tham số generic `T`
đã được điền vào bởi phần triển khai của `File::open` với kiểu của
giá trị thành công, `std::fs::File`, đó là một file handle. Kiểu của `E` được sử dụng trong
giá trị lỗi là `std::io::Error`. Kiểu trả về này có nghĩa là lời gọi đến
`File::open` có thể thành công và trả về một file handle mà chúng ta có thể đọc hoặc
ghi vào. Lời gọi hàm cũng có thể thất bại: ví dụ, file có thể không
tồn tại, hoặc chúng ta có thể không có quyền truy cập file. Hàm `File::open`
cần có một cách để cho chúng ta biết liệu nó thành công hay thất bại và đồng
thời đưa cho chúng ta hoặc là file handle hoặc là thông tin lỗi. Thông
tin này chính xác là những gì enum `Result` truyền đạt.

Trong trường hợp `File::open` thành công, giá trị trong biến
`greeting_file_result` sẽ là một thực thể (instance) của `Ok` chứa một file handle.
Trong trường hợp nó thất bại, giá trị trong `greeting_file_result` sẽ là một
thực thể của `Err` chứa nhiều thông tin hơn về loại lỗi đã
xảy ra.

Chúng ta cần thêm mã vào Listing 9-3 để thực hiện các hành động khác nhau tùy thuộc
vào giá trị mà `File::open` trả về. Listing 9-4 cho thấy một cách để xử lý
`Result` bằng cách sử dụng một công cụ cơ bản, biểu thức `match` mà chúng ta đã thảo luận trong
Chương 6.

<Listing number="9-4" file-name="src/main.rs" caption="Sử dụng biểu thức `match` để xử lý các biến thể `Result` có thể được trả về">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Lưu ý rằng, giống như enum `Option`, enum `Result` và các biến thể của nó đã được
đưa vào phạm vi bởi prelude, vì vậy chúng ta không cần chỉ định `Result::`
trước các biến thể `Ok` và `Err` trong các nhánh `match`.

Khi kết quả là `Ok`, mã này sẽ trả về giá trị `file` bên trong từ
biến thể `Ok`, và sau đó chúng ta gán giá trị file handle đó cho biến
`greeting_file`. Sau `match`, chúng ta có thể sử dụng file handle để đọc hoặc ghi.

Nhánh còn lại của `match` xử lý trường hợp chúng ta nhận được một giá trị `Err` từ
`File::open`. Trong ví dụ này, chúng ta đã chọn gọi macro `panic!`. Nếu
không có file nào tên là _hello.txt_ trong thư mục hiện tại và chúng ta chạy
mã này, chúng ta sẽ thấy kết quả sau từ macro `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Như thường lệ, kết quả này cho chúng ta biết chính xác điều gì đã xảy ra.

### Khớp với các Lỗi khác nhau

Mã trong Listing 9-4 sẽ `panic!` bất kể lý do tại sao `File::open` thất bại.
Tuy nhiên, chúng ta muốn thực hiện các hành động khác nhau cho các lý do thất bại khác nhau. Nếu
`File::open` thất bại vì file không tồn tại, chúng ta muốn tạo file đó
và trả về handle cho file mới. Nếu `File::open` thất bại vì bất kỳ lý do
nào khác—ví dụ, vì chúng ta không có quyền mở file—chúng ta vẫn
muốn mã `panic!` theo cùng cách nó đã làm trong Listing 9-4. Để làm điều này, chúng ta
thêm một biểu thức `match` lồng bên trong, được hiển thị trong Listing 9-5.

<Listing number="9-5" file-name="src/main.rs" caption="Xử lý các loại lỗi khác nhau theo những cách khác nhau">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

Kiểu của giá trị mà `File::open` trả về bên trong biến thể `Err` là
`io::Error`, đây là một struct được cung cấp bởi thư viện tiêu chuẩn. Struct này
có một phương thức `kind` mà chúng ta có thể gọi để lấy một giá trị `io::ErrorKind`. Enum
`io::ErrorKind` được cung cấp bởi thư viện tiêu chuẩn và có các biến thể
đại diện cho các loại lỗi khác nhau có thể xảy ra từ một thao tác `io`.
Biến thể chúng ta muốn sử dụng là `ErrorKind::NotFound`, nó cho biết
file chúng ta đang cố gắng mở chưa tồn tại. Vì vậy chúng ta khớp trên
`greeting_file_result`, nhưng chúng ta cũng có một match bên trong trên `error.kind()`.

Điều kiện chúng ta muốn kiểm tra trong match bên trong là liệu giá trị được trả về
bởi `error.kind()` có phải là biến thể `NotFound` của enum `ErrorKind` hay không. Nếu đúng,
chúng ta cố gắng tạo file với `File::create`. Tuy nhiên, vì `File::create`
cũng có thể thất bại, chúng ta cần một nhánh thứ hai trong biểu thức `match` bên trong. Khi
file không thể được tạo, một thông báo lỗi khác sẽ được in ra. Nhánh thứ hai của
`match` bên ngoài vẫn giữ nguyên, vì vậy chương trình sẽ panic với bất kỳ lỗi nào ngoại trừ
lỗi thiếu file.

> #### Các phương án thay thế cho việc sử dụng `match` với `Result<T, E>`
>
> Có thật nhiều `match`! Biểu thức `match` rất hữu ích nhưng cũng rất
> sơ khai. Trong Chương 13, bạn sẽ tìm hiểu về closure, thứ được sử dụng
> với nhiều phương thức được định nghĩa trên `Result<T, E>`. Những phương thức này có thể
> súc tích hơn việc sử dụng `match` khi xử lý các giá trị `Result<T, E>` trong mã của bạn.
>
> Ví dụ, đây là một cách khác để viết cùng một logic như được hiển thị trong Listing
> 9-5, lần này sử dụng closure và phương thức `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> Mặc dù mã này có hành vi giống như Listing 9-5, nó không chứa
> bất kỳ biểu thức `match` nào và dễ đọc hơn. Hãy quay lại ví dụ này
> sau khi bạn đã đọc Chương 13, và tra cứu phương thức `unwrap_or_else` trong
> tài liệu thư viện tiêu chuẩn. Còn rất nhiều phương thức như thế này có thể dọn dẹp các
> biểu thức `match` lồng nhau khổng lồ khi bạn đang xử lý lỗi.

{{#quiz ../quizzes/ch09-02-recoverable-errors-sec1.toml}}

#### Các lối tắt để Panic khi có Lỗi: `unwrap` và `expect`

Sử dụng `match` hoạt động đủ tốt, nhưng nó có thể hơi dài dòng và không phải lúc nào cũng
truyền đạt tốt ý định. Kiểu `Result<T, E>` có nhiều phương thức trợ giúp
được định nghĩa trên nó để thực hiện các tác vụ cụ thể hơn. Phương thức `unwrap` là một
phương thức lối tắt được triển khai giống như biểu thức `match` chúng ta đã viết trong
Listing 9-4. Nếu giá trị `Result` là biến thể `Ok`, `unwrap` sẽ trả về
giá trị bên trong `Ok`. Nếu `Result` là biến thể `Err`, `unwrap` sẽ
gọi macro `panic!` cho chúng ta. Đây là một ví dụ về `unwrap` trong thực tế:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Nếu chúng ta chạy mã này mà không có file _hello.txt_, chúng ta sẽ thấy một thông báo lỗi từ
lời gọi `panic!` mà phương thức `unwrap` thực hiện:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Tương tự, phương thức `expect` cho phép chúng ta chọn thông báo lỗi `panic!`.
Sử dụng `expect` thay vì `unwrap` và cung cấp các thông báo lỗi tốt có thể truyền đạt
ý định của bạn và giúp việc truy vết nguồn gốc của một panic dễ dàng hơn. Cú pháp của
`expect` trông như thế này:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

Chúng ta sử dụng `expect` theo cùng cách với `unwrap`: để trả về file handle hoặc gọi
macro `panic!`. Thông báo lỗi được sử dụng bởi `expect` trong lời gọi đến `panic!`
sẽ là tham số mà chúng ta truyền vào `expect`, thay vì thông báo `panic!`
mặc định mà `unwrap` sử dụng. Đây là kết quả trông như thế nào:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Trong mã chất lượng sản xuất, hầu hết những người dùng Rust chọn `expect` thay vì
`unwrap` và đưa ra nhiều ngữ cảnh hơn về lý do tại sao thao tác đó được mong đợi là luôn
thành công. Bằng cách đó, nếu các giả định của bạn bị chứng minh là sai, bạn có thêm
thông tin để sử dụng trong việc gỡ lỗi.

### Lan truyền Lỗi

Khi phần triển khai của một hàm gọi một thứ gì đó có thể thất bại, thay vì
xử lý lỗi ngay trong chính hàm đó, bạn có thể trả về lỗi cho
mã gọi nó để nó có thể quyết định phải làm gì. Đây được gọi là _lan truyền_ (propagating)
lỗi và mang lại nhiều quyền kiểm soát hơn cho mã gọi, nơi có thể có nhiều
thông tin hoặc logic quyết định cách xử lý lỗi hơn là những gì
bạn có sẵn trong ngữ cảnh mã của mình.

Ví dụ, Listing 9-6 cho thấy một hàm đọc username từ một file. Nếu
file không tồn tại hoặc không thể đọc được, hàm này sẽ trả về những lỗi đó
cho mã đã gọi hàm.

<Listing number="9-6" file-name="src/main.rs" caption="Một hàm trả về lỗi cho mã gọi bằng cách sử dụng `match`平衡">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Hàm này có thể được viết theo cách ngắn hơn nhiều, nhưng chúng ta sẽ bắt đầu bằng
cách thực hiện thủ công để khám phá việc xử lý lỗi; ở phần cuối,
chúng ta sẽ chỉ ra cách ngắn hơn. Hãy xem xét kiểu trả về của hàm
trước: `Result<String, io::Error>`. Điều này có nghĩa là hàm đang trả về một
giá trị kiểu `Result<T, E>`, trong đó tham số generic `T` đã được điền vào
với kiểu cụ thể `String` và kiểu generic `E` đã được điền vào với kiểu
cụ thể `io::Error`.

Nếu hàm này thành công mà không có bất kỳ vấn đề gì, mã gọi hàm này
sẽ nhận được một giá trị `Ok` chứa một `String`—chính là `username` mà
hàm này đã đọc từ file. Nếu hàm này gặp bất kỳ vấn đề nào,
mã gọi sẽ nhận được một giá trị `Err` chứa một thực thể của `io::Error`
có chứa nhiều thông tin hơn về các vấn đề đó là gì. Chúng ta đã chọn
`io::Error` làm kiểu trả về của hàm này vì đó tình cờ là
kiểu của giá trị lỗi được trả về từ cả hai thao tác mà chúng ta đang gọi trong
thân hàm này có thể thất bại: hàm `File::open` và phương thức
`read_to_string`.

Thân hàm bắt đầu bằng cách gọi hàm `File::open`. Sau đó chúng ta
xử lý giá trị `Result` bằng một `match` tương tự như `match` trong Listing 9-4.
Nếu `File::open` thành công, file handle trong biến pattern `file`
trở thành giá trị trong biến có thể thay đổi `username_file` và hàm
tiếp tục. Trong trường hợp `Err`, thay vì gọi `panic!`, chúng ta sử dụng từ khóa `return`
để trả về sớm ra khỏi hàm hoàn toàn và truyền giá trị lỗi
từ `File::open`, bây giờ nằm trong biến pattern `e`, quay lại mã gọi như là
giá trị lỗi của hàm này.

Vì vậy, nếu chúng ta có một file handle trong `username_file`, hàm sau đó tạo một
`String` mới trong biến `username` và gọi phương thức `read_to_string` trên
file handle trong `username_file` để đọc nội dung của file vào
`username`. Phương thức `read_to_string` cũng trả về một `Result` vì nó
có thể thất bại, ngay cả khi `File::open` đã thành công. Vì vậy chúng ta cần một `match` khác để
xử lý `Result` đó: nếu `read_to_string` thành công, thì hàm của chúng ta đã
thành công, và chúng ta trả về username từ file hiện đang nằm trong `username`
được bao bọc trong một `Ok`. Nếu `read_to_string` thất bại, chúng ta trả về giá trị lỗi theo
cùng cách mà chúng ta đã trả về giá trị lỗi trong `match` xử lý
giá trị trả về của `File::open`. Tuy nhiên, chúng ta không cần nói rõ ràng
`return`, vì đây là biểu thức cuối cùng trong hàm.

Mã gọi mã này sau đó sẽ xử lý việc nhận được hoặc là một giá trị `Ok`
chứa một username hoặc là một giá trị `Err` chứa một `io::Error`.
Việc quyết định làm gì với những giá trị đó là tùy thuộc vào mã gọi. Nếu mã gọi
nhận được một giá trị `Err`, nó có thể gọi `panic!` và làm dừng chương trình, sử dụng một
username mặc định, hoặc tra cứu username từ một nơi khác không phải là file, chẳng
hạn. Chúng ta không có đủ thông tin về những gì mã gọi thực sự đang cố gắng
làm, vì vậy chúng ta lan truyền tất cả thông tin thành công hoặc lỗi lên phía trên để
nó xử lý một cách thích hợp.

Mô hình lan truyền lỗi này rất phổ biến trong Rust đến nỗi Rust cung cấp
toán tử dấu hỏi chấm `?` để làm điều này dễ dàng hơn.

#### Một lối tắt để lan truyền lỗi: Toán tử `?`

Listing 9-7 cho thấy một bản triển khai của `read_username_from_file` có
chức năng tương tự như trong Listing 9-6, nhưng bản triển khai này sử dụng toán tử
`?`.

<Listing number="9-7" file-name="src/main.rs" caption="Một hàm trả về lỗi cho mã gọi bằng cách sử dụng toán tử `?`平衡">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

Dấu `?` đặt sau một giá trị `Result` được định nghĩa để hoạt động gần như cùng cách
với các biểu thức `match` mà chúng ta đã định nghĩa để xử lý các giá trị `Result` trong Listing
9-6. Nếu giá trị của `Result` là một `Ok`, giá trị bên trong `Ok` sẽ
được trả về từ biểu thức này, và chương trình sẽ tiếp tục. Nếu giá trị
là một `Err`, `Err` đó sẽ được trả về từ toàn bộ hàm như thể chúng ta đã
sử dụng từ khóa `return` để giá trị lỗi được lan truyền đến mã gọi.

Có một sự khác biệt giữa những gì biểu thức `match` từ Listing 9-6 thực hiện
và những gì toán tử `?` thực hiện: các giá trị lỗi mà toán tử `?` được gọi trên
chúng sẽ đi qua hàm `from`, được định nghĩa trong trait `From` trong
thư viện tiêu chuẩn, vốn được sử dụng để chuyển đổi các giá trị từ kiểu này sang kiểu khác.
Khi toán tử `?` gọi hàm `from`, kiểu lỗi nhận được sẽ được
chuyển đổi thành kiểu lỗi được định nghĩa trong kiểu trả về của hàm
hiện tại. Điều này hữu ích khi một hàm trả về một kiểu lỗi để đại diện cho
tất cả các cách mà một hàm có thể thất bại, ngay cả khi các phần có thể thất bại vì nhiều lý do
khác nhau.

Ví dụ, chúng ta có thể thay đổi hàm `read_username_from_file` trong Listing
9-7 để trả về một kiểu lỗi tùy chỉnh tên là `OurError` mà chúng ta định nghĩa. Nếu chúng ta cũng
định nghĩa `impl From<io::Error> for OurError` để xây dựng một thực thể của
`OurError` từ một `io::Error`, thì các lời gọi toán tử `?` trong thân của
`read_username_from_file` sẽ gọi `from` và chuyển đổi các kiểu lỗi mà không
cần thêm bất kỳ mã nào vào hàm.

Trong ngữ cảnh của Listing 9-7, dấu `?` ở cuối lời gọi `File::open` sẽ
trả về giá trị bên trong một `Ok` cho biến `username_file`. Nếu một lỗi
xảy ra, toán tử `?` sẽ trả về sớm ra khỏi toàn bộ hàm và đưa
bất kỳ giá trị `Err` nào cho mã gọi. Điều tương tự cũng áp dụng cho dấu `?` ở
cuối lời gọi `read_to_string`.

Toán tử `?` loại bỏ rất nhiều mã rườm rà (boilerplate) và làm cho phần triển khai của hàm này
đơn giản hơn. Chúng ta thậm chí có thể rút ngắn mã này hơn nữa bằng cách chuỗi các
lời gọi phương thức ngay sau dấu `?`, như được hiển thị trong Listing 9-8.

<Listing number="9-8" file-name="src/main.rs" caption="Chuỗi các lời gọi phương thức sau toán tử `?`平衡">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

Chúng ta đã di chuyển việc tạo `String` mới trong `username` lên đầu
hàm; phần đó không thay đổi. Thay vì tạo một biến
`username_file`, chúng ta đã chuỗi lời gọi đến `read_to_string` trực tiếp vào
kết quả của `File::open("hello.txt")?`. Chúng ta vẫn có một dấu `?` ở cuối lời gọi
`read_to_string`, và chúng ta vẫn trả về một giá trị `Ok` chứa `username`
khi cả `File::open` và `read_to_string` đều thành công thay vì trả về
lỗi. Chức năng một lần nữa giống như trong Listing 9-6 và Listing 9-7;
đây chỉ là một cách viết khác, tiện lợi hơn.

Listing 9-9 cho thấy một cách để làm điều này thậm chí còn ngắn hơn bằng cách sử dụng `fs::read_to_string`.

<Listing number="9-9" file-name="src/main.rs" caption="Sử dụng `fs::read_to_string` thay vì mở rồi mới đọc file">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Đọc một file vào một chuỗi là một thao tác khá phổ biến, vì vậy thư viện
tiêu chuẩn cung cấp hàm `fs::read_to_string` tiện lợi giúp mở
file, tạo một `String` mới, đọc nội dung của file, đặt nội dung
vào `String` đó, và trả về nó. Tất nhiên, việc sử dụng `fs::read_to_string`
không cho chúng ta cơ hội để giải thích tất cả về xử lý lỗi, vì vậy chúng ta đã thực hiện
theo cách dài hơn trước.

#### Nơi Toán tử `?` Có thể được Sử dụng

Toán tử `?` chỉ có thể được sử dụng trong các hàm có kiểu trả về tương thích
với giá trị mà dấu `?` được sử dụng trên đó. Điều này là do toán tử `?` được định nghĩa
để thực hiện việc trả về sớm một giá trị ra khỏi hàm, theo cùng cách
như biểu thức `match` chúng ta đã định nghĩa trong Listing 9-6. Trong Listing 9-6,
`match` đã sử dụng một giá trị `Result`, và nhánh trả về sớm đã trả về một
giá trị `Err(e)`. Kiểu trả về của hàm phải là một `Result` để
nó tương thích với lệnh `return` này.

Trong Listing 9-10, hãy xem lỗi chúng ta sẽ nhận được nếu chúng ta sử dụng toán tử `?`
trong một hàm `main` có kiểu trả về không tương thích với kiểu của
giá trị chúng ta sử dụng `?` trên đó.

<Listing number="9-10" file-name="src/main.rs" caption="Cố gắng sử dụng `?` trong hàm `main` trả về `()` sẽ không biên dịch được.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Mã này mở một file, việc này có thể thất bại. Toán tử `?` theo sau
giá trị `Result` được trả về bởi `File::open`, nhưng hàm `main` này có kiểu trả về là
`()`, không phải `Result`. Khi chúng ta biên dịch mã này, chúng ta nhận được thông báo lỗi
sau:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Lỗi này chỉ ra rằng chúng ta chỉ được phép sử dụng toán tử `?` trong một
hàm trả về `Result`, `Option`, hoặc một kiểu khác triển khai
`FromResidual`.

Để khắc phục lỗi này, bạn có hai lựa chọn. Một lựa chọn là thay đổi kiểu trả về
của hàm để tương thích với giá trị bạn đang sử dụng toán tử `?` trên đó
miễn là bạn không có hạn chế nào ngăn cản việc đó. Lựa chọn khác là
sử dụng một `match` hoặc một trong các phương thức của `Result<T, E>` để xử lý `Result<T, E>`
theo bất kỳ cách nào phù hợp.

Thông báo lỗi cũng đề cập rằng `?` cũng có thể được sử dụng với các giá trị `Option<T>`.
Giống như khi sử dụng `?` trên `Result`, bạn chỉ có thể sử dụng `?` trên `Option` trong một
hàm trả về một `Option`. Hành vi của toán tử `?` khi được gọi
trên một `Option<T>` tương tự như hành vi của nó khi được gọi trên một `Result<T, E>`:
nếu giá trị là `None`, `None` sẽ được trả về sớm từ hàm tại
điểm đó. Nếu giá trị là `Some`, giá trị bên trong `Some` là giá trị kết quả
của biểu thức, và hàm tiếp tục. Listing 9-11 có một ví dụ về một hàm tìm ký tự cuối cùng của dòng đầu tiên trong đoạn văn bản đã cho.

<Listing number="9-11" caption="Sử dụng toán tử `?` trên một giá trị `Option<T>`平衡">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Hàm này trả về `Option<char>` vì có khả năng là có một
ký tự ở đó, nhưng cũng có khả năng là không có. Mã này lấy đối số lát cắt chuỗi
`text` và gọi phương thức `lines` trên nó, phương thức này trả về
một iterator trên các dòng trong chuỗi. Vì hàm này muốn
kiểm tra dòng đầu tiên, nó gọi `next` trên iterator để lấy giá trị đầu tiên
từ iterator. Nếu `text` là chuỗi trống, lời gọi `next` này sẽ
trả về `None`, trong trường hợp đó chúng ta sử dụng `?` để dừng và trả về `None` từ
`last_char_of_first_line`. Nếu `text` không phải là chuỗi trống, `next` sẽ
trả về một giá trị `Some` chứa một lát cắt chuỗi của dòng đầu tiên trong `text`.

Dấu `?` trích xuất lát cắt chuỗi, và chúng ta có thể gọi `chars` trên lát cắt chuỗi đó
để lấy một iterator các ký tự của nó. Chúng ta quan tâm đến ký tự cuối cùng trong
dòng đầu tiên này, vì vậy chúng ta gọi `last` để trả về mục cuối cùng trong iterator.
Đây là một `Option` vì có khả năng dòng đầu tiên là chuỗi trống; ví dụ, nếu `text` bắt đầu bằng một dòng trống nhưng có các ký tự trên
các dòng khác, như trong `"\nhi"`. Tuy nhiên, nếu có một ký tự cuối cùng trên dòng đầu tiên, nó sẽ được trả về trong biến thể `Some`. Toán tử `?` ở giữa
cho chúng ta một cách súc tích để diễn đạt logic này, cho phép chúng ta triển khai
hàm trong một dòng. Nếu chúng ta không thể sử dụng toán tử `?` trên `Option`, chúng ta sẽ
phải triển khai logic này bằng nhiều lời gọi phương thức hơn hoặc một biểu thức `match`.

Lưu ý rằng bạn có thể sử dụng toán tử `?` trên một `Result` trong một hàm trả về
`Result`, và bạn có thể sử dụng toán tử `?` trên một `Option` trong một hàm trả về
`Option`, nhưng bạn không thể trộn lẫn chúng. Toán tử `?` sẽ không
tự động chuyển đổi một `Result` thành một `Option` hoặc ngược lại; trong những trường hợp đó,
bạn có thể sử dụng các phương thức như phương thức `ok` trên `Result` hoặc phương thức `ok_or` trên
`Option` để thực hiện việc chuyển đổi một cách rõ ràng.

Cho đến nay, tất cả các hàm `main` chúng ta đã sử dụng đều trả về `()`. Hàm `main` là
đặc biệt vì nó là điểm vào và điểm ra của một chương trình thực thi,
và có những hạn chế về kiểu trả về của nó để chương trình
hoạt động như mong đợi.

May mắn thay, `main` cũng có thể trả về một `Result<(), E>`. Listing 9-12 có mã
từ Listing 9-10, nhưng chúng ta đã thay đổi kiểu trả về của `main` thành
`Result<(), Box<dyn Error>>` và thêm một giá trị trả về `Ok(())` vào cuối. Mã
này bây giờ sẽ biên dịch được.

<Listing number="9-12" file-name="src/main.rs" caption="Thay đổi `main` để trả về `Result<(), E>` cho phép sử dụng toán tử `?` trên các giá trị `Result`.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

Kiểu `Box<dyn Error>` là một _trait object_, chúng ta sẽ nói về nó trong [“Sử dụng
Trait Object Cho phép các Giá trị thuộc các Kiểu Khác nhau”][trait-objects]<!--
ignore --> ở Chương 18. Hiện tại, bạn có thể hiểu `Box<dyn Error>` có nghĩa là “bất kỳ
loại lỗi nào.” Việc sử dụng `?` trên một giá trị `Result` trong một hàm `main` với
kiểu lỗi `Box<dyn Error>` được phép vì nó cho phép bất kỳ giá trị `Err` nào được
trả về sớm. Mặc dù thân của hàm `main` này sẽ chỉ bao giờ
trả về các lỗi kiểu `std::io::Error`, bằng cách chỉ định `Box<dyn Error>`, chữ ký này sẽ tiếp tục đúng ngay cả khi có thêm nhiều mã trả về các
lỗi khác được thêm vào thân của `main`.

Khi một hàm `main` trả về một `Result<(), E>`, chương trình thực thi sẽ thoát với
một giá trị `0` nếu `main` trả về `Ok(())` và sẽ thoát với một giá trị khác không nếu
`main` trả về một giá trị `Err`. Các chương trình thực thi được viết bằng C trả về các số nguyên khi
chúng thoát: các chương trình thoát thành công trả về số nguyên `0`, và các chương trình
có lỗi trả về một số nguyên khác `0`. Rust cũng trả về các số nguyên từ
các chương trình thực thi để tương thích với quy ước này.

Hàm `main` có thể trả về bất kỳ kiểu nào triển khai [trait
`std::process::Termination`][termination]<!-- ignore -->, vốn chứa
một hàm `report` trả về một `ExitCode`. Hãy tham khảo tài liệu thư viện tiêu chuẩn
để biết thêm thông tin về việc triển khai trait `Termination` cho
các kiểu của riêng bạn.

Bây giờ chúng ta đã thảo luận về các chi tiết của việc gọi `panic!` hoặc trả về `Result`,
hãy quay lại chủ đề về cách quyết định cái nào là phù hợp để sử dụng trong
các trường hợp nào.

{{#quiz ../quizzes/ch09-02-recoverable-errors-sec2.toml}}

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[termination]: ../std/process/trait.Termination.html
