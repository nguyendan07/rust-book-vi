## Định nghĩa Module để Kiểm soát Phạm vi và Tính Riêng tư

Trong phần này, chúng ta sẽ nói về các module và các phần khác của hệ thống module,
cụ thể là _đường dẫn_ (paths), cho phép bạn đặt tên cho các mục; từ khóa `use` giúp đưa một
đường dẫn vào phạm vi; và từ khóa `pub` để làm cho các mục trở nên công khai. Chúng ta cũng sẽ thảo luận về
từ khóa `as`, các package bên ngoài và toán tử glob.

### Bảng Tra cứu nhanh về Module (Modules Cheat Sheet)

Trước khi đi vào chi tiết về các module và đường dẫn, ở đây chúng tôi cung cấp một tài liệu tham khảo
nhanh về cách các module, đường dẫn, từ khóa `use` và từ khóa `pub` hoạt động
trong trình biên dịch, và cách hầu hết các nhà phát triển tổ chức mã nguồn của họ. Chúng ta sẽ đi
qua các ví dụ về từng quy tắc này trong suốt chương này, nhưng đây là một
nơi tuyệt vời để tham khảo như một lời nhắc nhở về cách các module hoạt động.

- **Bắt đầu từ crate root**: Khi biên dịch một crate, trình biên dịch trước tiên
  sẽ tìm trong file crate root (thường là _src/lib.rs_ cho một library crate hoặc
  _src/main.rs_ cho một binary crate) để tìm mã nguồn cần biên dịch.
- **Khai báo các module**: Trong file crate root, bạn có thể khai báo các module mới;
  giả sử bạn khai báo một module “garden” với `mod garden;`. Trình biên dịch sẽ tìm
  kiếm mã nguồn của module ở những nơi sau:
    - Trực tiếp (inline), bên trong các dấu ngoặc nhọn thay thế cho dấu chấm phẩy sau `mod
garden`
    - Trong file _src/garden.rs_
    - Trong file _src/garden/mod.rs_
- **Khai báo các module con (submodules)**: Trong bất kỳ file nào khác ngoài crate root, bạn có thể
  khai báo các module con. Ví dụ, bạn có thể khai báo `mod vegetables;` trong
  _src/garden.rs_. Trình biên dịch sẽ tìm kiếm mã nguồn của module con bên trong
  thư mục được đặt tên theo module cha ở những nơi sau:
    - Trực tiếp, ngay sau `mod vegetables`, bên trong các dấu ngoặc nhọn thay vì
      dấu chấm phẩy
    - Trong file _src/garden/vegetables.rs_
    - Trong file _src/garden/vegetables/mod.rs_
- **Đường dẫn đến mã nguồn trong các module**: Khi một module đã là một phần của crate, bạn có thể
  tham chiếu đến mã nguồn trong module đó từ bất kỳ nơi nào khác trong cùng crate đó, miễn là
  các quy tắc về tính riêng tư cho phép, bằng cách sử dụng đường dẫn đến mã nguồn. Ví dụ, một
  kiểu `Asparagus` trong module garden vegetables sẽ được tìm thấy tại
  `crate::garden::vegetables::Asparagus`.
- **Riêng tư so với Công khai (Private vs. public)**: Mã nguồn bên trong một module là riêng tư đối với các module
  cha của nó theo mặc định. Để làm cho một module trở nên công khai, hãy khai báo nó với `pub mod`
  thay vì `mod`. Để làm cho các mục bên trong một module công khai cũng trở nên công khai, hãy sử dụng
  `pub` trước các khai báo của chúng.
- **Từ khóa `use`**: Trong một phạm vi, từ khóa `use` tạo ra các lối tắt (shortcuts) đến
  các mục để giảm bớt việc lặp lại các đường dẫn dài. Trong bất kỳ phạm vi nào có thể tham chiếu đến
  `crate::garden::vegetables::Asparagus`, bạn có thể tạo một lối tắt với `use
crate::garden::vegetables::Asparagus;` và từ đó trở đi bạn chỉ cần
  viết `Asparagus` để sử dụng kiểu đó trong phạm vi.

Ở đây, chúng ta tạo một binary crate tên là `backyard` để minh họa các quy tắc này.
Thư mục của crate, cũng được đặt tên là `backyard`, chứa các file và
thư mục sau:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

File crate root trong trường hợp này là _src/main.rs_, và nó chứa:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

Dòng `pub mod garden;` bảo trình biên dịch bao gồm mã nguồn mà nó tìm thấy trong
_src/garden.rs_, đó là:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Ở đây, `pub mod vegetables;` có nghĩa là mã nguồn trong _src/garden/vegetables.rs_ cũng
được bao gồm. Mã nguồn đó là:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Bây giờ hãy đi sâu vào chi tiết của các quy tắc này và chứng minh chúng trong thực tế!

### Nhóm Mã nguồn Liên quan trong các Module

_Module_ cho phép chúng ta tổ chức mã nguồn trong một crate để dễ đọc và dễ tái sử dụng.
Module cũng cho phép chúng ta kiểm soát _tính riêng tư_ (privacy) của các mục vì mã nguồn bên trong một
module là riêng tư theo mặc định. Các mục riêng tư là các chi tiết triển khai nội bộ
không có sẵn để sử dụng bên ngoài. Chúng ta có thể chọn làm cho các module và các mục
bên trong chúng trở nên công khai, điều này giúp để lộ chúng để cho phép mã nguồn bên ngoài sử dụng và phụ thuộc
vào chúng.

Ví dụ, hãy viết một library crate cung cấp chức năng của một
nhà hàng. Chúng ta sẽ định nghĩa chữ ký (signatures) của các hàm nhưng để trống phần thân của chúng
để tập trung vào việc tổ chức mã nguồn hơn là việc
triển khai một nhà hàng.

Trong ngành nhà hàng, một số bộ phận của nhà hàng được gọi là
_front of house_ (phía trước) và những bộ phận khác là _back of house_ (phía sau). Front of house là nơi
khách hàng hiện diện; phần này bao gồm nơi tiếp tân xếp chỗ cho khách, nhân viên phục vụ
nhận yêu cầu gọi món và thanh toán, và nhân viên pha chế đồ uống. Back of house là nơi
các đầu bếp làm việc trong bếp, nhân viên rửa bát dọn dẹp và quản lý làm
công việc hành chính.

Để cấu trúc crate của chúng ta theo cách này, chúng ta có thể tổ chức các hàm của nó thành các module
lồng nhau. Tạo một thư viện mới tên là `restaurant` bằng cách chạy `cargo new
restaurant --lib`. Sau đó nhập mã nguồn trong Listing 7-1 vào _src/lib.rs_ để
định nghĩa một số module và chữ ký hàm; mã này là phần front of house.

<Listing number="7-1" file-name="src/lib.rs" caption="Một module `front_of_house` chứa các module khác, sau đó chứa các hàm">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

Chúng ta định nghĩa một module bằng từ khóa `mod` theo sau là tên của module
(trong trường hợp này là `front_of_house`). Phần thân của module sau đó nằm bên trong các dấu ngoặc
nhọn. Bên trong các module, chúng ta có thể đặt các module khác, như trong trường hợp này với các
module `hosting` và `serving`. Các module cũng có thể chứa các định nghĩa cho các
mục khác, chẳng hạn như struct, enum, hằng số, trait, và như trong Listing 7-1,
các hàm.

Bằng cách sử dụng các module, chúng ta có thể nhóm các định nghĩa liên quan lại với nhau và đặt tên cho lý do
tại sao chúng liên quan. Các lập trình viên sử dụng mã nguồn này có thể điều hướng mã nguồn dựa trên các
nhóm thay vì phải đọc qua tất cả các định nghĩa, giúp dễ dàng
tìm thấy các định nghĩa liên quan đến họ hơn. Các lập trình viên thêm chức năng mới
vào mã nguồn này sẽ biết nơi đặt mã để giữ cho chương trình được tổ chức.

Trước đó, chúng ta đã đề cập rằng _src/main.rs_ và _src/lib.rs_ được gọi là các crate
root. Lý do cho tên gọi của chúng là nội dung của một trong hai file này tạo thành
một module tên là `crate` ở gốc của cấu trúc module của crate,
được gọi là _cây module_ (module tree).

Listing 7-2 hiển thị cây module cho cấu trúc trong Listing 7-1.

<Listing number="7-2" caption="Cây module cho mã nguồn trong Listing 7-1">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Cây này cho thấy cách một số module lồng bên trong các module khác; ví dụ,
`hosting` lồng bên trong `front_of_house`. Cây cũng cho thấy một số module
là _anh em_ (siblings), nghĩa là chúng được định nghĩa trong cùng một module; `hosting` và
`serving` là anh em được định nghĩa bên trong `front_of_house`. Nếu module A được
chứa bên trong module B, chúng ta nói rằng module A là _con_ (child) của module B và
module B là _cha_ (parent) của module A. Lưu ý rằng toàn bộ cây module
được bắt nguồn từ module ẩn tên là `crate`.

Cây module có thể gợi nhớ cho bạn về cây thư mục của hệ thống tệp trên
máy tính của bạn; đây là một sự so sánh rất xác đáng! Giống như các thư mục trong một hệ thống tệp,
bạn sử dụng các module để tổ chức mã nguồn của mình. Và giống như các file trong một thư mục, chúng ta
cần một cách để tìm thấy các module của mình.

{{#quiz ../quizzes/ch07-02-modules.toml}}
