# Programming a Guessing Game

Hãy bắt đầu với Rust bằng cách làm một dự án thực hành cùng nhau! Chương này giới thiệu một vài khái niệm phổ biến trong Rust bằng cách cho bạn thấy cách sử dụng chúng trong một chương trình thực tế. Bạn sẽ học về `let`, `match`, methods, associated functions, external crates, và nhiều hơn nữa! Trong các chương tiếp theo, chúng ta sẽ khám phá những ý tưởng này chi tiết hơn. Ở chương này, bạn sẽ chỉ thực hành các kiến thức cơ bản.

Chúng ta sẽ triển khai một bài toán cổ điển cho người mới bắt đầu: trò chơi đoán số. Cách chơi như sau: chương trình sẽ sinh một số nguyên ngẫu nhiên từ 1 đến 100. Sau đó nó sẽ yêu cầu người chơi nhập một dự đoán. Sau khi nhập một dự đoán, chương trình sẽ cho biết dự đoán đó quá thấp hay quá cao. Nếu dự đoán đúng, trò chơi sẽ in thông báo chúc mừng và thoát.

## Setting Up a New Project

Để thiết lập một dự án mới, vào thư mục _projects_ mà bạn đã tạo ở Chương 1 và tạo một dự án mới bằng Cargo, như sau:

```console
$ cargo new guessing_game
$ cd guessing_game
```

Lệnh đầu tiên, `cargo new`, nhận tên dự án (`guessing_game`) làm đối số đầu tiên. Lệnh thứ hai chuyển đến thư mục dự án mới.

Hãy xem file _Cargo.toml_ được sinh ra:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Như bạn đã thấy ở Chương 1, `cargo new` tạo một chương trình “Hello, world!” cho bạn. Hãy xem file _src/main.rs_:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Bây giờ hãy biên dịch chương trình “Hello, world!” này và chạy nó trong cùng một bước bằng lệnh `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Lệnh `run` rất hữu ích khi bạn cần lặp nhanh trên một dự án, như chúng ta sẽ làm trong trò chơi này, kiểm tra từng phiên bản trước khi chuyển sang bước tiếp theo.

Mở lại file _src/main.rs_. Bạn sẽ viết tất cả mã trong file này.

## Processing a Guess

Phần đầu của chương trình đoán số sẽ hỏi đầu vào từ người dùng, xử lý đầu vào đó, và kiểm tra rằng đầu vào có đúng định dạng mong đợi. Để bắt đầu, chúng ta sẽ cho phép người chơi nhập một dự đoán. Nhập mã ở Listing 2-1 vào _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Code that gets a guess from the user and prints it">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Mã này chứa nhiều thông tin, vì vậy hãy cùng đi qua từng dòng. Để lấy đầu vào từ người dùng rồi in kết quả ra, chúng ta cần đưa thư viện `io` input/output vào phạm vi. Thư viện `io` nằm trong standard library, gọi là `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Theo mặc định, Rust có một tập hợp các mục được định nghĩa trong standard library mà nó đưa vào phạm vi của mọi chương trình. Tập hợp này gọi là _prelude_, và bạn có thể xem tất cả trong đó [trong tài liệu standard library][prelude].

Nếu một kiểu bạn muốn dùng không nằm trong prelude, bạn phải đưa kiểu đó vào phạm vi rõ ràng bằng một câu lệnh `use`. Việc sử dụng thư viện `std::io` cung cấp cho bạn nhiều tính năng hữu ích, bao gồm khả năng chấp nhận đầu vào từ người dùng.

Như bạn đã thấy ở Chương 1, hàm `main` là điểm vào chương trình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

Cú pháp `fn` khai báo một hàm mới; các dấu ngoặc đơn, `()`, biểu thị không có tham số; và dấu ngoặc nhọn, `{`, bắt đầu thân hàm.

Như bạn cũng đã học ở Chương 1, `println!` là một macro in một chuỗi ra màn hình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Mã này in một dòng nhắc nói trò chơi là gì và yêu cầu nhập.

### Storing Values with Variables

Tiếp theo, chúng ta sẽ tạo một _biến_ để lưu đầu vào của người dùng, như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Bây giờ chương trình trở nên thú vị! Có rất nhiều thứ diễn ra trong dòng nhỏ này. Chúng ta dùng câu lệnh `let` để tạo biến. Đây là một ví dụ khác:

```rust,ignore
let apples = 5;
```

Dòng này tạo một biến mới tên là `apples` và gán giá trị 5 cho nó. Trong Rust, biến mặc định là immutable, nghĩa là khi đã gán giá trị cho biến, giá trị đó sẽ không thay đổi. Chúng ta sẽ bàn kỹ khái niệm này trong [“Variables and Mutability”][variables-and-mutability]<!-- ignore --> ở Chương 3. Để biến thành mutable, ta thêm `mut` trước tên biến:

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> Note: Cú pháp `//` bắt đầu một chú thích tiếp tục đến cuối dòng. Rust bỏ qua mọi thứ trong chú thích. Chúng ta sẽ thảo luận chi tiết hơn về chú thích trong [Chương 3][comments]<!-- ignore -->.

Quay lại chương trình đoán số, giờ bạn biết `let mut guess` sẽ khai báo một biến mutable tên `guess`. Dấu bằng (`=`) báo cho Rust biết chúng ta muốn gán một cái gì đó cho biến. Ở bên phải dấu bằng là giá trị được gán cho `guess`, là kết quả của việc gọi `String::new`, một hàm trả về một thể hiện mới của `String`. [`String`][string]<!-- ignore --> là một kiểu chuỗi do standard library cung cấp, có khả năng mở rộng, mã hóa UTF-8.

Cú pháp `::` trong `String::new` biểu thị rằng `new` là một associated function của kiểu `String`. Một _associated function_ là một hàm được triển khai trên một kiểu; ở đây `new` tạo một chuỗi rỗng mới. Bạn sẽ thấy hàm `new` trên nhiều kiểu vì đó là tên phổ biến cho hàm tạo giá trị mới.

Toàn bộ dòng `let mut guess = String::new();` đã tạo một biến mutable hiện được gán một thể hiện `String` mới và rỗng. Whew!

### Receiving User Input

Nhớ rằng chúng ta đã đưa chức năng input/output từ standard library vào phạm vi với `use std::io;` ở đầu chương trình. Bây giờ chúng ta sẽ gọi hàm `stdin` từ module `io`, cho phép xử lý đầu vào từ người dùng:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Nếu chúng ta chưa import module `io` với `use std::io;` ở đầu chương trình, chúng ta vẫn có thể dùng hàm này bằng cách viết `std::io::stdin`. Hàm `stdin` trả về một thể hiện của [`std::io::Stdin`][iostdin]<!-- ignore -->, là kiểu đại diện cho một handle tới input chuẩn của terminal.

Tiếp theo, dòng `.read_line(&mut guess)` gọi method `read_line` trên handle input chuẩn để lấy đầu vào từ người dùng. Chúng ta cũng truyền `&mut guess` làm đối số cho `read_line` để cho nó biết chuỗi nào sẽ lưu đầu vào người dùng. Công việc đầy đủ của `read_line` là lấy những gì người dùng gõ vào standard input và nối vào chuỗi (không ghi đè nội dung hiện có), nên chúng ta truyền chuỗi đó như một đối số. Đối số chuỗi phải là mutable để method có thể thay đổi nội dung chuỗi.

Dấu `&` chỉ ra rằng đối số này là một _reference_, cho phép nhiều phần mã truy cập cùng một dữ liệu mà không cần sao chép nhiều lần. References là tính năng phức tạp, và một trong những lợi thế lớn của Rust là cách dùng references an toàn và dễ dàng. Bạn không cần hiểu nhiều chi tiết để hoàn thành chương trình này. Hiện tại, bạn chỉ cần biết references mặc định là immutable, vì vậy bạn phải viết `&mut guess` thay vì `&guess` để làm nó mutable. (Chương 4 sẽ giải thích references kỹ hơn.)

<!-- Old heading. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Handling Potential Failure with `Result`

Chúng ta vẫn đang thảo luận dòng mã này. Chúng ta đang nói về phần thứ ba của dòng lệnh hợp lệ về mặt logic. Phần tiếp theo là method này:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Chúng ta có thể viết dòng này như sau:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Tuy nhiên, một dòng dài khó đọc, nên tốt hơn là chia nhỏ. Thường thì nên chèn dấu xuống dòng và khoảng trắng khác để giúp tách các dòng dài khi gọi method theo cú pháp `.method_name()`. Bây giờ hãy bàn về dòng này.

Như đã đề cập, `read_line` đặt những gì người dùng nhập vào chuỗi mà chúng ta truyền, nhưng nó cũng trả về một giá trị `Result`. [`Result`][result]<!-- ignore --> là một [_enumeration_][enums]<!-- ignore -->, thường gọi là _enum_, là một kiểu có thể ở một trong nhiều trạng thái. Chúng ta gọi mỗi trạng thái có thể là một _variant_.

[Chương 6][enums]<!-- ignore --> sẽ trình bày enum chi tiết hơn. Mục đích của các kiểu `Result` là để mã hóa thông tin xử lý lỗi.

Các biến thể của `Result` là `Ok` và `Err`. Variant `Ok` chỉ ra rằng thao tác thành công, và nó chứa giá trị tạo ra. Variant `Err` nghĩa là thao tác thất bại và chứa thông tin về lý do thất bại.

Các giá trị kiểu `Result`, giống như mọi kiểu khác, có các phương thức được định nghĩa trên chúng. Một thể hiện của `Result` có method [`expect`][expect]<!-- ignore --> mà bạn có thể gọi. Nếu thể hiện này là `Err`, `expect` sẽ khiến chương trình dừng và in thông báo mà bạn truyền làm đối số. Nếu `read_line` trả về `Err`, nó có thể là lỗi từ hệ điều hành. Nếu thể hiện `Result` là `Ok`, `expect` sẽ lấy giá trị trong `Ok` và trả về giá trị đó để bạn sử dụng. Trong trường hợp này, giá trị đó là số byte trong đầu vào của người dùng.

Nếu bạn không gọi `expect`, chương trình vẫn biên dịch nhưng bạn sẽ nhận được một cảnh báo:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust cảnh báo rằng bạn chưa sử dụng giá trị `Result` trả về từ `read_line`, ám chỉ rằng chương trình chưa xử lý lỗi có thể xảy ra.

Cách đúng để loại bỏ cảnh báo là thực sự viết mã xử lý lỗi, nhưng trong trường hợp này chúng ta chỉ muốn chương trình dừng khi xảy ra vấn đề, nên có thể dùng `expect`. Bạn sẽ học cách phục hồi từ lỗi trong [Chương 9][recover]<!-- ignore -->.

### Printing Values with `println!` Placeholders

Ngoài dấu ngoặc nhọn đóng, chỉ còn một dòng nữa để bàn trong mã tới lúc này:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Dòng này in chuỗi giờ chứa đầu vào của người dùng. `{}` là một placeholder: hãy nghĩ `{}` như đôi càng nhỏ giữ một giá trị. Khi in giá trị của một biến, tên biến có thể đặt bên trong cặp ngoặc nhọn. Khi in kết quả của một biểu thức, đặt cặp ngoặc nhọn rỗng trong chuỗi định dạng, rồi theo sau chuỗi định dạng bằng danh sách các biểu thức phân tách bằng dấu phẩy sẽ in vào từng placeholder theo thứ tự tương ứng. Việc in một biến và kết quả của một biểu thức trong một lần gọi `println!` trông như sau:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Mã này sẽ in `x = 5 and y + 2 = 12`.

### Testing the First Part

Hãy thử phần đầu tiên của trò chơi. Chạy bằng `cargo run`:

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

Tại thời điểm này, phần đầu của trò chơi đã xong: chúng ta đang nhận dữ liệu từ bàn phím rồi in ra.

## Generating a Secret Number

Tiếp theo, chúng ta cần sinh một số bí mật mà người dùng sẽ dự đoán. Số bí mật nên khác nhau mỗi lần để trò chơi thú vị khi chơi nhiều lần. Chúng ta sẽ dùng một số ngẫu nhiên từ 1 đến 100 để trò chơi không quá khó. Rust chưa có sẵn chức năng số ngẫu nhiên trong standard library. Tuy nhiên, đội ngũ Rust cung cấp crate [`rand`][randcrate] với chức năng đó.

### Using a Crate to Get More Functionality

Hãy nhớ rằng crate là một tập hợp các file mã nguồn Rust. Dự án chúng ta đang xây là một _binary crate_, tức là một executable. Crate `rand` là một _library crate_, chứa mã được dùng trong chương trình khác và không thể chạy độc lập.

Sự quản lý các crate ngoài rất hiệu quả của Cargo là điểm mạnh của nó. Trước khi viết mã dùng `rand`, chúng ta cần sửa file _Cargo.toml_ để thêm crate `rand` làm dependency. Mở file đó và thêm dòng sau vào cuối, dưới header `[dependencies]` mà Cargo đã tạo cho bạn. Hãy chắc chắn ghi `rand` chính xác như ở đây, với số phiên bản này, nếu không các ví dụ trong tutorial có thể không hoạt động:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

Trong file _Cargo.toml_, mọi thứ theo sau một header là phần thuộc về section đó cho tới khi một section khác bắt đầu. Trong `[dependencies]` bạn nói Cargo biết dự án phụ thuộc những crate ngoài nào và phiên bản nào của chúng. Ở đây, chúng ta chỉ định crate `rand` với semantic version `0.8.5`. Cargo hiểu [Semantic Versioning][semver]<!-- ignore --> (SemVer), một chuẩn cho việc viết số phiên bản. Specifier `0.8.5` thực ra là viết tắt của `^0.8.5`, nghĩa là bất kỳ phiên bản nào ít nhất là 0.8.5 nhưng nhỏ hơn 0.9.0.

Cargo coi những phiên bản này có API công khai tương thích với 0.8.5, và chỉ định này đảm bảo bạn sẽ lấy bản vá mới nhất mà vẫn biên dịch được với mã trong chương này. Mọi phiên bản 0.9.0 trở lên không đảm bảo có cùng API với ví dụ sau đây.

Bây giờ, không thay đổi mã, hãy build dự án, như trong Listing 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="The output from running `cargo build` after adding the rand crate as a dependency">

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

Bạn có thể thấy các số phiên bản khác nhau (nhưng tất cả sẽ tương thích với mã, nhờ SemVer!) và các dòng khác nhau (tùy hệ điều hành), và thứ tự các dòng có thể khác.

Khi chúng ta thêm một dependency ngoài, Cargo tải các phiên bản mới nhất của mọi thứ dependency đó cần từ _registry_, tức là bản sao dữ liệu từ [Crates.io][cratesio]. Crates.io là nơi mọi người đăng dự án mã nguồn mở Rust để người khác dùng.

Sau khi cập nhật registry, Cargo kiểm tra section `[dependencies]` và tải về bất kỳ crate nào được liệt kê mà chưa có. Trong trường hợp này, mặc dù chỉ liệt kê `rand`, Cargo cũng tải về các crate khác mà `rand` phụ thuộc. Sau khi tải về, Rust biên dịch chúng rồi biên dịch dự án với các dependency có sẵn.

Nếu bạn chạy `cargo build` ngay lập tức lần nữa mà không thay đổi gì, bạn sẽ không thấy gì ngoài dòng `Finished`. Cargo biết nó đã tải và biên dịch dependencies, và bạn không thay đổi _Cargo.toml_. Cargo cũng biết bạn chưa thay đổi mã, nên không biên dịch lại. Không có gì để làm, nó thoát.

Nếu bạn mở _src/main.rs_, thay đổi một chút, lưu và build lại, bạn chỉ sẽ thấy hai dòng:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Những dòng này cho thấy Cargo chỉ cập nhật build với thay đổi nhỏ trên _src/main.rs_. Dependencies của bạn không thay đổi, nên Cargo biết có thể tái sử dụng những gì đã tải và biên dịch.

#### Ensuring Reproducible Builds with the _Cargo.lock_ File

Cargo có cơ chế đảm bảo bạn có thể rebuild cùng một artifact mỗi lần: Cargo sẽ chỉ dùng các phiên bản dependencies bạn chỉ định cho tới khi bạn nói khác. Ví dụ, giả sử tuần tới phiên bản 0.8.6 của crate `rand` ra mắt, và phiên bản đó có bản sửa lỗi quan trọng nhưng cũng mang theo một regression làm hỏng mã của bạn. Để xử lý, Rust tạo file _Cargo.lock_ lần đầu bạn chạy `cargo build`, vì vậy giờ chúng ta có file này trong thư mục _guessing_game_.

Khi build dự án lần đầu, Cargo tìm tất cả các phiên bản dependencies phù hợp và ghi chúng vào _Cargo.lock_. Khi build trong tương lai, Cargo sẽ thấy file _Cargo.lock_ tồn tại và dùng các phiên bản được liệt kê đó thay vì suy luận lại. Điều này cho phép bạn có một build có thể tái tạo. Nói cách khác, dự án của bạn sẽ vẫn dùng 0.8.5 trừ khi bạn nâng cấp rõ ràng, nhờ _Cargo.lock_. Vì _Cargo.lock_ quan trọng cho build có thể tái tạo, nó thường được commit vào source control cùng mã dự án.

#### Updating a Crate to Get a New Version

Khi bạn muốn cập nhật crate, Cargo cung cấp lệnh `update`, lệnh này sẽ bỏ qua _Cargo.lock_ và tìm các phiên bản mới nhất phù hợp với chỉ định trong _Cargo.toml_. Cargo rồi sẽ ghi các phiên bản đó vào _Cargo.lock_. Trong trường hợp này, Cargo chỉ tìm các phiên bản > 0.8.5 và < 0.9.0. Nếu crate `rand` phát hành hai phiên bản mới 0.8.6 và 0.9.0, bạn sẽ thấy như sau nếu chạy `cargo update`:

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

Cargo bỏ qua bản 0.9.0. Lúc này bạn cũng sẽ thấy thay đổi trong _Cargo.lock_ ghi rằng phiên bản `rand` đang dùng là 0.8.6. Để dùng `rand` phiên bản 0.9.0 hoặc bất kỳ phiên bản nào trong series 0.9._x_, bạn phải cập nhật _Cargo.toml_ như sau:

```toml
[dependencies]
rand = "0.9.0"
```

Lần tiếp theo bạn chạy `cargo build`, Cargo sẽ cập nhật registry và đánh giá lại yêu cầu `rand` theo phiên bản mới bạn chỉ định.

Còn nhiều điều để nói về [Cargo][doccargo]<!-- ignore --> và [hệ sinh thái của nó][doccratesio]<!-- ignore -->, mà chúng ta sẽ thảo luận ở Chương 14, nhưng hiện tại chỉ cần biết đến đây. Cargo làm cho việc tái sử dụng thư viện trở nên dễ dàng, nên cộng đồng Rust viết các dự án nhỏ lắp ghép từ nhiều package.

### Generating a Random Number

Hãy bắt đầu dùng `rand` để sinh số cần đoán. Bước tiếp theo là cập nhật _src/main.rs_, như trong Listing 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Adding code to generate a random number">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Đầu tiên chúng ta thêm dòng `use rand::Rng;`. Trait `Rng` định nghĩa các method mà các random number generators triển khai, và trait này phải nằm trong phạm vi để ta có thể dùng các method đó. Chương 10 sẽ trình bày trait chi tiết.

Tiếp theo, chúng ta thêm hai dòng ở giữa. Ở dòng đầu, ta gọi hàm `rand::thread_rng` để lấy generator ngẫu nhiên mà ta sẽ dùng: một generator cục bộ cho luồng hiện tại và được seed bởi hệ điều hành. Sau đó ta gọi method `gen_range` trên generator ngẫu nhiên. Method này do trait `Rng` định nghĩa mà chúng ta đã đưa vào phạm vi bằng `use rand::Rng;`. `gen_range` nhận một biểu thức range làm đối số và sinh một số ngẫu nhiên trong range đó. Dạng range chúng ta dùng ở đây có dạng `start..=end` và bao gồm cả hai đầu, nên ta phải chỉ `1..=100` để lấy số từ 1 đến 100.

> Note: Bạn sẽ không phải lúc nào cũng biết trait nào dùng và method hay hàm nào gọi từ một crate, nên mỗi crate có tài liệu chỉ dẫn cách dùng. Một tính năng hay của Cargo là chạy `cargo doc --open` sẽ xây dựng tài liệu của mọi dependency cục bộ và mở nó trong trình duyệt. Nếu bạn quan tâm các chức năng khác của crate `rand`, ví dụ, chạy `cargo doc --open` và nhấp `rand` trong sidebar bên trái.

Dòng mới thứ hai in số bí mật. Điều này hữu ích trong khi phát triển để có thể test, nhưng chúng ta sẽ xóa nó trong phiên bản cuối. Trò chơi không còn vui nếu chương trình in đáp án ngay khi bắt đầu!

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

Bạn sẽ nhận được các số ngẫu nhiên khác nhau, và tất cả đều trong khoảng từ 1 đến 100. Tuyệt!

## Comparing the Guess to the Secret Number

Bây giờ chúng ta có đầu vào người dùng và một số ngẫu nhiên, ta có thể so sánh chúng. Bước này được thể hiện ở Listing 2-4. Lưu ý mã này hiện chưa biên dịch được, như ta sẽ giải thích.

<Listing number="2-4" file-name="src/main.rs" caption="Handling the possible return values of comparing two numbers">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Đầu tiên chúng ta thêm một câu lệnh `use` khác, đưa một kiểu tên `std::cmp::Ordering` vào phạm vi từ standard library. Kiểu `Ordering` là một enum khác và có các variant `Less`, `Greater`, và `Equal`. Đây là ba kết quả có thể xảy ra khi so sánh hai giá trị.

Rồi chúng ta thêm năm dòng mới ở dưới dùng kiểu `Ordering`. Method `cmp` so sánh hai giá trị và có thể gọi trên bất cứ thứ gì có thể so sánh. Nó nhận một reference tới giá trị muốn so sánh với: ở đây là so sánh `guess` với `secret_number`. Sau đó nó trả về một variant của enum `Ordering` mà ta đã đưa vào phạm vi bằng `use`. Ta dùng một biểu thức [`match`][match]<!-- ignore --> để quyết định làm gì tiếp theo dựa trên variant `Ordering` trả về từ `cmp` khi so sánh `guess` và `secret_number`.

Một biểu thức `match` gồm các _arms_. Một arm bao gồm một _pattern_ để so khớp, và mã sẽ chạy nếu giá trị đưa cho `match` phù hợp với pattern đó. Rust lấy giá trị đưa cho `match` và so sánh theo thứ tự từng arm. Patterns và cấu trúc `match` là các tính năng mạnh mẽ của Rust: chúng cho phép bạn diễn tả nhiều tình huống và đảm bảo bạn xử lý đầy đủ. Những tính năng này sẽ được trình bày trong Chương 6 và Chương 19.

Hãy đi qua một ví dụ với biểu thức `match` ta dùng ở đây. Giả sử người dùng đoán 50 và số bí mật ngẫu nhiên lần này là 38.

Khi mã so sánh 50 với 38, method `cmp` sẽ trả về `Ordering::Greater` vì 50 lớn hơn 38. Biểu thức `match` nhận giá trị `Ordering::Greater` và bắt đầu kiểm tra pattern của từng arm. Nó nhìn arm đầu tiên với pattern `Ordering::Less` và thấy `Ordering::Greater` không khớp `Ordering::Less`, nên bỏ qua arm đó và chuyển sang arm kế. Pattern arm kế là `Ordering::Greater`, và nó khớp với `Ordering::Greater`! Mã trong arm đó sẽ thực thi và in `Too big!` ra màn hình. Biểu thức `match` kết thúc sau khi tìm khớp đầu tiên, nên sẽ không xem arm cuối cùng trong kịch bản này.

Tuy nhiên, mã ở Listing 2-4 vẫn chưa biên dịch. Hãy thử:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Lỗi chính là _mismatched types_. Rust có hệ thống kiểu mạnh và tĩnh. Tuy nhiên, nó cũng có suy luận kiểu. Khi viết `let mut guess = String::new()`, Rust suy luận `guess` là `String` và không bắt phải ghi kiểu. `secret_number`, ngược lại, là một kiểu số. Một vài kiểu số trong Rust có thể có giá trị giữa 1 và 100: `i32`, một số 32-bit; `u32`, một số 32-bit không dấu; `i64`, 64-bit; cũng như các kiểu khác. Nếu không chỉ định, Rust mặc định là `i32`, đó là kiểu của `secret_number` nếu không có thông tin kiểu khác. Lý do lỗi là Rust không thể so sánh một `String` với một kiểu số.

Cuối cùng, chúng ta muốn chuyển `String` mà chương trình đọc được sang kiểu số để có thể so sánh số học với số bí mật. Ta làm điều đó bằng cách thêm dòng này vào thân hàm `main`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Dòng là:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Chúng ta tạo biến tên `guess`. Nhưng khoan, chương trình đã có một biến tên `guess` rồi? Đúng, nhưng may mắn là Rust cho phép ta shadow giá trị trước đó của `guess` bằng một giá trị mới. _Shadowing_ cho phép tái sử dụng tên biến `guess` thay vì bắt phải tạo hai biến khác nhau, ví dụ `guess_str` và `guess`. Chúng ta sẽ trình bày chi tiết hơn ở [Chương 3][shadowing]<!-- ignore -->, nhưng hiện tại, biết rằng tính năng này thường dùng khi bạn muốn chuyển một giá trị từ kiểu này sang kiểu khác.

Chúng ta gán biến mới này bằng biểu thức `guess.trim().parse()`. `guess` trong biểu thức tham chiếu đến biến `guess` ban đầu chứa đầu vào dạng chuỗi. Method `trim` trên `String` loại bỏ khoảng trắng ở đầu và cuối, điều cần thiết trước khi chuyển chuỗi sang `u32`, vì `u32` chỉ chứa ký tự số. Người dùng phải nhấn <kbd>enter</kbd> để `read_line` nhận dự đoán, điều này thêm ký tự newline vào chuỗi. Ví dụ, nếu người dùng gõ <kbd>5</kbd> và nhấn <kbd>enter</kbd>, `guess` trông như `5\n`. `\n` biểu diễn “newline.” (Trên Windows, nhấn <kbd>enter</kbd> tạo carriage return và newline, `\r\n`.) `trim` loại bỏ `\n` hoặc `\r\n`, chỉ còn `5`.

Method [`parse` trên chuỗi][parse]<!-- ignore --> chuyển chuỗi sang kiểu khác. Ở đây, ta dùng nó để chuyển từ chuỗi sang số. Ta cần cho Rust biết chính xác kiểu số mong muốn bằng cách ghi `let guess: u32`. Dấu hai chấm (`:`) sau `guess` báo Rust rằng chúng ta chú thích kiểu cho biến. Rust có vài kiểu số; `u32` ở đây là một số nguyên 32-bit không dấu. Đây là lựa chọn hợp lý cho một số nguyên dương nhỏ. Bạn sẽ học về các kiểu số khác trong [Chương 3][integers]<!-- ignore -->.

Thêm nữa, chú thích `u32` trong ví dụ này và việc so sánh với `secret_number` khiến Rust suy luận `secret_number` cũng nên là `u32`. Vì vậy bây giờ so sánh sẽ là giữa hai giá trị cùng kiểu!

Method `parse` chỉ hoạt động trên các ký tự có thể hợp lý chuyển thành số và vì vậy có thể gây lỗi. Ví dụ, nếu chuỗi chứa `A👍%`, sẽ không thể chuyển thành số. Vì có thể thất bại, `parse` trả về một kiểu `Result`, tương tự như `read_line`. Ta sẽ xử lý `Result` này bằng `expect` như trước. Nếu `parse` trả về `Err` vì không thể tạo số, `expect` sẽ làm chương trình dừng và in thông báo. Nếu `parse` chuyển chuỗi thành số thành công, nó sẽ trả về `Ok` và `expect` sẽ trả giá trị số mà ta cần.

Hãy chạy chương trình giờ:

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

Tuyệt! Dù có khoảng trắng trước dự đoán, chương trình vẫn nhận ra người dùng đoán 76. Chạy vài lần để kiểm tra hành vi với các kiểu đầu vào: đoán đúng, đoán lớn hơn, đoán nhỏ hơn.

Chúng ta đã có hầu hết trò chơi hoạt động, nhưng người dùng chỉ có một lần đoán. Hãy thay đổi bằng cách thêm vòng lặp!

## Allowing Multiple Guesses with Looping

Từ khóa `loop` tạo một vòng lặp vô hạn. Chúng ta sẽ thêm một loop để cho người dùng thêm cơ hội đoán:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Như bạn thấy, chúng ta đã di chuyển mọi thứ từ prompt nhập dự đoán trở đi vào trong một loop. Hãy chắc chắn thụt lề các dòng trong loop thêm bốn khoảng trắng và chạy chương trình lại. Chương trình bây giờ sẽ hỏi dự đoán liên tục, điều này giới thiệu một vấn đề mới: có vẻ như người dùng không thể thoát!

Người dùng luôn có thể ngắt chương trình bằng phím tắt <kbd>ctrl</kbd>-<kbd>c</kbd>. Nhưng có một cách khác để thoát, như đã đề cập ở phần `parse` trong [“Comparing the Guess to the Secret Number”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->: nếu người dùng nhập một giá trị không phải số, chương trình sẽ bị crash. Ta có thể tận dụng điều đó để cho phép người dùng thoát, như sau:

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

Gõ `quit` sẽ thoát trò chơi, nhưng như bạn thấy, nhập bất kỳ chuỗi không phải số nào cũng làm thoát. Điều này không tối ưu; ta muốn chương trình cũng dừng khi đoán đúng.

### Quitting After a Correct Guess

Hãy lập trình để trò chơi thoát khi người dùng thắng bằng cách thêm câu lệnh `break`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Thêm dòng `break` sau `You win!` làm chương trình thoát vòng lặp khi người dùng đoán chính xác. Thoát vòng lặp cũng có nghĩa là thoát chương trình, vì loop là phần cuối của `main`.

### Handling Invalid Input

Để tinh chỉnh hành vi hơn nữa, thay vì làm chương trình crash khi người dùng nhập không phải số, ta hãy làm trò chơi bỏ qua đầu vào không hợp lệ để người dùng tiếp tục đoán. Ta có thể làm điều đó bằng cách thay đổi dòng nơi `guess` được chuyển từ `String` sang `u32`, như trong Listing 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Ignoring a non-number guess and asking for another guess instead of crashing the program">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Chúng ta chuyển từ gọi `expect` sang một biểu thức `match` để từ việc crash khi lỗi sang xử lý lỗi. Nhớ rằng `parse` trả về `Result` và `Result` là enum có các variant `Ok` và `Err`. Ở đây ta dùng `match`, như đã sử dụng với kết quả `Ordering` của method `cmp`.

Nếu `parse` có thể chuyển chuỗi thành số, nó sẽ trả về `Ok` chứa số đó. Giá trị `Ok` này sẽ khớp với pattern `Ok(num)` ở arm đầu, và biểu thức `match` sẽ chỉ trả về giá trị `num` mà `parse` tạo ra và gán vào biến `guess` mới.

Nếu `parse` không thể chuyển chuỗi thành số, nó sẽ trả `Err` chứa thông tin lỗi. Giá trị `Err` sẽ không khớp `Ok(num)` ở arm đầu, nhưng nó sẽ khớp `Err(_)` ở arm hai. Dấu gạch dưới `_` là một catch-all; ở đây ta nói rằng muốn khớp mọi `Err`, bất kể thông tin bên trong. Vì vậy chương trình sẽ thực thi mã ở arm hai, `continue`, nghĩa là đi sang vòng lặp tiếp theo và yêu cầu một dự đoán khác. Vậy là chương trình bỏ qua mọi lỗi mà `parse` có thể gặp!

Giờ mọi thứ trong chương trình nên hoạt động như mong đợi. Hãy thử:

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

Tuyệt vời! Với một tinh chỉnh nhỏ cuối cùng, chúng ta sẽ hoàn thành trò chơi đoán số. Nhớ rằng chương trình vẫn in số bí mật. Điều đó hữu ích để test, nhưng làm hỏng trò chơi. Hãy xóa `println!` in số bí mật. Listing 2-6 cho thấy mã hoàn chỉnh.

<Listing number="2-6" file-name="src/main.rs" caption="Complete guessing game code">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Tại thời điểm này, bạn đã xây dựng thành công trò chơi đoán số. Xin chúc mừng!

## Summary

Dự án này là cách thực hành giới thiệu nhiều khái niệm Rust mới: `let`, `match`, functions, việc dùng crate ngoài, và nhiều hơn nữa. Trong vài chương kế tiếp, bạn sẽ học các khái niệm này chi tiết hơn. Chương 3 trình bày các khái niệm mà hầu hết ngôn ngữ lập trình có, như biến, kiểu dữ liệu, và hàm, và cho thấy cách dùng chúng trong Rust. Chương 4 khám phá ownership, một tính năng làm Rust khác biệt. Chương 5 thảo luận structs và cú pháp method, và Chương 6 giải thích cách enums hoạt động.

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
