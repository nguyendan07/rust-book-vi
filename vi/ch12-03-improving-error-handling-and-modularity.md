## Tái Cấu Trúc để Cải Thiện Tính Module và Xử Lý Lỗi

Để cải thiện chương trình của mình, chúng ta sẽ khắc phục bốn vấn đề liên quan đến
cấu trúc của chương trình và cách nó xử lý các lỗi tiềm ẩn. Đầu tiên, hàm `main`
của chúng ta hiện thực hiện hai nhiệm vụ: phân tích cú pháp các đối số và đọc các tệp. Khi
chương trình của chúng ta phát triển, số lượng các nhiệm vụ riêng biệt mà hàm `main` xử lý sẽ
tăng lên. Khi một hàm đảm nhận thêm nhiều trách nhiệm, nó trở nên khó suy luận hơn,
khó kiểm thử hơn và khó thay đổi hơn mà không làm hỏng một trong các
phần của nó. Tốt nhất là nên tách biệt chức năng để mỗi hàm chịu trách nhiệm cho
một nhiệm vụ.

Vấn đề này cũng liên quan đến vấn đề thứ hai: mặc dù `query` và `file_path`
là các biến cấu hình cho chương trình của chúng ta, các biến như `contents` lại được sử dụng
để thực hiện logic của chương trình. `main` càng dài, chúng ta càng cần đưa nhiều biến
vào phạm vi; chúng ta càng có nhiều biến trong phạm vi, thì càng khó để
theo dõi mục đích của từng biến. Tốt nhất là nhóm các
biến cấu hình vào một cấu trúc để làm rõ mục đích của chúng.

Vấn đề thứ ba là chúng ta đã sử dụng `expect` để in một thông báo lỗi khi
việc đọc tệp thất bại, nhưng thông báo lỗi chỉ in ra `Should have been
able to read the file`. Việc đọc một tệp có thể thất bại theo nhiều cách: ví
dụ, tệp có thể bị thiếu, hoặc chúng ta có thể không có quyền để mở nó.
Hiện tại, bất kể tình huống nào, chúng ta đều in cùng một thông báo lỗi cho
mọi thứ, điều này sẽ không cung cấp cho người dùng bất kỳ thông tin nào!

Thứ tư, chúng ta sử dụng `expect` để xử lý một lỗi, và nếu người dùng chạy chương trình của chúng ta
mà không chỉ định đủ các đối số, họ sẽ nhận được một lỗi `index out of bounds`
từ Rust mà không giải thích rõ ràng vấn đề. Sẽ tốt nhất nếu tất cả các
mã xử lý lỗi nằm ở một nơi để những người bảo trì trong tương lai chỉ cần một nơi
để tham khảo mã nếu logic xử lý lỗi cần thay đổi. Việc có tất cả các
mã xử lý lỗi ở một nơi cũng sẽ đảm bảo rằng chúng ta đang in các thông báo
có ý nghĩa đối với người dùng cuối của mình.

Hãy giải quyết bốn vấn đề này bằng cách tái cấu trúc (refactoring) dự án của chúng ta.

### Phân Tách các Mối Bận Tâm cho Các Dự Án Nhị Phân

Vấn đề tổ chức về việc phân bổ trách nhiệm cho nhiều nhiệm vụ cho
hàm `main` là phổ biến đối với nhiều dự án nhị phân (binary projects). Do đó, cộng đồng Rust
đã phát triển các hướng dẫn để phân tách các mối bận tâm (separation of concerns) riêng biệt của một
chương trình nhị phân khi `main` bắt đầu trở nên lớn. Quá trình này có các
bước sau:

- Chia chương trình của bạn thành một tệp _main.rs_ và một tệp _lib.rs_ và chuyển
  logic chương trình của bạn sang _lib.rs_.
- Miễn là logic phân tích cú pháp dòng lệnh của bạn còn nhỏ, nó có thể vẫn nằm trong
  _main.rs_.
- Khi logic phân tích cú pháp dòng lệnh bắt đầu trở nên phức tạp, hãy trích xuất nó
  từ _main.rs_ và chuyển nó sang _lib.rs_.

Các trách nhiệm còn lại trong hàm `main` sau quá trình này
nên được giới hạn ở các việc sau:

- Gọi logic phân tích cú pháp dòng lệnh với các giá trị đối số
- Thiết lập bất kỳ cấu hình nào khác
- Gọi một hàm `run` trong _lib.rs_
- Xử lý lỗi nếu `run` trả về một lỗi

Mô hình này là về việc phân tách các mối bận tâm: _main.rs_ xử lý việc chạy
chương trình và _lib.rs_ xử lý tất cả logic của nhiệm vụ đang thực hiện. Bởi vì bạn
không thể kiểm thử trực tiếp hàm `main`, cấu trúc này cho phép bạn kiểm thử tất cả
logic chương trình của mình bằng cách chuyển nó vào các hàm trong _lib.rs_. Mã
còn lại trong _main.rs_ sẽ đủ nhỏ để xác minh tính chính xác của nó bằng cách
đọc nó. Hãy làm lại chương trình của chúng ta bằng cách tuân theo quy trình này.

#### Trích Xuất Bộ Phân Tích Cú Pháp Đối Số

Chúng ta sẽ trích xuất chức năng phân tích cú pháp các đối số vào một hàm mà
`main` sẽ gọi để chuẩn bị cho việc chuyển logic phân tích cú pháp dòng lệnh sang
_src/lib.rs_. Liệt kê 12-5 cho thấy phần bắt đầu mới của `main` gọi một
hàm mới `parse_config`, hàm mà chúng ta sẽ tạm thời định nghĩa trong _src/main.rs_.

<Listing number="12-5" file-name="src/main.rs" caption="Trích xuất một hàm `parse_config` từ `main`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Chúng ta vẫn đang thu thập các đối số dòng lệnh vào một vector, nhưng thay vì
gán giá trị đối số tại chỉ số 1 cho biến `query` và
giá trị đối số tại chỉ số 2 cho biến `file_path` trong hàm `main`,
chúng ta truyền toàn bộ vector cho hàm `parse_config`. Hàm
`parse_config` sau đó giữ logic xác định đối số nào
nằm trong biến nào và truyền các giá trị trở lại `main`. Chúng ta vẫn tạo
các biến `query` và `file_path` trong `main`, nhưng `main` không còn
trách nhiệm xác định các đối số dòng lệnh và biến
tương ứng như thế nào.

Việc làm lại này có vẻ như là hơi quá mức đối với chương trình nhỏ của chúng ta, nhưng chúng ta đang tái cấu trúc
theo các bước nhỏ, tăng dần. Sau khi thực hiện thay đổi này, hãy chạy lại chương trình để
xác minh rằng việc phân tích cú pháp đối số vẫn hoạt động. Tốt nhất là nên kiểm tra tiến độ của bạn
thường xuyên, để giúp xác định nguyên nhân của các vấn đề khi chúng xảy ra.

#### Nhóm Các Giá Trị Cấu Hình

Chúng ta có thể thực hiện một bước nhỏ khác để cải thiện hàm `parse_config` hơn nữa.
Tại thời điểm này, chúng ta đang trả về một tuple, nhưng sau đó chúng ta ngay lập tức tách
tuple đó thành các phần riêng lẻ một lần nữa. Đây là một dấu hiệu cho thấy có lẽ chúng ta chưa có
sự trừu tượng hóa phù hợp.

Một chỉ báo khác cho thấy vẫn còn chỗ để cải thiện là phần `config`
của `parse_config`, nó ám chỉ rằng hai giá trị chúng ta trả về có liên quan với nhau và
đều là một phần của một giá trị cấu hình. Chúng ta hiện không truyền đạt
ý nghĩa này trong cấu trúc của dữ liệu ngoài việc nhóm hai giá trị thành
một tuple; thay vào đó chúng ta sẽ đặt hai giá trị vào một struct và đặt cho mỗi
trường của struct một cái tên có ý nghĩa. Làm như vậy sẽ giúp những người
bảo trì mã này trong tương lai dễ dàng hiểu được các giá trị khác nhau liên quan đến
nhau như thế nào và mục đích của chúng là gì.

Liệt kê 12-6 cho thấy những cải tiến đối với hàm `parse_config`.

<Listing number="12-6" file-name="src/main.rs" caption="Tái cấu trúc `parse_config` để trả về một instance của một struct `Config`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm một struct tên là `Config` được định nghĩa để có các trường tên là `query` và
`file_path`. Chữ ký của `parse_config` giờ đây chỉ ra rằng nó trả về một
giá trị `Config`. Trong thân hàm `parse_config`, nơi chúng ta từng trả về
các string slice tham chiếu đến các giá trị `String` trong `args`, giờ đây chúng ta định nghĩa `Config`
chứa các giá trị `String` được sở hữu (owned). Biến `args` trong `main` là chủ sở hữu của
các giá trị đối số và chỉ cho phép hàm `parse_config` mượn
chúng, điều đó có nghĩa là chúng ta sẽ vi phạm các quy tắc mượn (borrowing rules) của Rust nếu `Config` cố gắng lấy
quyền sở hữu các giá trị trong `args`.

Có một số cách chúng ta có thể quản lý dữ liệu `String`; con đường dễ dàng nhất,
mặc dù hơi kém hiệu quả, là gọi phương thức `clone` trên các giá trị.
Điều này sẽ tạo ra một bản sao đầy đủ của dữ liệu để instance `Config` sở hữu, việc này
tốn nhiều thời gian và bộ nhớ hơn so với việc lưu trữ một tham chiếu đến dữ liệu chuỗi.
Tuy nhiên, việc nhân bản dữ liệu cũng làm cho mã của chúng ta rất đơn giản vì chúng ta
không phải quản lý lifetime của các tham chiếu; trong hoàn cảnh này,
hy sinh một chút hiệu năng để đạt được sự đơn giản là một sự đánh đổi xứng đáng.

> ### Những Đánh Đổi của Việc Sử Dụng `clone`
>
> Có một xu hướng trong nhiều người dùng Rust là tránh sử dụng `clone` để sửa chữa
> các vấn đề về quyền sở hữu vì chi phí thời gian chạy (runtime cost) của nó. Trong
> [Chương 13][ch13]<!-- ignore -->, bạn sẽ học cách sử dụng các phương thức hiệu quả hơn
> trong loại tình huống này. Nhưng hiện tại, việc sao chép một vài
> chuỗi để tiếp tục đạt được tiến bộ là không sao vì bạn sẽ chỉ thực hiện các bản sao này
> một lần và đường dẫn tệp cũng như chuỗi truy vấn của bạn rất nhỏ. Tốt hơn là nên có
> một chương trình hoạt động nhưng hơi kém hiệu quả còn hơn là cố gắng tối ưu hóa mã quá mức
> ngay trong lần viết đầu tiên. Khi bạn trở nên có kinh nghiệm hơn với Rust, việc
> bắt đầu với giải pháp hiệu quả nhất sẽ dễ dàng hơn, nhưng hiện tại, việc
> gọi `clone` là hoàn toàn có thể chấp nhận được.

Chúng ta đã cập nhật `main` để nó đặt instance của `Config` được trả về bởi
`parse_config` vào một biến tên là `config`, và chúng ta đã cập nhật mã
trước đây sử dụng các biến `query` và `file_path` riêng biệt để giờ đây nó sử dụng
các trường trên struct `Config` thay thế.

Bây giờ mã của chúng ta truyền đạt rõ ràng hơn rằng `query` và `file_path` có liên quan đến nhau và
mục đích của chúng là cấu hình cách chương trình sẽ hoạt động. Bất kỳ mã nào
sử dụng các giá trị này đều biết cách tìm chúng trong instance `config` trong các trường
được đặt tên theo mục đích của chúng.

#### Tạo một Constructor cho `Config`

Cho đến nay, chúng ta đã trích xuất logic chịu trách nhiệm phân tích cú pháp các đối số dòng lệnh
từ `main` và đặt nó vào hàm `parse_config`. Làm như vậy đã
giúp chúng ta thấy rằng các giá trị `query` và `file_path` có liên quan đến nhau, và
mối quan hệ đó nên được truyền đạt trong mã của chúng ta. Sau đó chúng ta đã thêm một struct `Config` để
đặt tên cho mục đích liên quan của `query` và `file_path` và để có thể trả về các
tên giá trị dưới dạng tên trường struct từ hàm `parse_config`.

Vì vậy, bây giờ mục đích của hàm `parse_config` là tạo một instance `Config`,
chúng ta có thể thay đổi `parse_config` từ một hàm thông thường thành một hàm
tên là `new` được liên kết với struct `Config`. Việc thực hiện thay đổi này
sẽ làm cho mã mang tính đặc trưng của Rust (idiomatic) hơn. Chúng ta có thể tạo các instance của các kiểu trong
thư viện chuẩn, chẳng hạn như `String`, bằng cách gọi `String::new`. Tương tự, bằng cách
thay đổi `parse_config` thành một hàm `new` liên kết với `Config`, chúng ta sẽ
có thể tạo các instance của `Config` bằng cách gọi `Config::new`. Liệt kê 12-7
cho thấy những thay đổi chúng ta cần thực hiện.

<Listing number="12-7" file-name="src/main.rs" caption="Thay đổi `parse_config` thành `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Chúng ta đã cập nhật `main` nơi chúng ta đang gọi `parse_config` để thay vào đó gọi
`Config::new`. Chúng ta đã đổi tên của `parse_config` thành `new` và di chuyển nó
vào trong một khối `impl`, khối này liên kết hàm `new` với `Config`. Hãy thử
biên dịch lại mã này để đảm bảo nó hoạt động.

### Sửa Lỗi Xử Lý Lỗi

Bây giờ chúng ta sẽ thực hiện việc sửa lỗi xử lý lỗi của mình. Hãy nhớ rằng việc cố gắng truy cập
các giá trị trong vector `args` tại chỉ số 1 hoặc chỉ số 2 sẽ khiến chương trình
panic nếu vector chứa ít hơn ba phần tử. Thử chạy chương trình
mà không có bất kỳ đối số nào; nó sẽ trông như thế này:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

Dòng `index out of bounds: the len is 1 but the index is 1` là một thông báo lỗi
dành cho các lập trình viên. Nó sẽ không giúp người dùng cuối của chúng ta hiểu được
họ nên làm gì thay thế. Hãy sửa lỗi đó ngay bây giờ.

#### Cải Thiện Thông Báo Lỗi

Trong Liệt kê 12-8, chúng ta thêm một kiểm tra trong hàm `new` để xác minh rằng
slice đủ dài trước khi truy cập chỉ số 1 và chỉ số 2. Nếu slice không
đủ dài, chương trình sẽ panic và hiển thị một thông báo lỗi tốt hơn.

<Listing number="12-8" file-name="src/main.rs" caption="Thêm một kiểm tra cho số lượng đối số">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Mã này tương tự như [hàm `Guess::new` mà chúng ta đã viết trong Liệt kê
9-13][ch9-custom-types]<!-- ignore -->, nơi chúng ta đã gọi `panic!` khi
đối số `value` nằm ngoài phạm vi của các giá trị hợp lệ. Thay vì kiểm tra
một phạm vi các giá trị ở đây, chúng ta đang kiểm tra xem độ dài của `args` có ít nhất là
`3` hay không và phần còn lại của hàm có thể hoạt động dưới giả định rằng
điều kiện này đã được đáp ứng. Nếu `args` có ít hơn ba phần tử, điều kiện này
sẽ là `true`, và chúng ta gọi macro `panic!` để kết thúc chương trình ngay lập tức.

Với vài dòng mã bổ sung này trong `new`, hãy chạy lại chương trình mà không có bất kỳ
đối số nào để xem lỗi bây giờ trông như thế nào:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Đầu ra này tốt hơn: bây giờ chúng ta đã có một thông báo lỗi hợp lý. Tuy nhiên, chúng ta cũng
có thông tin dư thừa mà chúng ta không muốn đưa cho người dùng của mình. Có lẽ
kỹ thuật chúng ta đã sử dụng trong Liệt kê 9-13 không phải là kỹ thuật tốt nhất để sử dụng ở đây: một lệnh gọi
đến `panic!` phù hợp cho một vấn đề lập trình hơn là một vấn đề sử dụng,
[như đã thảo luận trong Chương 9][ch9-error-guidelines]<!-- ignore -->. Thay vào đó,
chúng ta sẽ sử dụng kỹ thuật khác mà bạn đã học ở Chương 9—[trả về một
`Result`][ch9-result]<!-- ignore --> để chỉ ra thành công hoặc một lỗi.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Trả Về một `Result` Thay Vì Gọi `panic!`

Thay vào đó, chúng ta có thể trả về một giá trị `Result` sẽ chứa một instance `Config` trong
trường hợp thành công và sẽ mô tả vấn đề trong trường hợp lỗi. Chúng ta cũng
sẽ đổi tên hàm từ `new` thành `build` vì nhiều
lập trình viên kỳ vọng các hàm `new` không bao giờ thất bại. Khi `Config::build` đang
giao tiếp với `main`, chúng ta có thể sử dụng kiểu `Result` để báo hiệu rằng đã có một
vấn đề. Sau đó chúng ta có thể thay đổi `main` để chuyển đổi một biến thể `Err` thành một
lỗi thực tế hơn cho người dùng của chúng ta mà không có đoạn văn bản xung quanh về `thread
'main'` và `RUST_BACKTRACE` mà lệnh gọi đến `panic!` gây ra.

Liệt kê 12-9 cho thấy những thay đổi chúng ta cần thực hiện đối với giá trị trả về của
hàm mà bây giờ chúng ta gọi là `Config::build` và thân của hàm cần thiết
để trả về một `Result`. Lưu ý rằng mã này sẽ không được biên dịch cho đến khi chúng ta cập nhật cả `main`,
việc mà chúng ta sẽ thực hiện trong danh sách tiếp theo.

<Listing number="12-9" file-name="src/main.rs" caption="Trả về một `Result` từ `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Hàm `build` của chúng ta trả về một `Result` với một instance `Config` trong trường hợp thành công
và một string literal trong trường hợp lỗi. Các giá trị lỗi của chúng ta sẽ luôn là
các string literal có lifetime `'static`.

Chúng ta đã thực hiện hai thay đổi trong thân hàm: thay vì gọi `panic!`
khi người dùng không truyền đủ các đối số, bây giờ chúng ta trả về một giá trị `Err`, và
chúng ta đã bọc giá trị trả về `Config` trong một `Ok`. Những thay đổi này làm cho
hàm tuân thủ chữ ký kiểu mới của nó.

Trả về một giá trị `Err` từ `Config::build` cho phép hàm `main`
xử lý giá trị `Result` được trả về từ hàm `build` và thoát khỏi
tiến trình một cách sạch sẽ hơn trong trường hợp lỗi.

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### Gọi `Config::build` và Xử Lý Lỗi

Để xử lý trường hợp lỗi và in ra một thông báo thân thiện với người dùng, chúng ta cần cập nhật
`main` để xử lý `Result` đang được trả về bởi `Config::build`, như được hiển thị trong
Liệt kê 12-10. Chúng ta cũng sẽ tước bỏ trách nhiệm thoát khỏi công cụ dòng lệnh
với một mã lỗi khác không từ `panic!` và thay vào đó tự mình triển khai nó.
Một trạng thái thoát (exit status) khác không là một quy ước để báo hiệu cho tiến trình đã
gọi chương trình của chúng ta rằng chương trình đã thoát với một trạng thái lỗi.

<Listing number="12-10" file-name="src/main.rs" caption="Thoát với một mã lỗi nếu việc xây dựng một `Config` thất bại">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Trong liệt kê này, chúng ta đã sử dụng một phương thức mà chúng ta chưa đề cập chi tiết:
`unwrap_or_else`, phương thức này được định nghĩa trên `Result<T, E>` bởi thư viện chuẩn.
Sử dụng `unwrap_or_else` cho phép chúng ta định nghĩa một số xử lý lỗi tùy chỉnh, không phải `panic!`.
Nếu `Result` là một giá trị `Ok`, hành vi của phương thức này tương tự
như `unwrap`: nó trả về giá trị bên trong mà `Ok` đang bọc. Tuy nhiên, nếu
giá trị là một giá trị `Err`, phương thức này gọi mã trong _closure_, đó là
một hàm ẩn danh mà chúng ta định nghĩa và truyền làm đối số cho `unwrap_or_else`.
Chúng ta sẽ tìm hiểu kỹ về closure trong [Chương 13][ch13]<!-- ignore -->. Hiện tại,
bạn chỉ cần biết rằng `unwrap_or_else` sẽ truyền giá trị bên trong của
`Err`, trong trường hợp này là chuỗi tĩnh `"not enough arguments"`
mà chúng ta đã thêm trong Liệt kê 12-9, vào closure của chúng ta trong đối số `err`
xuất hiện giữa các thanh dọc. Mã trong closure sau đó có thể sử dụng
giá trị `err` khi nó chạy.

Chúng ta đã thêm một dòng `use` mới để đưa `process` từ thư viện chuẩn vào
phạm vi. Mã trong closure sẽ được chạy trong trường hợp lỗi chỉ có
hai dòng: chúng ta in giá trị `err` và sau đó gọi `process::exit`. Hàm
`process::exit` sẽ dừng chương trình ngay lập tức và trả về
số được truyền làm mã trạng thái thoát. Điều này tương tự như việc
xử lý dựa trên `panic!` mà chúng ta đã sử dụng trong Liệt kê 12-8, nhưng chúng ta không còn nhận được tất cả
đầu ra bổ sung. Hãy thử xem:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Tuyệt vời! Đầu ra này thân thiện hơn nhiều đối với người dùng của chúng ta.

### Trích Xuất Logic từ `main`

Bây giờ chúng ta đã hoàn thành việc tái cấu trúc phần phân tích cú pháp cấu hình, hãy chuyển sang
logic của chương trình. Như chúng ta đã nêu trong [“Phân Tách các Mối Bận Tâm cho Các Dự Án Nhị
Phân”](#separation-of-concerns-for-binary-projects)<!-- ignore -->, chúng ta sẽ
trích xuất một hàm tên là `run` sẽ giữ tất cả logic hiện có trong
hàm `main` mà không liên quan đến việc thiết lập cấu hình hoặc xử lý
lỗi. Khi hoàn tất, `main` sẽ ngắn gọn và dễ xác minh bằng cách
kiểm tra trực quan, và chúng ta sẽ có thể viết các kiểm thử cho tất cả logic khác.

Liệt kê 12-11 cho thấy hàm `run` được trích xuất. Hiện tại, chúng ta chỉ đang thực hiện
cải tiến nhỏ, tăng dần là trích xuất hàm. Chúng ta vẫn
định nghĩa hàm trong _src/main.rs_.

<Listing number="12-11" file-name="src/main.rs" caption="Trích xuất một hàm `run` chứa phần còn lại của logic chương trình">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

Hàm `run` giờ đây chứa tất cả logic còn lại từ `main`, bắt đầu
từ việc đọc tệp. Hàm `run` nhận instance `Config` làm
đối số.

#### Trả Về Lỗi từ Hàm `run`

Với logic chương trình còn lại được tách biệt vào hàm `run`, chúng ta có thể
cải thiện việc xử lý lỗi, như chúng ta đã làm với `Config::build` trong Liệt kê 12-9.
Thay vì cho phép chương trình panic bằng cách gọi `expect`, hàm
`run` sẽ trả về một `Result<T, E>` khi có sự cố xảy ra. Điều này sẽ cho phép
chúng ta củng cố thêm logic xoay quanh việc xử lý lỗi vào `main` theo một
cách thân thiện với người dùng. Liệt kê 12-12 cho thấy những thay đổi chúng ta cần thực hiện đối với
chữ ký và thân hàm `run`.

<Listing number="12-12" file-name="src/main.rs" caption="Thay đổi hàm `run` để trả về `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Chúng ta đã thực hiện ba thay đổi quan trọng ở đây. Đầu tiên, chúng ta đã thay đổi kiểu trả về của
hàm `run` thành `Result<(), Box<dyn Error>>`. Hàm này trước đây
trả về kiểu unit, `()`, và chúng ta giữ nó làm giá trị được trả về trong
trường hợp `Ok`.

Đối với kiểu lỗi, chúng ta đã sử dụng _trait object_ `Box<dyn Error>` (và chúng ta đã
đưa `std::error::Error` vào phạm vi bằng một câu lệnh `use` ở đầu tệp).
Chúng ta sẽ tìm hiểu về trait object trong [Chương 18][ch18]<!-- ignore -->. Hiện tại,
chỉ cần biết rằng `Box<dyn Error>` có nghĩa là hàm sẽ trả về một kiểu
triển khai trait `Error`, nhưng chúng ta không phải chỉ định cụ thể kiểu nào
mà giá trị trả về sẽ là. Điều này cho phép chúng ta linh hoạt trả về các giá trị lỗi có
thể thuộc các kiểu khác nhau trong các trường hợp lỗi khác nhau. Từ khóa `dyn` là viết tắt
của _dynamic_ (động).

Thứ hai, chúng ta đã loại bỏ lời gọi đến `expect` để thay thế bằng toán tử `?`, như chúng ta
đã nói trong [Chương 9][ch9-question-mark]<!-- ignore -->. Thay vì
`panic!` khi gặp lỗi, `?` sẽ trả về giá trị lỗi từ hàm hiện tại
cho người gọi xử lý.

Thứ ba, hàm `run` bây giờ trả về một giá trị `Ok` trong trường hợp thành công.
Chúng ta đã khai báo kiểu thành công của hàm `run` là `()` trong chữ ký,
điều đó có nghĩa là chúng ta cần bọc giá trị kiểu unit trong giá trị `Ok`. Cú pháp
`Ok(())` này thoạt nhìn có vẻ hơi lạ, nhưng sử dụng `()` như thế này là
cách chuẩn tắc (idiomatic) để chỉ ra rằng chúng ta đang gọi `run` chỉ vì các tác dụng phụ (side effects) của nó;
nó không trả về một giá trị chúng ta cần.

Khi bạn chạy mã này, nó sẽ được biên dịch nhưng sẽ hiển thị một cảnh báo:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust cho chúng ta biết rằng mã của chúng ta đã bỏ qua giá trị `Result` và giá trị `Result` đó
có thể chỉ ra rằng một lỗi đã xảy ra. Nhưng chúng ta không kiểm tra xem có
lỗi hay không, và trình biên dịch nhắc nhở chúng ta rằng có lẽ chúng ta đã định
có một số mã xử lý lỗi ở đây! Hãy khắc phục vấn đề đó ngay bây giờ.

#### Xử Lý Lỗi Được Trả Về từ `run` trong `main`

Chúng ta sẽ kiểm tra các lỗi và xử lý chúng bằng kỹ thuật tương tự như kỹ thuật chúng ta đã sử dụng
với `Config::build` trong Liệt kê 12-10, nhưng có một chút khác biệt:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Chúng ta sử dụng `if let` thay vì `unwrap_or_else` để kiểm tra xem `run` có trả về một
giá trị `Err` hay không và gọi `process::exit(1)` nếu nó xảy ra. Hàm `run`
không trả về một giá trị mà chúng ta muốn `unwrap` theo cùng cách mà
`Config::build` trả về instance `Config`. Bởi vì `run` trả về `()` trong
trường hợp thành công, chúng ta chỉ quan tâm đến việc phát hiện lỗi, vì vậy chúng ta không cần
`unwrap_or_else` để trả về giá trị đã được unwrapped, cái mà chỉ là `()`.

Thân của các hàm `if let` và `unwrap_or_else` là giống nhau trong
cả hai trường hợp: chúng ta in lỗi và thoát.

### Chia Tách Mã vào một Library Crate

Dự án `minigrep` của chúng ta trông ổn cho đến nay! Bây giờ chúng ta sẽ chia tách
tệp _src/main.rs_ và đặt một số mã vào tệp _src/lib.rs_. Bằng cách đó, chúng ta
có thể kiểm thử mã và có một tệp _src/main.rs_ với ít trách nhiệm hơn.

Hãy chuyển tất cả mã không nằm trong hàm `main` từ _src/main.rs_ sang
_src/lib.rs_:

- Định nghĩa hàm `run`
- Các câu lệnh `use` liên quan
- Định nghĩa struct `Config`
- Định nghĩa hàm `Config::build`

Nội dung của _src/lib.rs_ nên có các chữ ký được hiển thị trong Liệt kê 12-13
(chúng ta đã bỏ qua thân của các hàm để cho ngắn gọn). Lưu ý rằng điều này sẽ không
biên dịch được cho đến khi chúng ta sửa đổi _src/main.rs_ trong Liệt kê 12-14.

<Listing number="12-13" file-name="src/lib.rs" caption="Chuyển `Config` và `run` vào *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs:here}}
```

</Listing>

Chúng ta đã sử dụng rộng rãi từ khóa `pub`: trên `Config`, trên các trường của nó và phương thức
`build` của nó, và trên hàm `run`. Bây giờ chúng ta có một crate thư viện (library crate) có
một API công khai mà chúng ta có thể kiểm thử!

Bây giờ chúng ta cần đưa mã chúng ta đã chuyển sang _src/lib.rs_ vào phạm vi của
crate nhị phân (binary crate) trong _src/main.rs_, như được hiển thị trong Liệt kê 12-14.

<Listing number="12-14" file-name="src/main.rs" caption="Sử dụng library crate `minigrep` trong *src/main.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Chúng ta thêm một dòng `use minigrep::Config` để đưa kiểu `Config` từ
crate thư viện vào phạm vi của crate nhị phân, và chúng ta thêm tiền tố cho hàm `run`
bằng tên crate của chúng ta. Bây giờ tất cả chức năng nên được kết nối và nên
hoạt động. Chạy chương trình với `cargo run` và đảm bảo mọi thứ hoạt động chính xác.

Phù! Đó là một khối lượng công việc lớn, nhưng chúng ta đã chuẩn bị sẵn sàng cho sự thành công trong
tương lai. Bây giờ việc xử lý lỗi dễ dàng hơn nhiều, và chúng ta đã làm cho mã có tính mô-đun hơn.
Hầu như tất cả công việc của chúng ta sẽ được thực hiện trong _src/lib.rs_ từ nay về sau.

Hãy tận dụng tính mô-đun mới tìm thấy này bằng cách làm một việc mà trước đây
khó thực hiện với mã cũ nhưng giờ đây lại dễ dàng với mã mới: chúng ta sẽ
viết một số kiểm thử!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
