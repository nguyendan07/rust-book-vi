## Hello, World!

Bây giờ bạn đã cài đặt Rust xong, đã đến lúc viết chương trình Rust đầu tiên của bạn.
Theo truyền thống khi học một ngôn ngữ mới, chúng ta sẽ viết một chương trình nhỏ in dòng chữ `Hello, world!` ra màn hình, và chúng ta cũng sẽ làm như vậy ở đây!

> Lưu ý: Cuốn sách này giả định bạn đã có kiến thức cơ bản về dòng lệnh (command line). Rust không đưa ra yêu cầu cụ thể nào về trình soạn thảo, bộ công cụ hay nơi lưu trữ mã nguồn của bạn, vì vậy nếu bạn thích sử dụng Môi trường Phát triển Tích hợp (IDE) thay vì dòng lệnh, hãy thoải mái sử dụng IDE yêu thích của bạn. Nhiều IDE hiện nay đã hỗ trợ Rust ở các mức độ khác nhau; hãy kiểm tra tài liệu của IDE để biết thêm chi tiết. Đội ngũ phát triển Rust đã và đang tập trung vào việc hỗ trợ IDE xuất sắc thông qua `rust-analyzer`. Xem [Phụ lục D][devtools]<!-- ignore --> để biết thêm chi tiết.

### Tạo Thư Mục Dự Án

Bạn sẽ bắt đầu bằng cách tạo một thư mục để lưu trữ mã nguồn Rust. Đối với Rust, mã nguồn của bạn nằm ở đâu không quan trọng, nhưng đối với các bài tập và dự án trong cuốn sách này, chúng tôi khuyên bạn nên tạo một thư mục _projects_ trong thư mục người dùng (home directory) của bạn và lưu giữ tất cả các dự án ở đó.

Mở terminal và nhập các lệnh sau để tạo một thư mục _projects_ và một thư mục cho dự án “Hello, world!” bên trong thư mục _projects_.

Đối với Linux, macOS, và PowerShell trên Windows, nhập các lệnh sau:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Đối với Windows CMD, nhập các lệnh sau:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Viết và Chạy một Chương Trình Rust

Tiếp theo, hãy tạo một tệp mã nguồn mới và đặt tên là _main.rs_. Các tệp mã nguồn Rust luôn kết thúc bằng phần mở rộng _.rs_. Nếu tên tệp của bạn có nhiều hơn một từ, quy ước là sử dụng dấu gạch dưới để phân tách chúng. Ví dụ: hãy dùng _hello_world.rs_ thay vì _helloworld.rs_.

Bây giờ hãy mở tệp _main.rs_ bạn vừa tạo và nhập đoạn mã trong Danh sách 1-1.

<Listing number="1-1" file-name="main.rs" caption="Một chương trình in ra `Hello, world!`">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

Lưu tệp và quay lại cửa sổ terminal của bạn trong thư mục _~/projects/hello_world_. Trên Linux hoặc macOS, nhập các lệnh sau để biên dịch và chạy tệp:

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

Bất kể bạn sử dụng hệ điều hành nào, chuỗi `Hello, world!` sẽ được in ra terminal. Nếu bạn không thấy kết quả này, hãy tham khảo lại phần [“Khắc phục sự cố”][troubleshooting]<!-- ignore --> trong mục Cài đặt để tìm cách nhận trợ giúp.

Nếu `Hello, world!` đã được in ra, xin chúc mừng! Bạn đã chính thức viết xong một chương trình Rust. Điều đó biến bạn thành một lập trình viên Rust — chào mừng bạn!

### Giải Phẫu Một Chương Trình Rust

Hãy cùng xem xét chi tiết chương trình “Hello, world!” này. Đây là mảnh ghép đầu tiên của bức tranh:

```rust
fn main() {

}
```

Những dòng này định nghĩa một hàm có tên là `main`. Hàm `main` rất đặc biệt: nó luôn là đoạn mã đầu tiên chạy trong mọi chương trình Rust thực thi được. Tại đây, dòng đầu tiên khai báo một hàm có tên là `main` không có tham số và không trả về giá trị nào. Nếu có tham số, chúng sẽ nằm bên trong dấu ngoặc đơn `()`.

Thân hàm được bao bọc trong cặp dấu ngoặc nhọn `{}`. Rust yêu cầu dấu ngoặc nhọn xung quanh tất cả các thân hàm. Phong cách viết mã chuẩn mực là đặt dấu ngoặc nhọn mở trên cùng một dòng với khai báo hàm, cách nhau một khoảng trắng.

> Lưu ý: Nếu bạn muốn tuân thủ phong cách chuẩn mực xuyên suốt các dự án Rust, bạn có thể sử dụng công cụ tự động định dạng mã có tên là `rustfmt` để định dạng mã của bạn theo phong cách nhất quán (xem thêm về `rustfmt` trong [Phụ lục D][devtools]<!-- ignore -->). Đội ngũ Rust đã tích hợp sẵn công cụ này trong bản phân phối Rust chuẩn giống như `rustc`, vì vậy nó đã có sẵn trên máy tính của bạn!

Thân hàm `main` chứa đoạn mã sau:

```rust
println!("Hello, world!");
```

Dòng này thực hiện toàn bộ công việc trong chương trình nhỏ này: in văn bản ra màn hình. Có ba chi tiết quan trọng cần lưu ý ở đây.

Thứ nhất, `println!` gọi một macro của Rust. Nếu nó gọi một hàm thông thường, nó sẽ được viết là `println` (không có dấu `!`). Macro trong Rust là một cách viết mã để sinh ra mã nhằm mở rộng cú pháp Rust, và chúng ta sẽ thảo luận chi tiết hơn về chúng trong [Chương 20][ch20-macros]<!-- ignore -->. Hiện tại, bạn chỉ cần biết rằng dấu `!` có nghĩa là bạn đang gọi một macro thay vì một hàm thông thường, và macro không phải lúc nào cũng tuân theo các quy tắc giống hệt như hàm.

Thứ hai, bạn thấy chuỗi `"Hello, world!"`. Chúng ta truyền chuỗi này làm đối số cho `println!`, và chuỗi sẽ được in ra màn hình.

Thứ ba, chúng ta kết thúc dòng bằng một dấu chấm phẩy (`;`), báo hiệu rằng biểu thức này đã kết thúc và biểu thức tiếp theo sẵn sàng bắt đầu. Hầu hết các dòng mã Rust đều kết thúc bằng dấu chấm phẩy.

### Biên Dịch và Chạy Là Hai Bước Tách Biệt

Bạn vừa chạy một chương trình mới tạo, vì vậy hãy xem xét kỹ từng bước trong quy trình này.

Trước khi chạy một chương trình Rust, bạn phải biên dịch nó bằng trình biên dịch Rust bằng cách nhập lệnh `rustc` và truyền cho nó tên tệp mã nguồn của bạn, như sau:

```console
$ rustc main.rs
```

Nếu bạn có nền tảng về C hoặc C++, bạn sẽ nhận thấy lệnh này tương tự như `gcc` hoặc `clang`. Sau khi biên dịch thành công, Rust sẽ xuất ra một tệp nhị phân có thể thực thi được.

Trên Linux, macOS, và PowerShell trên Windows, bạn có thể thấy tệp thực thi bằng cách nhập lệnh `ls` trong terminal:

```console
$ ls
main  main.rs
```

Trên Linux và macOS, bạn sẽ thấy hai tệp. Với PowerShell trên Windows, bạn sẽ thấy cùng ba tệp như khi dùng CMD. Với CMD trên Windows, bạn sẽ nhập lệnh sau:

```cmd
> dir /B %= tùy chọn /B chỉ hiển thị tên tệp =%
main.exe
main.pdb
main.rs
```

Lệnh này hiển thị tệp mã nguồn với phần mở rộng _.rs_, tệp thực thi (_main.exe_ trên Windows, nhưng là _main_ trên các hệ điều hành khác), và trên Windows sẽ có thêm một tệp chứa thông tin gỡ lỗi (debugging information) với phần mở rộng _.pdb_. Từ đây, bạn chạy tệp _main_ hoặc _main.exe_ như sau:

```console
$ ./main # hoặc .\main trên Windows
```

Nếu _main.rs_ là chương trình “Hello, world!” của bạn, dòng lệnh này sẽ in `Hello, world!` ra terminal của bạn.

Nếu bạn quen thuộc hơn với các ngôn ngữ động như Ruby, Python, hoặc JavaScript, bạn có thể chưa quen với việc biên dịch và chạy một chương trình như hai bước tách biệt. Rust là một ngôn ngữ _biên dịch trước_ (ahead-of-time compiled), nghĩa là bạn có thể biên dịch chương trình và gửi tệp thực thi đó cho người khác, và họ có thể chạy nó ngay cả khi máy của họ không hề cài đặt Rust. Nếu bạn đưa cho ai đó một tệp _.rb_, _.py_, hoặc _.js_, họ bắt buộc phải cài đặt môi trường thực thi Ruby, Python, hoặc JavaScript tương ứng. Nhưng trong các ngôn ngữ đó, bạn chỉ cần một lệnh duy nhất để vừa thông dịch vừa chạy chương trình. Mọi thứ đều là sự đánh đổi trong thiết kế ngôn ngữ lập trình.

Chỉ biên dịch bằng `rustc` là đủ đối với các chương trình đơn giản, nhưng khi dự án của bạn phát triển lớn hơn, bạn sẽ muốn quản lý tất cả các tùy chọn cấu hình và giúp việc chia sẻ mã nguồn với người khác trở nên dễ dàng. Tiếp theo, chúng tôi sẽ giới thiệu cho bạn công cụ Cargo, công cụ sẽ đồng hành cùng bạn để viết các chương trình Rust trong thế giới thực.

{{#quiz ../quizzes/ch01-02-hello-world.toml}}

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
