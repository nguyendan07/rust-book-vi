## Hello, Cargo!

Cargo là hệ thống build và công cụ quản lý gói (package manager) chính thức của Rust. Hầu hết các Rustacean đều sử dụng công cụ này để quản lý các dự án Rust của họ vì Cargo xử lý rất nhiều tác vụ thay bạn, chẳng hạn như biên dịch mã nguồn, tải xuống các thư viện mà mã của bạn phụ thuộc vào, và biên dịch các thư viện đó. (Chúng tôi gọi các thư viện mà mã nguồn của bạn cần là các _dependencies_ - gói phụ thuộc).

Các chương trình Rust đơn giản nhất, giống như chương trình chúng ta đã viết cho đến nay, không có bất kỳ gói phụ thuộc nào. Nếu chúng ta xây dựng dự án “Hello, world!” bằng Cargo, nó sẽ chỉ sử dụng phần tính năng của Cargo để xử lý việc biên dịch mã. Khi bạn viết các chương trình Rust phức tạp hơn, bạn sẽ cần thêm các gói phụ thuộc, và nếu bạn bắt đầu dự án bằng Cargo, việc thêm phụ thuộc sẽ dễ dàng hơn rất nhiều.

Vì đại đa số các dự án Rust đều sử dụng Cargo, phần còn lại của cuốn sách này giả định rằng bạn cũng đang sử dụng Cargo. Cargo được cài đặt sẵn cùng với Rust nếu bạn sử dụng các trình cài đặt chính thức đã thảo luận trong phần [“Cài đặt”][installation]<!-- ignore -->. Nếu bạn đã cài đặt Rust bằng cách khác, hãy kiểm tra xem Cargo đã được cài đặt chưa bằng cách nhập lệnh sau vào terminal:

```console
$ cargo --version
```

Nếu bạn thấy số phiên bản hiển thị ra, nghĩa là bạn đã có Cargo! Nếu bạn thấy lỗi như `command not found`, hãy tham khảo tài liệu về phương pháp cài đặt của bạn để biết cách cài đặt Cargo riêng biệt.

### Tạo Dự Án Bằng Cargo

Hãy tạo một dự án mới bằng Cargo và xem nó khác với dự án “Hello, world!” ban đầu của chúng ta như thế nào. Điều hướng trở lại thư mục _projects_ của bạn (hoặc bất kỳ nơi nào bạn chọn để lưu mã nguồn). Sau đó, trên bất kỳ hệ điều hành nào, hãy chạy lệnh:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

Lệnh đầu tiên tạo một thư mục và dự án mới có tên là _hello_cargo_. Chúng ta đã đặt tên cho dự án là _hello_cargo_, và Cargo tạo các tệp của nó trong một thư mục cùng tên.

Đi vào thư mục _hello_cargo_ và liệt kê các tệp. Bạn sẽ thấy rằng Cargo đã tự động tạo cho chúng ta hai tệp và một thư mục: tệp _Cargo.toml_ và thư mục _src_ chứa tệp _main.rs_ bên trong.

Nó cũng đã khởi tạo sẵn một kho lưu trữ Git mới cùng với một tệp _.gitignore_. Các tệp Git sẽ không được tạo nếu bạn chạy `cargo new` bên trong một kho lưu trữ Git đã tồn tại; bạn có thể ghi đè hành vi này bằng cách sử dụng `cargo new --vcs=git`.

> Lưu ý: Git là một hệ thống quản lý phiên bản phổ biến. Bạn có thể yêu cầu `cargo new` sử dụng một hệ thống quản lý phiên bản khác hoặc không sử dụng hệ thống nào bằng cách dùng cờ `--vcs`. Chạy `cargo new --help` để xem các tùy chọn khả dụng.

Mở _Cargo.toml_ trong trình soạn thảo văn bản bạn chọn. Nó sẽ trông tương tự như đoạn mã trong Danh sách 1-2.

<Listing number="1-2" file-name="Cargo.toml" caption="Nội dung của *Cargo.toml* được tạo bởi `cargo new`">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Tệp này có định dạng [_TOML_][toml]<!-- ignore --> (_Tom’s Obvious, Minimal Language_), đây là định dạng cấu hình chuẩn của Cargo.

Dòng đầu tiên, `[package]`, là tiêu đề phần cho biết các câu lệnh tiếp theo đang cấu hình một gói (package). Khi chúng ta thêm thông tin vào tệp này, chúng ta sẽ thêm các phần khác.

Ba dòng tiếp theo thiết lập thông tin cấu hình mà Cargo cần để biên dịch chương trình của bạn: tên gói, phiên bản, và ấn bản (edition) Rust được sử dụng. Chúng ta sẽ nói về khóa `edition` trong [Phụ lục E][appendix-e]<!-- ignore -->.

Dòng cuối cùng, `[dependencies]`, là phần mở đầu để bạn liệt kê bất kỳ gói phụ thuộc nào của dự án. Trong Rust, các gói mã nguồn được gọi là các _crates_. Chúng ta sẽ chưa cần bất kỳ crate nào khác cho dự án này, nhưng chúng ta sẽ cần trong dự án đầu tiên ở Chương 2, vì vậy chúng ta sẽ sử dụng phần dependencies đó sau.

Bây giờ hãy mở _src/main.rs_ và quan sát:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo đã tự động tạo sẵn một chương trình “Hello, world!” cho bạn, giống hệt như chương trình chúng ta đã viết trong Danh sách 1-1! Cho đến nay, điểm khác biệt giữa dự án thủ công của chúng ta và dự án do Cargo tạo ra là Cargo đã đặt mã nguồn trong thư mục _src_ và chúng ta có một tệp cấu hình _Cargo.toml_ ở thư mục gốc.

Cargo quy định rằng các tệp mã nguồn của bạn phải nằm bên trong thư mục _src_. Thư mục cấp cao nhất của dự án chỉ dành cho các tệp README, thông tin giấy phép (license), tệp cấu hình và bất kỳ thứ gì khác không phải mã nguồn chương trình. Sử dụng Cargo giúp bạn tổ chức các dự án của mình một cách ngăn nắp và khoa học. Mọi thứ đều có vị trí rõ ràng của nó.

Nếu bạn bắt đầu một dự án mà không sử dụng Cargo như chúng ta đã làm với dự án “Hello, world!”, bạn có thể dễ dàng chuyển đổi nó thành một dự án sử dụng Cargo. Chỉ cần di chuyển mã nguồn của dự án vào thư mục _src_ và tạo một tệp _Cargo.toml_ phù hợp. Một cách đơn giản để tạo tệp _Cargo.toml_ đó là chạy lệnh `cargo init`, lệnh này sẽ tự động khởi tạo cho bạn.

### Biên Dịch và Chạy Dự Án Cargo

Bây giờ hãy xem điều gì khác biệt khi chúng ta xây dựng và chạy chương trình “Hello, world!” với Cargo! Từ thư mục _hello_cargo_, hãy biên dịch dự án của bạn bằng cách nhập lệnh sau:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Lệnh này tạo một tệp thực thi trong thư mục _target/debug/hello_cargo_ (hoặc _target\debug\hello_cargo.exe_ trên Windows) thay vì nằm ngay trong thư mục hiện tại của bạn. Vì cấu hình build mặc định là bản build debug, Cargo đặt tệp nhị phân trong một thư mục có tên là _debug_. Bạn có thể chạy tệp thực thi bằng lệnh này:

```console
$ ./target/debug/hello_cargo # hoặc .\target\debug\hello_cargo.exe trên Windows
Hello, world!
```

Nếu mọi việc suôn sẻ, dòng chữ `Hello, world!` sẽ được in ra terminal. Chạy `cargo build` lần đầu tiên cũng khiến Cargo tạo một tệp mới ở thư mục gốc: _Cargo.lock_. Tệp này theo dõi các phiên bản chính xác của các gói phụ thuộc trong dự án của bạn. Dự án này hiện chưa có phụ thuộc, vì vậy tệp còn khá đơn giản. Bạn không bao giờ cần phải chỉnh sửa tệp này thủ công; Cargo sẽ tự động quản lý nội dung của nó cho bạn.

Chúng ta vừa biên dịch một dự án bằng `cargo build` và chạy nó bằng `./target/debug/hello_cargo`, nhưng chúng ta cũng có thể sử dụng `cargo run` để vừa biên dịch vừa chạy tệp thực thi kết quả chỉ trong một lệnh duy nhất:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Sử dụng `cargo run` tiện lợi hơn nhiều so với việc phải nhớ chạy `cargo build` rồi gõ toàn bộ đường dẫn đến tệp nhị phân, vì vậy hầu hết các lập trình viên đều sử dụng `cargo run` khi phát triển.

Hãy chú ý rằng lần này chúng ta không thấy thông báo Cargo đang biên dịch `hello_cargo`. Cargo đã tự nhận biết rằng các tệp mã nguồn không hề thay đổi, vì vậy nó không biên dịch lại mà chỉ chạy trực tiếp tệp nhị phân đã có sẵn. Nếu bạn chỉnh sửa mã nguồn, Cargo sẽ tự động biên dịch lại dự án trước khi chạy, và bạn sẽ thấy kết quả như sau:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo cũng cung cấp một lệnh gọi là `cargo check`. Lệnh này nhanh chóng kiểm tra mã nguồn của bạn để đảm bảo nó biên dịch được mà không tạo ra tệp thực thi:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Tại sao bạn lại không muốn tạo ra một tệp thực thi? Thông thường, `cargo check` nhanh hơn `cargo build` rất nhiều vì nó bỏ qua bước tạo ra tệp nhị phân. Nếu bạn liên tục kiểm tra mã trong khi viết, việc sử dụng `cargo check` sẽ tăng tốc đáng kể quy trình làm việc để bạn biết liệu mã của mình có gặp lỗi biên dịch hay không! Do đó, nhiều Rustacean chạy `cargo check` định kỳ trong quá trình viết chương trình để đảm bảo mã hợp lệ, sau đó mới chạy `cargo build` khi họ thực sự sẵn sàng chạy thử chương trình.

Hãy cùng tóm tắt những gì chúng ta đã học về Cargo:

- Chúng ta có thể tạo một dự án mới bằng lệnh `cargo new`.
- Chúng ta có thể biên dịch dự án bằng `cargo build`.
- Chúng ta có thể biên dịch và chạy dự án trong một bước duy nhất bằng `cargo run`.
- Chúng ta có thể kiểm tra lỗi mã nguồn nhanh chóng mà không cần tạo tệp nhị phân bằng `cargo check`.
- Thay vì lưu kết quả build trong cùng thư mục với mã nguồn, Cargo lưu trữ nó trong thư mục _target/debug_.

Một lợi thế bổ sung của việc sử dụng Cargo là các câu lệnh đều giống hệt nhau trên mọi hệ điều hành bạn sử dụng. Vì vậy, từ thời điểm này, chúng tôi sẽ không cần cung cấp các hướng dẫn riêng biệt cho Linux, macOS và Windows nữa.

### Biên Dịch Cho Bản Phát Hành (Building for Release)

Khi dự án của bạn đã hoàn thiện và sẵn sàng phát hành, bạn có thể sử dụng `cargo build --release` để biên dịch với các tối ưu hóa hiệu năng cao nhất. Lệnh này sẽ tạo một tệp thực thi trong _target/release_ thay vì _target/debug_. Các tối ưu hóa này giúp mã Rust của bạn chạy nhanh hơn rất nhiều, nhưng việc bật chúng sẽ kéo dài thời gian biên dịch chương trình. Đây là lý do tại sao có hai cấu hình (profiles) khác nhau: một cho quá trình phát triển (development) khi bạn muốn biên dịch lại nhanh và thường xuyên, và một để xây dựng chương trình cuối cùng (release) gửi cho người dùng, nơi chương trình không cần build lại liên tục và sẽ chạy với tốc độ tối đa. Nếu bạn đang đo điểm chuẩn (benchmark) thời gian chạy của mã, hãy đảm bảo chạy `cargo build --release` và đo trên tệp thực thi trong _target/release_.

### Cargo Như Một Chuẩn Quy Ước

Với các dự án đơn giản, Cargo chưa thể hiện hết giá trị so với việc chỉ dùng `rustc`, nhưng nó sẽ chứng minh giá trị to lớn khi các chương trình của bạn trở nên phức tạp hơn. Khi các chương trình mở rộng ra nhiều tệp hoặc cần các thư viện phụ thuộc, việc để Cargo điều phối quá trình build sẽ dễ dàng hơn nhiều.

Mặc dù dự án `hello_cargo` rất đơn giản, giờ đây nó đã sử dụng phần lớn các công cụ thực tế mà bạn sẽ gắn bó trong suốt sự nghiệp lập trình Rust của mình. Trên thực tế, để làm việc trên bất kỳ dự án Rust mã nguồn mở có sẵn nào, bạn chỉ cần sử dụng các lệnh sau để lấy mã nguồn qua Git, chuyển đến thư mục của dự án đó và biên dịch:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Để biết thêm thông tin chi tiết về Cargo, hãy xem [tài liệu chính thức của Cargo][cargo].

{{#quiz ../quizzes/ch01-03-hello-cargo.toml}}

## Tóm Tắt

Bạn đã có một khởi đầu tuyệt vời trên hành trình chinh phục Rust! Trong chương này, bạn đã học được cách:

- Cài đặt phiên bản Rust ổn định mới nhất bằng `rustup`
- Cập nhật lên phiên bản Rust mới hơn
- Mở và tra cứu tài liệu cài đặt cục bộ
- Viết và chạy chương trình “Hello, world!” bằng `rustc` trực tiếp
- Tạo, quản lý và chạy một dự án mới theo các quy ước chuẩn của Cargo

Đây là thời điểm lý tưởng để xây dựng một chương trình thực tế hơn nhằm làm quen với việc đọc và viết mã Rust. Vì vậy, trong Chương 2, chúng ta sẽ cùng nhau xây dựng một trò chơi đoán số. Nếu bạn muốn bắt đầu bằng việc tìm hiểu các khái niệm lập trình phổ biến trong Rust trước, hãy đọc Chương 3 rồi sau đó quay lại Chương 2.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
