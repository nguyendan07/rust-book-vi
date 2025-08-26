## Phụ lục B: Toán tử và Ký hiệu

Phụ lục này chứa một bảng chú giải về cú pháp của Rust, bao gồm các toán tử và các ký hiệu khác xuất hiện một mình hoặc trong ngữ cảnh của đường dẫn, kiểu tổng quát, ràng buộc trait, macro, thuộc tính, chú thích, tuple và dấu ngoặc.

### Toán tử

Bảng B-1 chứa các toán tử trong Rust, một ví dụ về cách toán tử xuất hiện trong ngữ cảnh, một giải thích ngắn, và liệu toán tử đó có thể được nạp chồng hay không. Nếu một toán tử có thể nạp chồng, trait tương ứng để nạp chồng được liệt kê.

<span class="caption">Bảng B-1: Toán tử</span>

| Toán tử                   | Ví dụ                                                   | Giải thích                                                             | Có thể nạp chồng? |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | ----------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Mở rộng macro                                                          |                   |
| `!`                       | `!expr`                                                 | Phủ định bit hoặc logic                                                 | `Not`             |
| `!=`                      | `expr != expr`                                          | So sánh khác nhau                                                        | `PartialEq`       |
| `%`                       | `expr % expr`                                           | Phần dư số học                                                           | `Rem`             |
| `%=`                      | `var %= expr`                                           | Phần dư và gán                                                           | `RemAssign`       |
| `&`                       | `&expr`, `&mut expr`                                    | Mượn                                                                    |                   |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Kiểu tham chiếu mượn                                                    |                   |
| `&`                       | `expr & expr`                                           | Phép toán AND theo bit                                                  | `BitAnd`          |
| `&=`                      | `var &= expr`                                           | Phép AND bit và gán                                                     | `BitAndAssign`    |
| `&&`                      | `expr && expr`                                          | AND logic có ngắt sớm                                                   |                   |
| `*`                       | `expr * expr`                                           | Phép nhân số học                                                         | `Mul`             |
| `*=`                      | `var *= expr`                                           | Phép nhân và gán                                                         | `MulAssign`       |
| `*`                       | `*expr`                                                 | Giải tham chiếu                                                          | `Deref`           |
| `*`                       | `*const type`, `*mut type`                              | Con trỏ thô                                                              |                   |
| `+`                       | `trait + trait`, `'a + trait`                           | Ràng buộc kiểu kết hợp                                                   |                   |
| `+`                       | `expr + expr`                                           | Phép cộng số học                                                         | `Add`             |
| `+=`                      | `var += expr`                                           | Phép cộng và gán                                                         | `AddAssign`       |
| `,`                       | `expr, expr`                                            | Phân cách đối số và phần tử                                               |                   |
| `-`                       | `- expr`                                                | Phủ định số học                                                          | `Neg`             |
| `-`                       | `expr - expr`                                           | Phép trừ số học                                                          | `Sub`             |
| `-=`                      | `var -= expr`                                           | Phép trừ và gán                                                          | `SubAssign`       |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Kiểu trả về của hàm và closure                                           |                   |
| `.`                       | `expr.ident`                                            | Truy cập trường                                                           |                   |
| `.`                       | `expr.ident(expr, ...)`                                 | Gọi phương thức                                                           |                   |
| `.`                       | `expr.0`, `expr.1`, etc.                                | Truy cập phần tử tuple theo chỉ số                                         |                   |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Biểu diễn phạm vi không bao gồm phần tử cuối (loại trừ phía phải)         | `PartialOrd`      |
| `..=`                     | `..=expr`, `expr..=expr`                                | Biểu diễn phạm vi bao gồm phần tử cuối                                     | `PartialOrd`      |
| `..`                      | `..expr`                                                | Cú pháp cập nhật literal struct                                            |                   |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | Ràng buộc mẫu “và phần còn lại”                                            |                   |
| `...`                     | `expr...expr`                                           | (Đã lỗi thời, dùng `..=` thay) Trong mẫu: mẫu phạm vi bao gồm               |                   |
| `/`                       | `expr / expr`                                           | Phép chia số học                                                          | `Div`             |
| `/=`                      | `var /= expr`                                           | Phép chia và gán                                                          | `DivAssign`       |
| `:`                       | `pat: type`, `ident: type`                              | Chỉ định/ghi chú kiểu                                                      |                   |
| `:`                       | `ident: expr`                                           | Khởi tạo trường struct                                                     |                   |
| `:`                       | `'a: loop {...}`                                        | Nhãn của vòng lặp                                                          |                   |
| `;`                       | `expr;`                                                 | Kết thúc câu lệnh và định nghĩa                                            |                   |
| `;`                       | `[...; len]`                                            | Một phần của cú pháp mảng kích thước cố định                                |                   |
| `<<`                      | `expr << expr`                                          | Dịch trái                                                                  | `Shl`             |
| `<<=`                     | `var <<= expr`                                          | Dịch trái và gán                                                           | `ShlAssign`       |
| `<`                       | `expr < expr`                                           | So sánh nhỏ hơn                                                             | `PartialOrd`      |
| `<=`                      | `expr <= expr`                                          | So sánh nhỏ hơn hoặc bằng                                                   | `PartialOrd`      |
| `=`                       | `var = expr`, `ident = type`                            | Gán / chỉ định                                                             |                   |
| `==`                      | `expr == expr`                                          | So sánh bằng                                                                | `PartialEq`       |
| `=>`                      | `pat => expr`                                           | Một phần của cú pháp nhánh match                                            |                   |
| `>`                       | `expr > expr`                                           | So sánh lớn hơn                                                             | `PartialOrd`      |
| `>=`                      | `expr >= expr`                                          | So sánh lớn hơn hoặc bằng                                                   | `PartialOrd`      |
| `>>`                      | `expr >> expr`                                          | Dịch phải                                                                  | `Shr`             |
| `>>=`                     | `var >>= expr`                                          | Dịch phải và gán                                                           | `ShrAssign`       |
| `@`                       | `ident @ pat`                                           | Ràng buộc mẫu                                                               |                   |
| `^`                       | `expr ^ expr`                                           | Phép XOR theo bit                                                           | `BitXor`          |
| `^=`                      | `var ^= expr`                                           | Phép XOR bit và gán                                                         | `BitXorAssign`    |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Các lựa chọn trong mẫu                                                      |                   |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | Phép OR theo bit                                                            | `BitOr`           |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | Phép OR bit và gán                                                          | `BitOrAssign`     |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | OR logic có ngắt sớm                                                        |                   |
| `?`                       | `expr?`                                                 | Truyền lỗi                                                                  |                   |

### Ký hiệu không phải toán tử

Danh sách sau chứa tất cả ký hiệu không hoạt động như toán tử; tức là, chúng không hành xử như lời gọi hàm hoặc phương thức.

Bảng B-2 hiển thị các ký hiệu xuất hiện độc lập và hợp lệ trong nhiều vị trí khác nhau.

<span class="caption">Bảng B-2: Cú pháp xuất hiện độc lập</span>

| Ký hiệu                                                                 | Giải thích                                                              |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `'ident`                                                               | Tên lifetime hoặc nhãn vòng lặp                                         |
| Digits immediately followed by `u8`, `i32`,  `f64`, `usize`, and so on | Hằng số số có kiểu cụ thể                                               |
| `"..."`                                                                | Hằng chuỗi                                                              |
| `r"..."`, `r#"..."#`, `r##"..."##`, etc.                               | Chuỗi thô, ký tự thoát không được xử lý                                 |
| `b"..."`                                                               | Chuỗi byte; tạo một mảng byte thay vì một chuỗi                         |
| `br"..."`, `br#"..."#`, `br##"..."##`, etc.                            | Chuỗi byte thô, kết hợp giữa chuỗi thô và chuỗi byte                    |
| `'...'`                                                                | Hằng ký tự                                                              |
| `b'...'`                                                               | Hằng byte ASCII                                                         |
| <code>&vert;...&vert; expr</code>                                      | Closure (hàm ẩn danh)                                                   |
| `!`                                                                    | Luôn là kiểu bottom rỗng cho các hàm phân kỳ                            |
| `_`                                                                    | Ràng buộc mẫu "bỏ qua"; cũng dùng để làm cho literal số nguyên dễ đọc   |

Bảng B-3 hiển thị các ký hiệu xuất hiện trong ngữ cảnh của một đường dẫn qua hệ thống module đến một mục.

<span class="caption">Bảng B-3: Cú pháp liên quan đường dẫn</span>

| Ký hiệu                                  | Giải thích                                                                                                                     |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `ident::ident`                          | Đường dẫn namespace                                                                                                             |
| `::path`                                | Đường dẫn bắt đầu từ extern prelude, nơi tất cả các crate được đặt gốc (tức là một đường dẫn tuyệt đối rõ ràng bao gồm tên crate) |
| `self::path`                            | Đường dẫn tương đối tới module hiện tại (tức là một đường dẫn tương đối rõ ràng)                                                  |
| `super::path`                           | Đường dẫn tương đối tới module cha của module hiện tại                                                                           |
| `type::ident`, `<type as trait>::ident` | Các hằng, hàm và kiểu liên kết                                                                                                  |
| `<type>::...`                           | Phần tử liên kết cho một kiểu không thể đặt tên trực tiếp (ví dụ `<&T>::...`, `<[T]>::...`, v.v.)                                 |
| `trait::method(...)`                    | Phân biệt lời gọi phương thức bằng cách nêu tên trait định nghĩa nó                                                              |
| `type::method(...)`                     | Phân biệt lời gọi phương thức bằng cách nêu tên kiểu mà phương thức đó được định nghĩa cho                                        |
| `<type as trait>::method(...)`          | Phân biệt lời gọi phương thức bằng cách nêu cả trait và kiểu                                                                    |

Bảng B-4 hiển thị các ký hiệu xuất hiện trong ngữ cảnh của việc sử dụng tham số kiểu tổng quát.

<span class="caption">Bảng B-4: Generics</span>

| Ký hiệu                         | Giải thích                                                                                                                              |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | Chỉ định tham số cho kiểu generic trong một kiểu (ví dụ, `Vec<u8>`)                                                                      |
| `path::<...>`, `method::<...>` | Chỉ định tham số generic cho kiểu, hàm, hoặc phương thức trong một biểu thức; thường gọi là turbofish (ví dụ, `"42".parse::<i32>()`) |
| `fn ident<...> ...`            | Định nghĩa hàm generic                                                                                                                   |
| `struct ident<...> ...`        | Định nghĩa struct generic                                                                                                                |
| `enum ident<...> ...`          | Định nghĩa enum generic                                                                                                                  |
| `impl<...> ...`                | Định nghĩa impl generic                                                                                                                  |
| `for<...> type`                | Ràng buộc lifetime bậc cao hơn                                                                                                           |
| `type<ident=type>`             | Một kiểu generic trong đó một hoặc nhiều associated type được gán cụ thể (ví dụ `Iterator<Item=T>`)                                      |

Bảng B-5 hiển thị các ký hiệu xuất hiện trong ngữ cảnh ràng buộc tham số kiểu tổng quát bằng trait.

<span class="caption">Bảng B-5: Ràng buộc trait</span>

| Ký hiệu                        | Giải thích                                                                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `T: U`                        | Tham số generic `T` bị ràng buộc phải là các kiểu thực hiện `U`                                                                            |
| `T: 'a`                       | Kiểu generic `T` phải tồn tại lâu hơn lifetime `'a` (nghĩa là kiểu không thể chứa tham chiếu có lifetime ngắn hơn `'a`)                    |
| `T: 'static`                  | Kiểu generic `T` không chứa tham chiếu mượn nào ngoài các tham chiếu `'static`                                                              |
| `'b: 'a`                      | Lifetime `'b` phải tồn tại lâu hơn lifetime `'a`                                                                                           |
| `T: ?Sized`                   | Cho phép tham số kiểu generic là một kiểu có kích thước động                                                                               |
| `'a + trait`, `trait + trait` | Ràng buộc kiểu kết hợp                                                                                                                     |

Bảng B-6 hiển thị các ký hiệu xuất hiện trong ngữ cảnh gọi hoặc định nghĩa macro và xác định thuộc tính trên một mục.

<span class="caption">Bảng B-6: Macros và Thuộc tính</span>

| Ký hiệu                                      | Giải thích        |
| ------------------------------------------- | ------------------ |
| `#[meta]`                                   | Thuộc tính ngoài   |
| `#![meta]`                                  | Thuộc tính nội bộ  |
| `$ident`                                    | Thay thế trong macro |
| `$ident:kind`                               | Biến metavariable trong macro |
| `$(...)...`                                 | Lặp trong macro    |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Gọi macro          |

Bảng B-7 hiển thị các ký hiệu tạo chú thích.

<span class="caption">Bảng B-7: Chú thích</span>

| Ký hiệu     | Giải thích                     |
| ---------- | ------------------------------- |
| `//`       | Chú thích dòng                  |
| `//!`      | Chú thích tài liệu dòng nội bộ  |
| `///`      | Chú thích tài liệu dòng ngoài   |
| `/*...*/`  | Chú thích khối                  |
| `/*!...*/` | Chú thích tài liệu khối nội bộ  |
| `/**...*/` | Chú thích tài liệu khối ngoài   |

Bảng B-8 hiển thị các ngữ cảnh sử dụng dấu ngoặc tròn.

<span class="caption">Bảng B-8: Dấu ngoặc tròn</span>

| Ký hiệu                   | Giải thích                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `()`                     | Tuple rỗng (còn gọi là unit), cả literal và kiểu                                              |
| `(expr)`                 | Biểu thức được đặt trong ngoặc                                                               |
| `(expr,)`                | Biểu thức tuple một phần tử                                                                  |
| `(type,)`                | Kiểu tuple một phần tử                                                                       |
| `(expr, ...)`            | Biểu thức tuple                                                                               |
| `(type, ...)`            | Kiểu tuple                                                                                    |
| `expr(expr, ...)`        | Biểu thức gọi hàm; cũng dùng để khởi tạo tuple `struct`s và biến thể `enum` dạng tuple       |

Bảng B-9 hiển thị các ngữ cảnh sử dụng dấu ngoặc nhọn.

<span class="caption">Bảng B-9: Dấu ngoặc nhọn</span>

| Ngữ cảnh      | Giải thích      |
| ------------ | ---------------- |
| `{...}`      | Biểu thức khối    |
| `Type {...}` | Literal struct    |

Bảng B-10 hiển thị các ngữ cảnh sử dụng dấu ngoặc vuông.

<span class="caption">Bảng B-10: Dấu ngoặc vuông</span>

| Ngữ cảnh                                            | Giải thích                                                                                                                   |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                            | Literal mảng                                                                                                                 |
| `[expr; len]`                                      | Literal mảng chứa `len` bản sao của `expr`                                                                                   |
| `[type; len]`                                      | Kiểu mảng chứa `len` phần tử kiểu `type`                                                                                     |
| `expr[expr]`                                       | Truy cập chỉ mục bộ sưu tập. Có thể nạp chồng (`Index`, `IndexMut`)                                                          |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Truy cập bộ sưu tập giống như slicing, sử dụng `Range`, `RangeFrom`, `RangeTo`, hoặc `RangeFull` làm “chỉ mục”                 |
