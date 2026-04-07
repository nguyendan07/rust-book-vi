## Đọc Một Tệp

Bây giờ chúng ta sẽ thêm chức năng để đọc tệp được chỉ định trong đối số `file_path`.
Đầu tiên chúng ta cần một tệp mẫu để kiểm thử: chúng ta sẽ sử dụng một tệp với một
lượng nhỏ văn bản trên nhiều dòng với một số từ lặp lại. Liệt kê 12-3
có một bài thơ của Emily Dickinson sẽ hoạt động tốt! Tạo một tệp tên là
_poem.txt_ ở cấp độ gốc của dự án, và nhập bài thơ “I’m Nobody!
Who are you?”

<Listing number="12-3" file-name="poem.txt" caption="Một bài thơ của Emily Dickinson tạo nên một trường hợp kiểm thử tốt.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Sau khi văn bản đã sẵn sàng, hãy chỉnh sửa _src/main.rs_ và thêm mã để đọc tệp, như
được hiển thị trong Liệt kê 12-4.

<Listing number="12-4" file-name="src/main.rs" caption="Đọc nội dung của tệp được chỉ định bởi đối số thứ hai">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Đầu tiên, chúng ta đưa một phần liên quan của thư viện chuẩn vào bằng một câu lệnh `use`:
chúng ta cần `std::fs` để xử lý các tệp.

Trong `main`, câu lệnh mới `fs::read_to_string` nhận `file_path`, mở
tệp đó và trả về một giá trị kiểu `std::io::Result<String>` chứa
nội dung của tệp.

Sau đó, chúng ta lại thêm một câu lệnh `println!` tạm thời để in giá trị
của `contents` sau khi tệp được đọc, để chúng ta có thể kiểm tra xem chương trình có
đang hoạt động cho đến nay hay không.

Hãy chạy mã này với bất kỳ chuỗi nào làm đối số dòng lệnh đầu tiên (vì
chúng ta chưa triển khai phần tìm kiếm) và tệp _poem.txt_ là
đối số thứ hai:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Tuyệt vời! Mã đã đọc và sau đó in nội dung của tệp. Nhưng mã
có một vài thiếu sót. Hiện tại, hàm `main` có nhiều
trách nhiệm: nhìn chung, các hàm sẽ rõ ràng và dễ bảo trì hơn nếu
mỗi hàm chỉ chịu trách nhiệm cho một ý tưởng duy nhất. Vấn đề khác là chúng ta
không xử lý lỗi tốt như chúng ta có thể. Chương trình vẫn còn nhỏ, vì vậy những
thiếu sót này không phải là vấn đề lớn, nhưng khi chương trình phát triển, sẽ khó để sửa
chúng một cách sạch sẽ. Một thực hành tốt là bắt đầu tái cấu trúc (refactoring) sớm khi
phát triển một chương trình vì việc tái cấu trúc lượng mã nhỏ hơn sẽ dễ dàng hơn nhiều. Chúng ta sẽ làm điều đó tiếp theo.
