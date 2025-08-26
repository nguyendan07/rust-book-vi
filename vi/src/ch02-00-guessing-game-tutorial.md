# Lập trình một trò chơi đoán số

Hãy bắt đầu với Rust bằng cách làm một dự án thực hành cùng nhau! Chương này giới thiệu cho bạn một vài khái niệm Rust phổ biến thông qua việc cho bạn thấy cách dùng chúng trong một chương trình thực tế. Bạn sẽ học về `let`, `match`, phương thức, hàm liên quan (associated functions), crate bên ngoài, và nhiều thứ khác! Trong các chương sau, chúng ta sẽ khám phá những ý tưởng này chi tiết hơn. Ở chương này, bạn sẽ thực hành những khái niệm cơ bản.

Chúng ta sẽ triển khai một bài toán cổ điển dành cho người mới học: một trò chơi đoán số. Cách chơi như sau: chương trình sẽ tạo ra một số nguyên ngẫu nhiên từ 1 đến 100. Sau đó chương trình sẽ yêu cầu người chơi nhập một dự đoán. Sau khi nhập, chương trình sẽ cho biết dự đoán quá thấp hay quá cao. Nếu dự đoán đúng, trò chơi sẽ in ra lời chúc mừng và thoát.

## Thiết lập dự án mới

Để tạo một dự án mới, vào thư mục _projects_ mà bạn đã tạo trong Chương 1 và tạo một dự án mới bằng Cargo, như sau:

```console
$ cargo new guessing_game
$ cd guessing_game
```

Lệnh đầu tiên, `cargo new`, lấy tên dự án (`guessing_game`) làm đối số. Lệnh thứ hai chuyển vào thư mục dự án mới.

Hãy nhìn vào tệp _Cargo.toml_ được tạo:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Như bạn đã thấy trong Chương 1, `cargo new` tạo ra một chương trình “Hello, world!” cho bạn. Hãy xem tệp _src/main.rs_:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Bây giờ hãy biên dịch chương trình “Hello, world!” này và chạy nó trong cùng một bước bằng lệnh `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Lệnh `run` rất hữu ích khi bạn cần lặp nhanh trên một dự án, như chúng ta sẽ làm trong trò chơi này, thử nghiệm từng bản lặp trước khi tiếp tục.

Mở lại tệp _src/main.rs_. Bạn sẽ viết toàn bộ mã trong tệp này.

## Xử lý một dự đoán

Phần đầu tiên của chương trình đoán số sẽ yêu cầu nhập dữ liệu từ người dùng, xử lý dữ liệu đó, và kiểm tra rằng dữ liệu ở dạng mong đợi. Để bắt đầu, ta sẽ cho phép người chơi nhập một dự đoán. Nhập mã ở Listing 2-1 vào _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Mã lấy dự đoán từ người dùng và in nó ra">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Mã này chứa nhiều thông tin, vậy hãy đi qua từng dòng. Để nhận dữ liệu người dùng rồi in kết quả, ta cần đưa thư viện `io` (input/output) vào phạm vi. Thư viện `io` đến từ thư viện chuẩn, gọi là `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Theo mặc định, Rust có một tập các mục được định nghĩa trong thư viện chuẩn mà nó đưa vào phạm vi của mọi chương trình. Tập này gọi là _prelude_, và bạn có thể xem tất cả trong tài liệu của thư viện chuẩn [ở đây][prelude].

Nếu một kiểu bạn muốn dùng không có trong prelude, bạn phải đưa kiểu đó vào phạm vi rõ ràng bằng một câu lệnh `use`. Việc dùng `std::io` cung cấp cho bạn nhiều tính năng hữu ích, bao gồm khả năng chấp nhận dữ liệu người dùng.

Như bạn đã thấy trong Chương 1, hàm `main` là điểm vào của chương trình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

Cú pháp `fn` khai báo một hàm mới; dấu ngoặc tròn `()` cho biết không có tham số; và dấu ngoặc nhọn `{` bắt đầu phần thân hàm.

Như bạn cũng đã học trong Chương 1, `println!` là một macro in một chuỗi ra màn hình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Dòng này in một lời nhắc nói rõ trò chơi là gì và yêu cầu người dùng nhập dữ liệu.

### Lưu trữ giá trị với biến

Tiếp theo, chúng ta sẽ tạo một _biến_ để lưu dữ liệu người dùng, như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Bây giờ chương trình bắt đầu thú vị! Có nhiều điều đang xảy ra trong một dòng nhỏ này. Ta dùng câu lệnh `let` để tạo biến. Đây là một ví dụ khác:

```rust,ignore
let apples = 5;
```

Dòng này tạo một biến mới tên là `apples` và gán nó giá trị 5. Trong Rust, biến mặc định là bất biến (immutable), nghĩa là khi ta gán giá trị cho biến, giá trị đó sẽ không thay đổi. Chúng ta sẽ thảo luận chi tiết về khái niệm này trong phần [“Variables and Mutability”][variables-and-mutability]<!-- ignore --> ở Chương 3. Để làm biến có thể thay đổi, ta thêm `mut` trước tên biến:

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> Lưu ý: Cú pháp `//` bắt đầu một chú thích (comment) chạy đến cuối dòng. Rust bỏ qua mọi thứ trong chú thích. Chúng ta sẽ bàn về chú thích kỹ hơn trong [Chương 3][comments]<!-- ignore -->.

Quay trở lại chương trình đoán số, bạn biết rằng `let mut guess` sẽ giới thiệu một biến có thể thay đổi tên là `guess`. Dấu bằng (`=`) nói với Rust rằng chúng ta muốn gán một giá trị cho biến. Ở phía bên phải của dấu bằng là kết quả của việc gọi `String::new`, một hàm trả về một thể hiện mới của `String`.
[`String`][string]<!-- ignore --> là một kiểu chuỗi do thư viện chuẩn cung cấp, là một mảng ký tự có thể tăng kích thước, mã hóa UTF-8.

Cú pháp `::` trong `String::new` chỉ ra rằng `new` là một hàm liên quan (associated function) của kiểu `String`. Một _associated function_ là một hàm được triển khai trên một kiểu; trong trường hợp này là `String`. Hàm `new` tạo một chuỗi mới rỗng. Bạn sẽ thấy hàm `new` trên nhiều kiểu vì đó là cái tên phổ biến cho hàm khởi tạo một giá trị mới.

Đầy đủ, dòng `let mut guess = String::new();` đã tạo một biến có thể thay đổi đang được gán một thể hiện `String` rỗng. Whew!

### Nhận dữ liệu người dùng

Hãy nhớ rằng chúng ta đã đưa chức năng input/output vào phạm vi với `use std::io;` ở đầu chương trình. Bây giờ ta sẽ gọi hàm `stdin` từ module `io`, hàm này cho phép ta xử lý dữ liệu từ người dùng:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Nếu chúng ta chưa nhập module `io` bằng `use std::io;` ở đầu chương trình, ta vẫn có thể dùng hàm này bằng cách viết đầy đủ là `std::io::stdin`. Hàm `stdin` trả về một thể hiện của [`std::io::Stdin`][iostdin]<!-- ignore -->, là một kiểu đại diện cho tay cầm (handle) vào đầu vào chuẩn (standard input) của terminal.

Tiếp theo, dòng `.read_line(&mut guess)` gọi phương thức `read_line` trên tay cầm đầu vào chuẩn để nhận dữ liệu người dùng. Chúng ta cũng truyền `&mut guess` làm đối số cho `read_line` để nói cho nó biết chuỗi nào sẽ lưu dữ liệu nhập. Toàn bộ nhiệm vụ của `read_line` là lấy những gì người dùng nhập vào và nối thêm vào chuỗi (không ghi đè nội dung), nên ta truyền chuỗi đó như một đối số. Đối số chuỗi cần phải là mutable để phương thức có thể thay đổi nội dung chuỗi.

Ký hiệu `&` cho biết đối số này là một _tham chiếu_ (reference), cho phép nhiều phần của mã truy cập cùng một dữ liệu mà không cần sao chép. Tham chiếu là một tính năng phức tạp, và một trong những lợi thế lớn của Rust là cách sử dụng tham chiếu an toàn và dễ dàng. Hiện tại bạn không cần biết quá nhiều chi tiết; bây giờ chỉ cần biết rằng, giống như biến, tham chiếu mặc định là bất biến. Do đó, bạn phải viết `&mut guess` thay vì `&guess` để làm cho nó có thể thay đổi. (Chương 4 sẽ giải thích tham chiếu chi tiết hơn.)

<!-- Old heading. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Xử lý khả năng thất bại với `Result`

Chúng ta vẫn đang bàn về dòng mã này. Ta đang thảo luận về phần thứ ba của dòng, nhưng lưu ý rằng nó vẫn là một dòng logic đơn. Phần tiếp theo là phương thức:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Chúng ta có thể đã viết đoạn mã này như sau:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Tuy nhiên, một dòng dài khó đọc, nên tốt nhất là chia ra. Thường thì khôn ngoan khi thêm dấu xuống dòng và khoảng trắng để giúp tách các dòng dài khi bạn gọi phương thức với cú pháp `.method_name()`. Bây giờ hãy bàn xem dòng này làm gì.

Như đã đề cập trước đó, `read_line` đặt những gì người dùng nhập vào chuỗi mà ta truyền cho nó, nhưng nó cũng trả về một giá trị `Result`. [`Result`][result]<!-- ignore --> là một [_enumeration_][enums]<!-- ignore -->, thường gọi là _enum_, là một kiểu có thể ở trong một trong nhiều trạng thái có thể. Ta gọi mỗi trạng thái có thể là một _variant_.

[Chương 6][enums]<!-- ignore --> sẽ nói chi tiết hơn về enum. Mục đích của kiểu `Result` là mã hoá thông tin xử lý lỗi.

Các variant của `Result` là `Ok` và `Err`. Variant `Ok` chỉ ra thao tác thành công, và chứa giá trị được tạo ra thành công. Variant `Err` có nghĩa thao tác thất bại, và chứa thông tin về cách hoặc lý do thất bại.

Các giá trị của kiểu `Result`, như mọi kiểu khác, có phương thức định nghĩa trên chúng. Một thể hiện của `Result` có phương thức [`expect`][expect] mà bạn có thể gọi. Nếu thể hiện `Result` này là một giá trị `Err`, `expect` sẽ làm chương trình bị sập và hiển thị thông điệp bạn truyền. Nếu `read_line` trả về `Err`, khả năng cao đó là do lỗi từ hệ điều hành. Nếu thể hiện `Result` này là `Ok`, `expect` sẽ lấy giá trị trả về bên trong `Ok` và trả chỉ giá trị đó để bạn dùng. Trong trường hợp này, giá trị đó là số byte trong đầu vào của người dùng.

Nếu bạn không gọi `expect`, chương trình sẽ biên dịch nhưng bạn sẽ nhận cảnh báo:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust cảnh báo rằng bạn đã không sử dụng giá trị `Result` trả về từ `read_line`, cho biết chương trình chưa xử lý một lỗi có thể xảy ra.

Cách đúng để dẹp bỏ cảnh báo là thực sự viết mã xử lý lỗi, nhưng trong trường hợp này chúng ta chỉ muốn chương trình sập khi có vấn đề, nên chúng ta có thể dùng `expect`. Bạn sẽ học về cách phục hồi từ lỗi ở [Chương 9][recover]<!-- ignore -->.

### In giá trị với placeholder của `println!`

Ngoài dấu ngoặc nhọn đóng `}`, chỉ còn một dòng nữa để bàn trong mã hiện tại:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Dòng này in chuỗi bây giờ chứa dữ liệu người dùng. `{}` là một placeholder: hãy nghĩ `{}` như hai càng cua nhỏ giữ một giá trị. Khi in giá trị của một biến, tên biến có thể nằm trong dấu ngoặc nhọn. Khi in kết quả của một biểu thức, đặt dấu ngoặc nhọn rỗng trong chuỗi định dạng, sau đó theo sau chuỗi định dạng bằng một danh sách các biểu thức ngăn cách bằng dấu phẩy để in vào mỗi placeholder theo thứ tự. In một biến và kết quả của một biểu thức trong một lần gọi `println!` sẽ trông như sau:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Đoạn mã này sẽ in `x = 5 and y + 2 = 12`.

### Kiểm thử phần đầu tiên

Hãy chạy thử phần đầu của trò chơi đoán số. Chạy nó bằng `cargo run`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

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

Tới lúc này, phần đầu của trò chơi xong: chúng ta đang nhận đầu vào từ bàn phím và in nó ra.

## Tạo một số bí mật

Tiếp theo, chúng ta cần tạo một số bí mật mà người dùng sẽ cố gắng đoán. Số bí mật nên khác nhau mỗi lần để trò chơi thú vị khi chơi nhiều lần. Chúng ta sẽ dùng một số ngẫu nhiên từ 1 đến 100 để trò chơi không quá khó. Rust hiện chưa bao gồm chức năng số ngẫu nhiên trong thư viện chuẩn. Tuy nhiên, nhóm Rust cung cấp một crate [`rand`][randcrate] có chức năng này.

### Sử dụng một crate để có thêm chức năng

Hãy nhớ rằng crate là một tập hợp các tệp mã nguồn Rust. Dự án chúng ta đang xây là một _binary crate_, tức là một thực thi. Crate `rand` là một _library crate_, chứa mã được dùng trong chương trình khác và không thể thực thi một mình.

Sự phối hợp các crate bên ngoài của Cargo là nơi Cargo thực sự nổi bật. Trước khi viết mã dùng `rand`, ta cần chỉnh tệp _Cargo.toml_ để thêm `rand` làm phụ thuộc. Mở tệp này và thêm dòng sau vào cuối, dưới header `[dependencies]` mà Cargo đã tạo sẵn. Hãy chắc chắn ghi `rand` chính xác như ở đây, với số phiên bản này, nếu không các ví dụ mã trong hướng dẫn có thể không hoạt động:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

Trong tệp _Cargo.toml_, mọi thứ theo sau một header đều thuộc phần đó cho tới khi một phần khác bắt đầu. Trong `[dependencies]` bạn khai báo những crate ngoài mà dự án phụ thuộc và các phiên bản bạn yêu cầu. Ở đây, ta chỉ định crate `rand` với specifier phiên bản `0.8.5`. Cargo hiểu [Semantic Versioning][semver]<!-- ignore --> (SemVer), là một tiêu chuẩn cho việc viết số phiên bản. Specifier `0.8.5` thực tế là viết tắt của `^0.8.5`, nghĩa là bất kỳ phiên bản nào ít nhất là 0.8.5 nhưng dưới 0.9.0.

Cargo xem những phiên bản này có API công khai tương thích với 0.8.5, và chỉ định này đảm bảo bạn sẽ nhận được bản vá mới nhất tương thích với mã trong chương này. Bất kỳ phiên bản 0.9.0 hay lớn hơn không đảm bảo cùng API với ví dụ này.

Bây giờ, không thay đổi mã, hãy build dự án, như trong Listing 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="Kết quả khi chạy `cargo build` sau khi thêm crate rand làm phụ thuộc">

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

Bạn có thể thấy các số phiên bản khác nhau (nhưng tất cả đều tương thích với mã, nhờ SemVer!) và các dòng khác nhau (tuỳ hệ điều hành), và các dòng có thể theo thứ tự khác nhau.

Khi ta thêm một phụ thuộc ngoài, Cargo tải các phiên bản mới nhất của mọi crate mà phụ thuộc đó cần từ _registry_, là bản sao dữ liệu từ [Crates.io][cratesio]. Crates.io là nơi cộng đồng Rust đăng các dự án mã nguồn mở để người khác dùng.

Sau khi cập nhật registry, Cargo kiểm tra phần `[dependencies]` và tải xuống bất kỳ crate nào được liệt kê mà chưa có. Trong trường hợp này, mặc dù ta chỉ liệt kê `rand` là phụ thuộc, Cargo cũng lấy các crate khác mà `rand` cần. Sau khi tải về, Rust biên dịch chúng rồi biên dịch dự án với các phụ thuộc sẵn sàng.

Nếu bạn chạy `cargo build` ngay lập tức lần nữa mà không thay đổi gì, bạn sẽ không thấy gì ngoài dòng `Finished`. Cargo biết nó đã tải và biên dịch các phụ thuộc, và bạn không thay đổi gì trong _Cargo.toml_. Cargo cũng biết bạn không thay đổi mã, nên nó không biên dịch lại. Khi không có gì để làm, nó chỉ thoát.

Nếu bạn mở tệp _src/main.rs_, thực hiện một thay đổi nhỏ rồi lưu và build lại, bạn chỉ thấy hai dòng:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Những dòng này cho thấy Cargo chỉ cập nhật build với thay đổi nhỏ của bạn trong tệp _src/main.rs_. Phụ thuộc không thay đổi, nên Cargo biết nó có thể tái sử dụng những gì đã tải và biên dịch trước đó.

#### Đảm bảo build có thể tái tạo với tệp _Cargo.lock_

Cargo có cơ chế đảm bảo bạn có thể xây dựng lại cùng một artifact mọi lúc: Cargo sẽ chỉ dùng các phiên bản phụ thuộc mà bạn đã chỉ định cho đến khi bạn cho biết khác. Ví dụ, giả sử tuần tới `rand` ra phiên bản 0.8.6 chứa bản vá quan trọng nhưng cũng có một lỗi làm hỏng mã của bạn. Để xử lý điều này, Rust tạo tệp _Cargo.lock_ lần đầu khi bạn chạy `cargo build`, vì vậy bây giờ ta có tệp này trong thư mục _guessing_game_.

Khi bạn build dự án lần đầu, Cargo tìm tất cả các phiên bản của các phụ thuộc phù hợp với yêu cầu và ghi chúng vào tệp _Cargo.lock_. Khi bạn build dự án trong tương lai, Cargo sẽ thấy _Cargo.lock_ tồn tại và sẽ dùng các phiên bản được chỉ ra trong đó thay vì tìm lại phiên bản. Điều này cho phép build có thể tái tạo. Nói cách khác, dự án của bạn sẽ ở lại ở 0.8.5 cho đến khi bạn chủ động nâng cấp, nhờ _Cargo.lock_. Vì tệp _Cargo.lock_ quan trọng cho build có thể tái tạo, nó thường được commit vào source control cùng mã nguồn.

#### Cập nhật crate để lấy phiên bản mới

Khi bạn muốn cập nhật một crate, Cargo cung cấp lệnh `update`, sẽ bỏ qua _Cargo.lock_ và tìm các phiên bản mới nhất phù hợp với yêu cầu trong _Cargo.toml_. Cargo sẽ ghi những phiên bản đó vào _Cargo.lock_. Trong trường hợp này, Cargo chỉ tìm phiên bản lớn hơn 0.8.5 và nhỏ hơn 0.9.0. Nếu crate `rand` phát hành hai phiên bản mới 0.8.6 và 0.9.0, bạn sẽ thấy như sau khi chạy `cargo update`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.9.0)
```

Cargo bỏ qua bản phát hành 0.9.0. Lúc này bạn sẽ thấy tệp _Cargo.lock_ thay đổi ghi rằng phiên bản `rand` bạn đang dùng là 0.8.6. Để dùng `rand` phiên bản 0.9.0 hoặc bất kỳ phiên bản 0.9._x_, bạn phải cập nhật _Cargo.toml_ như sau:

```toml
[dependencies]
rand = "0.9.0"
```

Lần sau bạn chạy `cargo build`, Cargo sẽ cập nhật registry crates.io và đánh giá lại yêu cầu `rand` theo phiên bản mới bạn chỉ định.

Còn rất nhiều điều để nói về [Cargo][doccargo]<!-- ignore --> và [hệ sinh thái của nó][doccratesio]<!-- ignore -->, sẽ được bàn ở Chương 14, nhưng hiện tại đó là tất cả những gì bạn cần biết. Cargo làm cho việc tái sử dụng thư viện trở nên dễ dàng, vì vậy Rustacean có thể viết những dự án nhỏ hơn được ghép từ nhiều package.

### Tạo một số ngẫu nhiên

Hãy bắt đầu dùng `rand` để sinh một số cần đoán. Bước tiếp theo là cập nhật _src/main.rs_, như ở Listing 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Thêm mã để sinh một số ngẫu nhiên">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Đầu tiên ta thêm dòng `use rand::Rng;`. Trait `Rng` định nghĩa các phương thức mà các bộ sinh số ngẫu nhiên implement, và trait này phải có trong phạm vi để ta dùng những phương thức đó. Chương 10 sẽ đề cập trait chi tiết.

Tiếp theo, ta thêm hai dòng ở giữa. Trong dòng đầu, ta gọi hàm `rand::thread_rng` để lấy bộ sinh số ngẫu nhiên mà ta sẽ dùng: bộ sinh cục bộ cho luồng (thread) hiện tại và được seed bởi hệ điều hành. Sau đó ta gọi phương thức `gen_range` trên bộ sinh số. Phương thức này được định nghĩa bởi trait `Rng` mà ta đã đưa vào phạm vi với `use rand::Rng;`. `gen_range` nhận một biểu thức phạm vi làm đối số và sinh một số ngẫu nhiên trong phạm vi đó. Kiểu biểu thức phạm vi ta dùng ở đây có dạng `start..=end` và bao gồm cả giá trị đầu và cuối, vì vậy ta cần chỉ `1..=100` để yêu cầu một số từ 1 đến 100.

> Lưu ý: Bạn sẽ không tự động biết trait nào cần dùng và phương thức/hàm nào cần gọi từ một crate, nên mỗi crate có tài liệu hướng dẫn cách dùng. Một tính năng thú vị của Cargo là chạy `cargo doc --open` sẽ xây dựng tài liệu do tất cả phụ thuộc cung cấp cục bộ và mở nó trong trình duyệt. Nếu bạn muốn khám phá chức năng khác trong crate `rand`, ví dụ, chạy `cargo doc --open` và nhấp `rand` ở thanh bên trái.

Dòng mới thứ hai in số bí mật. Điều này hữu ích khi phát triển để kiểm thử, nhưng ta sẽ xoá nó ở phiên bản cuối cùng. Không hay gì nếu chương trình in luôn đáp án ngay khi bắt đầu!

Thử chạy chương trình vài lần:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

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

Bạn nên thấy các số ngẫu nhiên khác nhau, và tất cả chúng nằm giữa 1 và 100. Tuyệt!

## So sánh dự đoán với số bí mật

Bây giờ ta có đầu vào người dùng và một số ngẫu nhiên, ta có thể so sánh chúng. Bước này được thể hiện trong Listing 2-4. Lưu ý rằng mã này chưa biên dịch được; ta sẽ giải thích.

<Listing number="2-4" file-name="src/main.rs" caption="Xử lý các giá trị trả về có thể xảy ra khi so sánh hai số">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Đầu tiên ta thêm một câu lệnh `use` nữa, đưa một kiểu tên là
`std::cmp::Ordering` vào phạm vi từ thư viện chuẩn. Kiểu `Ordering` là một enum khác và có các variant `Less`, `Greater`, và `Equal`. Đây là ba kết quả có thể khi bạn so sánh hai giá trị.

Sau đó ta thêm năm dòng mới ở cuối dùng kiểu `Ordering`. Phương thức `cmp` so sánh hai giá trị và có thể gọi trên bất cứ thứ gì có thể so sánh. Nó nhận một tham chiếu tới thứ bạn muốn so sánh với: ở đây so sánh `guess` với `secret_number`. Sau đó nó trả về một variant của enum `Ordering` mà ta đã đưa vào phạm vi với câu lệnh `use`. Ta dùng một biểu thức [`match`][match]<!-- ignore --> để quyết định hành động tiếp theo dựa trên variant `Ordering` được trả về từ lời gọi `cmp` với `guess` và `secret_number`.

Một biểu thức `match` được tạo từ các _arms_. Một arm gồm một _pattern_ để so khớp, và mã sẽ chạy nếu giá trị đưa vào `match` khớp với pattern đó. Rust lấy giá trị đưa vào `match` và lần lượt so khớp với từng pattern. Pattern và cấu trúc `match` là các tính năng mạnh mẽ của Rust: chúng cho phép bạn diễn tả nhiều tình huống mã có thể gặp và đảm bảo bạn xử lý hết chúng. Các tính năng này sẽ được trình bày trong Chương 6 và Chương 19.

Hãy minh hoạ với ví dụ dùng biểu thức `match` ở đây. Giả sử người dùng đoán 50 và số bí mật lần này là 38.

Khi mã so sánh 50 với 38, phương thức `cmp` sẽ trả về `Ordering::Greater` vì 50 lớn hơn 38. Biểu thức `match` nhận giá trị `Ordering::Greater` và bắt đầu kiểm tra pattern của mỗi arm. Nó xem arm đầu tiên có pattern `Ordering::Less`, và thấy giá trị `Ordering::Greater` không khớp `Ordering::Less`, nên bỏ qua mã trong arm đó và chuyển sang arm tiếp theo. Arm tiếp theo có pattern `Ordering::Greater`, và điều này _khớp_ `Ordering::Greater`! Mã liên quan trong arm đó sẽ thực thi và in `Too big!` ra màn hình. Biểu thức `match` dừng sau khớp đầu tiên thành công, nên nó sẽ không xét arm cuối trong tình huống này.

Tuy nhiên, mã ở Listing 2-4 sẽ chưa biên dịch. Hãy thử nó:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Cốt lõi của lỗi nói rằng có _mismatched types_. Rust có hệ thống kiểu mạnh và tĩnh. Tuy nhiên, nó cũng có suy luận kiểu. Khi ta viết `let mut guess = String::new()`, Rust có thể suy ra `guess` là `String` và không bắt ta viết kiểu. `secret_number`, mặt khác, là một kiểu số. Một vài kiểu số của Rust có thể có giá trị giữa 1 và 100: `i32`, một số 32-bit có dấu; `u32`, một số 32-bit không dấu; `i64`, 64-bit; và những kiểu khác. Nếu không chỉ rõ, Rust mặc định là `i32`, là kiểu của `secret_number` trừ khi bạn thêm thông tin kiểu khác khiến Rust suy ra khác. Lý do lỗi là Rust không thể so sánh một chuỗi và một kiểu số.

Cuối cùng, ta muốn chuyển `String` đọc được thành một kiểu số để có thể so sánh số học với số bí mật. Ta làm điều đó bằng cách thêm dòng này vào thân hàm `main`:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Dòng là:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Ta tạo một biến tên `guess`. Nhưng khoan, chương trình không phải đã có biến `guess` rồi sao? Có, nhưng may mắn là Rust cho phép ta che bóng (shadow) giá trị trước đó của `guess` bằng một giá trị mới. _Shadowing_ cho phép tái sử dụng tên biến `guess` thay vì buộc tạo hai biến khác nhau, ví dụ `guess_str` và `guess`. Chúng ta sẽ bàn kỹ hơn về điều này trong [Chương 3][shadowing]<!-- ignore -->, nhưng hiện tại, biết rằng tính năng này thường dùng khi bạn muốn chuyển một giá trị từ kiểu này sang kiểu khác.

Ta gán biến mới này bằng biểu thức `guess.trim().parse()`. `guess` trong biểu thức này là biến `guess` ban đầu chứa chuỗi nhập. Phương thức `trim` trên một thể hiện `String` sẽ loại bỏ khoảng trắng ở đầu và cuối, điều cần làm trước khi chuyển chuỗi thành `u32`, vì `u32` chỉ chứa dữ liệu số. Người dùng phải nhấn <kbd>enter</kbd> để hoàn tất `read_line`, điều này thêm một ký tự xuống dòng vào chuỗi. Ví dụ, nếu người dùng gõ <kbd>5</kbd> và nhấn <kbd>enter</kbd>, `guess` trông như `5\n`. Ký tự `\n` biểu thị “new line”. (Trên Windows, nhấn <kbd>enter</kbd> tạo ra `\r\n`.) Phương thức `trim` loại bỏ `\n` hoặc `\r\n`, còn lại chỉ `5`.

Phương thức [`parse` trên chuỗi][parse]<!-- ignore --> chuyển chuỗi thành một kiểu khác. Ở đây ta dùng nó để chuyển từ chuỗi sang số. Ta cần cho Rust biết chính xác kiểu số muốn bằng cách viết `let guess: u32`. Dấu hai chấm (`:`) sau `guess` cho Rust biết ta sẽ chú thích kiểu của biến. Rust có một vài kiểu số; `u32` ở đây là số nguyên không dấu 32-bit. Đây là lựa chọn mặc định tốt cho số dương nhỏ. Bạn sẽ học về các kiểu số khác ở [Chương 3][integers]<!-- ignore -->.

Thêm vào đó, chú thích `u32` trong ví dụ này và việc so sánh với `secret_number` khiến Rust suy ra `secret_number` cũng nên là `u32`. Vì vậy giờ đây phép so sánh sẽ là giữa hai giá trị cùng kiểu!

Phương thức `parse` chỉ hoạt động trên các ký tự có thể hợp lý chuyển thành số và do đó có thể gây lỗi. Ví dụ, nếu chuỗi chứa `A👍%`, không thể chuyển sang số. Vì nó có thể thất bại, `parse` trả về một kiểu `Result`, giống như `read_line`. Ta sẽ xử lý `Result` này cùng cách bằng cách gọi `expect`. Nếu `parse` trả về `Err` vì không thể chuyển chuỗi thành số, lời gọi `expect` sẽ làm chương trình sập và in thông điệp ta truyền. Nếu `parse` thành công, nó trả về `Ok` chứa số mà ta cần.

Hãy chạy chương trình bây giờ:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

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

Tốt! Ngay cả khi có khoảng trắng trước dự đoán, chương trình vẫn hiểu là 76. Chạy chương trình vài lần để kiểm tra hành vi khác nhau với các loại đầu vào: đoán đúng, đoán quá cao, và đoán quá thấp.

Chúng ta đã có hầu hết trò chơi hoạt động, nhưng người dùng chỉ được phép đoán một lần. Hãy thay đổi bằng cách thêm một vòng lặp!

## Cho phép nhiều lần đoán bằng vòng lặp

Từ khoá `loop` tạo một vòng lặp vô hạn. Ta sẽ thêm một `loop` để cho người dùng nhiều cơ hội đoán hơn:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Như bạn thấy, ta đã chuyển mọi thứ từ lời nhắc nhập dự đoán trở đi vào trong một vòng lặp. Hãy đảm bảo thụt lề các dòng bên trong vòng lặp thêm bốn khoảng trắng mỗi dòng và chạy chương trình lại. Chương trình giờ sẽ hỏi tiếp dự đoán mãi, điều này thực sự gây ra một vấn đề mới: có vẻ như người dùng không thể thoát!

Người dùng luôn có thể dừng chương trình bằng tổ hợp phím <kbd>ctrl</kbd>-<kbd>c</kbd>. Nhưng còn một cách khác để thoát con quái vật này, như đã nhắc trong phần `parse` ở [“So sánh dự đoán với số bí mật”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->: nếu người dùng nhập một giá trị không phải số, chương trình sẽ sập. Ta có thể tận dụng điều đó để cho phép người dùng thoát, như sau:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Gõ `quit` sẽ thoát trò chơi, nhưng như bạn thấy, nhập bất kỳ chuỗi không phải số nào cũng khiến chương trình thoát. Điều này không lý tưởng; chúng ta muốn chương trình cũng dừng khi người chơi đoán đúng.

### Thoát sau khi đoán đúng

Hãy lập trình để trò chơi thoát khi người dùng thắng bằng cách thêm một câu lệnh `break`:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Thêm dòng `break` sau `You win!` khiến chương trình thoát vòng lặp khi người dùng đoán đúng. Thoát vòng lặp cũng có nghĩa thoát `main`, vì vòng lặp là phần cuối cùng của `main`.

### Xử lý đầu vào không hợp lệ

Để tinh chỉnh hành vi hơn nữa, thay vì để chương trình sập khi người dùng nhập không phải số, ta sẽ làm cho trò chơi bỏ qua đầu vào không hợp lệ để người dùng tiếp tục đoán. Ta có thể làm điều đó bằng cách thay đổi dòng nơi `guess` được chuyển từ `String` sang `u32`, như trong Listing 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Bỏ qua dự đoán không phải số và yêu cầu nhập lại thay vì làm chương trình sập">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Chúng ta chuyển từ gọi `expect` sang biểu thức `match` để từ trạng thái sập khi có lỗi sang xử lý lỗi. Hãy nhớ rằng `parse` trả về một kiểu `Result` và `Result` là một enum có các variant `Ok` và `Err`. Ta đang dùng `match` ở đây, như đã làm với kết quả `Ordering` từ phương thức `cmp`.

Nếu `parse` có thể chuyển chuỗi thành số, nó sẽ trả về một giá trị `Ok` chứa số. Giá trị `Ok` này sẽ khớp với pattern của arm đầu tiên `Ok(num)`, và biểu thức `match` sẽ trả `num`, giá trị mà `parse` tạo ra, và gán cho biến `guess` mới.

Nếu `parse` không thể chuyển chuỗi thành số, nó sẽ trả về `Err` chứa thông tin lỗi. Giá trị `Err` không khớp `Ok(num)` nhưng khớp `Err(_)` của arm thứ hai. Dấu gạch dưới `_` là giá trị bắt mọi; ở ví dụ này, ta nói muốn khớp mọi giá trị `Err` bất kể thông tin bên trong. Vậy chương trình sẽ thực thi mã của arm thứ hai là `continue`, khiến chương trình chuyển sang vòng lặp tiếp theo và yêu cầu dự đoán khác. Hiệu quả là chương trình bỏ qua mọi lỗi `parse`!

Bây giờ mọi thứ trong chương trình nên hoạt động như mong đợi. Hãy thử:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

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

Tuyệt vời! Với một chỉnh sửa nhỏ cuối cùng, chúng ta sẽ hoàn thành trò chơi đoán số. Nhớ rằng chương trình vẫn in số bí mật. Điều đó hữu ích để test, nhưng phá hỏng trò chơi. Hãy xoá `println!` in số bí mật. Listing 2-6 cho thấy mã hoàn chỉnh.

<Listing number="2-6" file-name="src/main.rs" caption="Mã hoàn chỉnh của trò chơi đoán số">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Tới đây, bạn đã xây dựng thành công trò chơi đoán số. Chúc mừng!

## Tóm tắt

Dự án này là cách thực hành nhằm giới thiệu nhiều khái niệm Rust mới: `let`, `match`, hàm, việc dùng crate ngoài, và hơn thế nữa. Trong vài chương tiếp theo, bạn sẽ học chi tiết về những khái niệm này. Chương 3 bàn về các khái niệm mà hầu hết ngôn ngữ lập trình có, như biến, kiểu dữ liệu, và hàm, và cho thấy cách dùng chúng trong Rust. Chương 4 khám phá ownership, một đặc tính làm Rust khác biệt. Chương 5 thảo luận về struct và cú pháp phương thức, và Chương 6 giải thích cách enum hoạt động.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
