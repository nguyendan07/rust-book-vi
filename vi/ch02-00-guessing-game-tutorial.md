# Lập trình Trò chơi Đoán số

Hãy bắt đầu khám phá Rust bằng cách cùng nhau thực hiện một dự án thực hành! Chương này sẽ giới thiệu cho bạn một vài khái niệm phổ biến trong Rust bằng cách chỉ cho bạn cách sử dụng chúng trong một chương trình thực tế. Bạn sẽ tìm hiểu về `let`, `match`, các phương thức (methods), các hàm liên kết (associated functions), các crate bên ngoài, và nhiều hơn thế nữa! Trong các chương tiếp theo, chúng ta sẽ tìm hiểu sâu hơn về các khái niệm này. Trong chương này, bạn sẽ thực hành các kiến thức nền tảng.

Chúng ta sẽ xây dựng một bài toán lập trình kinh điển cho người mới bắt đầu: trò chơi đoán số. Cách thức hoạt động như sau: chương trình sẽ tạo một số nguyên ngẫu nhiên từ 1 đến 100. Sau đó, nó sẽ nhắc người chơi nhập một số dự đoán. Sau khi người chơi nhập, chương trình sẽ cho biết số dự đoán đó quá nhỏ hay quá lớn. Nếu đoán đúng, trò chơi sẽ in thông báo chúc mừng và thoát ra.

> **Lưu ý:** Không có câu đố (quiz) nào trong chương này, vì mục đích chính là giúp bạn cảm nhận và làm quen với ngôn ngữ.

## Thiết lập Dự án Mới

Để thiết lập một dự án mới, hãy đi tới thư mục _projects_ mà bạn đã tạo ở Chương 1 và tạo một dự án mới bằng Cargo như sau:

```console
$ cargo new guessing_game
$ cd guessing_game
```

Lệnh đầu tiên, `cargo new`, nhận tên của dự án (`guessing_game`) làm đối số đầu tiên. Lệnh thứ hai chuyển vào thư mục của dự án mới.

Hãy xem tệp _Cargo.toml_ vừa được tạo:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Như bạn đã thấy trong Chương 1, `cargo new` tạo sẵn một chương trình “Hello, world!” cho bạn. Hãy kiểm tra tệp _src/main.rs_:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Bây giờ hãy biên dịch chương trình “Hello, world!” này và chạy nó trong cùng một bước bằng lệnh `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Lệnh `run` rất tiện lợi khi bạn cần lặp lại và thử nghiệm nhanh một dự án, như chúng ta sẽ làm trong trò chơi này: nhanh chóng kiểm tra từng bước trước khi chuyển sang bước tiếp theo.

Mở lại tệp _src/main.rs_. Bạn sẽ viết toàn bộ mã nguồn trong tệp này.

## Xử lý Lượt Đoán

Phần đầu tiên của chương trình đoán số sẽ yêu cầu người dùng nhập dữ liệu, xử lý dữ liệu đầu vào đó và kiểm tra xem dữ liệu có đúng định dạng mong đợi hay không. Để bắt đầu, chúng ta sẽ cho phép người chơi nhập vào một con số dự đoán. Hãy nhập đoạn mã trong Danh sách 2-1 vào _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Mã nguồn nhận một dự đoán từ người dùng và in ra">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Đoạn mã này chứa rất nhiều thông tin, vì vậy hãy cùng đi qua từng dòng một. Để nhận dữ liệu đầu vào từ người dùng và sau đó in kết quả ra màn hình, chúng ta cần đưa thư viện nhập/xuất `io` vào phạm vi (scope). Thư viện `io` đến từ thư viện chuẩn của Rust, được gọi là `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Theo mặc định, Rust có một tập hợp các mục được định nghĩa trong thư viện chuẩn mà nó tự động đưa vào phạm vi của mọi chương trình. Tập hợp này được gọi là _prelude_, và bạn có thể xem mọi thứ có trong nó [trong tài liệu thư viện chuẩn][prelude].

Nếu một kiểu dữ liệu bạn muốn sử dụng không nằm trong prelude, bạn phải đưa kiểu đó vào phạm vi một cách tường minh bằng câu lệnh `use`. Sử dụng thư viện `std::io` cung cấp cho bạn một số tính năng hữu ích, bao gồm khả năng nhận dữ liệu đầu vào từ người dùng.

Như bạn đã thấy trong Chương 1, hàm `main` là điểm bắt đầu (entry point) của chương trình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

Cú pháp `fn` dùng để khai báo một hàm mới; dấu ngoặc đơn `()` cho biết hàm không nhận tham số nào; và dấu ngoặc nhọn `{` mở đầu thân hàm.

Như bạn cũng đã học trong Chương 1, `println!` là một macro in một chuỗi văn bản ra màn hình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Đoạn mã này in một lời nhắc cho biết trò chơi là gì và yêu cầu người dùng nhập số.

### Lưu trữ Giá trị bằng Biến

Tiếp theo, chúng ta sẽ tạo một _biến_ (variable) để lưu trữ dữ liệu người dùng nhập vào, như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Bây giờ chương trình đang trở nên thú vị hơn! Có rất nhiều điều diễn ra trong dòng mã ngắn này. Chúng ta sử dụng câu lệnh `let` để tạo biến. Đây là một ví dụ khác:

```rust,ignore
let apples = 5;
```

Dòng này tạo một biến mới có tên là `apples` và liên kết (bind) nó với giá trị 5. Trong Rust, các biến theo mặc định là **bất biến** (immutable), nghĩa là một khi chúng ta đã gán cho biến một giá trị, giá trị đó sẽ không thay đổi. Chúng ta sẽ thảo luận chi tiết về khái niệm này trong phần [“Biến và Tính Khả Biến”][variables-and-mutability]<!-- ignore --> ở Chương 3. Để làm cho một biến có thể thay đổi được (khả biến - mutable), chúng ta thêm `mut` vào trước tên biến:

```rust,ignore
let apples = 5; // bất biến (immutable)
let mut bananas = 5; // khả biến (mutable)
```

> Lưu ý: Cú pháp `//` bắt đầu một chú thích (comment) kéo dài cho đến hết dòng. Rust bỏ qua mọi thứ trong các chú thích. Chúng ta sẽ thảo luận về chú thích chi tiết hơn trong [Chương 3][comments]<!-- ignore -->.

Quay trở lại chương trình trò chơi đoán số, bây giờ bạn biết rằng `let mut guess` sẽ khai báo một biến khả biến có tên là `guess`. Dấu bằng (`=`) báo cho Rust biết chúng ta muốn liên kết một giá trị với biến ngay bây giờ. Ở bên phải dấu bằng là giá trị mà `guess` được liên kết, đó là kết quả của việc gọi `String::new`, một hàm trả về một thể hiện (instance) mới của kiểu `String`. [`String`][string]<!-- ignore --> là một kiểu chuỗi do thư viện chuẩn cung cấp, có thể mở rộng kích thước và được mã hóa dưới dạng văn bản UTF-8.

Cú pháp `::` trong dòng `::new` cho biết rằng `new` là một hàm liên kết (associated function) của kiểu `String`. Một _hàm liên kết_ là một hàm được triển khai trên một kiểu dữ liệu, trong trường hợp này là `String`. Hàm `new` này tạo ra một chuỗi mới, rỗng. Bạn sẽ thấy hàm `new` trên rất nhiều kiểu dữ liệu vì đây là tên gọi quy ước phổ biến cho một hàm tạo ra một giá trị mới thuộc kiểu nào đó.

Tóm lại, dòng `let mut guess = String::new();` đã tạo ra một biến khả biến hiện đang liên kết với một thể hiện chuỗi `String` mới và rỗng.

### Nhận Dữ Liệu Đầu Vào từ Người Dùng

Hãy nhớ lại rằng chúng ta đã nạp tính năng nhập/xuất từ thư viện chuẩn bằng `use std::io;` ở dòng đầu tiên của chương trình. Bây giờ chúng ta sẽ gọi hàm `stdin` từ module `io`, hàm này cho phép chúng ta xử lý dữ liệu nhập từ người dùng:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Nếu chúng ta không nạp module `io` bằng `use std::io;` ở đầu chương trình, chúng ta vẫn có thể sử dụng hàm này bằng cách viết lệnh gọi hàm dưới dạng `std::io::stdin`. Hàm `stdin` trả về một thể hiện của [`std::io::Stdin`][iostdin]<!-- ignore -->, đây là một kiểu đại diện cho bộ xử lý luồng đầu vào chuẩn (standard input) của terminal.

Tiếp theo, dòng `.read_line(&mut guess)` gọi phương thức [`read_line`][read_line]<!-- ignore --> trên bộ xử lý đầu vào chuẩn để nhận dữ liệu từ người dùng. Chúng ta cũng truyền `&mut guess` làm đối số cho `read_line` để cho nó biết chuỗi nào sẽ lưu trữ dữ liệu đầu vào. Toàn bộ nhiệm vụ của `read_line` là lấy bất cứ thứ gì người dùng nhập vào luồng đầu vào chuẩn và nối (append) nó vào một chuỗi (mà không ghi đè nội dung cũ), vì vậy chúng ta truyền chuỗi đó làm đối số. Đối số chuỗi cần phải là khả biến (`mut`) để phương thức có thể thay đổi nội dung của chuỗi.

Ký tự `&` cho biết rằng đối số này là một **tham chiếu** (reference), cho phép nhiều phần khác nhau trong mã của bạn cùng truy cập vào một vùng dữ liệu mà không cần phải sao chép dữ liệu đó vào bộ nhớ nhiều lần. Tham chiếu là một tính năng mạnh mẽ và an toàn trong Rust. Bạn không cần phải biết quá nhiều chi tiết ngay bây giờ để hoàn thành chương trình này. Hiện tại, tất cả những gì bạn cần biết là giống như biến, các tham chiếu theo mặc định là bất biến. Do đó, bạn cần viết `&mut guess` thay vì `&guess` để làm cho nó có thể thay đổi được. (Chương 4 sẽ giải thích sâu hơn về tham chiếu).

<a id="handling-potential-failure-with-the-result-type"></a>

### Xử lý Lỗi Tiềm Ẩn với `Result`

Chúng ta vẫn đang xem xét dòng mã này. Bây giờ chúng ta đang thảo luận về phần thứ ba, nhưng lưu ý rằng nó vẫn là một phần của một dòng mã logic duy nhất:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Chúng ta có thể viết dòng mã này thành:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Tuy nhiên, một dòng quá dài rất khó đọc, vì vậy tốt nhất là nên ngắt dòng. Bạn nên thêm ký tự xuống dòng và khoảng trắng để giúp phân tách các dòng dài khi gọi phương thức với cú pháp `.method_name()`. Bây giờ hãy cùng thảo luận xem dòng này làm gì.

Như đã đề cập trước đó, `read_line` đưa những gì người dùng nhập vào chuỗi chúng ta truyền cho nó, nhưng nó cũng trả về một giá trị kiểu `Result`. [`Result`][result]<!-- ignore --> là một [_enumeration_][enums]<!-- ignore -->, thường được gọi tắt là _enum_, đây là một kiểu dữ liệu có thể ở một trong nhiều trạng thái có thể có. Chúng ta gọi mỗi trạng thái có thể có đó là một _variant_ (biến thể).

[Chương 6][enums]<!-- ignore --> sẽ đề cập chi tiết hơn về enum. Mục đích của kiểu `Result` này là mã hóa thông tin xử lý lỗi.

Các variant của `Result` là `Ok` và `Err`. Variant `Ok` cho biết thao tác đã thành công và nó chứa giá trị được tạo ra thành công bên trong. Variant `Err` có nghĩa là thao tác đã thất bại, và nó chứa thông tin về cách thức hoặc lý do tại sao thao tác đó thất bại.

Các giá trị của kiểu `Result`, giống như các giá trị của bất kỳ kiểu nào khác, đều có các phương thức được định nghĩa trên chúng. Một thể hiện của `Result` có một [phương thức `expect`][expect]<!-- ignore --> mà bạn có thể gọi. Nếu thể hiện `Result` này là một giá trị `Err`, `expect` sẽ khiến chương trình bị sập (crash / panic) và hiển thị thông báo mà bạn đã truyền làm đối số cho `expect`. Nếu phương thức `read_line` trả về một `Err`, nhiều khả năng đó là kết quả của một lỗi đến từ hệ điều hành bên dưới. Nếu thể hiện `Result` này là một giá trị `Ok`, `expect` sẽ lấy giá trị trả về mà `Ok` đang nắm giữ và trả lại chính giá trị đó cho bạn để bạn có thể sử dụng. Trong trường hợp này, giá trị đó là số byte trong dữ liệu đầu vào của người dùng.

Nếu bạn không gọi `expect`, chương trình vẫn sẽ biên dịch được, nhưng bạn sẽ nhận được một cảnh báo (warning):

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust cảnh báo rằng bạn chưa sử dụng giá trị `Result` trả về từ `read_line`, cho thấy chương trình chưa xử lý một lỗi có thể xảy ra.

Cách đúng đắn để loại bỏ cảnh báo này là viết mã xử lý lỗi thực sự, nhưng trong trường hợp của chúng ta, chúng ta chỉ muốn dừng chương trình ngay khi có sự cố xảy ra, vì vậy chúng ta có thể sử dụng `expect`. Bạn sẽ tìm hiểu về cách phục hồi từ các lỗi trong [Chương 9][recover]<!-- ignore -->.

### In Giá Trị bằng Ký Hiệu Giữ Chỗ `println!`

Bỏ qua dấu ngoặc nhọn đóng, chỉ còn một dòng nữa cần thảo luận trong đoạn mã cho đến thời điểm này:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Dòng này in chuỗi hiện chứa dữ liệu đầu vào của người dùng ra màn hình. Cặp dấu ngoặc nhọn `{}` là một ký hiệu giữ chỗ (placeholder): hãy hình dung `{}` giống như chiếc càng cua nhỏ giữ một giá trị tại vị trí đó. Khi in giá trị của một biến, tên biến có thể được đặt trực tiếp bên trong dấu ngoặc nhọn. Khi in kết quả của một biểu thức, hãy đặt dấu ngoặc nhọn rỗng `{}` trong chuỗi định dạng, sau đó đặt danh sách các biểu thức cách nhau bằng dấu phẩy theo đúng thứ tự. Việc in một biến và kết quả của một biểu thức trong một lệnh gọi `println!` sẽ trông như thế này:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Đoạn mã này sẽ in ra dòng chữ `x = 5 and y + 2 = 12`.

### Kiểm Thử Phần Đầu Tiên

Hãy cùng kiểm tra phần đầu tiên của trò chơi đoán số. Chạy chương trình bằng `cargo run`:

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Tại thời điểm này, phần đầu tiên của trò chơi đã hoàn thành: chúng ta đã nhận được dữ liệu đầu vào từ bàn phím và in nó ra màn hình.

## Tạo Số Bí Mật

Tiếp theo, chúng ta cần tạo một số bí mật mà người dùng sẽ cố gắng đoán. Số bí mật phải khác nhau mỗi lần chơi để trò chơi thú vị khi chơi nhiều lần. Chúng ta sẽ sử dụng một số ngẫu nhiên từ 1 đến 100 để trò chơi không quá khó. Rust chưa bao gồm chức năng tạo số ngẫu nhiên trong thư viện chuẩn của nó. Tuy nhiên, đội ngũ phát triển Rust cung cấp một [crate `rand`][randcrate] chứa chức năng này.

### Sử Dụng Crate để Mở Rộng Tính Năng

Hãy nhớ rằng một crate là một tập hợp các tệp mã nguồn Rust. Dự án chúng ta đang xây dựng là một _binary crate_, tức là một tệp thực thi. Crate `rand` là một _library crate_, chứa mã được thiết kế để sử dụng trong các chương trình khác và không thể tự chạy độc lập.

Khả năng phối hợp các crate bên ngoài chính là điểm sáng của Cargo. Trước khi có thể viết mã sử dụng `rand`, chúng ta cần chỉnh sửa tệp _Cargo.toml_ để thêm crate `rand` làm một gói phụ thuộc (dependency). Mở tệp đó ngay bây giờ và thêm dòng sau vào cuối, bên dưới tiêu đề mục `[dependencies]` mà Cargo đã tạo sẵn:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

Trong tệp _Cargo.toml_, mọi thứ theo sau một tiêu đề đều thuộc về phần đó cho đến khi một phần khác bắt đầu. Trong `[dependencies]`, bạn cho Cargo biết dự án của bạn phụ thuộc vào những crate bên ngoài nào và bạn yêu cầu phiên bản nào của các crate đó. Trong trường hợp này, chúng ta chỉ định crate `rand` với phiên bản theo chuẩn semantic versioning là `0.8.5`. Cargo hiểu [Semantic Versioning][semver]<!-- ignore --> (thường gọi là _SemVer_), đây là một tiêu chuẩn viết số phiên bản. Chỉ định `0.8.5` thực chất là cách viết tắt của `^0.8.5`, nghĩa là bất kỳ phiên bản nào tối thiểu là 0.8.5 nhưng thấp hơn 0.9.0.

Cargo coi các phiên bản này có API công khai tương thích với phiên bản 0.8.5, và chỉ định này đảm bảo bạn sẽ nhận được bản phát hành vá lỗi mới nhất mà vẫn biên dịch được với mã trong chương này. Bất kỳ phiên bản 0.9.0 trở lên nào đều không được đảm bảo có cùng API như những gì các ví dụ sau đây sử dụng.

Bây giờ, mà không cần thay đổi bất kỳ dòng mã nào, hãy biên dịch dự án như được hiển thị trong Danh sách 2-2.

<Listing number="2-2" caption="Kết quả từ việc chạy `cargo build` sau khi thêm crate rand làm gói phụ thuộc">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

Bạn có thể thấy các số phiên bản khác (nhưng tất cả chúng đều tương thích với mã nhờ SemVer!) và các dòng thông báo khác nhau (tùy thuộc vào hệ điều hành), và các dòng có thể xuất hiện theo thứ tự khác.

Khi chúng ta thêm một gói phụ thuộc bên ngoài, Cargo sẽ tải phiên bản mới nhất của mọi thứ mà gói phụ thuộc đó cần từ _registry_, là một bản sao dữ liệu từ [Crates.io][cratesio]. Crates.io là nơi mọi người trong hệ sinh thái Rust đăng tải các dự án Rust mã nguồn mở của họ để người khác sử dụng.

Sau khi cập nhật registry, Cargo kiểm tra mục `[dependencies]` và tải xuống bất kỳ crate nào được liệt kê mà chưa được tải về máy. Trong trường hợp này, mặc dù chúng ta chỉ liệt kê `rand` làm phụ thuộc, Cargo cũng đã tự động tải về các crate khác mà `rand` cần để hoạt động. Sau khi tải xuống các crate, Rust sẽ biên dịch chúng và sau đó biên dịch dự án với các phụ thuộc đã sẵn sàng.

Nếu bạn chạy lại `cargo build` ngay lập tức mà không thực hiện bất kỳ thay đổi nào, bạn sẽ không thấy bất kỳ dòng đầu ra nào ngoài dòng `Finished`. Cargo biết nó đã tải xuống và biên dịch các phụ thuộc, và bạn chưa thay đổi bất kỳ điều gì về chúng trong tệp _Cargo.toml_. Cargo cũng biết bạn chưa thay đổi mã nguồn của mình, vì vậy nó cũng không biên dịch lại.

#### Đảm bảo Khả năng Tái Tạo Bản Build với Tệp _Cargo.lock_

Cargo có một cơ chế đảm bảo rằng bạn có thể xây dựng lại cùng một kết quả chính xác mỗi khi bạn hoặc bất kỳ ai khác biên dịch mã của bạn: Cargo sẽ chỉ sử dụng các phiên bản phụ thuộc bạn đã chỉ định cho đến khi bạn yêu cầu khác. Ví dụ: giả sử tuần tới phiên bản 0.8.6 của crate `rand` ra mắt và phiên bản đó chứa một bản sửa lỗi quan trọng, nhưng nó cũng chứa một lỗi hồi quy (regression) làm hỏng mã của bạn. Để xử lý việc này, Rust tạo tệp _Cargo.lock_ trong lần đầu tiên bạn chạy `cargo build`.

Khi bạn xây dựng một dự án lần đầu tiên, Cargo tìm ra tất cả các phiên bản của các gói phụ thuộc phù hợp với tiêu chí và sau đó ghi chúng vào tệp _Cargo.lock_. Khi bạn xây dựng dự án trong tương lai, Cargo sẽ thấy tệp _Cargo.lock_ đã tồn tại và sẽ sử dụng các phiên bản được chỉ định trong đó thay vì phải tính toán lại phiên bản từ đầu. Điều này cho phép bạn tự động có một bản build có thể tái tạo (reproducible build). Tệp _Cargo.lock_ thường được lưu vào hệ thống quản lý phiên bản (Git) cùng với toàn bộ mã nguồn dự án.

#### Cập nhật một Crate lên Phiên bản Mới

Khi bạn _thực sự_ muốn cập nhật một crate, Cargo cung cấp lệnh `update`, lệnh này sẽ bỏ qua tệp _Cargo.lock_ và tìm ra tất cả các phiên bản mới nhất phù hợp với chỉ định của bạn trong _Cargo.toml_:

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.9.0)
```

Cargo bỏ qua bản phát hành 0.9.0 vì nó nằm ngoài phạm vi `^0.8.5`. Để sử dụng phiên bản 0.9.0 hoặc bất kỳ phiên bản nào trong chuỗi 0.9._x_, bạn sẽ phải cập nhật tệp _Cargo.toml_ trực tiếp:

```toml
[dependencies]
rand = "0.9.0"
```

### Tạo Số Ngẫu Nhiên

Bây giờ hãy sử dụng `rand` để tạo một số ngẫu nhiên. Hãy cập nhật _src/main.rs_ như được hiển thị trong Danh sách 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Thêm mã nguồn để tạo số ngẫu nhiên">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Đầu tiên, chúng ta thêm dòng `use rand::Rng;`. Trait `Rng` định nghĩa các phương thức mà các bộ tạo số ngẫu nhiên triển khai, và trait này phải nằm trong phạm vi (scope) để chúng ta có thể sử dụng các phương thức đó. Chương 10 sẽ đề cập chi tiết về trait.

Tiếp theo, chúng ta gọi hàm `rand::thread_rng` để lấy bộ tạo số ngẫu nhiên cục bộ cho luồng thực thi hiện tại và được khởi tạo seed bởi hệ điều hành. Sau đó, chúng ta gọi phương thức `gen_range` trên bộ tạo số ngẫu nhiên đó. Phương thức `gen_range` nhận một biểu thức range làm đối số và tạo ra một số ngẫu nhiên trong khoảng đó. Dạng biểu thức range chúng ta sử dụng ở đây có dạng `start..=end` và bao gồm cả cận dưới lẫn cận trên, vì vậy chúng ta chỉ định `1..=100` để yêu cầu một số từ 1 đến 100.

> Lưu ý: Một tính năng tuyệt vời khác của Cargo là chạy lệnh `cargo doc --open` sẽ xây dựng toàn bộ tài liệu hướng dẫn cục bộ do tất cả các gói phụ thuộc cung cấp và mở nó trong trình duyệt của bạn.

Dòng mới thứ hai in số bí mật ra màn hình. Điều này hữu ích trong quá trình phát triển để chúng ta có thể kiểm thử trò chơi, nhưng chúng ta sẽ xóa nó khỏi phiên bản cuối cùng.

Hãy chạy thử chương trình vài lần:

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Bạn sẽ nhận được các số ngẫu nhiên khác nhau, và tất cả chúng đều nằm trong khoảng từ 1 đến 100. Tuyệt vời!

## So Sánh Số Dự Đoán với Số Bí Mật

Bây giờ chúng ta đã có dữ liệu đầu vào của người dùng và một số ngẫu nhiên, chúng ta có thể so sánh chúng. Bước đó được hiển thị trong Danh sách 2-4. Lưu ý rằng đoạn mã này sẽ chưa biên dịch được ngay, như chúng tôi sẽ giải thích.

<Listing number="2-4" file-name="src/main.rs" caption="Xử lý các giá trị trả về có thể có khi so sánh hai số">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Đầu tiên, chúng ta thêm một câu lệnh `use` khác để đưa kiểu `std::cmp::Ordering` từ thư viện chuẩn vào phạm vi. Kiểu `Ordering` là một enum khác và có các variant là `Less`, `Greater`, và `Equal`. Đây là ba kết quả có thể xảy ra khi bạn so sánh hai giá trị.

Sau đó, chúng ta thêm năm dòng mới ở cuối sử dụng kiểu `Ordering`. Phương thức `cmp` so sánh hai giá trị và có thể được gọi trên bất kỳ thứ gì có thể so sánh được. Nó nhận một tham chiếu đến bất kỳ thứ gì bạn muốn so sánh cùng: ở đây nó đang so sánh `guess` với `secret_number`. Sau đó, nó trả về một variant của enum `Ordering`. Chúng ta sử dụng một biểu thức [`match`][match]<!-- ignore --> để quyết định việc cần làm tiếp theo dựa trên variant nào của `Ordering` được trả về.

Một biểu thức `match` được tạo thành từ các _nhánh_ (arms). Một nhánh bao gồm một _mẫu_ (pattern) để khớp với giá trị, và đoạn mã sẽ được chạy nếu giá trị được cung cấp cho `match` khớp với mẫu của nhánh đó. Rust lấy giá trị được đưa vào `match` và lần lượt duyệt qua từng mẫu của các nhánh. Mẫu và cấu trúc `match` là những tính năng cực kỳ mạnh mẽ trong Rust, sẽ được đề cập chi tiết trong Chương 6 và Chương 19.

Tuy nhiên, đoạn mã trong Danh sách 2-4 vẫn chưa biên dịch được. Hãy thử biên dịch:

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Trọng tâm của lỗi là **không khớp kiểu dữ liệu** (_mismatched types_). Rust có một hệ thống kiểu tĩnh và mạnh mẽ. Tuy nhiên, nó cũng có khả năng suy luận kiểu (type inference). Khi chúng ta viết `let mut guess = String::new()`, Rust có thể suy luận rằng `guess` phải là một `String` và không bắt chúng ta phải viết kiểu một cách tường minh. Mặt khác, `secret_number` lại là một kiểu số. Một số kiểu số trong Rust có thể chứa giá trị từ 1 đến 100: `i32` (số nguyên có dấu 32-bit), `u32` (số nguyên không dấu 32-bit), `i64` (số nguyên 64-bit), v.v. Trừ khi được chỉ định khác, Rust mặc định chọn `i32`. Lý do của lỗi biên dịch là Rust **không thể so sánh giữa một kiểu chuỗi và một kiểu số**.

Do đó, chúng ta cần chuyển đổi `String` mà chương trình đọc được thành một kiểu số để có thể so sánh số học với số bí mật. Chúng ta thực hiện điều đó bằng cách thêm dòng này vào thân hàm `main`:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Dòng mã đó là:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Chúng ta tạo một biến có tên là `guess`. Nhưng khoan đã, chương trình chẳng phải đã có một biến tên là `guess` rồi sao? Đúng vậy, nhưng Rust cho phép chúng ta **che khuất** (shadow) giá trị trước đó của `guess` bằng một biến mới. Tính năng _Shadowing_ cho phép chúng ta tái sử dụng tên biến `guess` thay vì buộc chúng ta phải tạo ra hai biến riêng biệt như `guess_str` và `guess`. Chúng ta sẽ tìm hiểu kỹ hơn về tính năng này trong [Chương 3][shadowing]<!-- ignore -->.

Chúng ta liên kết biến mới này với biểu thức `guess.trim().parse()`. Biến `guess` trong biểu thức đề cập đến biến `guess` ban đầu chứa chuỗi đầu vào. Phương thức `trim` trên một thể hiện `String` sẽ loại bỏ mọi khoảng trắng ở đầu và cuối chuỗi, cũng như ký tự xuống dòng `\n` hoặc `\r\n` sinh ra khi người dùng nhấn phím <kbd>Enter</kbd>.

Phương thức [`parse`][parse]<!-- ignore --> chuyển đổi một chuỗi thành một kiểu dữ liệu khác. Ở đây, chúng ta sử dụng nó để chuyển đổi từ chuỗi thành số nguyên `u32` thông qua chú thích kiểu tường minh `let guess: u32`.

Phương thức `parse` trả về một kiểu `Result` vì nó có thể thất bại (ví dụ nếu người dùng nhập chữ cái `A👍%`). Chúng ta tiếp tục dùng `.expect(...)` để xử lý: nếu `parse` trả về `Err`, chương trình sẽ dừng lại và hiển thị thông báo lỗi; nếu thành công (`Ok`), `expect` sẽ trả về con số nguyên `u32`.

Hãy chạy lại chương trình:

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Tuyệt vời! Ngay cả khi có khoảng trắng trước số đoán, chương trình vẫn nhận diện chính xác số 76 và đưa ra so sánh `Too big!`.

## Cho Phép Đoán Nhiều Lần Bằng Vòng Lặp

Từ khóa `loop` tạo ra một vòng lặp vô hạn. Chúng ta sẽ thêm một vòng lặp để cho người dùng nhiều cơ hội đoán số hơn:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Chương trình bây giờ sẽ liên tục hỏi số dự đoán tiếp theo. Người dùng có thể nhấn tổ hợp phím <kbd>Ctrl</kbd>-<kbd>C</kbd> để ngắt chương trình, hoặc nhập vào một ký tự không phải số để làm chương trình thoát ra qua lệnh `expect`.

### Thoát Sau Khi Đoán Đúng

Hãy lập trình để trò chơi kết thúc khi người dùng chiến thắng bằng cách thêm câu lệnh `break`:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Thêm dòng `break` sau `You win!` sẽ khiến chương trình thoát khỏi vòng lặp khi người dùng đoán đúng số bí mật. Thoát khỏi vòng lặp cũng đồng nghĩa với việc kết thúc chương trình vì vòng lặp là phần cuối cùng của hàm `main`.

### Xử Lý Dữ Liệu Đầu Vào Không Hợp Lệ

Để hoàn thiện trải nghiệm trò chơi hơn nữa, thay vì làm sập chương trình khi người dùng nhập dữ liệu không phải là số, hãy làm cho trò chơi bỏ qua đầu vào không hợp lệ để người chơi có thể tiếp tục đoán. Chúng ta có thể làm điều đó bằng cách thay đổi dòng chuyển đổi `guess` từ `String` sang `u32` bằng biểu thức `match`, như trong Danh sách 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Bỏ qua đầu vào không phải là số và yêu cầu đoán lại thay vì làm sập chương trình">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Chúng ta chuyển từ việc gọi `expect` sang biểu thức `match` để chuyển từ việc làm sập chương trình sang chủ động xử lý lỗi. Nếu `parse` thành công và trả về `Ok(num)`, nhánh đầu tiên sẽ trả về giá trị `num`. Nếu `parse` thất bại và trả về `Err(_)`, nhánh thứ hai sẽ thực thi từ khóa `continue`, yêu cầu chương trình chuyển ngay sang vòng lặp tiếp theo của `loop` và tiếp tục yêu cầu người dùng nhập lại! Dấu gạch dưới `_` là một giá trị bắt tất cả (catch-all), khớp với mọi loại lỗi mà không quan tâm chi tiết lỗi là gì.

Bây giờ mọi thứ trong chương trình sẽ hoạt động hoàn hảo như mong đợi:

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Tuyệt vời! Chỉ cần một chỉnh sửa nhỏ cuối cùng: xóa dòng `println!` in ra số bí mật ban đầu. Danh sách 2-6 hiển thị mã nguồn hoàn chỉnh cuối cùng.

<Listing number="2-6" file-name="src/main.rs" caption="Mã nguồn hoàn chỉnh của trò chơi đoán số">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Đến đây, bạn đã xây dựng thành công trò chơi đoán số hoàn chỉnh đầu tiên trong Rust. Xin chúc mừng!

## Tóm Tắt

Dự án thực hành này đã giúp bạn làm quen trực quan với nhiều khái niệm cốt lõi của Rust: `let`, `match`, hàm, phương thức, cách sử dụng các crate bên ngoài và xử lý lỗi cơ bản. Trong các chương tiếp theo, bạn sẽ tìm hiểu chi tiết hơn về các khái niệm này:
- Chương 3 đề cập đến các khái niệm quen thuộc trong lập trình như biến, kiểu dữ liệu, hàm và luồng điều khiển.
- Chương 4 khám phá quyền sở hữu (Ownership) — tính năng độc đáo định hình nên sức mạnh của Rust.
- Chương 5 thảo luận về struct và cú pháp phương thức.
- Chương 6 giải thích cách thức hoạt động của enum và khớp mẫu (pattern matching).

[prelude]: https://doc.rust-lang.org/std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: https://doc.rust-lang.org/std/string/struct.String.html
[iostdin]: https://doc.rust-lang.org/std/io/struct.Stdin.html
[read_line]: https://doc.rust-lang.org/std/io/struct.Stdin.html#method.read_line
[result]: https://doc.rust-lang.org/std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: https://doc.rust-lang.org/std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: https://doc.rust-lang.org/std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
