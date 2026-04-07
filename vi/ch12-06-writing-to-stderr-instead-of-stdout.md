## Ghi các Thông báo Lỗi ra Standard Error thay vì Standard Output

Tại thời điểm này, chúng ta đang ghi tất cả đầu ra của mình vào terminal bằng cách sử dụng
macro `println!`. Trong hầu hết các terminal, có hai loại đầu ra: _đầu ra tiêu chuẩn_
(`stdout`) cho thông tin chung và _lỗi tiêu chuẩn_ (`stderr`) cho
các thông báo lỗi. Sự phân biệt này cho phép người dùng chọn hướng đầu ra thành công
của một chương trình vào một tệp nhưng vẫn in các thông báo lỗi ra màn hình.

Macro `println!` chỉ có khả năng in ra đầu ra tiêu chuẩn, vì vậy chúng ta phải
sử dụng thứ khác để in ra lỗi tiêu chuẩn.

### Kiểm tra xem các Lỗi đang được ghi vào đâu

Đầu tiên, hãy quan sát cách nội dung được in bởi `minigrep` hiện đang được
ghi vào đầu ra tiêu chuẩn, bao gồm bất kỳ thông báo lỗi nào mà chúng ta muốn ghi vào
lỗi tiêu chuẩn thay thế. Chúng ta sẽ làm điều đó bằng cách chuyển hướng luồng đầu ra tiêu chuẩn
vào một tệp trong khi cố ý gây ra lỗi. Chúng ta sẽ không chuyển hướng luồng
lỗi tiêu chuẩn, vì vậy bất kỳ nội dung nào được gửi đến lỗi tiêu chuẩn sẽ tiếp tục hiển thị trên
màn hình.

Các chương trình dòng lệnh được mong đợi sẽ gửi các thông báo lỗi đến luồng lỗi tiêu chuẩn
để chúng ta vẫn có thể thấy các thông báo lỗi trên màn hình ngay cả khi chúng ta chuyển hướng
luồng đầu ra tiêu chuẩn vào một tệp. Chương trình của chúng ta hiện tại chưa thực hiện tốt điều này:
chúng ta sắp thấy rằng nó lưu thông báo lỗi vào một tệp thay thế!

Để chứng minh hành vi này, chúng ta sẽ chạy chương trình với `>` và đường dẫn tệp,
_output.txt_, mà chúng ta muốn chuyển hướng luồng đầu ra tiêu chuẩn vào. Chúng ta sẽ
không truyền bất kỳ đối số nào, điều này sẽ gây ra lỗi:

```console
$ cargo run > output.txt
```

Cú pháp `>` nói với shell ghi nội dung của đầu ra tiêu chuẩn vào
_output.txt_ thay vì màn hình. Chúng ta không thấy thông báo lỗi mà chúng ta đã
mong đợi được in ra màn hình, vì vậy điều đó có nghĩa là nó chắc chắn đã nằm trong
tệp. Đây là những gì _output.txt_ chứa:

```text
Problem parsing arguments: not enough arguments
```

Đúng vậy, thông báo lỗi của chúng ta đang được in ra đầu ra tiêu chuẩn. Sẽ hữu ích hơn nhiều
nếu các thông báo lỗi như thế này được in ra lỗi tiêu chuẩn để chỉ
dữ liệu từ một lần chạy thành công mới nằm trong tệp. Chúng ta sẽ thay đổi điều đó.

### In các Lỗi ra Standard Error

Chúng ta sẽ sử dụng mã trong Liệt kê 12-24 để thay đổi cách in các thông báo lỗi.
Do việc tái cấu trúc mà chúng ta đã thực hiện trước đó trong chương này, tất cả mã
in thông báo lỗi đều nằm trong một hàm, `main`. Thư viện tiêu chuẩn cung cấp
macro `eprintln!` in ra luồng lỗi tiêu chuẩn, vì vậy hãy thay đổi
hai nơi chúng ta đang gọi `println!` để in lỗi thành sử dụng `eprintln!`
thay thế.

<Listing number="12-24" file-name="src/main.rs" caption="Ghi các thông báo lỗi ra lỗi tiêu chuẩn thay vì đầu ra tiêu chuẩn bằng cách sử dụng `eprintln!` house">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Bây giờ hãy chạy lại chương trình theo cùng một cách, không có bất kỳ đối số nào và
chuyển hướng đầu ra tiêu chuẩn bằng `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Bây giờ chúng ta thấy lỗi trên màn hình và _output.txt_ không chứa gì, đó là
hành vi mà chúng ta mong đợi ở các chương trình dòng lệnh.

Hãy chạy lại chương trình với các đối số không gây ra lỗi nhưng vẫn
chuyển hướng đầu ra tiêu chuẩn vào một tệp, như sau:

```console
$ cargo run -- to poem.txt > output.txt
```

Chúng ta sẽ không thấy bất kỳ đầu ra nào trên terminal, và _output.txt_ sẽ chứa
kết quả của chúng ta:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Điều này chứng minh rằng bây giờ chúng ta đang sử dụng đầu ra tiêu chuẩn cho đầu ra thành công
và lỗi tiêu chuẩn cho đầu ra lỗi một cách thích hợp.

## Tổng kết

Chương này đã tóm tắt một số khái niệm chính mà bạn đã học cho đến nay và
đề cập đến cách thực hiện các thao tác I/O phổ biến trong Rust. Bằng cách sử dụng các đối số
dòng lệnh, tệp, biến môi trường và macro `eprintln!` để in
lỗi, giờ đây bạn đã sẵn sàng để viết các ứng dụng dòng lệnh. Kết hợp với
các khái niệm trong các chương trước, mã của bạn sẽ được tổ chức tốt, lưu trữ dữ liệu
hiệu quả trong các cấu trúc dữ liệu thích hợp, xử lý lỗi tốt và được
kiểm thử kỹ lưỡng.

Tiếp theo, chúng ta sẽ khám phá một số tính năng của Rust chịu ảnh hưởng bởi các ngôn ngữ
lập trình hàm: closure và iterator.
