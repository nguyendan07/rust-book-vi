## Phụ lục A: Từ khóa

Danh sách sau đây chứa các từ khóa được dành riêng cho việc sử dụng hiện tại hoặc trong tương lai của ngôn ngữ Rust. Do đó, chúng không thể được sử dụng làm định danh (ngoại trừ định danh thô như chúng ta sẽ thảo luận trong phần “[Định danh thô][raw-identifiers]<!-- ignore -->”). Định danh là tên của các hàm, biến, tham số, trường struct, module, crate, hằng số, macro, giá trị tĩnh, thuộc tính, kiểu, trait, hoặc lifetime.

[raw-identifiers]: #raw-identifiers

### Các từ khóa hiện đang được sử dụng

Sau đây là danh sách các từ khóa hiện đang được sử dụng, cùng với mô tả chức năng của chúng.

- `as` - thực hiện ép kiểu nguyên thủy, phân biệt trait cụ thể chứa một item, hoặc đổi tên các item trong câu lệnh `use`
- `async` - trả về một `Future` thay vì chặn luồng hiện tại
- `await` - tạm dừng thực thi cho đến khi kết quả của một `Future` sẵn sàng
- `break` - thoát khỏi vòng lặp ngay lập tức
- `const` - định nghĩa các item hằng số hoặc con trỏ thô hằng số
- `continue` - tiếp tục với vòng lặp tiếp theo
- `crate` - trong một đường dẫn module, tham chiếu đến gốc của crate
- `dyn` - điều phối động đến một đối tượng trait
- `else` - phương án dự phòng cho các cấu trúc luồng điều khiển `if` và `if let`
- `enum` - định nghĩa một enumeration (kiểu liệt kê)
- `extern` - liên kết một hàm hoặc biến bên ngoài
- `false` - literal Boolean false
- `fn` - định nghĩa một hàm hoặc kiểu con trỏ hàm
- `for` - lặp qua các item từ một iterator, triển khai một trait, hoặc chỉ định một lifetime bậc cao hơn
- `if` - rẽ nhánh dựa trên kết quả của một biểu thức điều kiện
- `impl` - triển khai chức năng nội tại hoặc của trait
- `in` - một phần của cú pháp vòng lặp `for`
- `let` - gán một biến
- `loop` - lặp vô điều kiện
- `match` - khớp một giá trị với các mẫu
- `mod` - định nghĩa một module
- `move` - làm cho một closure chiếm quyền sở hữu tất cả các biến mà nó nắm giữ
- `mut` - biểu thị tính khả biến trong tham chiếu, con trỏ thô, hoặc gán mẫu
- `pub` - biểu thị khả năng truy cập công khai trong các trường struct, khối `impl`, hoặc module
- `ref` - gán bằng tham chiếu
- `return` - trả về từ hàm
- `Self` - một bí danh kiểu cho kiểu mà chúng ta đang định nghĩa hoặc triển khai
- `self` - chủ thể của phương thức hoặc module hiện tại
- `static` - biến toàn cục hoặc lifetime kéo dài toàn bộ quá trình thực thi chương trình
- `struct` - định nghĩa một cấu trúc (structure)
- `super` - module cha của module hiện tại
- `trait` - định nghĩa một trait
- `true` - literal Boolean true
- `type` - định nghĩa một bí danh kiểu hoặc kiểu liên kết
- `union` - định nghĩa một [union][union]<!-- ignore -->; chỉ là một từ khóa khi được sử dụng trong một khai báo union
- `unsafe` - biểu thị mã, hàm, trait, hoặc triển khai không an toàn
- `use` - đưa các ký hiệu vào phạm vi; chỉ định các nắm giữ chính xác cho các giới hạn generic và lifetime
- `where` - biểu thị các mệnh đề ràng buộc một kiểu
- `while` - lặp có điều kiện dựa trên kết quả của một biểu thức

[union]: ../reference/items/unions.html

### Các từ khóa được dành riêng cho tương lai

Các từ khóa sau đây chưa có chức năng nào nhưng được Rust dành riêng cho việc sử dụng tiềm năng trong tương lai.

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### Định danh thô (Raw identifiers)

_Raw identifiers_ là cú pháp cho phép bạn sử dụng các từ khóa ở những nơi mà chúng thường không được phép. Bạn sử dụng một định danh thô bằng cách thêm tiền tố `r#` vào trước một từ khóa.

Ví dụ, `match` là một từ khóa. Nếu bạn cố gắng biên dịch hàm sau đây sử dụng `match` làm tên của nó:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
  haystack.contains(needle)
}
```

bạn sẽ nhận được lỗi này:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

Lỗi cho thấy bạn không thể sử dụng từ khóa `match` làm định danh hàm. Để sử dụng `match` làm tên hàm, bạn cần sử dụng cú pháp định danh thô, như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
  haystack.contains(needle)
}

fn main() {
  assert!(r#match("foo", "foobar"));
}
```

Mã này sẽ biên dịch mà không có lỗi nào. Lưu ý tiền tố `r#` trên tên hàm trong định nghĩa của nó cũng như nơi hàm được gọi trong `main`.

Định danh thô cho phép bạn sử dụng bất kỳ từ nào bạn chọn làm định danh, ngay cả khi từ đó là một từ khóa được dành riêng. Điều này cho chúng ta nhiều tự do hơn để chọn tên định danh, cũng như cho phép chúng ta tích hợp với các chương trình được viết bằng một ngôn ngữ mà những từ này không phải là từ khóa. Ngoài ra, định danh thô cho phép bạn sử dụng các thư viện được viết bằng một phiên bản Rust khác với phiên bản mà crate của bạn sử dụng. Ví dụ, `try` không phải là từ khóa trong phiên bản 2015 nhưng lại là từ khóa trong các phiên bản 2018, 2021 và 2024. Nếu bạn phụ thuộc vào một thư viện được viết bằng phiên bản 2015 và có một hàm `try`, bạn sẽ cần sử dụng cú pháp định danh thô, `r#try` trong trường hợp này, để gọi hàm đó từ mã của bạn trên các phiên bản sau này. Xem [Phụ lục E][appendix-e]<!-- ignore --> để biết thêm thông tin về các phiên bản.

[appendix-e]: appendix-05-editions.html
