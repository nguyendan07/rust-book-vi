## Phát triển Chức năng của Thư viện bằng Phát triển Hướng Kiểm thử

Bây giờ chúng ta đã trích xuất logic vào _src/lib.rs_ và để lại việc thu thập
đối số cũng như xử lý lỗi trong _src/main.rs_, việc viết các bài kiểm thử cho
chức năng cốt lõi của mã nguồn trở nên dễ dàng hơn nhiều. Chúng ta có thể gọi
các hàm trực tiếp với các đối số khác nhau và kiểm tra các giá trị trả về mà
không cần phải gọi chương trình binary của mình từ dòng lệnh.

Trong phần này, chúng ta sẽ thêm logic tìm kiếm vào chương trình `minigrep` bằng
quy trình phát triển hướng kiểm thử (test-driven development - TDD) với các bước
sau:

1. Viết một bài kiểm thử thất bại và chạy nó để đảm bảo nó thất bại vì lý do bạn
   mong đợi.
2. Viết hoặc sửa đổi vừa đủ mã để làm cho bài kiểm thử mới vượt qua.
3. Tái cấu trúc (Refactor) mã bạn vừa thêm hoặc thay đổi và đảm bảo các bài
   kiểm thử tiếp tục vượt qua.
4. Lặp lại từ bước 1!

Mặc dù đây chỉ là một trong nhiều cách để viết phần mềm, TDD có thể giúp định
hướng thiết kế mã. Việc viết bài kiểm thử trước khi viết mã giúp bài kiểm thử
vượt qua sẽ giúp duy trì độ bao phủ kiểm thử cao trong suốt quá trình.

Chúng ta sẽ thực hiện TDD cho việc triển khai chức năng thực sự thực hiện việc
tìm kiếm chuỗi truy vấn trong nội dung tệp và tạo ra danh sách các dòng
khớp với truy vấn đó. Chúng ta sẽ thêm chức năng này vào một hàm gọi là
`search`.

### Viết một Kiểm thử Thất bại

Vì chúng ta không cần chúng nữa, hãy xóa các câu lệnh `println!` khỏi
_src/lib.rs_ và _src/main.rs_ mà chúng ta đã sử dụng để kiểm tra hành vi của
chương trình. Sau đó, trong _src/lib.rs_, chúng ta sẽ thêm một module `tests`
với một hàm kiểm thử, như chúng ta đã làm trong [Chương 11][ch11-anatomy]<!-- ignore -->.
Hàm kiểm thử chỉ định hành vi mà chúng ta muốn hàm `search` có: nó sẽ nhận một
truy vấn và văn bản cần tìm kiếm, và nó sẽ chỉ trả về những dòng từ văn bản
có chứa truy vấn đó. Liệt kê 12-15 hiển thị bài kiểm thử này, cái mà sẽ chưa
thể biên dịch được.

<Listing number="12-15" file-name="src/lib.rs" caption="Tạo một kiểm thử thất bại cho hàm `search` mà chúng ta mong muốn có">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Bài kiểm thử này tìm kiếm chuỗi `"duct"`. Văn bản chúng ta đang tìm kiếm gồm ba
dòng, chỉ một trong số đó chứa `"duct"` (lưu ý rằng dấu gạch chéo ngược sau
dấu ngoặc kép mở nói với Rust không đặt ký tự dòng mới ở đầu nội dung của
hằng chuỗi này). Chúng ta khẳng định rằng giá trị trả về từ hàm `search`
chứa duy nhất dòng mà chúng ta mong đợi.

Chúng ta vẫn chưa thể chạy bài kiểm thử này và xem nó thất bại vì bài kiểm thử
thậm chí còn không biên dịch được: hàm `search` chưa tồn tại! Theo các nguyên
tắc TDD, chúng ta sẽ thêm vừa đủ mã để bài kiểm thử có thể biên dịch và chạy
bằng cách thêm một định nghĩa của hàm `search` luôn trả về một vector trống,
như được hiển thị trong Liệt kê 12-16. Sau đó, bài kiểm thử sẽ biên dịch và
thất bại vì một vector trống không khớp với một vector chứa dòng `"safe,
fast, productive."`

<Listing number="12-16" file-name="src/lib.rs" caption="Định nghĩa vừa đủ hàm `search` để bài kiểm thử của chúng ta có thể biên dịch">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cần định nghĩa một lifetime `'a` rõ ràng trong chữ ký của
`search` và sử dụng lifetime đó với đối số `contents` và giá trị trả về. Nhớ
lại trong [Chương 10][ch10-lifetimes]<!-- ignore --> rằng các tham số lifetime
chỉ định lifetime của đối số nào được kết nối với lifetime của giá trị trả về.
Trong trường hợp này, chúng ta chỉ ra rằng vector được trả về nên chứa các lát
cắt chuỗi tham chiếu đến các lát cắt của đối số `contents` (thay vì đối số
`query`).

Nói cách khác, chúng ta cho Rust biết rằng dữ liệu được trả về bởi hàm `search`
sẽ sống lâu bằng dữ liệu được truyền vào hàm `search` trong đối số `contents`.
Điều này rất quan trọng! Dữ liệu được tham chiếu _bởi_ một lát cắt cần phải
hợp lệ để tham chiếu đó hợp lệ; nếu trình biên dịch giả định chúng ta đang tạo
các lát cắt chuỗi từ `query` thay vì `contents`, nó sẽ thực hiện kiểm tra an
toàn không chính xác.

Nếu chúng ta quên các chú thích lifetime và cố gắng biên dịch hàm này, chúng ta
sẽ nhận được lỗi này:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust không thể biết chúng ta cần cái nào trong hai đối số, vì vậy chúng ta cần
nói cho nó một cách rõ ràng. Bởi vì `contents` là đối số chứa tất cả văn bản
của chúng ta và chúng ta muốn trả về các phần của văn bản đó khớp với nhau,
chúng ta biết `contents` là đối số nên được kết nối với giá trị trả về bằng
cú pháp lifetime.

Các ngôn ngữ lập trình khác không yêu cầu bạn kết nối các đối số với các giá trị
trả về trong chữ ký, nhưng việc thực hành này sẽ trở nên dễ dàng hơn theo thời
gian. Bạn có thể muốn so sánh ví dụ này với các ví dụ trong phần [“Xác thực
tham chiếu với Lifetimes”][validating-references-with-lifetimes]<!-- ignore -->
trong Chương 10.

Bây giờ hãy chạy bài kiểm thử:

```console
{{#include ../listings/ch12-an-io-project/listing-12-16/output.txt}}
```

Tuyệt vời, bài kiểm thử thất bại, đúng như chúng ta mong đợi. Hãy làm cho bài
kiểm thử vượt qua nào!

### Viết Mã để Vượt qua Bài Kiểm thử

Hiện tại, bài kiểm thử của chúng ta đang thất bại vì chúng ta luôn trả về một
vector trống. Để khắc phục điều đó và triển khai `search`, chương trình của
chúng ta cần thực hiện các bước sau:

1. Lặp qua từng dòng của nội dung.
2. Kiểm tra xem dòng đó có chứa chuỗi truy vấn của chúng ta hay không.
3. Nếu có, hãy thêm nó vào danh sách các giá trị chúng ta sẽ trả về.
4. Nếu không, không làm gì cả.
5. Trả về danh sách các kết quả khớp.

Hãy thực hiện từng bước, bắt đầu với việc lặp qua các dòng.

#### Lặp qua các Dòng với Phương thức `lines`

Rust có một phương thức hữu ích để xử lý việc lặp qua từng dòng của chuỗi,
được đặt tên một cách thuận tiện là `lines`, hoạt động như được hiển thị trong
Liệt kê 12-17. Lưu ý rằng đoạn mã này vẫn chưa thể biên dịch được.

<Listing number="12-17" file-name="src/lib.rs" caption="Lặp qua từng dòng trong `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

Phương thức `lines` trả về một trình lặp. Chúng ta sẽ nói chi tiết về các trình
lặp trong [Chương 13][ch13-iterators]<!-- ignore -->, nhưng hãy nhớ lại rằng
bạn đã thấy cách sử dụng trình lặp này trong [Liệt kê 3-5][ch3-iter]<!-- ignore -->,
nơi chúng ta đã sử dụng vòng lặp `for` với một trình lặp để chạy một số mã trên
mỗi mục trong một bộ sưu tập.

#### Tìm kiếm Query trong mỗi Dòng

Tiếp theo, chúng ta sẽ kiểm tra xem dòng hiện tại có chứa chuỗi truy vấn của
chúng ta hay không. May mắn thay, các chuỗi có một phương thức hữu ích tên là
`contains` để thực hiện việc này cho chúng ta! Thêm một lời gọi đến phương thức
`contains` trong hàm `search`, như được hiển thị trong Liệt kê 12-18. Lưu ý
rằng đoạn mã này vẫn chưa thể biên dịch được.

<Listing number="12-18" file-name="src/lib.rs" caption="Thêm chức năng để xem liệu dòng đó có chứa chuỗi trong `query` hay không">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

Tại thời điểm này, chúng ta đang xây dựng chức năng. Để mã có thể biên dịch,
chúng ta cần trả về một giá trị từ thân hàm như chúng ta đã chỉ ra trong chữ ký
hàm.

#### Lưu trữ các Dòng Khớp

Để hoàn thành hàm này, chúng ta cần một cách để lưu trữ các dòng khớp mà chúng
ta muốn trả về. Để làm được điều đó, chúng ta có thể tạo một vector có thể thay
đổi trước vòng lặp `for` và gọi phương thức `push` để lưu trữ một `line` vào
vector đó. Sau vòng lặp `for`, chúng ta trả về vector đó, như được hiển thị
trong Liệt kê 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Lưu trữ các dòng khớp để chúng ta có thể trả về chúng">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Bây giờ hàm `search` sẽ chỉ trả về các dòng có chứa `query`, và bài kiểm thử
của chúng ta sẽ vượt qua. Hãy chạy bài kiểm thử:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Bài kiểm thử của chúng ta đã vượt qua, vì vậy chúng ta biết nó hoạt động!

Tại thời điểm này, chúng ta có thể xem xét các cơ hội để tái cấu trúc việc
triển khai hàm search trong khi vẫn giữ cho các bài kiểm thử vượt qua để duy
trì cùng một chức năng. Mã trong hàm search không quá tệ, nhưng nó không tận
dụng được một số tính năng hữu ích của các trình lặp. Chúng ta sẽ quay lại ví
dụ này trong [Chương 13][ch13-iterators]<!-- ignore -->, nơi chúng ta sẽ khám
phá chi tiết về các trình lặp và xem cách cải thiện nó.

#### Sử dụng Hàm `search` trong Hàm `run`

Bây giờ hàm `search` đã hoạt động và được kiểm thử, chúng ta cần gọi `search`
từ hàm `run` của mình. Chúng ta cần truyền giá trị `config.query` và `contents`
mà `run` đọc được từ tệp vào hàm `search`. Sau đó, `run` sẽ in từng dòng được
trả về từ `search`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/src/lib.rs:here}}
```

Chúng ta vẫn đang sử dụng vòng lặp `for` để lấy từng dòng từ `search` và in nó.

Bây giờ toàn bộ chương trình sẽ hoạt động! Hãy thử nó, trước tiên với một từ
sẽ trả về chính xác một dòng từ bài thơ của Emily Dickinson: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Tuyệt! Bây giờ hãy thử một từ sẽ khớp với nhiều dòng, như _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Và cuối cùng, hãy đảm bảo rằng chúng ta không nhận được bất kỳ dòng nào khi tìm
kiếm một từ không có ở bất kỳ đâu trong bài thơ, chẳng hạn như _monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Xuất sắc! Chúng ta đã xây dựng phiên bản mini của riêng mình cho một công cụ
cổ điển và học được rất nhiều về cách cấu trúc các ứng dụng. Chúng ta cũng đã
học được một chút về nhập và xuất tệp, lifetimes, kiểm thử và phân tích dòng
lệnh.

Để hoàn thiện dự án này, chúng ta sẽ trình bày ngắn gọn cách làm việc với các
biến môi trường và cách in ra lỗi tiêu chuẩn, cả hai đều hữu ích khi bạn viết
các chương trình dòng lệnh.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
