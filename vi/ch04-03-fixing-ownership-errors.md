## Sửa Các Lỗi Về Quyền Sở Hữu

Học cách sửa một lỗi về quyền sở hữu (ownership error) là một kỹ năng cốt lõi khi lập trình với Rust. Khi bộ kiểm tra mượn (borrow checker) từ chối mã của bạn, bạn nên xử lý như thế nào? Trong phần này, chúng ta sẽ phân tích một số tình huống thực tế (case studies) về các lỗi quyền sở hữu phổ biến nhất, nguyên nhân sâu xa của chúng và các chiến lược chuẩn mực để khắc phục.

Một nguyên tắc chung là phân biệt xem một chương trình *thực sự không an toàn* (có nguy cơ gây lỗi bộ nhớ) hay *thực ra an toàn nhưng bị trình biên dịch từ chối vì quá bảo thủ*.

### Tình Huống 1: Trả Về Một Tham Chiếu Đến Stack

Đây là ví dụ kinh điển về việc cố gắng trả về một tham chiếu trỏ tới biến cục bộ trên stack:

```rust,ignore,does_not_compile
fn return_a_string() -> &String {
    let s = String::from("Hello world");
    &s
}
```

**Tại sao chương trình này không an toàn?** Vì biến `s` được cấp phát trong khung stack của hàm `return_a_string`. Khi hàm kết thúc, khung stack bị thu hồi và chuỗi `s` bị giải phóng ngay lập tức. Tham chiếu trả về sẽ trỏ vào một vùng nhớ rác không còn hợp lệ.

Tùy vào mục đích của ứng dụng, dưới đây là 4 cách giải quyết chuẩn mực:

1. **Chuyển quyền sở hữu (Move) ra ngoài hàm** (cách phổ biến nhất):
   ```rust
   fn return_a_string() -> String {
       let s = String::from("Hello world");
       s
   }
   ```
2. **Trả về một chuỗi ký tự tĩnh (`&'static str`)** nếu chuỗi là hằng số và không cần cấp phát động trên heap:
   ```rust
   fn return_a_string() -> &'static str {
       "Hello world"
   }
   ```
3. **Sử dụng con trỏ đếm tham chiếu (`Rc<String>`)** nếu nhiều nơi cần cùng đọc chuỗi mà không biết trước nơi nào sẽ giải phóng chuỗi cuối cùng:
   ```rust
   use std::rc::Rc;
   fn return_a_string() -> Rc<String> {
       let s = Rc::new(String::from("Hello world"));
       Rc::clone(&s)
   }
   ```
4. **Để bên gọi hàm truyền vào một tham chiếu khả biến** (`&mut String`) để hàm ghi trực tiếp kết quả vào đó (giúp tái sử dụng bộ đệm):
   ```rust
   fn return_a_string(output: &mut String) {
       output.replace_range(.., "Hello world");
   }
   ```

### Tình Huống 2: Không Đủ Quyền Hạn (Thay Đổi Dữ Liệu Qua Tham Chiếu Bất Biến)

Giả sử chúng ta muốn viết một hàm nối thêm danh xưng vào một danh sách các phần của tên:

```aquascope,permissions,stepper,boundaries,shouldFail
fn stringify_name_with_title(name: &Vec<String>) -> String {
    name.push(String::from("Esq."));`{}` // Lỗi: name chỉ là tham chiếu bất biến!
    let full = name.join(" ");
    full
}
```

Hàm này bị Borrow Checker từ chối vì `name` là tham chiếu bất biến (`&Vec<String>`), không có quyền ghi (@Perm{write}). Nếu hàm này được phép `push`, nó có thể làm thay đổi kích thước vector và vô hiệu hóa các tham chiếu khác đang trỏ vào vector đó bên ngoài hàm gọi.

**Cách khắc phục chuẩn mực:**
Thay vì đổi kiểu tham số thành `&mut Vec<String>` (sẽ làm thay đổi ngoài ý muốn dữ liệu của bên gọi) hoặc `Vec<String>` (sẽ cướp mất quyền sở hữu của bên gọi), cách viết chuẩn mực theo phong cách Rust (*idiomatic Rust*) là **chỉ đọc dữ liệu đầu vào và tạo chuỗi kết quả mới**:

```rust,ignore
fn stringify_name_with_title(name: &Vec<String>) -> String {
    let mut full = name.join(" ");
    full.push_str(" Esq.");
    full
}
```

Giải pháp này vừa an toàn, vừa giữ nguyên vẹn dữ liệu của bên gọi, lại tránh được các phép sao chép mảng không cần thiết.

### Tình Huống 3: Vừa Lặp Qua Vừa Thay Đổi Vector

Xét ví dụ tìm chuỗi dài nhất trong vector và thêm các chuỗi thỏa mãn điều kiện vào vector đó:

```aquascope,permissions,stepper,boundaries,shouldFail
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {`(focus,paths:*dst)`
    let largest: &String =
      dst.iter().max_by_key(|s| s.len()).unwrap();`(focus,paths:*dst)`
    for s in src {
        if s.len() > largest.len() {
            dst.push(s.clone());`{}` // Lỗi: dst.push làm mất hiệu lực tham chiếu `largest`!
        }
    }
}
```

Lệnh `let largest = ...` đang mượn bất biến `dst`. Khi vòng lặp gọi `dst.push(...)`, vector có thể cấp phát lại bộ nhớ heap và làm tham chiếu `largest` trỏ vào vùng nhớ rác!

**Cách giải quyết tối ưu:**
Vì chúng ta chỉ cần **độ dài** của phần tử lớn nhất để so sánh chứ không cần giữ tham chiếu đến bản thân chuỗi đó, ta chỉ cần sao chép giá trị độ dài (`usize` là kiểu có `Copy`):

```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest_len: usize = dst.iter().max_by_key(|s| s.len()).unwrap().len();
    for s in src {
        if s.len() > largest_len {
            dst.push(s.clone());
        }
    }
}
```

Sau khi lấy được `largest_len`, tham chiếu mượn trên `dst` lập tức kết thúc, cho phép `dst.push(...)` thực thi hoàn toàn an toàn và đạt hiệu năng tối đa.

### Tình Huống 4: Sao Chép vs Di Chuyển Khỏi Một Collection

Khi truy cập một phần tử từ vector:

```rust
let v: Vec<i32> = vec![0, 1, 2];
let n: i32 = v[0]; // Hợp lệ vì i32 triển khai trait Copy
```

Đoạn mã trên hoàn toàn hợp lệ vì `i32` không sở hữu vùng nhớ heap và có triển khai trait `Copy`.

Nhưng nếu vector chứa `String`:

```rust,ignore,does_not_compile
let v: Vec<String> = vec![String::from("Hello world")];
let s: String = v[0]; // LỖI: cannot move out of index of Vec
```

`String` là kiểu dữ liệu sở hữu bộ nhớ heap và không có `Copy`. Nếu phép gán trên được phép, cả `v` và `s` sẽ cùng nghĩ mình sở hữu chuỗi `"Hello world"`, dẫn đến lỗi giải phóng bộ nhớ kép (*Double Free*).

**Cách khắc phục:**
1. **Dùng tham chiếu nếu chỉ cần đọc:** `let s: &str = &v[0];`
2. **Clone nếu cần một bản sao độc lập:** `let s: String = v[0].clone();`
3. **Dùng `v.remove(0)` hoặc `v.pop()`** nếu bạn thực sự muốn rút phần tử đó ra khỏi vector và nhận quyền sở hữu.

### Tình Huống 5: Thay Đổi Nhiều Phần Tử Không Trùng Lặp Của Mảng

Xét trường hợp bạn muốn mượn khả biến 2 phần tử khác nhau trong cùng một mảng:

```rust,ignore,does_not_compile
let mut a = [0, 1, 2, 3];
let x = &mut a[1];
let y = &mut a[2]; // Lỗi: cannot borrow `a` as mutable more than once at a time
```

Về mặt logic thực tế, sửa `a[1]` và `a[2]` là an toàn vì chúng nằm ở hai ô nhớ khác nhau. Nhưng Borrow Checker chỉ nhìn mảng như một thực thể duy nhất (`a[_]`) nên từ chối mã này.

**Cách giải quyết chuẩn mực:**
Sử dụng hàm [`split_at_mut`][split_at_mut] từ thư viện chuẩn của Rust để chia mảng thành hai nửa an toàn:

```rust
let mut a = [0, 1, 2, 3];
let (a_left, a_right) = a.split_at_mut(2);
let x = &mut a_left[1]; // trỏ vào a[1]
let y = &mut a_right[0]; // trỏ vào a[2]
*x += *y;
```

> [!NOTE]
> **Ghi chú về thiết kế an toàn:**
> Bên dưới hàm `split_at_mut`, các lập trình viên thư viện chuẩn Rust sử dụng con trỏ thô bên trong khối `unsafe` đã được kiểm thử và chứng minh toán học nghiêm ngặt, sau đó bọc lại thành một hàm an toàn cho người dùng. Đây là cách Rust cho phép bạn vừa đạt hiệu năng tối đa vừa an toàn tuyệt đối.

### Tóm Tắt

Khi gặp lỗi từ Borrow Checker:
1. **Xác định nguyên nhân cốt lõi:** Vùng nhớ này cần sống bao lâu? Ai là người sở hữu nó?
2. **Áp dụng đúng chiến lược:** Dùng tham chiếu (`&T`) khi chỉ cần đọc, dùng `.clone()` khi cần bản sao riêng biệt, hoặc rút ngắn thời gian mượn (lifetime) bằng cách sao chép các kiểu dữ liệu nguyên thủy (như `len()`).

[rc]: https://doc.rust-lang.org/std/rc/index.html
[cells]: https://doc.rust-lang.org/std/cell/index.html
[split_at_mut]: https://doc.rust-lang.org/std/primitive.slice.html#method.split_at_mut
[unsafe]: ch19-01-unsafe-rust.html
[`Vec::remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.remove
[`slice::join`]: https://doc.rust-lang.org/std/primitive.slice.html#method.join
[iterators]: ch13-02-iterators.html
[closures]: ch13-01-closures.html

[^safe-subset]: Áp dụng cho tập hợp mã nguồn an toàn (safe Rust).
