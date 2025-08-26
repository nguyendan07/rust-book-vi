## Phụ lục D - Các công cụ phát triển hữu ích

Trong phụ lục này, chúng ta sẽ nói về một số công cụ phát triển hữu ích mà dự án Rust cung cấp. Chúng ta sẽ xem xét định dạng tự động, các cách nhanh chóng để áp dụng các bản sửa lỗi cảnh báo, một linter và tích hợp với các IDE.

### Định dạng tự động với `rustfmt`

Công cụ `rustfmt` định dạng lại mã của bạn theo phong cách mã của cộng đồng. Nhiều dự án hợp tác sử dụng `rustfmt` để ngăn chặn các cuộc tranh cãi về việc sử dụng phong cách nào khi viết Rust: mọi người đều định dạng mã của mình bằng công cụ này.

Các bản cài đặt Rust bao gồm `rustfmt` theo mặc định, vì vậy bạn đã có sẵn các chương trình `rustfmt` và `cargo-fmt` trên hệ thống của mình. Hai lệnh này tương tự như `rustc` và `cargo` ở chỗ `rustfmt` cho phép kiểm soát chi tiết hơn và `cargo-fmt` hiểu các quy ước của một dự án sử dụng Cargo. Để định dạng bất kỳ dự án Cargo nào, hãy nhập như sau:

```console
$ cargo fmt
```

Chạy lệnh này sẽ định dạng lại tất cả mã Rust trong crate hiện tại. Điều này chỉ nên thay đổi phong cách mã, không phải ngữ nghĩa của mã. Để biết thêm thông tin về `rustfmt`, hãy xem [tài liệu của nó][rustfmt].

### Sửa mã của bạn với `rustfix`

Công cụ `rustfix` được bao gồm trong các bản cài đặt Rust và có thể tự động sửa các cảnh báo của trình biên dịch có cách khắc phục rõ ràng và có khả năng là điều bạn muốn. Có lẽ bạn đã từng thấy các cảnh báo của trình biên dịch trước đây. Ví dụ, hãy xem xét đoạn mã này:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
  let mut x = 42;
  println!("{x}");
}
```

Ở đây, chúng ta định nghĩa biến `x` là có thể thay đổi, nhưng chúng ta không bao giờ thực sự thay đổi nó. Rust cảnh báo chúng ta về điều đó:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

Cảnh báo đề nghị chúng ta xóa từ khóa `mut`. Chúng ta có thể tự động áp dụng đề xuất đó bằng công cụ `rustfix` bằng cách chạy lệnh `cargo fix`:

```console
$ cargo fix
  Checking myprogram v0.1.0 (file:///projects/myprogram)
    Fixing src/main.rs (1 fix)
  Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Khi chúng ta xem lại tệp _src/main.rs_, chúng ta sẽ thấy rằng `cargo fix` đã thay đổi mã:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
  let x = 42;
  println!("{x}");
}
```

Biến `x` bây giờ là bất biến, và cảnh báo không còn xuất hiện nữa.

Bạn cũng có thể sử dụng lệnh `cargo fix` để chuyển đổi mã của mình giữa các phiên bản Rust khác nhau. Các phiên bản được đề cập trong [Phụ lục E][editions].

### Nhiều lints hơn với Clippy

Công cụ Clippy là một tập hợp các lints để phân tích mã của bạn để bạn có thể phát hiện các lỗi phổ biến và cải thiện mã Rust của mình. Clippy được bao gồm trong các bản cài đặt Rust tiêu chuẩn.

Để chạy các lints của Clippy trên bất kỳ dự án Cargo nào, hãy nhập như sau:

```console
$ cargo clippy
```

Ví dụ, giả sử bạn viết một chương trình sử dụng một giá trị gần đúng của một hằng số toán học, chẳng hạn như pi, như chương trình này:

<Listing file-name="src/main.rs">

```rust
fn main() {
  let x = 3.1415;
  let r = 8.0;
  println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Chạy `cargo clippy` trên dự án này sẽ dẫn đến lỗi sau:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Lỗi này cho bạn biết rằng Rust đã có một hằng số `PI` chính xác hơn được định nghĩa, và chương trình của bạn sẽ đúng hơn nếu bạn sử dụng hằng số đó. Sau đó, bạn sẽ thay đổi mã của mình để sử dụng hằng số `PI`.

Đoạn mã sau không gây ra bất kỳ lỗi hoặc cảnh báo nào từ Clippy:

<Listing file-name="src/main.rs">

```rust
fn main() {
  let x = std::f64::consts::PI;
  let r = 8.0;
  println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Để biết thêm thông tin về Clippy, hãy xem [tài liệu của nó][clippy].

### Tích hợp IDE bằng `rust-analyzer`

Để hỗ trợ tích hợp IDE, cộng đồng Rust khuyến nghị sử dụng [`rust-analyzer`][rust-analyzer]<!-- ignore -->. Công cụ này là một bộ các tiện ích tập trung vào trình biên dịch, sử dụng [Giao thức Máy chủ Ngôn ngữ (Language Server Protocol)][lsp]<!--
ignore -->, là một đặc tả để các IDE và ngôn ngữ lập trình giao tiếp với nhau. Các máy khách khác nhau có thể sử dụng `rust-analyzer`, chẳng hạn như [plug-in Rust analyzer cho Visual Studio Code][vscode].

Truy cập [trang chủ][rust-analyzer]<!-- ignore --> của dự án `rust-analyzer` để xem hướng dẫn cài đặt, sau đó cài đặt hỗ trợ máy chủ ngôn ngữ trong IDE cụ thể của bạn. IDE của bạn sẽ có thêm các khả năng như tự động hoàn thành, nhảy đến định nghĩa và hiển thị lỗi nội tuyến.

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
