## Kiểm soát cách chạy các Test

Giống như `cargo run` biên dịch mã của bạn và sau đó chạy bản thực thi kết quả,
`cargo test` biên dịch mã của bạn ở chế độ test và chạy bản thực thi test
kết quả. Hành vi mặc định của bản thực thi được tạo bởi `cargo test` là chạy
tất cả các test song song và thu thập đầu ra (capture output) được tạo ra trong quá trình chạy test,
ngăn chặn đầu ra hiển thị và giúp dễ dàng đọc đầu ra liên quan đến
kết quả test hơn. Tuy nhiên, bạn có thể chỉ định các tùy chọn dòng lệnh
để thay đổi hành vi mặc định này.

Một số tùy chọn dòng lệnh dành cho `cargo test`, và một số dành cho bản thực thi
test kết quả. Để phân tách hai loại đối số này, bạn liệt kê các đối số dành
cho `cargo test` theo sau là dấu phân cách `--` và sau đó là những đối số dành cho
bản thực thi test. Chạy `cargo test --help` hiển thị các tùy chọn bạn có thể sử dụng
với `cargo test`, và chạy `cargo test -- --help` hiển thị các tùy chọn bạn
có thể sử dụng sau dấu phân cách. Các tùy chọn đó cũng được ghi lại trong
[phần “Tests”][tests] của [cuốn sách rustc][rustc].

[tests]: https://doc.rust-lang.org/rustc/tests/index.html
[rustc]: https://doc.rust-lang.org/rustc/index.html

### Chạy các Test song song hoặc tuần tự

Khi bạn chạy nhiều test, theo mặc định chúng chạy song song bằng cách sử dụng các luồng (threads),
nghĩa là chúng kết thúc việc chạy nhanh hơn và bạn nhận được phản hồi sớm hơn. Bởi vì các
test đang chạy cùng một lúc, bạn phải đảm bảo các test của mình không phụ thuộc
vào nhau hoặc vào bất kỳ trạng thái chia sẻ nào, bao gồm cả môi trường chia sẻ, chẳng hạn
như thư mục làm việc hiện tại hoặc các biến môi trường.

Ví dụ, giả sử mỗi test của bạn chạy một đoạn mã tạo ra một tệp trên đĩa
tên là _test-output.txt_ và ghi một số dữ liệu vào tệp đó. Sau đó, mỗi test đọc
dữ liệu trong tệp đó và khẳng định rằng tệp chứa một giá trị cụ thể,
giá trị này khác nhau trong mỗi test. Bởi vì các test chạy cùng lúc, một
test có thể ghi đè lên tệp trong khoảng thời gian giữa lúc một test khác đang ghi và
đang đọc tệp. Test thứ hai sau đó sẽ thất bại, không phải vì mã
sai mà vì các test đã can thiệp lẫn nhau khi chạy
song song. Một giải pháp là đảm bảo mỗi test ghi vào một tệp khác nhau;
một giải pháp khác là chạy các test từng cái một.

Nếu bạn không muốn chạy các test song song hoặc nếu bạn muốn kiểm soát chi tiết
hơn về số lượng luồng được sử dụng, bạn có thể gửi cờ `--test-threads`
và số lượng luồng bạn muốn sử dụng cho bản thực thi test. Hãy xem
ví dụ sau:

```console
$ cargo test -- --test-threads=1
```

Chúng ta đặt số lượng luồng test thành `1`, yêu cầu chương trình không sử dụng bất kỳ
sự song song nào. Chạy các test bằng một luồng sẽ mất nhiều thời gian hơn chạy
song song, nhưng các test sẽ không can thiệp lẫn nhau nếu chúng chia sẻ
trạng thái.

### Hiển thị đầu ra của hàm

Theo mặc định, nếu một test vượt qua, thư viện test của Rust sẽ thu thập bất cứ thứ gì được in ra
đầu ra tiêu chuẩn (standard output). Ví dụ, nếu chúng ta gọi `println!` trong một test và test đó
vượt qua, chúng ta sẽ không thấy đầu ra của `println!` trong terminal; chúng ta sẽ chỉ thấy
dòng cho biết test đã vượt qua. Nếu một test thất bại, chúng ta sẽ thấy bất cứ điều gì đã được
in ra đầu ra tiêu chuẩn cùng với phần còn lại của thông báo thất bại.

Ví dụ, Liệt kê 11-10 có một hàm ngớ ngẩn in ra giá trị tham số của nó
và trả về 10, cũng như một test vượt qua và một test thất bại.

<Listing number="11-10" file-name="src/lib.rs" caption="Các test cho một hàm gọi `println!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Khi chúng ta chạy các test này với `cargo test`, chúng ta sẽ thấy đầu ra sau:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Lưu ý rằng không có chỗ nào trong đầu ra này chúng ta thấy `I got the value 4`,
vốn được in khi test vượt qua chạy. Đầu ra đó đã bị thu thập. Đầu
ra từ test bị thất bại, `I got the value 8`, xuất hiện trong phần
tóm tắt đầu ra test, phần này cũng hiển thị nguyên nhân gây ra thất bại của test.

Nếu chúng ta cũng muốn xem các giá trị được in cho các test vượt qua, chúng ta có thể yêu cầu Rust
hiển thị cả đầu ra của các test thành công bằng `--show-output`:

```console
$ cargo test -- --show-output
```

Khi chúng ta chạy lại các test trong Liệt kê 11-10 với cờ `--show-output`, chúng ta
thấy đầu ra sau:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Chạy một tập hợp con các Test theo tên

Đôi khi, việc chạy toàn bộ bộ test (test suite) có thể mất nhiều thời gian. Nếu bạn đang làm việc trên
mã ở một khu vực cụ thể, bạn có thể chỉ muốn chạy các test liên quan đến
đoạn mã đó. Bạn có thể chọn test nào để chạy bằng cách truyền cho `cargo test` tên
hoặc các tên của (các) test bạn muốn chạy dưới dạng một đối số.

Để minh họa cách chạy một tập hợp con các test, trước tiên chúng ta sẽ tạo ba test cho
hàm `add_two` của mình, như được hiển thị trong Liệt kê 11-11, và chọn cái nào để chạy.

<Listing number="11-11" file-name="src/lib.rs" caption="Ba test với ba cái tên khác nhau">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Nếu chúng ta chạy các test mà không truyền bất kỳ đối số nào, như chúng ta đã thấy trước đó, tất cả các
test sẽ chạy song song:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Chạy các Test đơn lẻ

Chúng ta có thể truyền tên của bất kỳ hàm test nào cho `cargo test` để chỉ chạy test đó:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Chỉ có test có tên `one_hundred` đã chạy; hai test còn lại không khớp
với tên đó. Đầu ra test cho chúng ta biết rằng chúng ta còn nhiều test khác không chạy bằng cách
hiển thị `2 filtered out` ở cuối.

Chúng ta không thể chỉ định tên của nhiều test theo cách này; chỉ giá trị đầu tiên
được đưa cho `cargo test` sẽ được sử dụng. Nhưng có một cách để chạy nhiều test.

#### Lọc để chạy nhiều Test

Chúng ta có thể chỉ định một phần của tên test, và bất kỳ test nào có tên khớp với giá trị đó
sẽ được chạy. Ví dụ, vì hai trong số các tên test của chúng ta chứa `add`, chúng ta có thể
chạy hai test đó bằng cách chạy `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Lệnh này đã chạy tất cả các test có `add` trong tên và lọc bỏ test
tên là `one_hundred`. Cũng lưu ý rằng module mà một test xuất hiện trở thành
một phần tên của test, vì vậy chúng ta có thể chạy tất cả các test trong một module bằng cách lọc
theo tên của module đó.

### Bỏ qua một số Test trừ khi được yêu cầu cụ thể

Đôi khi một vài test cụ thể có thể rất tốn thời gian để thực thi, vì vậy bạn
có thể muốn loại trừ chúng trong hầu hết các lần chạy `cargo test`. Thay vì
liệt kê dưới dạng đối số tất cả các test bạn muốn chạy, thay vào đó bạn có thể chú thích các
test tốn thời gian bằng cách sử dụng thuộc tính `ignore` để loại trừ chúng, như được hiển thị
ở đây:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

Sau `#[test]`, chúng ta thêm dòng `#[ignore]` vào test mà chúng ta muốn loại trừ.
Bây giờ khi chúng ta chạy các test của mình, `it_works` chạy, nhưng `expensive_test` thì không:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

Hàm `expensive_test` được liệt kê là `ignored`. Nếu chúng ta chỉ muốn chạy
các test bị bỏ qua, chúng ta có thể sử dụng `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Bằng cách kiểm soát test nào được chạy, bạn có thể đảm bảo kết quả `cargo test` của mình
sẽ được trả về nhanh chóng. Khi bạn ở thời điểm mà việc kiểm tra
kết quả của các test `ignored` là hợp lý và bạn có thời gian để chờ kết quả,
bạn có thể chạy `cargo test -- --ignored` thay thế. Nếu bạn muốn chạy tất cả các test
cho dù chúng có bị bỏ qua hay không, bạn có thể chạy `cargo test -- --include-ignored`.

{{#quiz ../quizzes/ch11-02-running-tests.toml}}
