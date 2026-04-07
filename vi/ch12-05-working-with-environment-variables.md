## Làm việc với các Biến môi trường

Chúng ta sẽ cải thiện `minigrep` bằng cách thêm một tính năng bổ sung: một tùy chọn cho
việc tìm kiếm không phân biệt hoa thường (case-insensitive) mà người dùng có thể bật thông qua một biến
môi trường (environment variable). Chúng ta có thể biến tính năng này thành một tùy chọn dòng lệnh và yêu cầu
người dùng nhập nó mỗi khi họ muốn áp dụng, nhưng thay vào đó, bằng cách biến nó thành một
biến môi trường, chúng ta cho phép người dùng thiết lập biến môi trường một lần
và tất cả các tìm kiếm của họ sẽ không phân biệt hoa thường trong phiên làm việc đó của terminal.

### Viết một Kiểm thử Thất bại cho Hàm `search_case_insensitive`

Đầu tiên, chúng ta thêm một hàm `search_case_insensitive` mới, hàm này sẽ được gọi khi
biến môi trường có giá trị. Chúng ta sẽ tiếp tục tuân theo quy trình TDD,
vì vậy bước đầu tiên một lần nữa là viết một bài kiểm thử thất bại. Chúng ta sẽ thêm một bài kiểm thử mới cho
hàm `search_case_insensitive` mới và đổi tên bài kiểm thử cũ từ
`one_result` thành `case_sensitive` để làm rõ sự khác biệt giữa hai
bài kiểm thử, như được hiển thị trong Liệt kê 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Thêm một bài kiểm thử thất bại mới cho hàm không phân biệt hoa thường mà chúng ta sắp thêm vào">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cũng đã chỉnh sửa `contents` của bài kiểm thử cũ. Chúng ta đã thêm một dòng mới
với văn bản `"Duct tape."` sử dụng chữ _D_ viết hoa, dòng này sẽ không khớp với truy vấn
`"duct"` khi chúng ta tìm kiếm theo cách có phân biệt hoa thường (case-sensitive). Việc thay đổi bài kiểm thử cũ
theo cách này giúp đảm bảo rằng chúng ta không vô tình làm hỏng chức năng tìm kiếm
có phân biệt hoa thường mà chúng ta đã triển khai. Bài kiểm thử này bây giờ sẽ vượt qua
và sẽ tiếp tục vượt qua khi chúng ta làm việc trên tính năng tìm kiếm không phân biệt hoa thường.

Bài kiểm thử mới cho việc tìm kiếm _không_ phân biệt hoa thường sử dụng `"rUsT"` làm truy vấn của nó. Trong
hàm `search_case_insensitive` mà chúng ta sắp thêm vào, truy vấn `"rUsT"`
nên khớp với dòng chứa `"Rust:"` với chữ _R_ viết hoa và khớp với
dòng `"Trust me."` mặc dù cả hai đều có cách viết hoa khác với truy vấn. Đây
là bài kiểm thử thất bại của chúng ta, và nó sẽ không thể biên dịch được vì chúng ta chưa định nghĩa
hàm `search_case_insensitive`. Bạn có thể thêm một triển khai khung (skeleton)
luôn trả về một vector trống, tương tự như cách chúng ta đã làm cho
hàm `search` trong Liệt kê 12-16 để thấy bài kiểm thử được biên dịch và thất bại.

### Triển khai Hàm `search_case_insensitive`

Hàm `search_case_insensitive`, được hiển thị trong Liệt kê 12-21, sẽ gần như
giống với hàm `search`. Sự khác biệt duy nhất là chúng ta sẽ chuyển `query` và mỗi
`line` thành chữ thường để bất kể kiểu chữ của các đối số đầu vào là gì,
chúng sẽ có cùng kiểu chữ khi chúng ta kiểm tra xem dòng đó có chứa truy vấn hay không.

<Listing number="12-21" file-name="src/lib.rs" caption="Định nghĩa hàm `search_case_insensitive` để chuyển truy vấn và dòng thành chữ thường trước khi so sánh chúng">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Đầu tiên, chúng ta chuyển chuỗi `query` thành chữ thường và lưu trữ nó trong một biến mới có
cùng tên, che bóng (shadowing) `query` ban đầu. Việc gọi `to_lowercase` trên truy vấn
là cần thiết để bất kể truy vấn của người dùng là `"rust"`, `"RUST"`,
`"Rust"`, hay `"rUsT"`, chúng ta sẽ coi truy vấn như thể nó là `"rust"` và
không phân biệt hoa thường. Mặc dù `to_lowercase` sẽ xử lý Unicode cơ bản, nhưng nó
sẽ không chính xác 100%. Nếu chúng ta đang viết một ứng dụng thực tế, chúng ta sẽ muốn thực hiện
nhiều công việc hơn ở đây, nhưng phần này là về các biến môi trường, không phải
Unicode, vì vậy chúng ta sẽ dừng lại ở đó.

Lưu ý rằng `query` bây giờ là một `String` thay vì một lát cắt chuỗi (string slice) vì việc gọi
`to_lowercase` tạo ra dữ liệu mới thay vì tham chiếu đến dữ liệu hiện có. Ví dụ, giả sử
truy vấn là `"rUsT"`: lát cắt chuỗi đó không chứa chữ `u` hoặc `t` viết thường
để chúng ta sử dụng, vì vậy chúng ta phải cấp phát một `String` mới chứa
`"rust"`. Khi chúng ta truyền `query` làm đối số cho phương thức `contains` bây giờ, chúng ta
cần thêm một dấu và (`&`) vì chữ ký của `contains` được định nghĩa để nhận
một lát cắt chuỗi.

Tiếp theo, chúng ta thêm một lời gọi đến `to_lowercase` trên mỗi `line` để chuyển tất cả
các ký tự thành chữ thường. Bây giờ chúng ta đã chuyển đổi `line` và `query` thành chữ thường, chúng ta sẽ
tìm thấy các kết quả khớp bất kể kiểu chữ của truy vấn là gì.

Hãy xem liệu triển khai này có vượt qua các bài kiểm thử không:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Tuyệt vời! Chúng đã vượt qua. Bây giờ, hãy gọi hàm `search_case_insensitive` mới
từ hàm `run`. Đầu tiên, chúng ta sẽ thêm một tùy chọn cấu hình vào struct `Config`
để chuyển đổi giữa tìm kiếm có phân biệt hoa thường và không phân biệt hoa thường. Việc thêm
trường này sẽ gây ra lỗi trình biên dịch vì chúng ta chưa khởi tạo trường này
ở bất kỳ đâu:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:here}}
```

Chúng ta đã thêm trường `ignore_case` để giữ một giá trị Boolean. Tiếp theo, chúng ta cần hàm `run`
kiểm tra giá trị của trường `ignore_case` và sử dụng giá trị đó để quyết định
nên gọi hàm `search` hay hàm `search_case_insensitive`,
như được hiển thị trong Liệt kê 12-22. Đoạn mã này vẫn chưa thể biên dịch được.

<Listing number="12-22" file-name="src/lib.rs" caption="Gọi `search` hoặc `search_case_insensitive` dựa trên giá trị trong `config.ignore_case` house">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:there}}
```

</Listing>

Cuối cùng, chúng ta cần kiểm tra biến môi trường. Các hàm để làm việc với
biến môi trường nằm trong module `env` trong thư viện tiêu chuẩn (standard library), vì vậy chúng ta đưa
module đó vào phạm vi (scope) ở đầu tệp _src/lib.rs_. Sau đó, chúng ta sẽ sử dụng hàm `var`
từ module `env` để kiểm tra xem có bất kỳ giá trị nào được thiết lập cho một biến
môi trường có tên là `IGNORE_CASE` hay không, như được hiển thị trong Liệt kê 12-23.

<Listing number="12-23" file-name="src/lib.rs" caption="Kiểm tra bất kỳ giá trị nào trong một biến môi trường có tên là `IGNORE_CASE` house">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/lib.rs:here}}
```

</Listing>

Ở đây, chúng ta tạo một biến mới, `ignore_case`. Để thiết lập giá trị của nó, chúng ta gọi hàm
`env::var` và truyền cho nó tên của biến môi trường `IGNORE_CASE`. Hàm
`env::var` trả về một `Result`, nó sẽ là biến thể `Ok` thành công
chứa giá trị của biến môi trường nếu biến môi trường đó
được thiết lập với bất kỳ giá trị nào. Nó sẽ trả về biến thể `Err` nếu biến môi trường không được thiết lập.

Chúng ta đang sử dụng phương thức `is_ok` trên `Result` để kiểm tra xem biến môi trường
có được thiết lập hay không, điều đó có nghĩa là chương trình nên thực hiện tìm kiếm không phân biệt hoa thường.
Nếu biến môi trường `IGNORE_CASE` không được thiết lập cho bất kỳ thứ gì, `is_ok` sẽ
trả về `false` và chương trình sẽ thực hiện tìm kiếm có phân biệt hoa thường. Chúng ta không
quan tâm đến _giá trị_ của biến môi trường, chỉ cần biết nó được thiết lập hay chưa được thiết lập,
vì vậy chúng ta đang kiểm tra `is_ok` thay vì sử dụng `unwrap`, `expect`, hoặc bất kỳ
phương thức nào khác mà chúng ta đã thấy trên `Result`.

Chúng ta truyền giá trị trong biến `ignore_case` cho instance `Config` để hàm
`run` có thể đọc giá trị đó và quyết định nên gọi
`search_case_insensitive` hay `search`, như chúng ta đã triển khai trong Liệt kê 12-22.

Hãy thử một lần xem sao! Đầu tiên, chúng ta sẽ chạy chương trình của mình mà không thiết lập biến môi trường
và với truy vấn `to`, truy vấn này sẽ khớp với bất kỳ dòng nào chứa từ
_to_ ở dạng chữ thường:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Có vẻ như nó vẫn hoạt động! Bây giờ hãy chạy chương trình với `IGNORE_CASE` được thiết lập
thành `1` nhưng với cùng truy vấn _to_:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Nếu bạn đang sử dụng PowerShell, bạn sẽ cần thiết lập biến môi trường và
chạy chương trình dưới dạng các lệnh riêng biệt:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Điều này sẽ làm cho `IGNORE_CASE` tồn tại trong phần còn lại của phiên làm việc shell của bạn.
Nó có thể được hủy bỏ bằng cmdlet `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Chúng ta sẽ nhận được các dòng chứa _to_ có thể có các chữ cái viết hoa:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Tuyệt vời, chúng ta cũng nhận được các dòng chứa _To_! Chương trình `minigrep` của chúng ta bây giờ có thể
thực hiện tìm kiếm không phân biệt hoa thường được điều khiển bởi một biến môi trường. Bây giờ bạn đã biết
cách quản lý các tùy chọn được thiết lập bằng cách sử dụng các đối số dòng lệnh hoặc các biến môi trường.

Một số chương trình cho phép các đối số _và_ các biến môi trường cho cùng một
cấu hình. Trong những trường hợp đó, các chương trình quyết định rằng cái này hoặc cái kia sẽ được ưu tiên.
Để thực hiện một bài tập khác cho riêng mình, hãy thử kiểm soát tính phân biệt hoa thường
thông qua đối số dòng lệnh hoặc biến môi trường. Hãy quyết định
xem đối số dòng lệnh hay biến môi trường nên được ưu tiên
nếu chương trình được chạy với một cái được thiết lập là có phân biệt hoa thường và một cái được thiết lập là
bỏ qua kiểu chữ.

Module `std::env` chứa nhiều tính năng hữu ích hơn để xử lý các
biến môi trường: hãy xem tài liệu của nó để biết những gì có sẵn.
