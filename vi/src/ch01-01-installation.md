## Cài đặt

Bước đầu tiên là cài đặt Rust. Chúng ta sẽ tải Rust thông qua `rustup`, một công cụ dòng lệnh để quản lý các phiên bản Rust và các công cụ liên quan. Bạn sẽ cần kết nối internet để tải xuống.

> Lưu ý: Nếu bạn không muốn sử dụng `rustup` vì một lý do nào đó, vui lòng xem trang [Các phương pháp cài đặt Rust khác][otherinstall] để biết thêm các tùy chọn.

Các bước sau đây sẽ cài đặt phiên bản ổn định mới nhất của trình biên dịch Rust. Đảm bảo về sự ổn định của Rust đảm bảo rằng tất cả các ví dụ trong sách có thể biên dịch được sẽ tiếp tục biên dịch được với các phiên bản Rust mới hơn. Đầu ra có thể khác một chút giữa các phiên bản vì Rust thường cải thiện thông báo lỗi và cảnh báo. Nói cách khác, bất kỳ phiên bản Rust ổn định, mới hơn nào bạn cài đặt bằng các bước này đều sẽ hoạt động như mong đợi với nội dung của cuốn sách này.

> ### Ký hiệu dòng lệnh
>
> Trong chương này và xuyên suốt cuốn sách, chúng tôi sẽ hiển thị một số lệnh được sử dụng trong terminal. Các dòng mà bạn nên nhập vào terminal đều bắt đầu bằng `$`. Bạn không cần phải gõ ký tự `$`; đó là dấu nhắc dòng lệnh được hiển thị để chỉ ra sự bắt đầu của mỗi lệnh. Các dòng không bắt đầu bằng `$` thường hiển thị đầu ra của lệnh trước đó. Ngoài ra, các ví dụ dành riêng cho PowerShell sẽ sử dụng `>` thay vì `$`.

### Cài đặt `rustup` trên Linux hoặc macOS

Nếu bạn đang sử dụng Linux hoặc macOS, hãy mở một terminal và nhập lệnh sau:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Lệnh này tải xuống một kịch bản và bắt đầu cài đặt công cụ `rustup`, công cụ này sẽ cài đặt phiên bản Rust ổn định mới nhất. Bạn có thể được nhắc nhập mật khẩu. Nếu cài đặt thành công, dòng sau sẽ xuất hiện:

```text
Rust is installed now. Great!
```

Bạn cũng sẽ cần một _linker_, là một chương trình mà Rust sử dụng để nối các kết quả đã biên dịch của nó thành một tệp duy nhất. Rất có thể bạn đã có một linker. Nếu bạn gặp lỗi linker, bạn nên cài đặt một trình biên dịch C, thường sẽ bao gồm một linker. Một trình biên dịch C cũng hữu ích vì một số gói Rust phổ biến phụ thuộc vào mã C và sẽ cần một trình biên dịch C.

Trên macOS, bạn có thể nhận một trình biên dịch C bằng cách chạy:

```console
$ xcode-select --install
```

Người dùng Linux thường nên cài đặt GCC hoặc Clang, theo tài liệu của bản phân phối của họ. Ví dụ, nếu bạn sử dụng Ubuntu, bạn có thể cài đặt gói `build-essential`.

### Cài đặt `rustup` trên Windows

Trên Windows, hãy truy cập [https://www.rust-lang.org/tools/install][install] và làm theo hướng dẫn để cài đặt Rust. Tại một thời điểm nào đó trong quá trình cài đặt, bạn sẽ được nhắc cài đặt Visual Studio. Điều này cung cấp một linker và các thư viện gốc cần thiết để biên dịch chương trình. Nếu bạn cần thêm trợ giúp với bước này, hãy xem [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]

Phần còn lại của cuốn sách này sử dụng các lệnh hoạt động trong cả _cmd.exe_ và PowerShell. Nếu có sự khác biệt cụ thể, chúng tôi sẽ giải thích nên sử dụng cái nào.

### Xử lý sự cố

Để kiểm tra xem bạn đã cài đặt Rust đúng cách chưa, hãy mở một shell và nhập dòng này:

```console
$ rustc --version
```

Bạn sẽ thấy số phiên bản, mã hash của commit và ngày commit cho phiên bản ổn định mới nhất đã được phát hành, theo định dạng sau:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Nếu bạn thấy thông tin này, bạn đã cài đặt Rust thành công! Nếu bạn không thấy thông tin này, hãy kiểm tra xem Rust có trong biến hệ thống `%PATH%` của bạn như sau.

Trong Windows CMD, sử dụng:

```console
> echo %PATH%
```

Trong PowerShell, sử dụng:

```powershell
> echo $env:Path
```

Trên Linux và macOS, sử dụng:

```console
$ echo $PATH
```

Nếu tất cả đều đúng và Rust vẫn không hoạt động, có một số nơi bạn có thể nhận được sự giúp đỡ. Tìm hiểu cách liên lạc với các Rustacean khác (một biệt danh ngớ ngẩn mà chúng tôi tự gọi mình) trên [trang cộng đồng][community].

### Cập nhật và Gỡ cài đặt

Sau khi Rust được cài đặt qua `rustup`, việc cập nhật lên phiên bản mới phát hành rất dễ dàng. Từ shell của bạn, chạy kịch bản cập nhật sau:

```console
$ rustup update
```

Để gỡ cài đặt Rust và `rustup`, hãy chạy kịch bản gỡ cài đặt sau từ shell của bạn:

```console
$ rustup self uninstall
```

### Tài liệu cục bộ

Việc cài đặt Rust cũng bao gồm một bản sao cục bộ của tài liệu để bạn có thể đọc nó ngoại tuyến. Chạy `rustup doc` để mở tài liệu cục bộ trong trình duyệt của bạn.

Bất cứ khi nào một kiểu hoặc hàm được cung cấp bởi thư viện chuẩn và bạn không chắc nó làm gì hoặc cách sử dụng nó, hãy sử dụng tài liệu giao diện lập trình ứng dụng (API) để tìm hiểu!

### Trình soạn thảo văn bản và Môi trường phát triển tích hợp

Cuốn sách này không đưa ra giả định nào về công cụ bạn sử dụng để viết mã Rust. Hầu như bất kỳ trình soạn thảo văn bản nào cũng có thể hoàn thành công việc! Tuy nhiên, nhiều trình soạn thảo văn bản và môi trường phát triển tích hợp (IDE) có hỗ trợ tích hợp cho Rust. Bạn luôn có thể tìm thấy một danh sách khá cập nhật của nhiều trình soạn thảo và IDE trên [trang công cụ][tools] trên trang web của Rust.

### Làm việc ngoại tuyến với cuốn sách này

Trong một số ví dụ, chúng tôi sẽ sử dụng các gói Rust ngoài thư viện chuẩn. Để thực hiện các ví dụ đó, bạn sẽ cần có kết nối internet hoặc đã tải xuống các phụ thuộc đó trước. Để tải xuống các phụ thuộc trước, bạn có thể chạy các lệnh sau. (Chúng tôi sẽ giải thích `cargo` là gì và mỗi lệnh này làm gì một cách chi tiết sau.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Điều này sẽ lưu vào bộ đệm các bản tải xuống cho các gói này để bạn không cần phải tải chúng xuống sau này. Sau khi bạn đã chạy lệnh này, bạn không cần giữ thư mục `get-dependencies`. Nếu bạn đã chạy lệnh này, bạn có thể sử dụng cờ `--offline` với tất cả các lệnh `cargo` trong phần còn lại của cuốn sách để sử dụng các phiên bản đã lưu trong bộ đệm này thay vì cố gắng sử dụng mạng.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
