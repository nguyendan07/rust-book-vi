## Tham chiếu (References) và Mượn (Borrowing)

Quyền sở hữu (Ownership), box, và hành động chuyển quyền (moves) cung cấp nền tảng vững chắc để lập trình an toàn với heap. Tuy nhiên, nếu hàm nào cũng bắt buộc phải chuyển quyền sở hữu (move-only), việc viết mã sẽ rất bất tiện. Ví dụ, giả sử bạn muốn đọc một vài chuỗi ký tự hai lần:

```aquascope,interpreter,shouldFail,horizontal
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");
    greet(m1, m2);`[]`
    let s = format!("{} {}", m1, m2);`[]` // Lỗi: m1 và m2 đã bị move!
}

fn greet(g1: String, g2: String) {
    println!("{} {}!", g1, g2);`[]`
}
```

Trong ví dụ này, việc gọi `greet` di chuyển quyền sở hữu từ `m1` và `m2` vào các tham số của hàm `greet`. Cả hai chuỗi đều bị giải phóng (drop) khi hàm `greet` kết thúc, và do đó không thể được sử dụng lại bên trong `main`. Nếu chúng ta cố gắng đọc chúng ở dòng `format!(..)`, chương trình sẽ gặp lỗi vì vùng nhớ đã bị giải phóng. Trình biên dịch Rust sẽ từ chối biên dịch chương trình này:

```text
error[E0382]: borrow of moved value: `m1`
 --> test.rs:5:30
 (...rest of the error...)
```

Để tránh việc hàm lấy mất quyền sở hữu, một giải pháp thủ công là `greet` phải trả lại quyền sở hữu của các chuỗi qua tuple:

```aquascope,interpreter,horizontal
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");`[]`
    let (m1_again, m2_again) = greet(m1, m2);
    let s = format!("{} {}", m1_again, m2_again);`[]`
}

fn greet(g1: String, g2: String) -> (String, String) {
    println!("{} {}!", g1, g2);
    (g1, g2)
}
```

Cách làm này rất dài dòng và cồng kềnh. Thay vào đó, Rust cung cấp một cơ chế thanh lịch và hiệu quả hơn nhiều: **Tham chiếu (References)**.

### Tham Chiếu Là Các Con Trỏ Không Sở Hữu (References Are Non-Owning Pointers)

Một **tham chiếu** (reference) là một loại con trỏ trỏ tới dữ liệu mà không nắm quyền sở hữu dữ liệu đó. Dưới đây là cách viết lại chương trình `greet` bằng tham chiếu:

```aquascope,interpreter,horizontal
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");`[]`
    greet(&m1, &m2);`[]` // chú ý dấu &
    let s = format!("{} {}", m1, m2);
}

fn greet(g1: &String, g2: &String) { // tham số nhận &String
    `[]`println!("{} {}!", g1, g2);
}
```

Biểu thức `&m1` sử dụng toán tử `&` (ampersand) để tạo ra một tham chiếu đến (hoặc "mượn" - borrow) `m1`. Kiểu dữ liệu của tham số `g1` trong `greet` là `&String`, mang ý nghĩa "một tham chiếu đến một `String`".

Quan sát tại L2: `g1` là một tham chiếu trên stack trỏ tới `m1` (cũng trên stack), và `m1` là một `String` quản lý bộ nhớ chứa `"Hello"` trên heap.

Biến `m1` sở hữu chuỗi `"Hello"`, trong khi `g1` **không sở hữu** dữ liệu này. Do đó, khi `greet` kết thúc tại L3, không có dữ liệu heap nào bị giải phóng — chỉ có khung stack của `greet` bị thu hồi. Điều này hoàn toàn nhất quán với nguyên tắc quyền sở hữu của Rust: vì `g1` không phải là chủ sở hữu, Rust sẽ không giải phóng `"Hello"` khi `g1` hết hạn.

Các tham chiếu là **các con trỏ không sở hữu** (non-owning pointers).

### Giải Tham Chiếu Một Con Trỏ Để Truy Cập Dữ Liệu (Dereferencing)

Để truy cập giá trị mà con trỏ đang trỏ tới, ta sử dụng toán tử **giải tham chiếu** (dereference), được viết bằng dấu hoa thị (`*`):

```aquascope,interpreter
# fn main() {
let mut x: Box<i32> = Box::new(1);
let a: i32 = *x;         // *x đọc giá trị trên heap, do đó a = 1
*x += 1;                 // *x ở vế trái thay đổi giá trị trên heap,
                         //     x bây giờ trỏ tới giá trị 2

let r1: &Box<i32> = &x;  // r1 trỏ tới x trên stack
let b: i32 = **r1;       // hai lần giải tham chiếu để đến giá trị heap

let r2: &i32 = &*x;      // r2 trỏ trực tiếp đến giá trị trên heap
let c: i32 = *r2;`[]`    // chỉ cần một lần giải tham chiếu
# }
```

Trong thực tế, bạn thường không cần phải viết dấu `*` thủ công vì Rust có cơ chế tự động giải tham chiếu (deref coercion) khi gọi phương thức qua toán tử dấu chấm (`.`):

```rust,ignore
# fn main()  {
let x: Box<i32> = Box::new(-1);
let x_abs1 = i32::abs(*x); // giải tham chiếu tường minh
let x_abs2 = x.abs();      // giải tham chiếu ngầm định
assert_eq!(x_abs1, x_abs2);

let s = String::from("Hello");
let s_len1 = str::len(&s); // tham chiếu tường minh
let s_len2 = s.len();      // tham chiếu ngầm định
assert_eq!(s_len1, s_len2);
# }
```

{{#quiz ../quizzes/ch04-02-references-sec1-basics.toml}}

### Rust Ngăn Chặn Tình Trạng Vừa Aliasing Vừa Thay Đổi Dữ Liệu

Con trỏ là một tính năng mạnh mẽ nhưng cũng tiềm ẩn rủi ro lớn vì chúng tạo ra hiện tượng **Aliasing** (nhiều biến/con trỏ cùng trỏ tới một vùng dữ liệu). Bản thân Aliasing là vô hại, nhưng khi kết hợp với **Mutation** (thay đổi dữ liệu), thảm họa bộ nhớ có thể xảy ra:

- Một biến có thể giải phóng hoặc cấp phát lại vùng nhớ dữ liệu, khiến biến khác bị "bỏ rơi" trỏ vào vùng nhớ không còn hợp lệ (*Use-after-free*).
- Một biến thay đổi dữ liệu đồng thời với biến khác (*Data Race*), gây lỗi không lường trước được trong môi trường đa luồng.

Để minh họa, hãy xem xét một `Vec<i32>`. Khi thực hiện `v.push(4)`, nếu vector đầy sức chứa (capacity), nó sẽ cấp phát một mảng mới lớn hơn trên heap, sao chép dữ liệu cũ sang và **giải phóng mảng ban đầu**:

```aquascope,interpreter,shouldFail,horizontal
#fn main() {
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &i32 = &v[2];`[]`
v.push(4);`[]`
println!("Third element is {}", *num);`[]`
#}
```

Nếu đoạn mã trên chạy được: tại L1, `num` trỏ tới phần tử thứ ba của mảng cũ. Khi `v.push(4)` được gọi tại L2, mảng cũ bị giải phóng. Đến L3, `*num` sẽ đọc vào vùng nhớ rác đã bị giải phóng (hành vi chưa xác định)!

Để ngăn chặn triệt để lỗi này, Rust tuân theo một nguyên tắc sắt đá:

> **Nguyên tắc An toàn Con trỏ:** Dữ liệu không bao giờ được phép vừa bị Alias (nhiều nơi cùng trỏ tới) vừa bị Thay đổi (Mutate) cùng một lúc.

Rust kiểm soát nguyên tắc này thông qua **Bộ kiểm tra mượn (Borrow Checker)**.

> [!NOTE]
> **Khác biệt quan trọng với Python:**
> Trong Python, bạn có thể thoải mái vừa lặp qua một list vừa `append` hoặc xóa phần tử trong list đó thông qua các biến khác nhau (Aliasing + Mutation), nhưng điều này rất dễ gây ra các bug logic âm thầm hoặc crash lúc chạy (`RuntimeError: dictionary changed size during iteration`).
> Rust ngăn chặn lỗi này ngay từ lúc biên dịch: **Tại một thời điểm, một vùng dữ liệu chỉ cho phép: HOẶC có nhiều tham chiếu chỉ đọc (`&T`), HOẶC có duy nhất MỘT tham chiếu có thể ghi (`&mut T`)**, tuyệt đối không thể có cả hai cùng lúc!

### Cơ Chế Quyền Hạn (Permissions) của Borrow Checker

Ý tưởng cốt lõi của Borrow Checker là mỗi đối tượng/vùng nhớ (**place**) có ba loại quyền hạn trên dữ liệu của nó:

-   **Read** (Đọc - @Perm{read}): Dữ liệu có thể được đọc hoặc sao chép.
-   **Write** (Ghi - @Perm{write}): Dữ liệu có thể được chỉnh sửa.
-   **Own** (Sở hữu - @Perm{own}): Dữ liệu có thể được chuyển giao (move) hoặc giải phóng (drop).

Các quyền này chỉ tồn tại trong quá trình phân tích tĩnh của trình biên dịch, không tốn bất kỳ chi phí nào lúc runtime. Khi bạn tạo một tham chiếu (mượn dữ liệu), **các quyền hạn này sẽ tạm thời bị chuyển giao hoặc khóa lại**:

```aquascope,permissions,stepper
#fn main() {
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &i32 = &v[2];
println!("Third element is {}", *num);
v.push(4);
#}
```

1. Ban đầu, `let mut v` có đủ cả 3 quyền: @Perm{read}@Perm{write}@Perm{own}.
2. Khi `let num = &v[2]` được tạo (mượn bất biến):
   - `v` bị **tạm khóa** quyền @Perm[lost]{write} và @Perm[lost]{own}, chỉ còn giữ quyền @Perm{read}.
   - `num` nhận quyền @Perm{read}@Perm{own}, còn `*num` có quyền @Perm{read}.
3. Sau khi dòng `println!` sử dụng `num` xong, thời gian sống (lifetime) của `num` kết thúc:
   - `v` được **hoàn trả lại** quyền @Perm[gained]{write} và @Perm[gained]{own}.
4. Lúc này `v.push(4)` mới được phép thực thi an toàn.

### Tham Chiếu Khả Biến (Mutable References)

Tham chiếu bất biến (`&T`) cho phép nhiều nơi cùng đọc (chia sẻ). Khi bạn cần sửa dữ liệu mà không muốn chuyển quyền sở hữu, Rust cung cấp **Tham chiếu khả biến** (`&mut T`):

```aquascope,permissions,stepper,boundaries
#fn main() {
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &mut i32 = &mut v[2];
*num += 1;
println!("Third element is {}", *num);
println!("Vector is now {:?}", v);
#}
```

Khi một tham chiếu khả biến `&mut` được tạo ra:
1. Biến gốc `v` bị khóa **toàn bộ mọi quyền** (@Perm[lost]{read}, @Perm[lost]{write}, @Perm[lost]{own}) — nó không thể được đọc hay ghi qua `v` trong khi đang bị mượn khả biến.
2. Vùng nhớ `*num` nhận được cả quyền @Perm{read} lẫn @Perm{write}.
3. Nhờ vậy, dữ liệu chỉ có thể được truy cập và chỉnh sửa duy nhất qua con trỏ `num` (*Unique Access*), loại bỏ hoàn toàn nguy cơ xung đột dữ liệu.

### Dữ Liệu Phải Sống Lâu Hơn (Outlive) Tham Chiếu Đến Nó

Nguyên tắc an toàn thứ hai của Borrow Checker là: **dữ liệu gốc phải luôn tồn tại lâu hơn bất kỳ tham chiếu nào trỏ đến nó**.

Nếu bạn cố tình trả về một tham chiếu trỏ tới biến cục bộ trên stack:

```rust,ignore,does_not_compile
fn return_a_string() -> &String {
    let s = String::from("Hello world");
    &s // LỖI: s sẽ bị giải phóng khi hàm kết thúc, tham chiếu sẽ trỏ vào hư không!
}
```

Trình biên dịch Rust sẽ từ chối biên dịch ngay lập tức với lỗi `missing lifetime specifier` vì biến `s` sẽ bị hủy (drop) khi hàm kết thúc, và tham chiếu trả về sẽ trở thành con trỏ rác (*Dangling Reference*).

{{#quiz ../quizzes/ch04-02-references-sec3-safety.toml}}

### Tóm Tắt

- **Tham chiếu (`&` và `&mut`)** là con trỏ không sở hữu, cho phép truy cập dữ liệu mà không làm di chuyển quyền sở hữu.
- **Quy tắc vay mượn của Rust**:
  1. Bạn có thể có **bất kỳ số lượng tham chiếu bất biến (`&T`)** nào, HOẶC **duy nhất một tham chiếu khả biến (`&mut T`)** tại một thời điểm.
  2. Tham chiếu phải luôn **hợp lệ** (dữ liệu gốc không được phép bị hủy trước tham chiếu).
- Tất cả các quy tắc này được đảm bảo 100% tại thời điểm biên dịch thông qua **Borrow Checker**, mang lại sự an toàn bộ nhớ tuyệt đối mà không cần Garbage Collector.

[`String::push_str`]: https://doc.rust-lang.org/std/string/struct.String.html#method.push_str
[`Vec`]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[`Vec::push`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.push
