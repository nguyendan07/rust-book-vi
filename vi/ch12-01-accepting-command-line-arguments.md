## Chấp Nhận Các Đối Số Dòng Lệnh

Hãy tạo một dự án mới với, như thường lệ, `cargo new`. Chúng ta sẽ gọi dự án của mình là
`minigrep` để phân biệt nó với công cụ `grep` mà bạn có thể đã có
trên hệ thống của mình.

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Nhiệm vụ đầu tiên là làm cho `minigrep` chấp nhận hai đối số dòng lệnh của nó:
đường dẫn tệp và một chuỗi để tìm kiếm. Nghĩa là, chúng ta muốn có thể chạy
chương trình của mình với `cargo run`, hai dấu gạch ngang để chỉ ra các đối số tiếp theo
dành cho chương trình của chúng ta thay vì dành cho `cargo`, một chuỗi để tìm kiếm và một đường dẫn đến
một tệp để tìm kiếm trong đó, giống như thế này:

```console
$ cargo run -- searchstring example-filename.txt
```

Hiện tại, chương trình được tạo bởi `cargo new` không thể xử lý các đối số mà chúng ta
đưa cho nó. Một số thư viện hiện có trên [crates.io](https://crates.io/) có thể giúp
viết một chương trình chấp nhận các đối số dòng lệnh, nhưng vì bạn mới
đang học khái niệm này, hãy tự mình triển khai khả năng này.

### Đọc Các Giá Trị Đối Số

Để cho phép `minigrep` đọc các giá trị của các đối số dòng lệnh mà chúng ta truyền cho
nó, chúng ta sẽ cần hàm `std::env::args` được cung cấp trong thư viện chuẩn của Rust.
Hàm này trả về một trình lặp (iterator) của các đối số dòng lệnh được truyền
cho `minigrep`. Chúng ta sẽ tìm hiểu kỹ về trình lặp trong [Chương 13][ch13]<!-- ignore
-->. Hiện tại, bạn chỉ cần biết hai chi tiết về trình lặp: trình lặp
tạo ra một chuỗi các giá trị và chúng ta có thể gọi phương thức `collect` trên một trình lặp
để biến nó thành một bộ sưu tập (collection), chẳng hạn như một vector, chứa tất cả các phần tử mà
trình lặp tạo ra.

Mã trong Liệt kê 12-1 cho phép chương trình `minigrep` của bạn đọc bất kỳ đối số dòng lệnh
nào được truyền cho nó, và sau đó thu thập các giá trị vào một vector.

<Listing number="12-1" file-name="src/main.rs" caption="Thu thập các đối số dòng lệnh vào một vector và in chúng ra">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Đầu tiên, chúng ta đưa module `std::env` vào phạm vi bằng câu lệnh `use` để chúng ta
có thể sử dụng hàm `args` của nó. Lưu ý rằng hàm `std::env::args` được
lồng trong hai cấp module. Như chúng ta đã thảo luận trong [Chương
7][ch7-idiomatic-use]<!-- ignore -->, trong những trường hợp mà hàm mong muốn
được lồng trong nhiều hơn một module, chúng ta đã chọn đưa module cha vào
phạm vi thay vì chính hàm đó. Bằng cách đó, chúng ta có thể dễ dàng sử dụng các hàm khác
từ `std::env`. Nó cũng ít mơ hồ hơn so với việc thêm `use std::env::args` và
sau đó gọi hàm chỉ bằng `args`, bởi vì `args` có thể dễ dàng bị
nhầm lẫn với một hàm được định nghĩa trong module hiện tại.

> ### Hàm `args` và Unicode Không Hợp Lệ
>
> Lưu ý rằng `std::env::args` sẽ gây ra panic nếu bất kỳ đối số nào chứa
> Unicode không hợp lệ. Nếu chương trình của bạn cần chấp nhận các đối số chứa Unicode
> không hợp lệ, hãy sử dụng `std::env::args_os` thay thế. Hàm đó trả về một trình lặp
> tạo ra các giá trị `OsString` thay vì các giá trị `String`. Chúng tôi đã chọn
> sử dụng `std::env::args` ở đây để đơn giản vì các giá trị `OsString` khác nhau theo từng
> nền tảng và phức tạp để làm việc hơn các giá trị `String`.

Trên dòng đầu tiên của `main`, chúng ta gọi `env::args`, và chúng ta ngay lập tức sử dụng
`collect` để chuyển trình lặp thành một vector chứa tất cả các giá trị được tạo ra
bởi trình lặp. Chúng ta có thể sử dụng hàm `collect` để tạo ra nhiều loại
bộ sưu tập, vì vậy chúng ta chú thích kiểu của `args` một cách rõ ràng để chỉ định rằng chúng ta
muốn một vector chứa các chuỗi. Mặc dù bạn rất hiếm khi cần chú thích kiểu trong Rust,
`collect` là một hàm mà bạn thường xuyên cần chú thích vì Rust
không thể suy luận loại bộ sưu tập mà bạn muốn.

Cuối cùng, chúng ta in vector bằng macro debug. Hãy thử chạy mã
trước tiên không có đối số và sau đó với hai đối số:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Lưu ý rằng giá trị đầu tiên trong vector là `"target/debug/minigrep"`, đó
là tên tệp nhị phân của chúng ta. Điều này khớp với hành vi của danh sách đối số trong
C, cho phép các chương trình sử dụng tên mà chúng được gọi trong quá trình thực thi.
Nó thường thuận tiện để có quyền truy cập vào tên chương trình trong trường hợp bạn muốn
in nó trong các thông báo hoặc thay đổi hành vi của chương trình dựa trên việc
bí danh dòng lệnh nào đã được sử dụng để gọi chương trình. Nhưng đối với mục đích của
chương này, chúng ta sẽ bỏ qua nó và chỉ lưu hai đối số chúng ta cần.

### Lưu Các Giá Trị Đối Số Trong Các Biến

Chương trình hiện có thể truy cập các giá trị được chỉ định làm đối số dòng
lệnh. Bây giờ chúng ta cần lưu giá trị của hai đối số vào các biến để
chúng ta có thể sử dụng các giá trị đó trong phần còn lại của chương trình. Chúng ta thực hiện điều đó trong Liệt kê
12-2.

<Listing number="12-2" file-name="src/main.rs" caption="Tạo các biến để giữ đối số truy vấn và đối số đường dẫn tệp">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Như chúng ta đã thấy khi in vector, tên của chương trình chiếm giá trị
đầu tiên trong vector tại `args[0]`, vì vậy chúng ta bắt đầu các đối số tại chỉ số 1.
Đối số đầu tiên `minigrep` nhận là chuỗi chúng ta đang tìm kiếm, vì vậy chúng ta đặt một
tham chiếu đến đối số đầu tiên vào biến `query`. Đối số thứ hai
sẽ là đường dẫn tệp, vì vậy chúng ta đặt một tham chiếu đến đối số thứ hai vào
biến `file_path`.

Chúng ta tạm thời in giá trị của các biến này để chứng minh rằng mã đang
hoạt động như chúng ta mong muốn. Hãy chạy lại chương trình này với các đối số `test`
và `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Tuyệt vời, chương trình đang hoạt động! Giá trị của các đối số chúng ta cần đang
được lưu vào đúng các biến. Sau này, chúng ta sẽ thêm một số xử lý lỗi để giải quyết
một số tình huống sai sót tiềm ẩn, chẳng hạn như khi người dùng không cung cấp
đối số; bây giờ, chúng ta sẽ bỏ qua tình huống đó và tập trung vào việc thêm khả năng
đọc tệp.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
