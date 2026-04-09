<!-- Old link, do not remove -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Cài đặt các tệp nhị phân với `cargo install`

Lệnh `cargo install` cho phép bạn cài đặt và sử dụng các crate nhị phân
tại địa phương. Điều này không nhằm mục đích thay thế các gói hệ thống; nó được thiết kế để trở thành một
cách thuận tiện cho các nhà phát triển Rust cài đặt các công cụ mà những người khác đã chia sẻ trên
[crates.io](https://crates.io/)<!-- ignore -->. Lưu ý rằng bạn chỉ có thể cài đặt
các gói có các mục tiêu nhị phân (binary targets). Một _mục tiêu nhị phân_ là chương trình có thể chạy được
được tạo ra nếu crate có tệp _src/main.rs_ hoặc một tệp khác được chỉ định
là một tệp nhị phân, trái ngược với mục tiêu thư viện không thể tự chạy được nhưng
phù hợp để đưa vào bên trong các chương trình khác. Thông thường, các crate có
thông tin trong tệp _README_ về việc liệu một crate là một thư viện, có một
mục tiêu nhị phân, hoặc cả hai.

Tất cả các tệp nhị phân được cài đặt với `cargo install` được lưu trữ trong thư mục _bin_
của gốc cài đặt. Nếu bạn đã cài đặt Rust bằng _rustup.rs_ và không có bất kỳ
cấu hình tùy chỉnh nào, thư mục này sẽ là _$HOME/.cargo/bin_. Hãy đảm bảo rằng
thư mục đó nằm trong `$PATH` để có thể chạy các chương trình bạn đã cài đặt bằng `cargo install`.

Ví dụ, trong Chương 12 chúng ta đã đề cập rằng có một bản triển khai bằng Rust của
công cụ `grep` được gọi là `ripgrep` để tìm kiếm các tệp. Để cài đặt `ripgrep`, chúng ta
có thể chạy lệnh sau:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

Dòng kế cuối của đầu ra hiển thị vị trí và tên của
tệp nhị phân đã cài đặt, trong trường hợp của `ripgrep` là `rg`. Miễn là
thư mục cài đặt nằm trong `$PATH` của bạn, như đã đề cập trước đó, bạn có thể
chạy `rg --help` và bắt đầu sử dụng một công cụ tìm kiếm tệp nhanh hơn, đậm chất Rust hơn!
