## Đường dẫn để Tham chiếu đến một Mục trong Cây Module (Paths for Referring to an Item in the Module Tree)

Để chỉ cho Rust nơi tìm một mục trong cây module, chúng ta sử dụng một đường dẫn (path) theo cùng
cách chúng ta sử dụng một đường dẫn khi điều hướng trong một hệ thống tệp. Để gọi một hàm, chúng ta cần
biết đường dẫn của nó.

Một đường dẫn có thể có hai dạng:

- Một _đường dẫn tuyệt đối_ (absolute path) là đường dẫn đầy đủ bắt đầu từ một crate root; đối với mã nguồn
  từ một crate bên ngoài, đường dẫn tuyệt đối bắt đầu bằng tên crate, và đối với
  mã nguồn từ crate hiện tại, nó bắt đầu bằng từ khóa `crate`.
- Một _đường dẫn tương đối_ (relative path) bắt đầu từ module hiện tại và sử dụng `self`, `super`, hoặc
  một mã định danh (identifier) trong module hiện tại.

Cả đường dẫn tuyệt đối và tương đối đều được theo sau bởi một hoặc nhiều mã định danh
phân cách bởi dấu hai chấm kép (`::`).

Quay lại Listing 7-1, giả sử chúng ta muốn gọi hàm `add_to_waitlist`.
Điều này cũng giống như việc hỏi: đường dẫn của hàm `add_to_waitlist` là gì?
Listing 7-3 chứa Listing 7-1 với một số module và hàm đã được lược bỏ.

Chúng ta sẽ chỉ ra hai cách để gọi hàm `add_to_waitlist` từ một hàm mới,
`eat_at_restaurant`, được định nghĩa trong crate root. Các đường dẫn này là chính xác, nhưng
vẫn còn một vấn đề khác sẽ ngăn ví dụ này biên dịch
như hiện tại. Chúng ta sẽ giải thích lý do ngay sau đây.

Hàm `eat_at_restaurant` là một phần của API công khai của library crate của chúng ta, vì vậy
chúng ta đánh dấu nó bằng từ khóa `pub`. Trong phần [“Công khai các Đường dẫn với Từ khóa
`pub`”][pub]<!-- ignore -->, chúng ta sẽ đi vào chi tiết hơn về `pub`.

<Listing number="7-3" file-name="src/lib.rs" caption="Gọi hàm `add_to_waitlist` bằng cách sử dụng đường dẫn tuyệt đối và tương đối">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

Lần đầu tiên chúng ta gọi hàm `add_to_waitlist` trong `eat_at_restaurant`,
chúng ta sử dụng một đường dẫn tuyệt đối. Hàm `add_to_waitlist` được định nghĩa trong cùng
crate với `eat_at_restaurant`, có nghĩa là chúng ta có thể sử dụng từ khóa `crate` để
bắt đầu một đường dẫn tuyệt đối. Sau đó, chúng ta bao gồm từng module kế tiếp cho đến khi
đến được `add_to_waitlist`. Bạn có thể tưởng tượng một hệ thống tệp với cùng
cấu trúc: chúng ta sẽ chỉ định đường dẫn `/front_of_house/hosting/add_to_waitlist` để
chạy chương trình `add_to_waitlist`; việc sử dụng tên `crate` để bắt đầu từ
crate root giống như việc sử dụng `/` để bắt đầu từ gốc của hệ thống tệp trong shell của bạn.

Lần thứ hai chúng ta gọi `add_to_waitlist` trong `eat_at_restaurant`, chúng ta sử dụng một
đường dẫn tương đối. Đường dẫn bắt đầu bằng `front_of_house`, tên của module
được định nghĩa cùng cấp độ trong cây module với `eat_at_restaurant`. Ở đây tương đương với
hệ thống tệp sẽ là sử dụng đường dẫn
`front_of_house/hosting/add_to_waitlist`. Bắt đầu bằng một tên module có nghĩa
rằng đường dẫn đó là tương đối.

Việc chọn sử dụng đường dẫn tương đối hay tuyệt đối là một quyết định bạn sẽ thực hiện
dựa trên dự án của mình, và nó phụ thuộc vào việc bạn có nhiều khả năng di chuyển
mã định nghĩa mục riêng biệt hay cùng với mã nguồn sử dụng mục đó. Ví dụ, nếu chúng ta di chuyển module `front_of_house` và
hàm `eat_at_restaurant` vào một module tên là `customer_experience`, chúng ta
cần cập nhật đường dẫn tuyệt đối đến `add_to_waitlist`, nhưng đường dẫn tương đối
vẫn hợp lệ. Tuy nhiên, nếu chúng ta di chuyển hàm `eat_at_restaurant`
riêng biệt vào một module tên là `dining`, đường dẫn tuyệt đối đến lời gọi
`add_to_waitlist` vẫn giữ nguyên, nhưng đường dẫn tương đối sẽ cần được
cập nhật. Sở thích của chúng tôi nói chung là chỉ định các đường dẫn tuyệt đối vì có
nhiều khả năng chúng ta sẽ muốn di chuyển các định nghĩa mã nguồn và các lời gọi mục một cách độc lập
với nhau.

Hãy thử biên dịch Listing 7-3 và tìm hiểu tại sao nó vẫn chưa biên dịch được! Các
lỗi chúng ta nhận được hiển thị trong Listing 7-4.

<Listing number="7-4" caption="Lỗi trình biên dịch khi xây dựng mã nguồn trong Listing 7-3">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

Các thông báo lỗi nói rằng module `hosting` là riêng tư (private). Nói cách khác, chúng ta
có các đường dẫn chính xác cho module `hosting` và hàm `add_to_waitlist`,
nhưng Rust không cho phép chúng ta sử dụng chúng vì nó không có quyền truy cập vào các
phần riêng tư. Trong Rust, tất cả các mục (hàm, phương thức, struct, enum,
module và hằng số) đều là riêng tư đối với các module cha theo mặc định. Nếu bạn muốn
làm cho một mục như một hàm hoặc struct trở nên riêng tư, bạn đặt nó vào một module.

Các mục trong một module cha không thể sử dụng các mục riêng tư bên trong các module con, nhưng
các mục trong các module con có thể sử dụng các mục trong các module tổ tiên của chúng. Điều này
là do các module con bao bọc và ẩn đi các chi tiết triển khai của chúng, nhưng các
module con có thể nhìn thấy ngữ cảnh mà chúng được định nghĩa. Để tiếp tục với phép
ẩn dụ của chúng ta, hãy nghĩ về các quy tắc riêng tư giống như văn phòng phía sau của một
nhà hàng: những gì diễn ra trong đó là riêng tư đối với khách hàng của nhà hàng, nhưng
những người quản lý văn phòng có thể nhìn thấy và làm mọi thứ trong nhà hàng mà họ điều hành.

Rust đã chọn để hệ thống module hoạt động theo cách này để việc ẩn đi các chi tiết triển khai
bên trong là mặc định. Bằng cách đó, bạn biết phần nào của mã nguồn bên trong
bạn có thể thay đổi mà không làm hỏng mã nguồn bên ngoài. Tuy nhiên, Rust cung cấp cho bạn
tùy chọn để để lộ các phần bên trong của mã nguồn của module con cho các module tổ tiên bên ngoài
bằng cách sử dụng từ khóa `pub` để làm cho một mục trở nên công khai (public).

### Công khai các Đường dẫn với Từ khóa `pub`

Hãy quay lại lỗi trong Listing 7-4 đã thông báo cho chúng ta rằng module `hosting` là
riêng tư. Chúng ta muốn hàm `eat_at_restaurant` trong module cha có
quyền truy cập vào hàm `add_to_waitlist` trong module con, vì vậy chúng ta đánh dấu
module `hosting` bằng từ khóa `pub`, như được hiển thị trong Listing 7-5.

<Listing number="7-5" file-name="src/lib.rs" caption="Khai báo module `hosting` là `pub` để sử dụng nó từ `eat_at_restaurant`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

Thật không may, mã nguồn trong Listing 7-5 vẫn dẫn đến các lỗi trình biên dịch, như
được hiển thị trong Listing 7-6.

<Listing number="7-6" caption="Lỗi trình biên dịch khi xây dựng mã nguồn trong Listing 7-5">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

Chuyện gì đã xảy ra? Thêm từ khóa `pub` vào trước `mod hosting` làm cho
module trở nên công khai. Với thay đổi này, nếu chúng ta có thể truy cập `front_of_house`, chúng ta có thể
truy cập `hosting`. Nhưng _nội dung_ của `hosting` vẫn là riêng tư; việc làm cho
module trở nên công khai không làm cho nội dung của nó trở nên công khai. Từ khóa `pub` trên một module
chỉ cho phép mã nguồn trong các module tổ tiên của nó tham chiếu đến nó, chứ không phải truy cập vào mã nguồn bên trong nó.
Bởi vì các module là các thùng chứa (containers), không có nhiều việc chúng ta có thể làm nếu chỉ làm cho
module trở nên công khai; chúng ta cần tiến xa hơn và chọn làm cho một hoặc nhiều
mục bên trong module trở nên công khai.

Các lỗi trong Listing 7-6 nói rằng hàm `add_to_waitlist` là riêng tư.
Các quy tắc về tính riêng tư áp dụng cho struct, enum, hàm và phương thức cũng như
các module.

Hãy cũng làm cho hàm `add_to_waitlist` trở nên công khai bằng cách thêm từ khóa `pub`
trước định nghĩa của nó, như trong Listing 7-7.

<Listing number="7-7" file-name="src/lib.rs" caption="Thêm từ khóa `pub` vào `mod hosting` và `fn add_to_waitlist` cho phép chúng ta gọi hàm từ `eat_at_restaurant`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

Bây giờ mã nguồn sẽ được biên dịch! Để thấy lý do tại sao việc thêm từ khóa `pub` cho phép chúng ta sử dụng
các đường dẫn này trong `eat_at_restaurant` liên quan đến các quy tắc riêng tư, chúng ta hãy xem xét
đường dẫn tuyệt đối và đường dẫn tương đối.

Trong đường dẫn tuyệt đối, chúng ta bắt đầu với `crate`, gốc của cây module của
crate. Module `front_of_house` được định nghĩa trong crate root. Trong khi
`front_of_house` không phải là công khai, bởi vì hàm `eat_at_restaurant` được
định nghĩa trong cùng module với `front_of_house` (nghĩa là `eat_at_restaurant`
và `front_of_house` là anh em), chúng ta có thể tham chiếu đến `front_of_house` từ
`eat_at_restaurant`. Tiếp theo là module `hosting` được đánh dấu bằng `pub`. Chúng ta có thể
truy cập module cha của `hosting`, vì vậy chúng ta có thể truy cập `hosting`. Cuối cùng,
hàm `add_to_waitlist` được đánh dấu bằng `pub` và chúng ta có thể truy cập module cha của nó,
vì vậy lời gọi hàm này hoạt động!

Trong đường dẫn tương đối, logic cũng tương tự như đường dẫn tuyệt đối ngoại trừ
bước đầu tiên: thay vì bắt đầu từ crate root, đường dẫn bắt đầu từ
`front_of_house`. Module `front_of_house` được định nghĩa trong cùng module
với `eat_at_restaurant`, vì vậy đường dẫn tương đối bắt đầu từ module mà
`eat_at_restaurant` được định nghĩa sẽ hoạt động. Sau đó, vì `hosting` và
`add_to_waitlist` được đánh dấu bằng `pub`, phần còn lại của đường dẫn hoạt động, và
lời gọi hàm này là hợp lệ!

Nếu bạn có kế hoạch chia sẻ library crate của mình để các dự án khác có thể sử dụng mã nguồn của bạn,
API công khai của bạn là hợp đồng của bạn với những người dùng crate của bạn, nó xác định cách
họ có thể tương tác với mã nguồn của bạn. Có nhiều cân nhắc xung quanh việc quản lý
các thay đổi đối với API công khai của bạn để giúp mọi người dễ dàng phụ thuộc vào
crate của bạn hơn. Những cân nhắc này nằm ngoài phạm vi của cuốn sách này; nếu bạn
quan tâm đến chủ đề này, hãy xem [Tài liệu hướng dẫn về Rust API][api-guidelines].

> #### Thực hành tốt nhất cho các Package có một Binary và một Library
>
> Chúng ta đã đề cập rằng một package có thể chứa cả một crate root binary _src/main.rs_
> cũng như một crate root library _src/lib.rs_, và cả hai crate sẽ có
> tên package theo mặc định. Thông thường, các package với mô hình chứa cả một library và một binary crate này sẽ chỉ có đủ mã nguồn trong
> binary crate để khởi động một tệp thực thi gọi mã nguồn được định nghĩa trong library
> crate. Điều này cho phép các dự án khác hưởng lợi từ hầu hết các chức năng mà
> package cung cấp vì mã nguồn của library crate có thể được chia sẻ.
>
> Cây module nên được định nghĩa trong _src/lib.rs_. Sau đó, bất kỳ mục công khai nào cũng có thể
> được sử dụng trong binary crate bằng cách bắt đầu các đường dẫn với tên của package.
> Binary crate trở thành một người dùng của library crate giống như một crate hoàn toàn
> bên ngoài sẽ sử dụng library crate: nó chỉ có thể sử dụng API công khai.
> Điều này giúp bạn thiết kế một API tốt; không chỉ bạn là tác giả, bạn còn là một
> khách hàng!
>
> Trong [Chương 12][ch12]<!-- ignore -->, chúng ta sẽ trình bày thực hành tổ chức này
> với một chương trình dòng lệnh sẽ chứa cả một binary crate
> và một library crate.

{{#quiz ../quizzes/ch07-03-paths-sec1.toml}}

### Bắt đầu Đường dẫn Tương đối với `super`

Chúng ta có thể xây dựng các đường dẫn tương đối bắt đầu trong module cha, thay vì
module hiện tại hoặc crate root, bằng cách sử dụng `super` ở đầu
đường dẫn. Điều này giống như việc bắt đầu một đường dẫn hệ thống tệp với cú pháp `..` có nghĩa
là đi đến thư mục cha. Việc sử dụng `super` cho phép chúng ta tham chiếu đến một mục
mà chúng ta biết là nằm trong module cha, điều này có thể giúp việc sắp xếp lại cây module
trở nên dễ dàng hơn khi module có liên quan chặt chẽ với module cha nhưng module cha
có thể được chuyển sang nơi khác trong cây module vào một ngày nào đó.

Hãy xem xét mã nguồn trong Listing 7-8 mô phỏng tình huống một đầu bếp
sửa một đơn hàng không chính xác và đích thân mang nó ra cho khách hàng.
Hàm `fix_incorrect_order` được định nghĩa trong module `back_of_house` gọi
hàm `deliver_order` được định nghĩa trong module cha bằng cách chỉ định đường dẫn đến
`deliver_order`, bắt đầu bằng `super`.

<Listing number="7-8" file-name="src/lib.rs" caption="Gọi một hàm bằng đường dẫn tương đối bắt đầu với `super`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

Hàm `fix_incorrect_order` nằm trong module `back_of_house`, vì vậy chúng ta có thể
sử dụng `super` để đi đến module cha của `back_of_house`, trong trường hợp này
là `crate`, gốc. Từ đó, chúng ta tìm kiếm `deliver_order` và thấy nó.
Thành công! Chúng ta nghĩ rằng module `back_of_house` và hàm `deliver_order`
có khả năng giữ nguyên mối quan hệ với nhau và được di chuyển
cùng nhau nếu chúng ta quyết định tổ chức lại cây module của crate. Do đó, chúng ta
sử dụng `super` để chúng ta sẽ có ít nơi cần cập nhật mã nguồn hơn trong tương lai nếu
mã nguồn này được chuyển sang một module khác.

### Công khai Struct và Enum

Chúng ta cũng có thể sử dụng `pub` để chỉ định các struct và enum là công khai, nhưng có
một vài chi tiết bổ sung cho việc sử dụng `pub` với struct và enum. Nếu chúng ta sử dụng `pub`
trước một định nghĩa struct, chúng ta làm cho struct đó trở nên công khai, nhưng các trường của struct
vẫn sẽ là riêng tư. Chúng ta có thể làm cho từng trường trở nên công khai hay không trên cơ sở từng trường hợp cụ thể. Trong Listing 7-9, chúng ta đã định nghĩa một struct `back_of_house::Breakfast` công khai
với một trường `toast` công khai nhưng một trường `seasonal_fruit` riêng tư. Điều này mô phỏng
trường hợp trong một nhà hàng nơi khách hàng có thể chọn loại bánh mì
đi kèm với bữa ăn, nhưng đầu bếp quyết định loại trái cây nào đi kèm với bữa ăn dựa
trên những gì đang trong mùa và còn hàng. Trái cây có sẵn thay đổi nhanh chóng, vì vậy
khách hàng không thể chọn trái cây hoặc thậm chí thấy loại trái cây nào họ sẽ nhận được.

<Listing number="7-9" file-name="src/lib.rs" caption="Một struct với một vài trường công khai và một vài trường riêng tư">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

Bởi vì trường `toast` trong struct `back_of_house::Breakfast` là công khai,
trong `eat_at_restaurant` chúng ta có thể ghi và đọc vào trường `toast` bằng cách sử dụng ký hiệu dấu chấm (dot notation). Lưu ý rằng chúng ta không thể sử dụng trường `seasonal_fruit` trong
`eat_at_restaurant`, vì `seasonal_fruit` là riêng tư. Hãy thử bỏ chú thích dòng
sửa đổi giá trị trường `seasonal_fruit` để xem bạn nhận được lỗi gì!

Ngoài ra, lưu ý rằng vì `back_of_house::Breakfast` có một trường riêng tư,
struct cần cung cấp một hàm liên kết (associated function) công khai để tạo một
thể hiện của `Breakfast` (chúng ta đã đặt tên nó là `summer` ở đây). Nếu `Breakfast` không
có một hàm như vậy, chúng ta không thể tạo một thể hiện của `Breakfast` trong
`eat_at_restaurant` vì chúng ta không thể đặt giá trị của trường `seasonal_fruit` riêng tư trong `eat_at_restaurant`.

Ngược lại, nếu chúng ta làm cho một enum trở nên công khai, tất cả các biến thể (variants) của nó sau đó đều là công khai. Chúng ta chỉ cần `pub` trước từ khóa `enum`, như được hiển thị trong Listing 7-10.

<Listing number="7-10" file-name="src/lib.rs" caption="Chỉ định một enum là công khai làm cho tất cả các biến thể của nó trở nên công khai.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

Bởi vì chúng ta đã làm cho enum `Appetizer` trở nên công khai, chúng ta có thể sử dụng các biến thể `Soup` và `Salad`
trong `eat_at_restaurant`.

Các enum không hữu ích lắm trừ khi các biến thể của chúng là công khai; sẽ thật phiền phức
nếu phải chú thích tất cả các biến thể enum bằng `pub` trong mọi trường hợp, vì vậy mặc định
cho các biến thể enum là công khai. Các struct thường hữu ích mà không cần các trường của chúng
phải là công khai, vì vậy các trường struct tuân theo quy tắc chung là mọi thứ
đều riêng tư theo mặc định trừ khi được chú thích bằng `pub`.

Còn một tình huống nữa liên quan đến `pub` mà chúng ta chưa đề cập, và đó là
tính năng hệ thống module cuối cùng của chúng ta: từ khóa `use`. Chúng ta sẽ đề cập riêng đến `use`
trước, sau đó chúng ta sẽ chỉ ra cách kết hợp `pub` và `use`.

{{#quiz ../quizzes/ch07-03-paths-sec2.toml}}

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
