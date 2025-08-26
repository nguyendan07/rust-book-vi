## Chào mừng đến với Cargo!

Cargo là hệ thống xây dựng và quản lý gói của Rust. Hầu hết các Rustacean (người dùng Rust) sử dụng công cụ này để quản lý các dự án Rust của họ vì Cargo xử lý rất nhiều tác vụ cho bạn, chẳng hạn như xây dựng mã của bạn, tải xuống các thư viện mà mã của bạn phụ thuộc vào, và xây dựng các thư viện đó. (Chúng ta gọi các thư viện mà mã của bạn cần là _dependencies_ (phụ thuộc).)

Các chương trình Rust đơn giản nhất, như chương trình chúng ta đã viết cho đến nay, không có bất kỳ phụ thuộc nào. Nếu chúng ta đã xây dựng dự án “Hello, world!” với Cargo, nó sẽ chỉ sử dụng phần của Cargo xử lý việc xây dựng mã của bạn. Khi bạn viết các chương trình Rust phức tạp hơn, bạn sẽ thêm các phụ thuộc, và nếu bạn bắt đầu một dự án bằng Cargo, việc thêm các phụ thuộc sẽ dễ dàng hơn nhiều.

Bởi vì phần lớn các dự án Rust đều sử dụng Cargo, phần còn lại của cuốn sách này giả định rằng bạn cũng đang sử dụng Cargo. Cargo được cài đặt cùng với Rust nếu bạn đã sử dụng các trình cài đặt chính thức được thảo luận trong phần [“Cài đặt”][installation]<!-- ignore -->. Nếu bạn đã cài đặt Rust thông qua một số phương tiện khác, hãy kiểm tra xem Cargo đã được cài đặt chưa bằng cách nhập lệnh sau vào terminal của bạn:

```console
$ cargo --version
```

Nếu bạn thấy một số phiên bản, bạn đã cài đặt nó! Nếu bạn thấy lỗi, chẳng hạn như `command not found`, hãy xem tài liệu về phương pháp cài đặt của bạn để xác định cách cài đặt Cargo riêng lẻ.

### Tạo một dự án với Cargo

Hãy tạo một dự án mới bằng Cargo và xem nó khác với dự án “Hello, world!” ban đầu của chúng ta như thế nào. Quay trở lại thư mục _projects_ của bạn (hoặc bất cứ nơi nào bạn quyết định lưu trữ mã của mình). Sau đó, trên bất kỳ hệ điều hành nào, hãy chạy lệnh sau:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

Lệnh đầu tiên tạo một thư mục và dự án mới có tên _hello_cargo_. Chúng ta đã đặt tên dự án của mình là _hello_cargo_, và Cargo tạo các tệp của nó trong một thư mục cùng tên.

Đi vào thư mục _hello_cargo_ và liệt kê các tệp. Bạn sẽ thấy rằng Cargo đã tạo ra hai tệp và một thư mục cho chúng ta: một tệp _Cargo.toml_ và một thư mục _src_ với một tệp _main.rs_ bên trong.

Nó cũng đã khởi tạo một kho chứa Git mới cùng với một tệp _.gitignore_. Các tệp Git sẽ không được tạo nếu bạn chạy `cargo new` trong một kho chứa Git hiện có; bạn có thể ghi đè hành vi này bằng cách sử dụng `cargo new --vcs=git`.

> Lưu ý: Git là một hệ thống quản lý phiên bản phổ biến. Bạn có thể thay đổi `cargo new` để sử dụng một hệ thống quản lý phiên bản khác hoặc không sử dụng hệ thống quản lý phiên bản nào bằng cách sử dụng cờ `--vcs`. Chạy `cargo new --help` để xem các tùy chọn có sẵn.

Mở _Cargo.toml_ trong trình soạn thảo văn bản bạn chọn. Nó sẽ trông tương tự như mã trong Liệt kê 1-2.

<Listing number="1-2" file-name="Cargo.toml" caption="Nội dung của *Cargo.toml* được tạo bởi `cargo new`">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Tệp này ở định dạng [_TOML_][toml]<!-- ignore --> (_Tom’s Obvious, Minimal Language_), là định dạng cấu hình của Cargo.

Dòng đầu tiên, `[package]`, là một tiêu đề mục cho biết rằng các câu lệnh sau đây đang cấu hình một gói. Khi chúng ta thêm nhiều thông tin hơn vào tệp này, chúng ta sẽ thêm các mục khác.

Ba dòng tiếp theo thiết lập thông tin cấu hình mà Cargo cần để biên dịch chương trình của bạn: tên, phiên bản và phiên bản Rust sẽ sử dụng. Chúng ta sẽ nói về khóa `edition` trong [Phụ lục E][appendix-e]<!-- ignore -->.

Dòng cuối cùng, `[dependencies]`, là phần bắt đầu của một mục để bạn liệt kê bất kỳ phụ thuộc nào của dự án. Trong Rust, các gói mã được gọi là _crates_. Chúng ta sẽ không cần bất kỳ crate nào khác cho dự án này, nhưng chúng ta sẽ cần trong dự án đầu tiên ở Chương 2, vì vậy chúng ta sẽ sử dụng mục phụ thuộc này sau.

Bây giờ hãy mở _src/main.rs_ và xem qua:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo đã tạo ra một chương trình “Hello, world!” cho bạn, giống hệt như chương trình chúng ta đã viết trong Liệt kê 1-1! Cho đến nay, sự khác biệt giữa dự án của chúng ta và dự án do Cargo tạo ra là Cargo đã đặt mã vào thư mục _src_ và chúng ta có một tệp cấu hình _Cargo.toml_ trong thư mục gốc.

Cargo mong đợi các tệp nguồn của bạn nằm trong thư mục _src_. Thư mục dự án cấp cao nhất chỉ dành cho các tệp README, thông tin giấy phép, tệp cấu hình và bất cứ thứ gì khác không liên quan đến mã của bạn. Sử dụng Cargo giúp bạn tổ chức các dự án của mình. Có một nơi cho mọi thứ, và mọi thứ đều ở đúng vị trí của nó.

Nếu bạn đã bắt đầu một dự án không sử dụng Cargo, như chúng ta đã làm với dự án “Hello, world!”, bạn có thể chuyển đổi nó thành một dự án sử dụng Cargo. Di chuyển mã dự án vào thư mục _src_ và tạo một tệp _Cargo.toml_ phù hợp. Một cách dễ dàng để có được tệp _Cargo.toml_ đó là chạy `cargo init`, nó sẽ tự động tạo tệp đó cho bạn.

### Xây dựng và Chạy một dự án Cargo

Bây giờ hãy xem có gì khác biệt khi chúng ta xây dựng và chạy chương trình “Hello, world!” với Cargo! Từ thư mục _hello_cargo_ của bạn, hãy xây dựng dự án của bạn bằng cách nhập lệnh sau:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Lệnh này tạo một tệp thực thi trong _target/debug/hello_cargo_ (hoặc _target\debug\hello_cargo.exe_ trên Windows) thay vì trong thư mục hiện tại của bạn. Bởi vì bản dựng mặc định là bản dựng gỡ lỗi (debug build), Cargo đặt tệp nhị phân vào một thư mục có tên là _debug_. Bạn có thể chạy tệp thực thi bằng lệnh này:

```console
$ ./target/debug/hello_cargo # hoặc .\target\debug\hello_cargo.exe trên Windows
Hello, world!
```

Nếu mọi việc suôn sẻ, `Hello, world!` sẽ được in ra terminal. Chạy `cargo build` lần đầu tiên cũng khiến Cargo tạo một tệp mới ở cấp cao nhất: _Cargo.lock_. Tệp này theo dõi các phiên bản chính xác của các phụ thuộc trong dự án của bạn. Dự án này không có phụ thuộc, vì vậy tệp này hơi thưa thớt. Bạn sẽ không bao giờ cần phải thay đổi tệp này theo cách thủ công; Cargo quản lý nội dung của nó cho bạn.

Chúng ta vừa xây dựng một dự án với `cargo build` và chạy nó với `./target/debug/hello_cargo`, nhưng chúng ta cũng có thể sử dụng `cargo run` để biên dịch mã và sau đó chạy tệp thực thi kết quả tất cả trong một lệnh:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Sử dụng `cargo run` tiện lợi hơn là phải nhớ chạy `cargo build` và sau đó sử dụng toàn bộ đường dẫn đến tệp nhị phân, vì vậy hầu hết các nhà phát triển đều sử dụng `cargo run`.

Lưu ý rằng lần này chúng ta không thấy đầu ra cho biết Cargo đang biên dịch `hello_cargo`. Cargo đã nhận ra rằng các tệp không thay đổi, vì vậy nó không xây dựng lại mà chỉ chạy tệp nhị phân. Nếu bạn đã sửa đổi mã nguồn của mình, Cargo sẽ xây dựng lại dự án trước khi chạy nó, và bạn sẽ thấy đầu ra này:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo cũng cung cấp một lệnh gọi là `cargo check`. Lệnh này nhanh chóng kiểm tra mã của bạn để đảm bảo nó biên dịch được nhưng không tạo ra một tệp thực thi:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Tại sao bạn lại không muốn có một tệp thực thi? Thông thường, `cargo check` nhanh hơn nhiều so với `cargo build` vì nó bỏ qua bước tạo ra một tệp thực thi. Nếu bạn liên tục kiểm tra công việc của mình trong khi viết mã, việc sử dụng `cargo check` sẽ tăng tốc quá trình cho bạn biết liệu dự án của bạn có còn biên dịch được hay không! Do đó, nhiều Rustacean chạy `cargo check` định kỳ khi họ viết chương trình để đảm bảo nó biên dịch được. Sau đó, họ chạy `cargo build` khi họ sẵn sàng sử dụng tệp thực thi.

Hãy tóm tắt lại những gì chúng ta đã học được về Cargo cho đến nay:

- Chúng ta có thể tạo một dự án bằng `cargo new`.
- Chúng ta có thể xây dựng một dự án bằng `cargo build`.
- Chúng ta có thể xây dựng và chạy một dự án trong một bước bằng `cargo run`.
- Chúng ta có thể xây dựng một dự án mà không tạo ra tệp nhị phân để kiểm tra lỗi bằng `cargo check`.
- Thay vì lưu kết quả của bản dựng trong cùng thư mục với mã của chúng ta, Cargo lưu trữ nó trong thư mục _target/debug_.

Một lợi thế bổ sung của việc sử dụng Cargo là các lệnh đều giống nhau bất kể bạn đang làm việc trên hệ điều hành nào. Vì vậy, tại thời điểm này, chúng tôi sẽ không còn cung cấp hướng dẫn cụ thể cho Linux và macOS so với Windows nữa.

### Xây dựng cho bản phát hành (Release)

Khi dự án của bạn cuối cùng đã sẵn sàng để phát hành, bạn có thể sử dụng `cargo build --release` để biên dịch nó với các tối ưu hóa. Lệnh này sẽ tạo một tệp thực thi trong _target/release_ thay vì _target/debug_. Các tối ưu hóa làm cho mã Rust của bạn chạy nhanh hơn, nhưng việc bật chúng sẽ kéo dài thời gian để chương trình của bạn biên dịch. Đây là lý do tại sao có hai hồ sơ khác nhau: một cho phát triển, khi bạn muốn xây dựng lại nhanh chóng và thường xuyên, và một hồ sơ khác để xây dựng chương trình cuối cùng bạn sẽ cung cấp cho người dùng mà sẽ không được xây dựng lại nhiều lần và sẽ chạy nhanh nhất có thể. Nếu bạn đang đo lường thời gian chạy của mã, hãy chắc chắn chạy `cargo build --release` và đo lường với tệp thực thi trong _target/release_.

### Cargo như một quy ước

Với các dự án đơn giản, Cargo không cung cấp nhiều giá trị hơn so với việc chỉ sử dụng `rustc`, nhưng nó sẽ chứng tỏ giá trị của mình khi các chương trình của bạn trở nên phức tạp hơn. Một khi các chương trình phát triển thành nhiều tệp hoặc cần một phụ thuộc, việc để Cargo điều phối việc xây dựng sẽ dễ dàng hơn nhiều.

Mặc dù dự án `hello_cargo` đơn giản, nó hiện đang sử dụng phần lớn các công cụ thực tế mà bạn sẽ sử dụng trong phần còn lại của sự nghiệp Rust của mình. Trên thực tế, để làm việc trên bất kỳ dự án hiện có nào, bạn có thể sử dụng các lệnh sau để lấy mã bằng Git, chuyển đến thư mục của dự án đó và xây dựng:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Để biết thêm thông tin về Cargo, hãy xem [tài liệu của nó][cargo].

## Tóm tắt

Bạn đã có một khởi đầu tuyệt vời trên hành trình Rust của mình! Trong chương này, bạn đã học cách:

- Cài đặt phiên bản ổn định mới nhất của Rust bằng `rustup`
- Cập nhật lên phiên bản Rust mới hơn
- Mở tài liệu được cài đặt cục bộ
- Viết và chạy chương trình “Hello, world!” trực tiếp bằng `rustc`
- Tạo và chạy một dự án mới bằng các quy ước của Cargo

Đây là thời điểm tuyệt vời để xây dựng một chương trình thực chất hơn để làm quen với việc đọc và viết mã Rust. Vì vậy, trong Chương 2, chúng ta sẽ xây dựng một chương trình trò chơi đoán số. Nếu bạn muốn bắt đầu bằng cách tìm hiểu cách các khái niệm lập trình phổ biến hoạt động trong Rust, hãy xem Chương 3 và sau đó quay lại Chương 2.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
