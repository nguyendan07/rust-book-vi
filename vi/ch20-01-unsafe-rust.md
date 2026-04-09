## Unsafe Rust

Tất cả mã nguồn chúng ta đã thảo luận cho đến nay đều có các đảm bảo an toàn bộ nhớ của Rust
được thực thi tại thời điểm biên dịch. Tuy nhiên, Rust có một ngôn ngữ thứ hai ẩn bên trong nó
mà không thực thi các đảm bảo an toàn bộ nhớ này: nó được gọi là _unsafe Rust_
và hoạt động giống như Rust thông thường, nhưng mang lại cho chúng ta thêm những siêu năng lực.

Unsafe Rust tồn tại vì, về bản chất, phân tích tĩnh (static analysis) là bảo thủ. Khi
trình biên dịch cố gắng xác định xem mã nguồn có duy trì các đảm bảo hay không,
thà rằng nó từ chối một số chương trình hợp lệ còn hơn là chấp nhận một số chương trình
không hợp lệ. Mặc dù mã nguồn _có thể_ ổn, nhưng nếu trình biên dịch Rust không có
đủ thông tin để tự tin, nó sẽ từ chối mã nguồn đó. Trong những trường hợp này,
bạn có thể sử dụng mã unsafe để nói với trình biên dịch rằng, “Hãy tin tôi, tôi biết mình đang
làm gì.” Tuy nhiên, hãy lưu ý rằng, bạn tự chịu rủi ro khi sử dụng unsafe Rust: nếu bạn
sử dụng mã unsafe không đúng cách, các vấn đề có thể xảy ra do mất an toàn bộ nhớ, chẳng hạn như
giải tham chiếu con trỏ null (null pointer dereferencing).

Một lý do khác khiến Rust có một bản ngã unsafe là vì phần cứng máy tính bên dưới
vốn dĩ không an toàn. Nếu Rust không cho phép bạn thực hiện các thao tác unsafe,
bạn không thể thực hiện một số tác vụ nhất định. Rust cần cho phép bạn lập trình hệ thống cấp thấp,
chẳng hạn như tương tác trực tiếp với hệ điều hành hoặc thậm chí là
viết hệ điều hành của riêng bạn. Làm việc với lập trình hệ thống cấp thấp
là một trong các mục tiêu của ngôn ngữ này. Hãy cùng khám phá những gì chúng ta có thể làm với unsafe
Rust và cách thực hiện nó.

### Các siêu năng lực của Unsafe

Để chuyển sang unsafe Rust, hãy sử dụng từ khóa `unsafe` và sau đó bắt đầu một khối mới
chứa mã unsafe. Bạn có thể thực hiện năm hành động trong unsafe Rust mà bạn
không thể làm trong Rust an toàn, mà chúng ta gọi là _các siêu năng lực của unsafe_. Những siêu năng lực đó
bao gồm khả năng để:

- Giải tham chiếu một con trỏ thô (raw pointer)
- Gọi một hàm hoặc phương thức unsafe
- Truy cập hoặc sửa đổi một biến tĩnh có thể thay đổi (mutable static variable)
- Thực thi một trait unsafe
- Truy cập các trường của một `union`

Điều quan trọng cần hiểu là `unsafe` không tắt trình kiểm tra mượn (borrow checker)
hoặc vô hiệu hóa bất kỳ kiểm tra an toàn nào khác của Rust: nếu bạn sử dụng một tham chiếu trong mã
unsafe, nó vẫn sẽ được kiểm tra. Từ khóa `unsafe` chỉ cung cấp cho bạn quyền truy cập vào
năm tính năng này, những tính năng mà sau đó không được trình biên dịch kiểm tra về an toàn
bộ nhớ. Bạn vẫn sẽ nhận được một mức độ an toàn nhất định bên trong một khối unsafe.

Ngoài ra, `unsafe` không có nghĩa là mã nguồn bên trong khối nhất thiết phải
nguy hiểm hoặc chắc chắn sẽ gặp các vấn đề về an toàn bộ nhớ: mục đích là
với tư cách là lập trình viên, bạn sẽ đảm bảo mã nguồn bên trong một khối `unsafe` sẽ
truy cập bộ nhớ theo cách hợp lệ.

Con người có thể sai lầm và lỗi sẽ xảy ra, nhưng bằng cách yêu cầu năm thao tác
unsafe này phải nằm bên trong các khối được chú thích bằng `unsafe`, bạn sẽ biết rằng
bất kỳ lỗi nào liên quan đến an toàn bộ nhớ phải nằm trong một khối `unsafe`. Hãy giữ
các khối `unsafe` nhỏ; bạn sẽ thấy biết ơn sau này khi điều tra các lỗi bộ
nhớ.

Để cô lập mã unsafe nhiều nhất có thể, tốt nhất là bao bọc mã đó
bên trong một trừu tượng an toàn và cung cấp một API an toàn, điều mà chúng ta sẽ thảo luận sau trong
chương này khi chúng ta xem xét các hàm và phương thức unsafe. Các phần của thư viện chuẩn
được triển khai dưới dạng các trừu tượng an toàn trên mã unsafe đã được
kiểm duyệt. Việc bao bọc mã unsafe trong một trừu tượng an toàn giúp ngăn chặn việc sử dụng `unsafe`
rò rỉ ra tất cả những nơi mà bạn hoặc người dùng của bạn có thể muốn sử dụng
chức năng được triển khai bằng mã `unsafe`, bởi vì việc sử dụng một trừu tượng
an toàn là an toàn.

Hãy lần lượt xem xét từng siêu năng lực trong số năm siêu năng lực của unsafe. Chúng ta cũng sẽ xem xét
một số trừu tượng cung cấp một giao diện an toàn cho mã unsafe.

### Giải tham chiếu một con trỏ thô

Trong [“Trình kiểm tra mượn tìm thấy các vi phạm quyền”][permission-violations]<!-- ignore --> ở Chương 4, chúng ta
đã mô tả cách trình biên dịch đảm bảo các tham chiếu luôn hợp lệ. Unsafe Rust có
hai kiểu mới được gọi là _con trỏ thô_ (raw pointers) tương tự như các tham chiếu. Giống như
các tham chiếu, con trỏ thô có thể là bất biến hoặc có thể thay đổi và được viết là `*const
T` và `*mut T`, tương ứng. Dấu hoa thị không phải là toán tử giải tham chiếu; nó là
một phần của tên kiểu. Trong ngữ cảnh của con trỏ thô, _bất biến_ có nghĩa là
con trỏ không thể được gán trực tiếp sau khi được giải tham chiếu.

Khác với các tham chiếu và con trỏ thông minh (smart pointers), con trỏ thô:

- Được phép bỏ qua các quy tắc mượn bằng cách có cả con trỏ bất biến và
  con trỏ có thể thay đổi hoặc nhiều con trỏ có thể thay đổi trỏ đến cùng một vị trí
- Không được đảm bảo trỏ đến bộ nhớ hợp lệ
- Được phép là null
- Không thực hiện bất kỳ việc tự động dọn dẹp nào

Bằng cách chọn không để Rust thực thi các đảm bảo này, bạn có thể từ bỏ
sự an toàn được đảm bảo để đổi lấy hiệu suất cao hơn hoặc khả năng
giao tiếp với một ngôn ngữ hoặc phần cứng khác nơi các đảm bảo của Rust không áp dụng.

Danh sách 20-1 cho thấy cách tạo một con trỏ thô bất biến và một con trỏ thô có thể thay đổi.

<Listing number="20-1" caption="Tạo các con trỏ thô bằng các toán tử mượn thô (raw borrow operators)">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta không bao gồm từ khóa `unsafe` trong mã này. Chúng ta có thể tạo
các con trỏ thô trong mã an toàn; chúng ta chỉ không thể giải tham chiếu các con trỏ thô bên ngoài một
khối unsafe, như bạn sẽ thấy trong giây lát.

Chúng ta đã tạo các con trỏ thô bằng cách sử dụng các toán tử mượn thô: `&raw const num`
tạo một con trỏ thô bất biến `*const i32`, và `&raw mut num` tạo một con trỏ thô
có thể thay đổi `*mut i32`. Bởi vì chúng ta đã tạo chúng trực tiếp từ một biến
cục bộ, chúng ta biết những con trỏ thô cụ thể này là hợp lệ, nhưng chúng ta không thể đưa ra
giả định đó về bất kỳ con trỏ thô nào.

Để chứng minh điều này, tiếp theo chúng ta sẽ tạo một con trỏ thô mà tính hợp lệ của nó chúng ta không thể
chắc chắn, bằng cách sử dụng `as` để ép kiểu một giá trị thay vì sử dụng các toán tử mượn
thô. Danh sách 20-2 cho thấy cách tạo một con trỏ thô đến một vị trí bất kỳ
trong bộ nhớ. Việc cố gắng sử dụng bộ nhớ tùy ý là không xác định (undefined): có thể có
dữ liệu tại địa chỉ đó hoặc có thể không, trình biên dịch có thể tối ưu hóa mã nguồn để
không có truy cập bộ nhớ nào, hoặc chương trình có thể kết thúc với một lỗi phân đoạn (segmentation
fault). Thông thường, không có lý do chính đáng nào để viết mã như thế này, đặc biệt là trong
những trường hợp bạn có thể sử dụng một toán tử mượn thô để thay thế, nhưng nó là khả thi.

<Listing number="20-2" caption="Tạo một con trỏ thô đến một địa chỉ bộ nhớ tùy ý">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Hãy nhớ rằng chúng ta có thể tạo các con trỏ thô trong mã an toàn, nhưng chúng ta không thể _giải tham chiếu_
các con trỏ thô và đọc dữ liệu đang được trỏ tới. Trong Danh sách 20-3, chúng ta sử dụng
toán tử giải tham chiếu `*` trên một con trỏ thô, việc này yêu cầu một khối `unsafe`.

<Listing number="20-3" caption="Giải tham chiếu các con trỏ thô bên trong một khối `unsafe` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Việc tạo một con trỏ không gây hại; chỉ khi chúng ta cố gắng truy cập giá trị mà
nó trỏ tới thì chúng ta mới có thể phải đối mặt với một giá trị không hợp lệ.

Cũng lưu ý rằng trong Danh sách 20-1 và 20-3, chúng đã tạo các con trỏ thô `*const i32` và `*mut i32`
cùng trỏ đến một vị trí bộ nhớ, nơi `num` được
lưu trữ. Nếu thay vào đó chúng ta cố gắng tạo một tham chiếu bất biến và một tham chiếu có thể thay đổi cho
`num`, mã nguồn sẽ không được biên dịch vì các quy tắc sở hữu của Rust không
cho phép một tham chiếu có thể thay đổi cùng lúc với bất kỳ tham chiếu bất biến nào. Với
các con trỏ thô, chúng ta có thể tạo một con trỏ có thể thay đổi và một con trỏ bất biến đến
cùng một vị trí và thay đổi dữ liệu thông qua con trỏ có thể thay đổi, có khả năng tạo ra
một cuộc đua dữ liệu (data race). Hãy cẩn thận!

Với tất cả những nguy hiểm này, tại sao bạn lại sử dụng con trỏ thô? Một trường hợp sử dụng chính
là khi giao tiếp với mã C, như bạn sẽ thấy trong phần tiếp theo,
[“Gọi một hàm hoặc
phương thức unsafe.”](#calling-an-unsafe-function-or-method)<!-- ignore --> Một trường hợp khác là
khi xây dựng các trừu tượng an toàn mà trình kiểm tra mượn không hiểu.
Chúng ta sẽ giới thiệu các hàm unsafe và sau đó xem xét một ví dụ về một trừu tượng
an toàn sử dụng mã unsafe.

### Gọi một hàm hoặc phương thức unsafe

Loại thao tác thứ hai bạn có thể thực hiện trong một khối unsafe là gọi
các hàm unsafe. Các hàm và phương thức unsafe trông hoàn toàn giống như các hàm và phương thức
thông thường, nhưng chúng có thêm một từ khóa `unsafe` trước phần còn lại của
định nghĩa. Từ khóa `unsafe` trong ngữ cảnh này cho biết hàm có
các yêu cầu mà chúng ta cần duy trì khi gọi hàm này, bởi vì Rust không thể
đảm bảo chúng ta đã đáp ứng các yêu cầu này. Bằng cách gọi một hàm unsafe bên trong một
khối `unsafe`, chúng ta đang nói rằng chúng ta đã đọc tài liệu của hàm này và
chúng ta chịu trách nhiệm duy trì các hợp đồng của hàm.

Đây là một hàm unsafe tên là `dangerous` không làm gì trong
thân hàm của nó:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Chúng ta phải gọi hàm `dangerous` bên trong một khối `unsafe` riêng biệt. Nếu chúng ta
cố gắng gọi `dangerous` mà không có khối `unsafe`, chúng ta sẽ gặp lỗi:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Với khối `unsafe`, chúng ta đang khẳng định với Rust rằng chúng ta đã đọc tài liệu của
hàm, chúng ta hiểu cách sử dụng nó đúng cách, và chúng ta đã xác minh rằng
chúng ta đang thực hiện đúng hợp đồng của hàm.

Để thực hiện các thao tác unsafe trong thân của một hàm unsafe, bạn vẫn cần
sử dụng một khối `unsafe`, giống như bên trong một hàm thông thường, và trình biên dịch
sẽ cảnh báo bạn nếu bạn quên. Điều này giúp giữ cho các khối `unsafe` nhỏ nhất
có thể, vì các thao tác unsafe có thể không cần thiết trong toàn bộ thân
hàm.

#### Tạo một trừu tượng an toàn trên mã unsafe

Chỉ vì một hàm chứa mã unsafe không có nghĩa là chúng ta cần đánh dấu
toàn bộ hàm là unsafe. Trên thực tế, việc bao bọc mã unsafe trong một hàm an toàn là
một sự trừu tượng phổ biến. Ví dụ, hãy nghiên cứu hàm `split_at_mut`
từ thư viện chuẩn, vốn yêu cầu một số mã unsafe. Chúng ta sẽ khám phá cách
chúng ta có thể triển khai nó. Phương thức an toàn này được định nghĩa trên các slice có thể thay đổi: nó nhận
một slice và tạo thành hai bằng cách chia slice tại chỉ số (index) được đưa vào như một
đối số. Danh sách 20-4 cho thấy cách sử dụng `split_at_mut`.

<Listing number="20-4" caption="Sử dụng hàm `split_at_mut` an toàn">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Chúng ta không thể triển khai hàm này chỉ bằng cách sử dụng Rust an toàn. Một nỗ lực có thể trông
giống như Danh sách 20-5, cái mà sẽ không được biên dịch. Để đơn giản, chúng ta sẽ
triển khai `split_at_mut` dưới dạng một hàm thay vì một phương thức và chỉ dành cho các slice
của các giá trị `i32` thay vì cho một kiểu generic `T`.

<Listing number="20-5" caption="Một nỗ lực triển khai `split_at_mut` chỉ sử dụng Rust an toàn">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Hàm này trước tiên lấy tổng độ dài của slice. Sau đó, nó xác nhận (assert) rằng
chỉ số được đưa vào như một tham số nằm trong slice bằng cách kiểm tra xem nó có
nhỏ hơn hoặc bằng độ dài hay không. Việc xác nhận có nghĩa là nếu chúng ta truyền một chỉ số
lớn hơn độ dài để chia slice, hàm sẽ panic
trước khi nó cố gắng sử dụng chỉ số đó.

Sau đó, chúng ta trả về hai slice có thể thay đổi trong một tuple: một từ đầu của
slice ban đầu đến chỉ số `mid` và một slice khác từ `mid` đến cuối của
slice.

Khi chúng ta cố gắng biên dịch mã trong Danh sách 20-5, chúng ta sẽ nhận được một lỗi.

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Trình kiểm tra mượn của Rust không thể hiểu rằng chúng ta đang mượn các phần khác nhau của
slice; nó chỉ biết rằng chúng ta đang mượn từ cùng một slice hai lần.
Việc mượn các phần khác nhau của một slice về cơ bản là ổn vì hai
slice không chồng lấn lên nhau, nhưng Rust không đủ thông minh để biết điều này. Khi chúng ta
biết mã nguồn là ổn, nhưng Rust thì không, đó là lúc cần tìm đến mã unsafe.

Danh sách 20-6 cho thấy cách sử dụng một khối `unsafe`, một con trỏ thô, và một số lời gọi
đến các hàm unsafe để làm cho việc triển khai `split_at_mut` hoạt động.

<Listing number="20-6" caption="Sử dụng mã unsafe trong việc triển khai hàm `split_at_mut` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Hãy nhớ lại từ [“Kiểu Slice”][the-slice-type]<!-- ignore --> ở Chương 4 rằng
các slice là một con trỏ đến một số dữ liệu và độ dài của slice. Chúng ta sử dụng phương thức `len`
để lấy độ dài của một slice và phương thức `as_mut_ptr` để truy cập
con trỏ thô của một slice. Trong trường hợp này, vì chúng ta có một slice có thể thay đổi cho các giá trị `i32`
, `as_mut_ptr` trả về một con trỏ thô với kiểu `*mut i32`, cái mà chúng ta đã
lưu trữ trong biến `ptr`.

Chúng ta giữ nguyên phần xác nhận rằng chỉ số `mid` nằm trong slice. Sau đó, chúng ta đi đến
mã unsafe: hàm `slice::from_raw_parts_mut` nhận một con trỏ thô
và một độ dài, và nó tạo ra một slice. Chúng ta sử dụng nó để tạo ra một slice bắt đầu
từ `ptr` và dài `mid` phần tử. Sau đó, chúng ta gọi phương thức `add` trên `ptr` với
`mid` như một đối số để lấy một con trỏ thô bắt đầu tại `mid`, và chúng ta tạo ra một
slice bằng cách sử dụng con trỏ đó và số lượng phần tử còn lại sau `mid` làm
độ dài.

Hàm `slice::from_raw_parts_mut` là unsafe bởi vì nó nhận một
con trỏ thô và phải tin rằng con trỏ này là hợp lệ. Phương thức `add` trên các con trỏ
thô cũng là unsafe bởi vì nó phải tin rằng vị trí offset cũng
là một con trỏ hợp lệ. Do đó, chúng ta đã phải đặt một khối `unsafe` xung quanh các lời gọi của chúng ta đến
`slice::from_raw_parts_mut` và `add` để chúng ta có thể gọi chúng. Bằng cách xem xét
mã nguồn và bằng cách thêm phần xác nhận rằng `mid` phải nhỏ hơn hoặc bằng
`len`, chúng ta có thể khẳng định rằng tất cả các con trỏ thô được sử dụng bên trong khối `unsafe`
sẽ là các con trỏ hợp lệ đến dữ liệu bên trong slice. Đây là một cách sử dụng `unsafe`
chấp nhận được và phù hợp.

Lưu ý rằng chúng ta không cần đánh dấu hàm `split_at_mut` kết quả là
`unsafe`, và chúng ta có thể gọi hàm này từ Rust an toàn. Chúng ta đã tạo ra một
trừu tượng an toàn cho mã unsafe với một triển khai của hàm có sử dụng
mã `unsafe` theo cách an toàn, bởi vì nó chỉ tạo ra các con trỏ hợp lệ từ
dữ liệu mà hàm này có quyền truy cập.

Ngược lại, việc sử dụng `slice::from_raw_parts_mut` trong Danh sách 20-7 có thể
sẽ gây crash khi slice được sử dụng. Mã này lấy một vị trí bộ nhớ tùy ý
và tạo ra một slice dài 10.000 phần tử.

<Listing number="20-7" caption="Tạo một slice từ một vị trí bộ nhớ tùy ý">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Chúng ta không sở hữu bộ nhớ tại vị trí tùy ý này, và không có gì đảm bảo
rằng slice mà mã này tạo ra chứa các giá trị `i32` hợp lệ. Việc cố gắng sử dụng
`values` như thể nó là một slice hợp lệ sẽ dẫn đến hành vi không xác định (undefined behavior).

#### Sử dụng các hàm `extern` để gọi mã bên ngoài

Đôi khi, mã Rust của bạn có thể cần tương tác với mã được viết bằng một ngôn ngữ khác.
Đối với việc này, Rust có từ khóa `extern` tạo điều kiện thuận lợi cho việc tạo ra
và sử dụng một _Foreign Function Interface (FFI)_ (Giao diện hàm ngoại). Một FFI là một cách để một
ngôn ngữ lập trình định nghĩa các hàm và cho phép một ngôn ngữ lập trình khác (ngoại lai)
gọi những hàm đó.

Danh sách 20-8 minh họa cách thiết lập tích hợp với hàm `abs`
từ thư viện chuẩn C. Các hàm được khai báo bên trong các khối `extern` thường
là không an toàn để gọi từ mã Rust, vì vậy các khối `extern` cũng phải được đánh dấu
`unsafe`. Lý do là các ngôn ngữ khác không thực thi các quy tắc và
đảm bảo của Rust, và Rust không thể kiểm tra chúng, vì vậy trách nhiệm thuộc về lập trình viên
để đảm bảo an toàn.

<Listing number="20-8" file-name="src/main.rs" caption="Khai báo và gọi một hàm `extern` được định nghĩa trong một ngôn ngữ khác">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

Bên trong khối `unsafe extern "C"`, chúng ta liệt kê tên và chữ ký của
các hàm bên ngoài từ một ngôn ngữ khác mà chúng ta muốn gọi. Phần `"C"` xác định
_giao diện nhị phân ứng dụng (application binary interface - ABI)_ nào mà hàm bên ngoài sử dụng: ABI
định nghĩa cách gọi hàm ở cấp độ assembly. ABI `"C"` là
phổ biến nhất và tuân theo ABI của ngôn ngữ lập trình C. Thông tin về tất cả
các ABI mà Rust hỗ trợ có sẵn trong [Tài liệu tham khảo Rust][ABI].

Mọi mục được khai báo bên trong một khối `unsafe extern` đều ngầm định là `unsafe`.
Tuy nhiên, một số hàm FFI _là_ an toàn để gọi. Ví dụ, hàm `abs`
từ thư viện chuẩn của C không có bất kỳ cân nhắc nào về an toàn bộ nhớ và chúng ta
biết nó có thể được gọi với bất kỳ `i32` nào. Trong những trường hợp như thế này, chúng ta có thể sử dụng từ khóa `safe`
để nói rằng hàm cụ thể này là an toàn để gọi mặc dù nó nằm trong
một khối `unsafe extern`. Một khi chúng ta thực hiện thay đổi đó, việc gọi nó không còn
yêu cầu một khối `unsafe` nữa, như được trình bày trong Danh sách 20-9.

<Listing number="20-9" file-name="src/main.rs" caption="Đánh dấu rõ ràng một hàm là `safe` bên trong một khối `unsafe extern` và gọi nó một cách an toàn">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Việc đánh dấu một hàm là `safe` không vốn dĩ làm cho nó an toàn! Thay vào đó, nó
giống như một lời hứa bạn đang thực hiện với Rust rằng nó _thật sự_ an toàn. Đó vẫn là
trách nhiệm của bạn để đảm bảo lời hứa đó được giữ vững!

> #### Gọi các hàm Rust từ các ngôn ngữ khác
>
> Chúng ta cũng có thể sử dụng `extern` để tạo một giao diện cho phép các ngôn ngữ khác
> gọi các hàm Rust. Thay vì tạo toàn bộ khối `extern`, chúng ta thêm từ khóa
> `extern` và chỉ định ABI sẽ sử dụng ngay trước từ khóa `fn` cho
> hàm có liên quan. Chúng ta cũng cần thêm một chú thích `#[unsafe(no_mangle)]`
> để yêu cầu trình biên dịch Rust không làm xáo trộn (mangle) tên của hàm này.
> _Mangling_ là khi một trình biên dịch thay đổi tên chúng ta đã đặt cho một hàm thành một
> tên khác chứa nhiều thông tin hơn cho các phần khác của
> quá trình biên dịch tiêu thụ nhưng ít dễ đọc hơn đối với con người. Mọi ngôn ngữ lập trình
> trình biên dịch ngôn ngữ đều xáo trộn tên hơi khác nhau, vì vậy để một hàm Rust
> có thể được đặt tên bởi các ngôn ngữ khác, chúng ta phải vô hiệu hóa việc xáo trộn tên của
> trình biên dịch Rust. Việc này là unsafe bởi vì có thể có sự xung đột tên giữa
> các thư viện nếu không có tính năng xáo trộn tích hợp, vì vậy trách nhiệm của chúng ta là đảm bảo
> rằng tên chúng ta chọn là an toàn để xuất ra mà không cần xáo trộn.
>
> Trong ví dụ sau, chúng ta làm cho hàm `call_from_c` có thể truy cập được từ
> mã C, sau khi nó được biên dịch thành một thư viện chia sẻ (shared library) và được liên kết từ C:
>
> ```rust
> #[unsafe(no_mangle)]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> Cách sử dụng `extern` này chỉ yêu cầu `unsafe` trong thuộc tính (attribute), không phải trên
> khối `extern`.

### Truy cập hoặc Sửa đổi một Biến tĩnh có thể thay đổi

Trong cuốn sách này, chúng ta vẫn chưa nói về các biến toàn cục (global variables), cái mà Rust có
hỗ trợ nhưng có thể gây rắc rối với các quy tắc sở hữu của Rust. Nếu hai luồng (thread)
đang truy cập cùng một biến toàn cục có thể thay đổi, nó có thể gây ra một cuộc đua dữ liệu.

Trong Rust, các biến toàn cục được gọi là các biến _tĩnh_ (static). Danh sách 20-10 cho thấy một
ví dụ về khai báo và sử dụng một biến tĩnh với một string slice làm
giá trị.

<Listing number="20-10" file-name="src/main.rs" caption="Định nghĩa và sử dụng một biến tĩnh bất biến">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Các biến tĩnh tương tự như các hằng số (constants), điều mà chúng ta đã thảo luận trong
[“Hằng số”][differences-between-variables-and-constants]<!-- ignore --> ở
Chương 3. Tên của các biến tĩnh theo quy ước là `SCREAMING_SNAKE_CASE`.
Các biến tĩnh chỉ có thể lưu trữ các tham chiếu với lifetime `'static`
, có nghĩa là trình biên dịch Rust có thể tự tìm ra lifetime và chúng ta
không bắt buộc phải chú thích nó một cách rõ ràng. Việc truy cập một biến tĩnh
bất biến là an toàn.

Một sự khác biệt nhỏ giữa các hằng số và các biến tĩnh bất biến là
các giá trị trong một biến tĩnh có một địa chỉ cố định trong bộ nhớ. Việc sử dụng giá trị
sẽ luôn truy cập vào cùng một dữ liệu. Mặt khác, các hằng số được phép
sao chép dữ liệu của chúng bất cứ khi nào chúng được sử dụng. Một sự khác biệt khác là các biến tĩnh
có thể thay đổi được. Việc truy cập và sửa đổi các biến tĩnh có thể thay đổi là
_unsafe_. Danh sách 20-11 cho thấy cách khai báo, truy cập và sửa đổi một biến tĩnh
có thể thay đổi tên là `COUNTER`.

<Listing number="20-11" file-name="src/main.rs" caption="Đọc từ hoặc ghi vào một biến tĩnh có thể thay đổi là unsafe">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Giống như các biến thông thường, chúng ta chỉ định khả năng thay đổi bằng cách sử dụng từ khóa `mut`. Bất kỳ
mã nguồn nào đọc hoặc ghi từ `COUNTER` đều phải nằm trong một khối `unsafe`. Mã
này biên dịch và in ra `COUNTER: 3` như chúng ta mong đợi vì nó là đơn
luồng. Việc có nhiều luồng truy cập `COUNTER` có khả năng dẫn đến các cuộc đua
dữ liệu, vì vậy nó là hành vi không xác định. Do đó, chúng ta cần đánh dấu toàn bộ
hàm là `unsafe`, và ghi lại các hạn chế về an toàn, để bất kỳ ai gọi
hàm đều biết những gì họ được và không được phép làm một cách an toàn.

Bất cứ khi nào chúng ta viết một hàm unsafe, theo quy ước là nên viết một bình luận
bắt đầu bằng `SAFETY` và giải thích những gì người gọi cần làm để gọi
hàm một cách an toàn. Tương tự, bất cứ khi nào chúng ta thực hiện một thao tác unsafe, theo
quy ước là nên viết một bình luận bắt đầu bằng `SAFETY` để giải thích cách các quy tắc
an toàn được duy trì.

Ngoài ra, trình biên dịch sẽ không cho phép bạn tạo các tham chiếu đến một biến tĩnh
có thể thay đổi. Bạn chỉ có thể truy cập nó thông qua một con trỏ thô, được tạo bằng một trong
các toán tử mượn thô. Điều đó bao gồm cả những trường hợp mà tham chiếu được tạo ra
vô hình, chẳng hạn như khi nó được sử dụng trong `println!` trong danh sách mã này. Yêu
cầu rằng các tham chiếu đến các biến tĩnh có thể thay đổi chỉ có thể được tạo thông qua
các con trỏ thô giúp làm cho các yêu cầu an toàn khi sử dụng chúng trở nên rõ ràng hơn.

Với dữ liệu có thể thay đổi có thể truy cập toàn cục, thật khó để đảm bảo không có
cuộc đua dữ liệu nào, đó là lý do tại sao Rust coi các biến tĩnh có thể thay đổi là
unsafe. Khi có thể, tốt hơn là sử dụng các kỹ thuật đồng thời và
các con trỏ thông minh an toàn với luồng mà chúng ta đã thảo luận trong Chương 16 để trình biên dịch kiểm tra
rằng việc truy cập dữ liệu từ các luồng khác nhau được thực hiện một cách an toàn.

### Thực thi một Trait Unsafe

Chúng ta có thể sử dụng `unsafe` để thực thi một trait unsafe. Một trait là unsafe khi có ít nhất
một trong các phương thức của nó có một số bất biến (invariant) mà trình biên dịch không thể xác minh.
Chúng ta khai báo rằng một trait là `unsafe` bằng cách thêm từ khóa `unsafe` trước `trait`
và cũng đánh dấu việc thực thi trait đó là `unsafe`, như được trình bày trong
Danh sách 20-12.

<Listing number="20-12" caption="Định nghĩa và thực thi một trait unsafe">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

Bằng cách sử dụng `unsafe impl`, chúng ta đang hứa rằng chúng ta sẽ duy trì các bất biến mà
trình biên dịch không thể xác minh.

Ví dụ, hãy nhớ lại các marker trait `Sync` và `Send` mà chúng ta đã thảo luận trong
[“Đồng thời có thể mở rộng với các trait `Sync` và `Send`
Traits”][extensible-concurrency-with-the-sync-and-send-traits]<!-- ignore --> ở
Chương 16: trình biên dịch tự động thực thi các trait này nếu các kiểu của chúng ta được
cấu thành hoàn toàn từ các kiểu khác có thực thi `Send` và `Sync`. Nếu chúng ta
thực thi một kiểu có chứa một kiểu không thực thi `Send` hoặc `Sync`,
chẳng hạn như các con trỏ thô, và chúng ta muốn đánh dấu kiểu đó là `Send` hoặc `Sync`, chúng ta phải
sử dụng `unsafe`. Rust không thể xác minh rằng kiểu của chúng ta duy trì các đảm bảo rằng nó có thể
được gửi an toàn giữa các luồng hoặc được truy cập từ nhiều luồng; do đó, chúng ta
cần thực hiện các kiểm tra đó một cách thủ công và chỉ định như vậy bằng `unsafe`.

### Truy cập các trường của một Union

Hành động cuối cùng chỉ hoạt động với `unsafe` là truy cập các trường của một union. Một
`union` tương tự như một `struct`, nhưng chỉ một trường được khai báo được sử dụng trong một
thực thể (instance) cụ thể tại một thời điểm. Các union chủ yếu được sử dụng để giao tiếp với
các union trong mã C. Việc truy cập các trường của union là unsafe bởi vì Rust không thể đảm bảo
kiểu dữ liệu hiện đang được lưu trữ trong thực thể union. Bạn có thể tìm hiểu
thêm về union trong [Tài liệu tham khảo Rust][unions].

### Sử dụng Miri để kiểm tra mã Unsafe

Khi viết mã unsafe, bạn có thể muốn kiểm tra xem những gì bạn đã viết
thực sự an toàn và chính xác hay không. Một trong những cách tốt nhất để làm điều đó là sử dụng
Miri, một công cụ chính thức của Rust để phát hiện hành vi không xác định. Trong khi
trình kiểm tra mượn là một công cụ _tĩnh_ hoạt động tại thời điểm biên dịch, Miri là một
công cụ _động_ hoạt động tại thời điểm thực thi. Nó kiểm tra mã của bạn bằng cách chạy
chương trình của bạn, hoặc bộ test của nó, và phát hiện khi bạn vi phạm các quy tắc mà nó
hiểu về cách Rust nên hoạt động.

Việc sử dụng Miri yêu cầu một bản build nightly của Rust (điều mà chúng ta nói nhiều hơn trong
[Phụ lục G: Rust được tạo ra như thế nào và “Nightly Rust”][nightly]). Bạn có thể cài đặt
cả phiên bản nightly của Rust và công cụ Miri bằng cách gõ `rustup +nightly
component add miri`. Việc này không thay đổi phiên bản Rust mà dự án của bạn
sử dụng; nó chỉ thêm công cụ vào hệ thống của bạn để bạn có thể sử dụng khi muốn.
Bạn có thể chạy Miri trên một dự án bằng cách gõ `cargo +nightly miri run` hoặc `cargo
+nightly miri test`.

Để thấy một ví dụ về việc công cụ này hữu ích như thế nào, hãy xem xét điều gì xảy ra khi chúng ta chạy nó
với Danh sách 20-11.

```console
{{#include ../listings/ch20-advanced-features/listing-20-11/output.txt}}
```

Miri cảnh báo chính xác rằng chúng ta có các tham chiếu dùng chung cho dữ liệu có thể thay đổi. Ở đây,
Miri chỉ đưa ra một cảnh báo vì điều này không được đảm bảo là hành vi không xác định
trong trường hợp này, và nó không cho chúng ta biết cách khắc phục vấn đề. Nhưng ít
nhất chúng ta biết có nguy cơ xảy ra hành vi không xác định và có thể suy nghĩ về cách
làm cho mã nguồn an toàn. Trong một số trường hợp, Miri cũng có thể phát hiện các lỗi hoàn toàn—các mẫu mã
chắc chắn là sai—và đưa ra các khuyến nghị về cách khắc phục
những lỗi đó.

Miri không bắt được mọi thứ bạn có thể làm sai khi viết mã unsafe. Miri
là một công cụ phân tích động, vì vậy nó chỉ bắt được các vấn đề với mã nguồn thực sự
được chạy. Điều đó có nghĩa là bạn sẽ cần sử dụng nó kết hợp với các kỹ thuật kiểm thử tốt
để tăng sự tự tin của bạn về mã unsafe mà bạn đã viết.
Miri cũng không bao quát mọi cách có thể khiến mã của bạn không an toàn (unsound).

Nói cách khác: Nếu Miri _thực sự_ bắt được một vấn đề, bạn biết là có bug, nhưng
chỉ vì Miri _không_ bắt được bug không có nghĩa là không có vấn đề gì. Nó
tuy nhiên có thể bắt được rất nhiều. Hãy thử chạy nó trên các ví dụ khác về mã unsafe trong
chương này và xem nó nói gì!

Bạn có thể tìm hiểu thêm về Miri tại [kho lưu trữ GitHub của nó][miri].

### Khi nào nên sử dụng mã Unsafe

Việc sử dụng `unsafe` để sử dụng một trong năm siêu năng lực vừa được thảo luận
không phải là sai hay thậm chí là bị phản đối, nhưng việc viết mã `unsafe`
chính xác thì khó khăn hơn vì trình biên dịch không thể giúp duy trì an toàn bộ nhớ. Khi bạn có một
lý do để sử dụng mã `unsafe`, bạn có thể làm như vậy, và việc có chú thích `unsafe` rõ ràng
giúp việc truy vết nguồn gốc của các vấn đề khi chúng xảy ra trở nên dễ dàng hơn.
Bất cứ khi nào bạn viết mã unsafe, bạn có thể sử dụng Miri để giúp bạn tự tin hơn
rằng mã bạn đã viết tuân thủ các quy tắc của Rust.

Để khám phá sâu hơn về cách làm việc hiệu quả với unsafe Rust, hãy đọc
hướng dẫn chính thức của Rust về chủ đề này, [Rustonomicon][nomicon].

{{#quiz ../quizzes/ch19-01-unsafe-rust.toml}}

[permission-violations]: ch04-02-references-and-borrowing.html#the-borrow-checker-finds-permission-violations
[ABI]: ../reference/items/external-blocks.html#abi
[differences-between-variables-and-constants]: ch03-01-variables-and-mutability.html#constants
[extensible-concurrency-with-the-sync-and-send-traits]: ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits
[the-slice-type]: ch04-04-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
