## Cài đặt

Bước đầu tiên là cài đặt Rust. Chúng ta sẽ tải Rust thông qua `rustup`, một công cụ dòng lệnh để quản lý các phiên bản Rust và các công cụ liên quan. Bạn sẽ cần kết nối internet để tải xuống.

> Lưu ý: Nếu bạn không muốn sử dụng `rustup` vì lý do nào đó, vui lòng xem [Trang các phương pháp cài đặt Rust khác][otherinstall] để biết thêm các tùy chọn.

Các bước sau đây sẽ cài đặt phiên bản ổn định (stable) mới nhất của trình biên dịch Rust. Các cam kết về tính ổn định của Rust đảm bảo rằng tất cả các ví dụ có thể biên dịch trong cuốn sách này sẽ tiếp tục biên dịch được với các phiên bản Rust mới hơn. Kết quả đầu ra có thể hơi khác nhau giữa các phiên bản vì Rust thường xuyên cải thiện các thông báo lỗi và cảnh báo. Nói cách khác, bất kỳ phiên bản Rust ổn định mới nào bạn cài đặt bằng các bước này đều sẽ hoạt động như mong đợi với nội dung của cuốn sách.

> ### Quy ước ký hiệu dòng lệnh
>
> Trong chương này và xuyên suốt cuốn sách, chúng tôi sẽ hiển thị một số lệnh được sử dụng trong terminal. Các dòng bạn nên nhập vào terminal đều bắt đầu bằng ký tự `$`. Bạn không cần nhập ký tự `$`; đó là dấu nhắc dòng lệnh (command prompt) được hiển thị để biểu thị điểm bắt đầu của mỗi lệnh. Các dòng không bắt đầu bằng `$` thường hiển thị kết quả đầu ra của lệnh trước đó. Ngoài ra, các ví dụ dành riêng cho PowerShell trên Windows sẽ sử dụng `>` thay vì `$`.

### Cài đặt `rustup` trên Linux hoặc macOS

Nếu bạn đang sử dụng Linux hoặc macOS, hãy mở terminal và nhập lệnh sau:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Lệnh này sẽ tải xuống một đoạn script và bắt đầu quá trình cài đặt công cụ `rustup`, công cụ này sẽ tự động cài đặt phiên bản ổn định mới nhất của Rust. Bạn có thể được yêu cầu nhập mật khẩu quản trị. Nếu cài đặt thành công, dòng thông báo sau sẽ xuất hiện:

```text
Rust is installed now. Great!
```

Bạn cũng sẽ cần một _linker_ (trình liên kết), một chương trình mà Rust sử dụng để liên kết các tệp đầu ra đã biên dịch thành một tệp thực thi duy nhất. Nhiều khả năng máy của bạn đã có sẵn linker. Nếu bạn gặp lỗi liên quan đến linker, bạn nên cài đặt một trình biên dịch C (C compiler), trình biên dịch này thường đi kèm với linker. Trình biên dịch C cũng rất hữu ích vì một số gói thư viện Rust phổ biến phụ thuộc vào mã C và sẽ cần đến trình biên dịch C.

Trên macOS, bạn có thể cài đặt trình biên dịch C bằng lệnh:

```console
$ xcode-select --install
```

Người dùng Linux thường nên cài đặt GCC hoặc Clang theo tài liệu hướng dẫn của bản phân phối (distribution) đang dùng. Ví dụ: nếu bạn dùng Ubuntu, bạn có thể cài đặt gói `build-essential`.

### Cài đặt `rustup` trên Windows

Trên Windows, hãy truy cập [https://www.rust-lang.org/tools/install][install] và làm theo hướng dẫn để tải và cài đặt Rust. Tại một thời điểm trong quá trình cài đặt, bạn sẽ được nhắc cài đặt Visual Studio. Điều này cung cấp linker và các thư viện gốc (native libraries) cần thiết để biên dịch chương trình. Nếu bạn cần thêm trợ giúp cho bước này, hãy xem [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc].

Phần còn lại của cuốn sách sử dụng các lệnh hoạt động trên cả _cmd.exe_ và PowerShell. Nếu có những điểm khác biệt cụ thể, chúng tôi sẽ giải thích rõ nên dùng lệnh nào.

### Khắc phục sự cố

Để kiểm tra xem bạn đã cài đặt Rust chính xác hay chưa, hãy mở terminal và nhập dòng lệnh sau:

```console
$ rustc --version
```

Bạn sẽ thấy số phiên bản, mã băm commit (commit hash) và ngày commit của phiên bản ổn định mới nhất đã được phát hành, theo định dạng sau:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Nếu bạn thấy thông tin này, bạn đã cài đặt Rust thành công! Nếu không thấy thông tin này, hãy kiểm tra xem đường dẫn tới Rust đã có trong biến môi trường hệ thống `PATH` của bạn chưa theo hướng dẫn sau:

Trên Windows CMD, sử dụng:

```console
> echo %PATH%
```

Trên PowerShell, sử dụng:

```powershell
> echo $env:Path
```

Trên Linux và macOS, sử dụng:

```console
$ echo $PATH
```

Nếu tất cả đều chính xác mà Rust vẫn không hoạt động, có nhiều nơi bạn có thể tìm kiếm sự trợ giúp. Hãy tìm cách kết nối với các Rustacean khác (biệt danh thân mật mà những người lập trình Rust tự gọi nhau) trên [trang cộng đồng Rust][community].

### Cập nhật và Gỡ cài đặt

Sau khi Rust được cài đặt qua `rustup`, việc cập nhật lên phiên bản mới phát hành rất dễ dàng. Từ terminal của bạn, chạy lệnh cập nhật sau:

```console
$ rustup update
```

Để gỡ cài đặt Rust và `rustup`, chạy lệnh gỡ cài đặt sau từ terminal:

```console
$ rustup self uninstall
```

### Tài liệu Ngoại tuyến Cục bộ

Bản cài đặt Rust cũng bao gồm một bản sao tài liệu cục bộ để bạn có thể đọc ngoại tuyến mà không cần kết nối mạng. Chạy lệnh `rustup doc` để mở tài liệu cục bộ trong trình duyệt của bạn.

Bất cứ khi nào có một kiểu dữ liệu hoặc hàm được thư viện chuẩn cung cấp mà bạn không chắc chắn nó làm gì hoặc cách sử dụng như thế nào, hãy tra cứu tài liệu Giao diện Lập trình Ứng dụng (API) để tìm hiểu!

### Trình soạn thảo Văn bản và IDE

Cuốn sách này không đưa ra giả định nào về các công cụ bạn sử dụng để viết mã Rust. Hầu như bất kỳ trình soạn thảo văn bản nào cũng có thể hoàn thành tốt công việc! Tuy nhiên, nhiều trình soạn thảo văn bản và Môi trường Phát triển Tích hợp (IDE) có hỗ trợ tích hợp sẵn rất tốt cho Rust. Bạn luôn có thể tìm thấy danh sách cập nhật các trình soạn thảo và IDE trên [trang công cụ][tools] của trang web Rust.

### Làm việc Ngoại tuyến với Cuốn Sách Này

Trong một số ví dụ, chúng ta sẽ sử dụng các gói thư viện Rust ngoài thư viện chuẩn. Để thực hành các ví dụ đó, bạn sẽ cần có kết nối internet hoặc đã tải trước các gói phụ thuộc đó. Để tải trước các gói phụ thuộc, bạn có thể chạy các lệnh sau (chúng tôi sẽ giải thích `cargo` là gì và chi tiết từng lệnh này sau):

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Thao tác này sẽ lưu vào bộ nhớ đệm (cache) các gói tải về để bạn không cần tải lại sau này. Sau khi chạy lệnh này, bạn không cần giữ lại thư mục `get-dependencies`. Khi đã chạy lệnh này, bạn có thể sử dụng cờ `--offline` với tất cả các lệnh `cargo` trong phần còn lại của cuốn sách để sử dụng các phiên bản đã được lưu trong bộ nhớ đệm thay vì kết nối qua mạng.

{{#quiz ../quizzes/ch01-01-installation.toml}}

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
