## Xin chào, Thế giới!

Bây giờ bạn đã cài đặt Rust, đã đến lúc viết chương trình Rust đầu tiên. Thường lệ khi học một ngôn ngữ mới là viết một chương trình nhỏ in ra dòng chữ `Hello, world!` lên màn hình, và chúng ta sẽ làm điều tương tự ở đây!

> Chú ý: Cuốn sách này giả định bạn đã quen cơ bản với dòng lệnh. Rust không đặt yêu cầu cụ thể về việc chỉnh sửa, công cụ hay nơi lưu mã nguồn của bạn, vì vậy nếu bạn thích dùng một môi trường phát triển tích hợp (IDE) thay vì dòng lệnh, cứ thoải mái dùng IDE ưa thích. Nhiều IDE hiện đã có mức độ hỗ trợ Rust; kiểm tra tài liệu của IDE để biết chi tiết. Nhóm Rust đang tập trung vào việc cải thiện hỗ trợ IDE tuyệt vời qua `rust-analyzer`. Xem [Appendix D][devtools]<!-- ignore --> để biết thêm chi tiết.

### Tạo Thư mục Dự án

Bạn sẽ bắt đầu bằng cách tạo một thư mục để lưu mã Rust. Rust không quan tâm mã của bạn nằm ở đâu, nhưng cho các bài tập và dự án trong cuốn sách này, chúng tôi gợi ý tạo một thư mục _projects_ trong thư mục nhà của bạn và giữ tất cả dự án ở đó.

Mở một terminal và nhập các lệnh sau để tạo thư mục _projects_ và một thư mục cho dự án “Hello, world!” bên trong thư mục _projects_.

Đối với Linux, macOS và PowerShell trên Windows, nhập:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Đối với CMD trên Windows, nhập:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Viết và Chạy Một Chương trình Rust

Tiếp theo, tạo một tệp nguồn mới và đặt tên là _main.rs_. Các tệp Rust luôn kết thúc bằng phần mở rộng _.rs_. Nếu bạn dùng nhiều hơn một từ trong tên tệp, quy ước là dùng dấu gạch dưới để phân tách chúng. Ví dụ, dùng _hello_world.rs_ thay vì _helloworld.rs_.

Bây giờ mở tệp _main.rs_ bạn vừa tạo và nhập mã trong Danh sách 1-1.

<Listing number="1-1" file-name="main.rs" caption="Một chương trình in `Hello, world!`">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

Lưu tệp và quay lại cửa sổ terminal trong thư mục _~/projects/hello_world_. Trên Linux hoặc macOS, nhập các lệnh sau để biên dịch và chạy tệp:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

Trên Windows, nhập lệnh `.\main` thay vì `./main`:

```powershell
> rustc main.rs
> .\main
Hello, world!
```

Bất kể hệ điều hành của bạn là gì, chuỗi `Hello, world!` sẽ được in ra terminal. Nếu bạn không thấy kết quả này, quay lại phần [“Troubleshooting”][troubleshooting]<!-- ignore --> trong phần Cài đặt để biết cách nhận trợ giúp.

Nếu `Hello, world!` xuất hiện, chúc mừng! Bạn đã chính thức viết một chương trình Rust. Điều đó biến bạn thành một lập trình viên Rust — chào mừng!

### Cấu trúc Một Chương trình Rust

Hãy xem xét chi tiết chương trình “Hello, world!” này. Đây là mảnh đầu tiên của bài toán:

```rust
fn main() {

}
```

Các dòng này định nghĩa một hàm tên là `main`. Hàm `main` là đặc biệt: nó luôn là mã đầu tiên chạy trong mọi chương trình thực thi Rust. Ở đây, dòng đầu khai báo một hàm tên `main` không có tham số và không trả về gì. Nếu có tham số, chúng sẽ nằm bên trong dấu ngoặc `()`.

Thân hàm được bao bởi `{}`. Rust yêu cầu dấu ngoặc nhọn quanh tất cả thân hàm. Việc đặt dấu ngoặc mở cùng dòng với khai báo hàm và để một khoảng trắng ở giữa là một phong cách tốt.

> Chú ý: Nếu bạn muốn tuân theo một phong cách chuẩn trên các dự án Rust, bạn có thể dùng công cụ định dạng tự động có tên `rustfmt` để định dạng mã theo một phong cách cụ thể (sẽ nói thêm về `rustfmt` trong [Appendix D][devtools]<!-- ignore -->). Nhóm Rust đã bao gồm công cụ này trong bản phân phối tiêu chuẩn, cũng như `rustc`, nên nó nên đã được cài trên máy của bạn!

Phần thân của hàm `main` chứa đoạn mã sau:

```rust
println!("Hello, world!");
```

Dòng này thực hiện toàn bộ công việc trong chương trình nhỏ này: nó in văn bản ra màn hình. Có ba chi tiết quan trọng cần lưu ý ở đây.

Thứ nhất, `println!` gọi một macro của Rust. Nếu nó gọi một hàm thay vì vậy, nó sẽ được viết là `println` (không có `!`). Macro trong Rust là một cách viết mã tạo ra mã để mở rộng cú pháp Rust, và chúng ta sẽ thảo luận chi tiết hơn trong [Chapter 20][ch20-macros]<!-- ignore -->. Hiện tại, bạn chỉ cần biết rằng việc có `!` nghĩa là bạn đang gọi một macro thay vì một hàm bình thường và macro không phải lúc nào cũng tuân theo các quy tắc giống hàm.

Thứ hai, bạn thấy chuỗi `"Hello, world!"`. Chúng ta truyền chuỗi này làm đối số cho `println!`, và chuỗi sẽ được in ra màn hình.

Thứ ba, chúng ta kết thúc dòng bằng dấu chấm phẩy (`;`), điều này chỉ ra rằng biểu thức này kết thúc và biểu thức tiếp theo có thể bắt đầu. Hầu hết các dòng mã Rust kết thúc bằng dấu chấm phẩy.

### Biên dịch và Chạy là Hai Bước Riêng Biệt

Bạn vừa chạy một chương trình mới tạo, vậy hãy xem xét từng bước trong quy trình.

Trước khi chạy một chương trình Rust, bạn phải biên dịch nó bằng trình biên dịch Rust bằng cách nhập lệnh `rustc` và truyền tên tệp nguồn, như sau:

```console
$ rustc main.rs
```

Nếu bạn có nền tảng C hoặc C++, bạn sẽ nhận thấy điều này tương tự `gcc` hoặc `clang`. Sau khi biên dịch thành công, Rust sẽ xuất ra một tệp thực thi nhị phân.

Trên Linux, macOS và PowerShell trên Windows, bạn có thể thấy tệp thực thi bằng cách nhập lệnh `ls` trong shell của bạn:

```console
$ ls
main  main.rs
```

Trên Linux và macOS, bạn sẽ thấy hai tệp. Với PowerShell trên Windows, bạn sẽ thấy cùng ba tệp như khi dùng CMD. Với CMD trên Windows, bạn sẽ nhập như sau:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

Điều này hiển thị tệp mã nguồn với phần mở rộng _.rs_, tệp thực thi (_main.exe_ trên Windows, nhưng _main_ trên các nền tảng khác), và, khi dùng Windows, một tệp chứa thông tin gỡ lỗi với phần mở rộng _.pdb_. Từ đây, bạn chạy tệp _main_ hoặc _main.exe_, như sau:

```console
$ ./main # or .\main on Windows
```

Nếu _main.rs_ của bạn là chương trình “Hello, world!”, dòng này sẽ in `Hello, world!` ra terminal của bạn.

Nếu bạn quen hơn với một ngôn ngữ động, như Ruby, Python, hoặc JavaScript, bạn có thể không quen việc biên dịch và chạy chương trình là hai bước riêng. Rust là một ngôn ngữ _biên dịch trước-thời-gian_ (ahead-of-time compiled), nghĩa là bạn có thể biên dịch một chương trình và đưa tệp thực thi cho người khác, và họ có thể chạy nó ngay cả khi không cài Rust. Nếu bạn đưa cho ai đó một tệp _.rb_, _.py_, hoặc _.js_, họ cần cài một trình triển khai Ruby, Python, hay JavaScript (tương ứng). Nhưng ở những ngôn ngữ đó, bạn chỉ cần một lệnh để biên dịch và chạy chương trình. Mọi thiết kế ngôn ngữ đều có sự đánh đổi.

Chỉ biên dịch bằng `rustc` là đủ cho các chương trình đơn giản, nhưng khi dự án của bạn lớn hơn, bạn sẽ muốn quản lý tất cả tùy chọn và dễ dàng chia sẻ mã. Tiếp theo, chúng ta sẽ giới thiệu công cụ Cargo, công cụ sẽ giúp bạn viết các chương trình Rust thực tế.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
