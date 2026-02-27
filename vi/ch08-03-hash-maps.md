## Lưu trữ các Khóa với các Giá trị Liên kết trong Bảng Băm (Storing Keys with Associated Values in Hash Maps)

Bộ sưu tập phổ biến cuối cùng của chúng ta là _bảng băm_ (hash map). Kiểu `HashMap<K, V>`
lưu trữ một ánh xạ các khóa kiểu `K` đến các giá trị kiểu `V` bằng cách sử dụng một _hàm băm_ (hashing
function), hàm này xác định cách nó đặt các khóa và giá trị này vào bộ nhớ.
Nhiều ngôn ngữ lập trình hỗ trợ loại cấu trúc dữ liệu này, nhưng chúng thường
sử dụng một cái tên khác, chẳng hạn như _hash_, _map_, _object_, _hash table_,
_dictionary_, hoặc _associative array_, chỉ để kể tên một vài ví dụ.

Bảng băm hữu ích khi bạn muốn tra cứu dữ liệu không phải bằng cách sử dụng chỉ số (index), như
bạn có thể làm với vector, mà bằng cách sử dụng một khóa có thể thuộc bất kỳ kiểu nào. Ví dụ,
trong một trò chơi, bạn có thể theo dõi điểm số của mỗi đội trong một bảng băm trong đó
mỗi khóa là tên của một đội và các giá trị là điểm số của mỗi đội. Khi biết một tên
đội, bạn có thể lấy lại điểm số của nó.

Chúng ta sẽ xem qua các API cơ bản của bảng băm trong phần này, nhưng còn nhiều điều thú vị khác
đang ẩn giấu trong các hàm được định nghĩa cho `HashMap<K, V>` bởi thư viện tiêu chuẩn.
Như mọi khi, hãy kiểm tra tài liệu thư viện tiêu chuẩn để biết thêm thông tin.

### Tạo một Bảng Băm Mới

Một cách để tạo một bảng băm trống là sử dụng `new` và thêm các phần tử bằng
`insert`. Trong Listing 8-20, chúng ta đang theo dõi điểm số của hai đội có
tên là _Blue_ và _Yellow_. Đội Blue bắt đầu với 10 điểm, và đội
Yellow bắt đầu với 50 điểm.

<Listing number="8-20" caption="Tạo một bảng băm mới và chèn một số khóa và giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Lưu ý rằng trước tiên chúng ta cần `use` `HashMap` từ phần collections của
thư viện tiêu chuẩn. Trong số ba bộ sưu tập phổ biến của chúng ta, bộ sưu tập này ít được
sử dụng nhất, vì vậy nó không được bao gồm trong các tính năng được đưa vào phạm vi
tự động trong prelude. Bảng băm cũng có ít sự hỗ trợ hơn từ
thư viện tiêu chuẩn; ví dụ, không có macro tích hợp sẵn nào để khởi tạo chúng.

Giống như vector, bảng băm lưu trữ dữ liệu của chúng trên heap. `HashMap` này có
các khóa kiểu `String` và các giá trị kiểu `i32`. Giống như vector, bảng băm có
tính đồng nhất (homogeneous): tất cả các khóa phải có cùng kiểu, và tất cả các giá trị
phải có cùng kiểu.

### Truy cập các Giá trị trong một Bảng Băm

Chúng ta có thể lấy một giá trị ra khỏi bảng băm bằng cách cung cấp khóa của nó cho phương thức `get`,
như được hiển thị trong Listing 8-21.

<Listing number="8-21" caption="Truy cập điểm số cho đội Blue được lưu trữ trong bảng băm">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Ở đây, `score` sẽ có giá trị liên kết với đội Blue, và
kết quả sẽ là `10`. Phương thức `get` trả về một `Option<&V>`; nếu không có
giá trị nào cho khóa đó trong bảng băm, `get` sẽ trả về `None`. Chương trình này
xử lý `Option` bằng cách gọi `copied` để lấy một `Option<i32>` thay vì một
`Option<&i32>`, sau đó dùng `unwrap_or` để đặt `score` bằng không nếu `scores` không
có mục nhập (entry) cho khóa đó.

Chúng ta có thể duyệt qua từng cặp khóa-giá trị trong một bảng băm theo cách tương tự như chúng ta
làm với vector, sử dụng vòng lặp `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Đoạn mã này sẽ in từng cặp theo một thứ tự tùy ý:

```text
Yellow: 50
Blue: 10
```

### Bảng Băm và Quyền sở hữu (Ownership)

Đối với các kiểu dữ liệu triển khai đặc tính `Copy`, như `i32`, các giá trị được sao chép
vào bảng băm. Đối với các giá trị có quyền sở hữu như `String`, các giá trị sẽ được di chuyển (moved) và
bảng băm sẽ là chủ sở hữu của các giá trị đó, như được minh họa trong Listing 8-22.

<Listing number="8-22" caption="Chỉ ra rằng các khóa và giá trị thuộc quyền sở hữu của bảng băm sau khi chúng được chèn vào">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

Chúng ta không thể sử dụng các biến `field_name` và `field_value` sau
khi chúng đã được di chuyển vào bảng băm bằng lời gọi đến `insert`.

Nếu chúng ta chèn các tham chiếu đến các giá trị vào bảng băm, các giá trị đó sẽ không được di chuyển
vào bảng băm. Các giá trị mà các tham chiếu trỏ đến phải hợp lệ trong ít
nhất là khoảng thời gian mà bảng băm còn hợp lệ. Chúng ta sẽ nói thêm về những vấn đề này trong
[“Xác thực các Tham chiếu với Lifetimes”][validating-references-with-lifetimes]<!-- ignore --> ở Chương 10.

### Cập nhật một Bảng Băm

Mặc dù số lượng các cặp khóa và giá trị có thể mở rộng, nhưng mỗi khóa duy nhất
chỉ có thể có một giá trị liên kết với nó tại một thời điểm (nhưng không phải ngược lại: ví
dụ, cả đội Blue và đội Yellow đều có thể có giá trị `10`
được lưu trữ trong bảng băm `scores`).

Khi bạn muốn thay đổi dữ liệu trong một bảng băm, bạn phải quyết định cách
xử lý trường hợp khi một khóa đã được gán một giá trị. Bạn có thể thay thế
giá trị cũ bằng giá trị mới, hoàn toàn bỏ qua giá trị cũ. Bạn có thể
giữ giá trị cũ và bỏ qua giá trị mới, chỉ thêm giá trị mới nếu
khóa _chưa_ có giá trị. Hoặc bạn có thể kết hợp giá trị cũ và
giá trị mới. Hãy cùng xem cách thực hiện từng cách này!

#### Ghi đè một Giá trị

Nếu chúng ta chèn một khóa và một giá trị vào một bảng băm và sau đó chèn chính khóa đó
với một giá trị khác, giá trị liên kết với khóa đó sẽ bị thay thế.
Mặc dù mã trong Listing 8-23 gọi `insert` hai lần, bảng băm sẽ
chỉ chứa một cặp khóa-giá trị vì chúng ta đang chèn giá trị cho khóa của đội Blue
trong cả hai lần.

<Listing number="8-23" caption="Thay thế một giá trị được lưu trữ với một khóa cụ thể">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Đoạn mã này sẽ in `{"Blue": 25}`. Giá trị ban đầu là `10` đã bị
ghi đè.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Chỉ thêm một Khóa và Giá trị Nếu Khóa chưa Hiện diện

Việc kiểm tra xem một khóa cụ thể đã tồn tại trong bảng băm với
một giá trị hay chưa và sau đó thực hiện các hành động sau đây là rất phổ biến: nếu khóa đó đã tồn tại trong
bảng băm, giá trị hiện tại sẽ được giữ nguyên; nếu khóa
đó không tồn tại, hãy chèn nó và một giá trị cho nó.

Bảng băm có một API đặc biệt cho việc này được gọi là `entry`, nó nhận khóa bạn
muốn kiểm tra làm tham số. Giá trị trả về của phương thức `entry` là một enum
gọi là `Entry` đại diện cho một giá trị có thể tồn tại hoặc không. Giả sử
chúng ta muốn kiểm tra xem khóa cho đội Yellow có giá trị liên kết
với nó hay không. Nếu không, chúng ta muốn chèn giá trị `50`, và tương tự cho
đội Blue. Sử dụng API `entry`, mã sẽ trông giống như Listing 8-24.

<Listing number="8-24" caption="Sử dụng phương thức `entry` để chỉ chèn nếu khóa chưa có giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

Phương thức `or_insert` trên `Entry` được định nghĩa để trả về một tham chiếu có thể thay đổi (mutable reference) đến
giá trị cho khóa `Entry` tương ứng nếu khóa đó tồn tại, và nếu không, nó
sẽ chèn tham số làm giá trị mới cho khóa này và trả về một tham chiếu có thể thay đổi
đến giá trị mới. Kỹ thuật này gọn gàng hơn nhiều so với việc tự viết
logic và ngoài ra, nó hoạt động tốt hơn với bộ kiểm tra mượn (borrow checker).

Chạy mã trong Listing 8-24 sẽ in `{"Yellow": 50, "Blue": 10}`. Lời gọi
`entry` đầu tiên sẽ chèn khóa cho đội Yellow với giá trị
`50` vì đội Yellow chưa có giá trị. Lời gọi `entry` thứ hai
sẽ không thay đổi bảng băm vì đội Blue đã có giá trị
`10`.

#### Cập nhật một Giá trị Dựa trên Giá trị Cũ

Một trường hợp sử dụng phổ biến khác cho bảng băm là tra cứu giá trị của một khóa và sau đó
cập nhật nó dựa trên giá trị cũ. Ví dụ, Listing 8-25 hiển thị mã
đếm số lần mỗi từ xuất hiện trong một đoạn văn bản. Chúng ta sử dụng một bảng băm với
các từ làm khóa và tăng giá trị để theo dõi số lần chúng ta thấy
từ đó. Nếu đây là lần đầu tiên chúng ta thấy một từ, trước tiên chúng ta sẽ chèn
giá trị `0`.

<Listing number="8-25" caption="Đếm số lần xuất hiện của các từ bằng một bảng băm lưu trữ các từ và số lượng">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Đoạn mã này sẽ in `{"world": 2, "hello": 1, "wonderful": 1}`. Bạn có thể thấy
các cặp khóa-giá trị tương tự được in theo một thứ tự khác: hãy nhớ lại từ [“Truy cập
các Giá trị trong một Bảng Băm”][access]<!-- ignore --> rằng việc duyệt qua một bảng băm
diễn ra theo thứ tự tùy ý.

Phương thức `split_whitespace` trả về một iterator trên các lát cắt con (subslices), được phân tách bằng
khoảng trắng, của giá trị trong `text`. Phương thức `or_insert` trả về một tham chiếu có thể thay đổi
(`&mut V`) đến giá trị cho khóa đã chỉ định. Ở đây, chúng ta lưu trữ tham chiếu
có thể thay đổi đó trong biến `count`, vì vậy để gán cho giá trị đó,
trước tiên chúng ta phải giải tham chiếu `count` bằng dấu sao (`*`). Tham chiếu
có thể thay đổi nằm ngoài phạm vi ở cuối vòng lặp `for`, vì vậy tất cả những
thay đổi này đều an toàn và được các quy tắc mượn cho phép.

### Hàm Băm (Hashing Functions)

Theo mặc định, `HashMap` sử dụng một hàm băm gọi là _SipHash_ có thể cung cấp
khả năng chống lại các cuộc tấn công từ chối dịch vụ (DoS) liên quan đến bảng
băm[^siphash]<!-- ignore -->. Đây không phải là thuật toán băm nhanh nhất hiện có,
nhưng sự đánh đổi để có bảo mật tốt hơn đi kèm với việc giảm
hiệu suất là xứng đáng. Nếu bạn phân tích hiệu năng (profile) mã của mình và thấy rằng hàm
băm mặc định quá chậm so với mục đích của bạn, bạn có thể chuyển sang một hàm khác
bằng cách chỉ định một bộ băm (hasher) khác. Một _hasher_ là một kiểu dữ liệu triển khai đặc tính
`BuildHasher`. Chúng ta sẽ nói về các đặc tính (traits) và cách triển khai chúng trong
[Chương 10][traits]<!-- ignore -->. Bạn không nhất thiết phải tự triển khai
bộ băm của riêng mình từ đầu; [crates.io](https://crates.io/)<!-- ignore -->
có các thư viện được chia sẻ bởi những người dùng Rust khác cung cấp các bộ băm triển khai nhiều
thuật toán băm phổ biến.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

{{#quiz ../quizzes/ch08-03-hashmap.toml}}

## Tóm tắt

Vector, chuỗi và bảng băm sẽ cung cấp một lượng lớn chức năng
cần thiết trong các chương trình khi bạn cần lưu trữ, truy cập và sửa đổi dữ liệu. Dưới đây là
một số bài tập mà bây giờ bạn đã có đủ khả năng để giải quyết:

1. Cho một danh sách các số nguyên, hãy sử dụng một vector và trả về trung vị (median - khi được sắp xếp,
   là giá trị ở vị trí giữa) và yếu vị (mode - giá trị xuất hiện thường xuyên
   nhất; một bảng băm sẽ hữu ích ở đây) của danh sách.
2. Chuyển đổi các chuỗi sang pig latin. Phụ âm đầu tiên của mỗi từ được chuyển xuống
   cuối từ và thêm _ay_, vì vậy _first_ trở thành _irst-fay_. Các từ
   bắt đầu bằng một nguyên âm thì được thêm _hay_ vào cuối thay thế (_apple_ trở thành
   _apple-hay_). Hãy lưu ý các chi tiết về mã hóa UTF-8!
3. Sử dụng một bảng băm và các vector, hãy tạo một giao diện văn bản để cho phép người dùng thêm
   tên nhân viên vào một phòng ban trong công ty; ví dụ, “Add Sally to
   Engineering” hoặc “Add Amir to Sales.” Sau đó, cho phép người dùng lấy danh sách tất cả
   mọi người trong một phòng ban hoặc tất cả mọi người trong công ty theo phòng ban, được sắp xếp
   theo bảng chữ cái.

Tài liệu API thư viện tiêu chuẩn mô tả các phương thức mà vector, chuỗi,
và bảng băm có, chúng sẽ hữu ích cho các bài tập này!

Chúng ta đang tiến tới các chương trình phức tạp hơn, trong đó các thao tác có thể thất bại, vì vậy đây
là thời điểm hoàn hảo để thảo luận về xử lý lỗi. Chúng ta sẽ thực hiện điều đó tiếp theo!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
