## Xuất bản một Crate lên Crates.io

Chúng ta đã sử dụng các gói từ [crates.io](https://crates.io/)<!-- ignore --> như là
các phụ thuộc (dependencies) của dự án của mình, nhưng bạn cũng có thể chia sẻ mã nguồn của mình với người khác
bằng cách xuất bản các gói của riêng bạn. Sổ đăng ký crate (crate registry) tại
[crates.io](https://crates.io/)<!-- ignore --> phân phối mã nguồn của các gói của bạn,
vì vậy nó chủ yếu lưu trữ mã nguồn mở.

Rust và Cargo có các tính năng giúp gói đã xuất bản của bạn dễ dàng được mọi người
tìm thấy và sử dụng hơn. Chúng ta sẽ thảo luận về một số tính năng này tiếp theo và sau đó giải thích
cách xuất bản một gói.

### Tạo các Chú thích Tài liệu Hữu ích

Việc lập tài liệu chính xác cho các gói của bạn sẽ giúp những người dùng khác biết cách thức và thời điểm
sử dụng chúng, vì vậy việc đầu tư thời gian để viết tài liệu là rất xứng đáng. Trong Chương
3, chúng ta đã thảo luận về cách chú thích mã Rust bằng hai dấu gạch chéo, `//`. Rust cũng có
một loại chú thích đặc biệt dành cho tài liệu, được gọi một cách thuận tiện là
_chú thích tài liệu_ (documentation comment), sẽ tạo ra tài liệu HTML. HTML
hiển thị nội dung của các chú thích tài liệu cho các mục API công khai dành cho
các lập trình viên quan tâm đến việc biết cách _sử dụng_ crate của bạn thay vì cách
crate của bạn được _triển khai_.

Các chú thích tài liệu sử dụng ba dấu gạch chéo, `///`, thay vì hai và hỗ trợ
ký hiệu Markdown để định dạng văn bản. Đặt các chú thích tài liệu ngay
trước mục mà chúng đang lập tài liệu. Liệt kê 14-1 hiển thị các chú thích tài liệu
cho một hàm `add_one` trong một crate tên là `my_crate`.

<Listing number="14-1" file-name="src/lib.rs" caption="Một chú thích tài liệu cho một hàm">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Ở đây, chúng ta đưa ra một mô tả về những gì hàm `add_one` thực hiện, bắt đầu một
phần với tiêu đề `Examples`, và sau đó cung cấp mã minh họa
cách sử dụng hàm `add_one`. Chúng ta có thể tạo tài liệu HTML từ
chú thích tài liệu này bằng cách chạy `cargo doc`. Lệnh này chạy công cụ
`rustdoc` được phân phối cùng với Rust và đặt tài liệu HTML đã tạo
vào thư mục _target/doc_.

Để thuận tiện, chạy `cargo doc --open` sẽ xây dựng HTML cho tài liệu của
crate hiện tại (cũng như tài liệu cho tất cả các phụ thuộc của
crate của bạn) và mở kết quả trong trình duyệt web. Điều hướng đến
hàm `add_one` và bạn sẽ thấy văn bản trong các chú thích tài liệu được
kết xuất như thế nào, như được hiển thị trong Hình 14-1:

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Hình 14-1: Tài liệu HTML cho hàm `add_one`</span>

#### Các phần thường được sử dụng

Chúng ta đã sử dụng tiêu đề Markdown `# Examples` trong Liệt kê 14-1 để tạo một phần
trong HTML với tiêu đề “Examples.” Dưới đây là một số phần khác mà các tác giả crate
thường sử dụng trong tài liệu của họ:

- **Panics**: Các tình huống mà hàm đang được lập tài liệu có thể
  gây ra panic. Những người gọi hàm không muốn chương trình của họ bị panic nên
  đảm bảo rằng họ không gọi hàm trong các tình huống này.
- **Errors**: Nếu hàm trả về một `Result`, việc mô tả các loại
  lỗi có thể xảy ra và những điều kiện nào có thể khiến những lỗi đó được
  trả về có thể hữu ích cho người gọi để họ có thể viết mã xử lý các
  loại lỗi khác nhau theo những cách khác nhau.
- **Safety**: Nếu hàm là `unsafe` để gọi (chúng ta thảo luận về sự không an toàn trong
  Chương 20), nên có một phần giải thích tại sao hàm đó không an toàn
  và bao gồm các bất biến (invariants) mà hàm mong muốn người gọi phải duy trì.

Hầu hết các chú thích tài liệu không cần tất cả các phần này, nhưng đây là
một danh sách kiểm tra tốt để nhắc nhở bạn về các khía cạnh của mã mà người dùng sẽ
quan tâm muốn biết.

#### Chú thích tài liệu dưới dạng các bài kiểm tra

Việc thêm các khối mã ví dụ trong các chú thích tài liệu của bạn có thể giúp minh họa
cách sử dụng thư viện của bạn và việc làm này mang lại một phần thưởng bổ sung: chạy `cargo test`
sẽ chạy các ví dụ mã trong tài liệu của bạn dưới dạng các bài kiểm tra! Không có gì
tốt hơn tài liệu có ví dụ. Nhưng không có gì tệ hơn các ví dụ
không hoạt động vì mã đã thay đổi kể từ khi tài liệu được
viết. Nếu chúng ta chạy `cargo test` với tài liệu cho hàm
`add_one` từ Liệt kê 14-1, chúng ta sẽ thấy một phần trong kết quả kiểm tra trông
như thế này:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Bây giờ, nếu chúng ta thay đổi hàm hoặc ví dụ sao cho `assert_eq!` trong
ví dụ gây ra panic và chạy lại `cargo test`, chúng ta sẽ thấy rằng các bài kiểm tra tài liệu (doc tests) bắt được
việc ví dụ và mã không còn khớp với nhau nữa!

#### Chú thích các mục chứa bên trong

Kiểu chú thích tài liệu `//!` thêm tài liệu vào mục chứa
chú thích thay vì vào các mục theo sau chú thích. Chúng ta thường sử dụng
những chú thích tài liệu này bên trong tệp gốc của crate (_src/lib.rs_ theo quy ước) hoặc
bên trong một mô-đun để lập tài liệu cho crate hoặc mô-đun đó một cách tổng thể.

Ví dụ, để thêm tài liệu mô tả mục đích của crate `my_crate`
chứa hàm `add_one`, chúng ta thêm các chú thích tài liệu
bắt đầu bằng `//!` vào đầu tệp _src/lib.rs_, như được hiển thị trong Liệt kê
14-2:

<Listing number="14-2" file-name="src/lib.rs" caption="Tài liệu cho toàn bộ crate `my_crate`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng không có bất kỳ mã nào sau dòng cuối cùng bắt đầu bằng `//!`. Bởi vì
chúng ta đã bắt đầu các chú thích bằng `//!` thay vì `///`, chúng ta đang lập tài liệu cho mục
chứa chú thích này thay vì một mục theo sau chú thích này. Trong
trường hợp này, mục đó là tệp _src/lib.rs_, vốn là gốc của crate. Những
chú thích này mô tả toàn bộ crate.

Khi chúng ta chạy `cargo doc --open`, các chú thích này sẽ hiển thị trên trang
đầu của tài liệu cho `my_crate` phía trên danh sách các mục công khai trong
crate, như được hiển thị trong Hình 14-2.

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Hình 14-2: Tài liệu được kết xuất cho `my_crate`,
bao gồm chú thích mô tả toàn bộ crate</span>

Các chú thích tài liệu bên trong các mục đặc biệt hữu ích để mô tả các crate và
mô-đun. Hãy sử dụng chúng để giải thích mục đích tổng thể của vật chứa để
giúp người dùng của bạn hiểu về tổ chức của crate.

{{#quiz ../quizzes/ch14-02-publishing-to-crates-io-sec1.toml}}

### Xuất khẩu một API công khai thuận tiện với `pub use`

Cấu trúc API công khai của bạn là một cân nhắc chính khi xuất bản một
crate. Những người sử dụng crate của bạn ít quen thuộc với cấu trúc này hơn bạn
và có thể gặp khó khăn khi tìm các phần họ muốn sử dụng nếu crate của bạn
có một hệ thống phân cấp mô-đun lớn.

Trong Chương 7, chúng ta đã đề cập đến cách làm cho các mục ở trạng thái công khai bằng từ khóa `pub`, và
đưa các mục vào một phạm vi bằng từ khóa `use`. Tuy nhiên, cấu trúc
có ý nghĩa đối với bạn trong khi bạn đang phát triển một crate có thể không thuận tiện
cho người dùng của bạn. Bạn có thể muốn tổ chức các struct của mình trong một hệ thống phân cấp
chứa nhiều cấp độ, nhưng sau đó những người muốn sử dụng một kiểu dữ liệu bạn đã
định nghĩa sâu trong hệ thống phân cấp có thể gặp khó khăn khi tìm hiểu xem kiểu đó có tồn tại hay không.
Họ cũng có thể cảm thấy khó chịu khi phải nhập `use`
`my_crate::some_module::another_module::UsefulType;` thay vì `use`
`my_crate::UsefulType;`.

Tin tốt là nếu cấu trúc _không_ thuận tiện cho người khác sử dụng
từ một thư viện khác, bạn không cần phải sắp xếp lại tổ chức nội bộ của mình:
thay vào đó, bạn có thể tái xuất khẩu (re-export) các mục để tạo ra một cấu trúc công khai khác
với cấu trúc riêng tư của bạn bằng cách sử dụng `pub use`. _Tái xuất khẩu_ lấy một mục công khai
ở một vị trí và làm cho nó ở trạng thái công khai ở một vị trí khác, như thể nó được
định nghĩa ở vị trí khác đó.

Ví dụ, giả sử chúng ta đã tạo một thư viện tên là `art` để mô hình hóa các khái niệm nghệ thuật.
Trong thư viện này có hai mô-đun: một mô-đun `kinds` chứa hai enum
tên là `PrimaryColor` và `SecondaryColor` và một mô-đun `utils` chứa một
hàm tên là `mix`, như được hiển thị trong Liệt kê 14-3:

<Listing number="14-3" file-name="src/lib.rs" caption="Một thư viện `art` với các mục được tổ chức thành các mô-đun `kinds` và `utils`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Hình 14-3 cho thấy trang đầu của tài liệu cho crate này được
tạo bởi `cargo doc` sẽ trông như thế nào:

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Hình 14-3: Trang đầu của tài liệu cho `art`
liệt kê các mô-đun `kinds` và `utils`</span>

Lưu ý rằng các kiểu `PrimaryColor` và `SecondaryColor` không được liệt kê trên
trang đầu, và hàm `mix` cũng vậy. Chúng ta phải nhấp vào `kinds` và `utils` để
thấy chúng.

Một crate khác phụ thuộc vào thư viện này sẽ cần các câu lệnh `use`
đưa các mục từ `art` vào phạm vi, chỉ định cấu trúc mô-đun
hiện đang được định nghĩa. Liệt kê 14-4 cho thấy một ví dụ về một crate
sử dụng các mục `PrimaryColor` và `mix` từ crate `art`:

<Listing number="14-4" file-name="src/main.rs" caption="Một crate sử dụng các mục của crate `art` với cấu trúc nội bộ của nó được xuất khẩu">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Tác giả của mã trong Liệt kê 14-4, người sử dụng crate `art`, đã phải
tìm ra rằng `PrimaryColor` nằm trong mô-đun `kinds` và `mix` nằm trong
mô-đun `utils`. Cấu trúc mô-đun của crate `art` phù hợp với
các nhà phát triển đang làm việc trên crate `art` hơn là những người sử dụng nó. Cấu trúc
nội bộ không chứa bất kỳ thông tin hữu ích nào cho ai đó đang cố gắng
hiểu cách sử dụng crate `art`, mà đúng hơn là gây ra sự nhầm lẫn vì
các nhà phát triển sử dụng nó phải tìm xem nên xem ở đâu, và phải chỉ định
tên mô-đun trong các câu lệnh `use`.

Để loại bỏ tổ chức nội bộ khỏi API công khai, chúng ta có thể sửa đổi
mã crate `art` trong Liệt kê 14-3 để thêm các câu lệnh `pub use` nhằm tái xuất khẩu
các mục ở cấp cao nhất, như được hiển thị trong Liệt kê 14-5:

<Listing number="14-5" file-name="src/lib.rs" caption="Thêm các câu lệnh `pub use` để tái xuất khẩu các mục">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

Tài liệu API mà `cargo doc` tạo ra cho crate này bây giờ sẽ liệt kê
và liên kết các mục tái xuất khẩu trên trang đầu, như được hiển thị trong Hình 14-4, giúp cho
các kiểu `PrimaryColor` và `SecondaryColor` và hàm `mix` dễ tìm hơn.

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Hình 14-4: Trang đầu của tài liệu cho `art`
liệt kê các mục tái xuất khẩu</span>

Người dùng crate `art` vẫn có thể thấy và sử dụng cấu trúc nội bộ từ Liệt kê
14-3 như được minh họa trong Liệt kê 14-4, hoặc họ có thể sử dụng
cấu trúc thuận tiện hơn trong Liệt kê 14-5, như được hiển thị trong Liệt kê 14-6:

<Listing number="14-6" file-name="src/main.rs" caption="Một chương trình sử dụng các mục tái xuất khẩu từ crate `art`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

Trong các trường hợp có nhiều mô-đun lồng nhau, việc tái xuất khẩu các kiểu dữ liệu ở cấp
cao nhất với `pub use` có thể tạo ra sự khác biệt đáng kể trong trải nghiệm của
những người sử dụng crate. Một cách sử dụng phổ biến khác của `pub use` là tái xuất khẩu
các định nghĩa của một phụ thuộc trong crate hiện tại để làm cho định nghĩa của crate đó
trở thành một phần của API công khai của crate của bạn.

Tạo ra một cấu trúc API công khai hữu ích mang tính nghệ thuật hơn là khoa học, và
bạn có thể lặp lại để tìm ra API phù hợp nhất cho người dùng của mình. Việc chọn `pub
use` mang lại cho bạn sự linh hoạt trong cách bạn cấu trúc crate của mình ở bên trong và
tách biệt cấu trúc nội bộ đó khỏi những gì bạn trình bày cho người dùng. Hãy xem
một số mã của các crate bạn đã cài đặt để xem liệu cấu trúc nội bộ của chúng
có khác với API công khai của chúng hay không.

### Thiết lập một tài khoản Crates.io

Trước khi bạn có thể xuất bản bất kỳ crate nào, bạn cần tạo một tài khoản trên
[crates.io](https://crates.io/)<!-- ignore --> và lấy một mã thông báo (token) API. Để làm như vậy,
hãy truy cập trang chủ tại [crates.io](https://crates.io/)<!-- ignore --> và đăng
nhập thông qua tài khoản GitHub. (Tài khoản GitHub hiện là một yêu cầu, nhưng
trang web có thể hỗ trợ các cách tạo tài khoản khác trong tương lai.) Sau khi
bạn đã đăng nhập, hãy truy cập cài đặt tài khoản của bạn tại
[https://crates.io/me/](https://crates.io/me/)<!-- ignore --> và lấy
khóa API của bạn. Sau đó chạy lệnh `cargo login` và dán khóa API của bạn khi được nhắc, như thế này:

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Lệnh này sẽ thông báo cho Cargo về mã thông báo API của bạn và lưu trữ nó cục bộ trong
_~/.cargo/credentials_. Lưu ý rằng mã thông báo này là một _bí mật_: đừng chia sẻ nó
với bất kỳ ai khác. Nếu bạn vô tình chia sẻ nó với bất kỳ ai vì bất kỳ lý do gì, bạn nên
thu hồi nó và tạo một mã thông báo mới trên [crates.io](https://crates.io/)<!-- ignore
-->.

### Thêm siêu dữ liệu vào một Crate mới

Giả sử bạn có một crate muốn xuất bản. Trước khi xuất bản, bạn sẽ cần
thêm một số siêu dữ liệu (metadata) trong phần `[package]` của tệp _Cargo.toml_
của crate.

Crate của bạn sẽ cần một cái tên duy nhất. Trong khi bạn đang làm việc trên một crate cục bộ,
bạn có thể đặt tên crate bất cứ thứ gì bạn muốn. Tuy nhiên, tên crate trên
[crates.io](https://crates.io/)<!-- ignore --> được cấp phát theo nguyên tắc ai đến trước được trước.
Một khi tên crate đã được lấy, không ai khác có thể xuất bản một crate
với tên đó. Trước khi cố gắng xuất bản một crate, hãy tìm kiếm tên bạn
muốn sử dụng. Nếu tên đó đã được sử dụng, bạn sẽ cần tìm một tên khác và
chỉnh sửa trường `name` trong tệp _Cargo.toml_ dưới phần `[package]` để
sử dụng tên mới để xuất bản, như sau:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Ngay cả khi bạn đã chọn một tên duy nhất, khi bạn chạy `cargo publish` để xuất bản
crate tại thời điểm này, bạn sẽ nhận được một cảnh báo và sau đó là một lỗi:

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

Điều này dẫn đến một lỗi vì bạn đang thiếu một số thông tin quan trọng: một
mô tả và giấy phép là bắt buộc để mọi người biết crate của bạn làm gì
và theo các điều khoản nào họ có thể sử dụng nó. Trong _Cargo.toml_, hãy thêm một mô tả chỉ
khoảng một hoặc hai câu, vì nó sẽ xuất hiện cùng với crate của bạn trong kết quả tìm kiếm.
Đối với trường `license`, bạn cần cung cấp một _giá trị định danh giấy phép_.
[Software Package Data Exchange (SPDX) của Linux Foundation][spdx] liệt kê các
định danh bạn có thể sử dụng cho giá trị này. Ví dụ, để chỉ định rằng bạn đã
cấp phép cho crate của mình bằng Giấy phép MIT, hãy thêm định danh `MIT`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Nếu bạn muốn sử dụng một giấy phép không xuất hiện trong SPDX, bạn cần đặt
văn bản của giấy phép đó vào một tệp, đưa tệp đó vào dự án của bạn, và sau đó
sử dụng `license-file` để chỉ định tên của tệp đó thay vì sử dụng
khóa `license`.

Hướng dẫn về việc giấy phép nào là phù hợp cho dự án của bạn nằm ngoài phạm vi
của cuốn sách này. Nhiều người trong cộng đồng Rust cấp phép cho các dự án của họ theo
cùng một cách như Rust bằng cách sử dụng giấy phép kép `MIT OR Apache-2.0`. Cách làm này
chứng minh rằng bạn cũng có thể chỉ định nhiều định danh giấy phép được phân tách
bởi `OR` để có nhiều giấy phép cho dự án của mình.

Với một tên duy nhất, phiên bản, mô tả của bạn và một giấy phép được thêm vào, tệp
_Cargo.toml_ cho một dự án đã sẵn sàng để xuất bản có thể trông như thế này:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Tài liệu của Cargo](https://doc.rust-lang.org/cargo/) mô tả các
siêu dữ liệu khác bạn có thể chỉ định để đảm bảo những người khác có thể khám phá và sử dụng crate của bạn
dễ dàng hơn.

### Xuất bản lên Crates.io

Bây giờ bạn đã tạo một tài khoản, lưu mã thông báo API, chọn tên cho
crate của mình và chỉ định siêu dữ liệu được yêu cầu, bạn đã sẵn sàng để xuất bản!
Việc xuất bản một crate sẽ tải một phiên bản cụ thể lên
[crates.io](https://crates.io/)<!-- ignore --> để người khác sử dụng.

Hãy cẩn thận, vì việc xuất bản là _vĩnh viễn_. Phiên bản đó không bao giờ có thể bị
ghi đè, và mã nguồn không thể bị xóa. Một mục tiêu chính của
[crates.io](https://crates.io/)<!-- ignore --> là hoạt động như một kho lưu trữ mã nguồn vĩnh viễn
để các bản dựng của tất cả các dự án phụ thuộc vào các crate từ
[crates.io](https://crates.io/)<!-- ignore --> sẽ tiếp tục hoạt động. Việc cho phép
xóa phiên bản sẽ khiến việc thực hiện mục tiêu đó trở nên bất khả thi. Tuy nhiên, không có
giới hạn về số lượng phiên bản crate bạn có thể xuất bản.

Chạy lệnh `cargo publish` một lần nữa. Bây giờ nó sẽ thành công:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Chúc mừng! Bây giờ bạn đã chia sẻ mã nguồn của mình với cộng đồng Rust, và
bất kỳ ai cũng có thể dễ dàng thêm crate của bạn làm phụ thuộc cho dự án của họ.

### Xuất bản một phiên bản mới của một Crate hiện có

Khi bạn đã thực hiện các thay đổi đối với crate của mình và sẵn sàng phát hành một phiên bản mới,
bạn thay đổi giá trị `version` được chỉ định trong tệp _Cargo.toml_ của mình và
xuất bản lại. Sử dụng [các quy tắc Semantic Versioning][semver] để quyết định xem
số phiên bản tiếp theo phù hợp là gì dựa trên các loại thay đổi bạn đã thực hiện.
Sau đó chạy `cargo publish` để tải lên phiên bản mới.

<!-- Old link, do not remove -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>

### Phản đối các phiên bản từ Crates.io với `cargo yank`

Mặc dù bạn không thể xóa các phiên bản trước đó của một crate, nhưng bạn có thể ngăn chặn bất kỳ
dự án nào trong tương lai thêm chúng làm một phụ thuộc mới. Điều này hữu ích khi một
phiên bản crate bị hỏng vì lý do này hay lý do khác. Trong những tình huống như vậy, Cargo
hỗ trợ việc yanking (thu hồi) một phiên bản crate.

_Yanking_ một phiên bản ngăn chặn các dự án mới phụ thuộc vào phiên bản đó trong khi
vẫn cho phép tất cả các dự án hiện tại đang phụ thuộc vào nó tiếp tục hoạt động. Về cơ bản, một
lệnh yank có nghĩa là tất cả các dự án có tệp _Cargo.lock_ sẽ không bị hỏng, và bất kỳ tệp
_Cargo.lock_ nào được tạo ra trong tương lai sẽ không sử dụng phiên bản đã bị yank.

Để yank một phiên bản của một crate, trong thư mục của crate mà bạn đã
xuất bản trước đó, hãy chạy `cargo yank` và chỉ định phiên bản bạn muốn
yank. Ví dụ, nếu chúng ta đã xuất bản một crate tên là `guessing_game` phiên bản
1.0.1 và chúng ta muốn yank nó, trong thư mục dự án của `guessing_game`, chúng ta sẽ
chạy:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Bằng cách thêm `--undo` vào lệnh, bạn cũng có thể hoàn tác một lệnh yank và cho phép các dự án
bắt đầu phụ thuộc vào một phiên bản một lần nữa:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Một lệnh yank _không_ xóa bất kỳ mã nào. Ví dụ, nó không thể xóa các bí mật
vô tình được tải lên. Nếu điều đó xảy ra, bạn phải đặt lại các bí mật đó ngay lập tức.

{{#quiz ../quizzes/ch14-02-publishing-to-crates-io-sec2.toml}}

[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
