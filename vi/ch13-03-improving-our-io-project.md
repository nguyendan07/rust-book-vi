## Cải thiện dự án I/O của chúng ta

Với kiến thức mới này về các iterator, chúng ta có thể cải thiện dự án I/O trong
Chương 12 bằng cách sử dụng các iterator để làm cho các vị trí trong mã rõ ràng và súc tích
hơn. Hãy xem cách các iterator có thể cải thiện việc triển khai hàm
`Config::build` và hàm `search` của chúng ta.

### Loại bỏ một `clone` bằng cách sử dụng một Iterator

Trong Danh sách 12-6, chúng ta đã thêm mã lấy một slice của các giá trị `String` và tạo ra
một instance của struct `Config` bằng cách truy cập vào slice bằng chỉ số và clone các
giá trị, cho phép struct `Config` sở hữu những giá trị đó. Trong Danh sách 13-17,
chúng ta đã tái hiện lại việc triển khai hàm `Config::build` như nó đã có
trong Danh sách 12-23.

<Listing number="13-17" file-name="src/lib.rs" caption="Tái hiện hàm `Config::build` từ Danh sách 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/lib.rs:ch13}}
```

</Listing>

Vào thời điểm đó, chúng tôi đã nói đừng lo lắng về các lời gọi `clone` không hiệu quả vì
chúng ta sẽ loại bỏ chúng trong tương lai. Vâng, thời điểm đó chính là bây giờ!

Chúng ta cần `clone` ở đây vì chúng ta có một slice với các phần tử `String` trong
tham số `args`, nhưng hàm `build` không sở hữu `args`. Để trả về
quyền sở hữu của một instance `Config`, chúng ta phải clone các giá trị từ các trường `query`
và `file_path` của `Config` để instance `Config` có thể sở hữu các giá trị của nó.

Với kiến thức mới về các iterator, chúng ta có thể thay đổi hàm `build` để
lấy quyền sở hữu của một iterator làm đối số thay vì mượn một slice.
Chúng ta sẽ sử dụng chức năng của iterator thay vì mã kiểm tra độ dài
của slice và truy cập vào các vị trí cụ thể. Điều này sẽ làm rõ những gì
hàm `Config::build` đang làm vì iterator sẽ truy cập các giá trị.

Một khi `Config::build` lấy quyền sở hữu của iterator và ngừng sử dụng các thao tác lập chỉ mục
có tính chất mượn, chúng ta có thể di chuyển các giá trị `String` từ iterator vào
`Config` thay vì gọi `clone` và tạo một phân bổ (allocation) mới.

#### Sử dụng trực tiếp Iterator được trả về

Mở tệp _src/main.rs_ trong dự án I/O của bạn, nó sẽ trông như thế này:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Đầu tiên chúng ta sẽ thay đổi phần bắt đầu của hàm `main` mà chúng ta đã có trong Danh sách
12-24 thành mã trong Danh sách 13-18, lần này sử dụng một iterator. Điều này
sẽ không biên dịch được cho đến khi chúng ta cập nhật cả `Config::build`.

<Listing number="13-18" file-name="src/main.rs" caption="Truyền giá trị trả về của `env::args` cho `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

Hàm `env::args` trả về một iterator! Thay vì thu thập các giá trị của
iterator vào một vector và sau đó truyền một slice cho `Config::build`, bây giờ
chúng ta đang truyền quyền sở hữu của iterator được trả về từ `env::args` trực tiếp
cho `Config::build`.

Tiếp theo, chúng ta cần cập nhật định nghĩa của `Config::build`. Trong tệp
_src/lib.rs_ của dự án I/O, hãy thay đổi chữ ký của `Config::build` thành
như Danh sách 13-19. Điều này vẫn sẽ không biên dịch được vì chúng ta cần cập nhật
phần thân hàm.

<Listing number="13-19" file-name="src/lib.rs" caption="Cập nhật chữ ký của `Config::build` để mong đợi một iterator">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/lib.rs:here}}
```

</Listing>

Tài liệu thư viện chuẩn cho hàm `env::args` cho thấy kiểu của iterator
mà nó trả về là `std::env::Args`, và kiểu đó triển khai trait `Iterator`
và trả về các giá trị `String`.

Chúng ta đã cập nhật chữ ký của hàm `Config::build` để tham số
`args` có một kiểu generic với các ràng buộc trait `impl Iterator<Item = String>`
thay vì `&[String]`. Việc sử dụng cú pháp `impl Trait` này mà chúng ta đã thảo luận trong
phần [“Trait làm tham số”][impl-trait]<!-- ignore --> của Chương 10
có nghĩa là `args` có thể là bất kỳ kiểu nào triển khai trait `Iterator` và
trả về các mục `String`.

Bởi vì chúng ta đang lấy quyền sở hữu của `args` và chúng ta sẽ thay đổi `args` bằng cách
lặp qua nó, chúng ta có thể thêm từ khóa `mut` vào đặc tả của
tham số `args` để làm cho nó có khả năng thay đổi.

#### Sử dụng các phương thức của Trait `Iterator` thay vì lập chỉ mục

Tiếp theo, chúng ta sẽ sửa phần thân của `Config::build`. Bởi vì `args` triển khai
trait `Iterator`, chúng ta biết mình có thể gọi phương thức `next` trên nó! Danh sách 13-20
cập nhật mã từ Danh sách 12-23 để sử dụng phương thức `next`.

<Listing number="13-20" file-name="src/lib.rs" caption="Thay đổi phần thân của `Config::build` để sử dụng các phương thức iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/lib.rs:here}}
```

</Listing>

Hãy nhớ rằng giá trị đầu tiên trong giá trị trả về của `env::args` là tên của
chương trình. Chúng ta muốn bỏ qua giá trị đó và chuyển đến giá trị tiếp theo, vì vậy đầu tiên chúng ta gọi
`next` và không làm gì với giá trị trả về. Sau đó chúng ta gọi `next` để lấy
giá trị chúng ta muốn đưa vào trường `query` của `Config`. Nếu `next` trả về `Some`,
chúng ta sử dụng một `match` để trích xuất giá trị. Nếu nó trả về `None`, điều đó có nghĩa là không có đủ
đối số được đưa vào và chúng ta trả về sớm với một giá trị `Err`. Chúng ta thực hiện điều tương tự
cho giá trị `file_path`.

### Làm cho mã rõ ràng hơn với các Adapter Iterator

Chúng ta cũng có thể tận dụng các iterator trong hàm `search` trong dự án I/O
của mình, được tái hiện ở đây trong Danh sách 13-21 như nó đã có trong Danh sách 12-19:

<Listing number="13-21" file-name="src/lib.rs" caption="Việc triển khai hàm `search` từ Danh sách 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Chúng ta có thể viết mã này theo cách súc tích hơn bằng cách sử dụng các phương thức adapter iterator.
Làm như vậy cũng giúp chúng ta tránh việc có một vector trung gian `results` có khả năng thay đổi. Phong cách
lập trình hàm thích giảm thiểu lượng trạng thái có khả năng thay đổi để
làm cho mã rõ ràng hơn. Việc loại bỏ trạng thái có khả năng thay đổi có thể cho phép một cải tiến trong tương lai
để việc tìm kiếm diễn ra song song, bởi vì chúng ta sẽ không phải quản lý
việc truy cập đồng thời vào vector `results`. Danh sách 13-22 cho thấy sự thay đổi này:

<Listing number="13-22" file-name="src/lib.rs" caption="Sử dụng các phương thức adapter iterator trong việc triển khai hàm `search`">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

Nhớ lại rằng mục đích của hàm `search` là trả về tất cả các dòng trong
`contents` có chứa `query`. Tương tự như ví dụ `filter` trong Danh sách
13-16, mã này sử dụng adapter `filter` để chỉ giữ lại những dòng mà
`line.contains(query)` trả về `true`. Sau đó chúng ta thu thập các dòng khớp
vào một vector khác bằng `collect`. Đơn giản hơn nhiều! Hãy thoải mái thực hiện
thay đổi tương tự để sử dụng các phương thức iterator trong hàm `search_case_insensitive`
nếu muốn.

### Lựa chọn giữa Vòng lặp hoặc Iterator

Câu hỏi logic tiếp theo là bạn nên chọn phong cách nào trong mã của riêng mình và
tại sao: triển khai ban đầu trong Danh sách 13-21 hay phiên bản sử dụng
iterator trong Danh sách 13-22. Hầu hết các lập trình viên Rust thích sử dụng
phong cách iterator. Lúc đầu có thể hơi khó để làm quen, nhưng khi bạn đã hiểu về
các adapter iterator khác nhau và những gì chúng làm, iterator có thể dễ hiểu
hơn. Thay vì loay hoay với các phần khác nhau của việc lặp và xây dựng
các vector mới, mã tập trung vào mục tiêu cấp cao của vòng lặp. Điều này
trừu tượng hóa một số mã thông thường để dễ dàng nhìn thấy các khái niệm
duy nhất đối với mã này, chẳng hạn như điều kiện lọc mà mỗi phần tử trong
iterator phải vượt qua.

Nhưng hai triển khai có thực sự tương đương không? Giả định trực quan có thể
là vòng lặp cấp thấp hơn sẽ nhanh hơn. Hãy cùng nói về hiệu suất.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
