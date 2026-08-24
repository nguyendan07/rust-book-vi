## Kiểu Slice

_Slice_ (lát cắt / phân đoạn) cho phép bạn tham chiếu tới một chuỗi liên tiếp các phần tử trong một [tập hợp dữ liệu (collection)](ch08-00-common-collections.md) thay vì toàn bộ tập hợp đó. Một slice là một loại tham chiếu đặc biệt, vì vậy nó là một con trỏ không nắm quyền sở hữu (non-owning pointer).

Để hiểu tại sao slice lại hữu ích, hãy cùng giải quyết một bài toán lập trình nhỏ: viết một hàm nhận vào một chuỗi các từ được phân tách bằng dấu cách và trả về từ đầu tiên tìm thấy trong chuỗi đó. Nếu chuỗi không chứa dấu cách nào, thì toàn bộ chuỗi chính là từ đầu tiên.

Nếu chưa có slice, chữ ký hàm có thể trông như sau:

```rust,ignore
fn first_word(s: &String) -> ?
```

Chúng ta nhận vào `&String` để mượn dữ liệu mà không lấy quyền sở hữu. Nhưng hàm nên trả về kiểu gì để biểu thị *một phần* của chuỗi? Chúng ta có thể thử trả về chỉ số byte (index) kết thúc của từ đầu tiên:

<Listing number="4-7" file-name="src/main.rs" caption="Hàm `first_word` trả về giá trị chỉ số byte kết thúc trong `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Để kiểm tra từng byte xem có phải dấu cách hay không, ta chuyển `String` thành một mảng các byte bằng `s.as_bytes()`, sau đó lặp qua từng phần tử bằng `.iter().enumerate()`.

Tuy nhiên, cách tiếp cận này có một lỗ hổng logic nghiêm trọng: giá trị trả về `usize` là một con số độc lập, không bị ràng buộc vào dữ liệu gốc `s`. Hãy xem Danh sách 4-8:

<Listing number="4-8" file-name="src/main.rs" caption="Lưu trữ chỉ số từ `first_word` và sau đó thay đổi nội dung của `String`">

```aquascope,interpreter+permissions,boundaries,stepper,horizontal
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {
    let mut s = String::from("hello world");`(focus)`
    let word = first_word(&s);`[]`
    s.clear();`[]``{}`
    // Lúc này word vẫn bằng 5, nhưng s đã hoàn toàn trống rỗng!
}
```

</Listing>

Chương trình trên biên dịch mà không gặp bất kỳ lỗi nào! Biến `word` vẫn giữ giá trị `5`, nhưng dữ liệu trong `s` đã bị xóa sạch bằng `s.clear()`. Nếu sau đó bạn dùng con số `5` này để truy xuất vào `s`, chương trình sẽ bị lỗi hoặc đọc dữ liệu sai lệch. Việc phải tự tay quản lý các chỉ số bắt đầu và kết thúc rất dễ gây ra các bug mất đồng bộ dữ liệu.

May mắn thay, Rust giải quyết triệt để bài toán này bằng: **String Slices**.

### String Slices

Một _string slice_ là một tham chiếu trỏ tới một phần của `String`:

```aquascope,interpreter
#fn main() {
let s = String::from("hello world");

let hello: &str = &s[0..5];
let world: &str = &s[6..11];
let s2: &String = &s; `[]`
#}
```

Cú pháp `[starting_index..ending_index]` tạo ra một slice bắt đầu từ `starting_index` đến trước `ending_index` (không bao gồm `ending_index`).

String slice là một **con trỏ béo (fat pointer)**: nó lưu trữ **2 thông tin** trên stack:
1. Con trỏ trỏ tới byte bắt đầu của slice trong chuỗi gốc.
2. Độ dài (`len`) của slice.

Nhờ lưu trữ trực tiếp con trỏ và độ dài, slice trỏ thẳng vào dữ liệu có sẵn mà hoàn toàn không cần sao chép hay cấp phát thêm bất kỳ bộ nhớ heap nào.

> [!NOTE]
> **Khác biệt cốt lõi với Python:**
> - Trong Python, khi bạn cắt chuỗi hoặc list: `sub = s[0:5]`, Python sẽ **cấp phát một vùng nhớ mới và sao chép toàn bộ dữ liệu** sang chuỗi/list mới.
> - Trong Rust, string slice `&s[0..5]` là một **tham chiếu Fat Pointer (zero-copy)** trỏ trực tiếp vào dữ liệu gốc kèm theo độ dài. Không có phép cấp phát hay sao chép bộ nhớ nào diễn ra.
> - Đặc biệt, vì slice là một tham chiếu mượn bất biến (`&str`), **Borrow Checker của Rust sẽ khóa quyền sửa đổi của chuỗi gốc `s`** trong suốt thời gian slice còn được sử dụng, loại bỏ 100% lỗi mất đồng bộ dữ liệu như ở Danh sách 4-8.

#### Cú Pháp Range Rút Gọn

Rust hỗ trợ viết gọn cú pháp phạm vi (`..`):
- Bắt đầu từ chỉ số 0: `&s[..2]` tương đương `&s[0..2]`
- Kéo dài đến hết chuỗi: `&s[3..]` tương đương `&s[3..s.len()]`
- Toàn bộ chuỗi: `&s[..]` tương đương `&s[0..s.len()]`

> _Lưu ý_: Chỉ số cắt của string slice bắt buộc phải nằm đúng ranh giới của các ký tự mã hóa UTF-8 hợp lệ. Nếu bạn cố gắng cắt ở giữa một ký tự nhiều byte, chương trình sẽ panic báo lỗi.

#### Viết Lại `first_word` Sử Dụng String Slice

Hãy viết lại hàm `first_word` với kiểu trả về là `&str`:

<Listing file-name="src/main.rs" caption="Hàm `first_word` trả về một string slice">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Khi sử dụng hàm này với đoạn mã sửa đổi chuỗi:

```aquascope,permissions,boundaries,stepper,shouldFail
#fn first_word(s: &String) -> &str {
#    let bytes = s.as_bytes();
#    for (i, &item) in bytes.iter().enumerate() {
#        if item == b' ' {
#            return &s[0..i];
#        }
#    }
#    &s[..]
#}
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);`(focus,paths:s)`
    s.clear();`{}` // LỖI BIÊN DỊCH: s không thể bị thay đổi khi đang bị mượn bất biến bởi `word`!
    println!("the first word is: {}", word);
}
```

Trình biên dịch Rust sẽ chặn đứng lỗi này ngay tại thời điểm biên dịch:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Vì `s.clear()` cần mượn khả biến (`&mut s`) để xóa nội dung chuỗi, trong khi `word` vẫn đang giữ tham chiếu bất biến (`&str`) để in ra ở dòng tiếp theo, Rust cấm hai tham chiếu này tồn tại cùng lúc và báo lỗi rõ ràng.

#### String Literals Thực Chất Là Các Slices

Bây giờ bạn có thể hiểu bản chất của các chuỗi cố định (string literals):

```rust
let s = "Hello, world!";
```

Kiểu của `s` ở đây chính là `&str` — một slice bất biến trỏ trực tiếp vào vùng nhớ tĩnh đã được nhúng sẵn bên trong tệp nhị phân đã biên dịch của chương trình.

#### String Slice Dưới Dạng Tham Số Hàm

Một lập trình viên Rust giàu kinh nghiệm sẽ luôn viết chữ ký hàm nhận `&str` thay vì `&String`:

<Listing number="4-9" file-name="src/main.rs" caption="Cải tiến hàm `first_word` bằng cách nhận tham số kiểu `&str`">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Việc nhận `&str` giúp hàm linh hoạt hơn rất nhiều: bạn có thể truyền thẳng một string literal `&str`, một slice `&s[0..5]`, hoặc truyền trực tiếp một tham chiếu `&String` (nhờ tính năng tự động ép kiểu Deref Coercion):

<Listing file-name="src/main.rs" caption="Sử dụng hàm `first_word` với cả String và string slice">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Các Kiểu Slice Khác

Ngoài chuỗi, bạn có thể tạo slice cho bất kỳ tập hợp dữ liệu nào, ví dụ như mảng số nguyên:

```rust
let a = [1, 2, 3, 4, 5];
let slice: &[i32] = &a[1..3];
assert_eq!(slice, &[2, 3]);
```

Slice này có kiểu là `&[i32]`, hoạt động theo nguyên lý Fat Pointer tương tự: lưu con trỏ trỏ tới phần tử đầu tiên và độ dài của phân đoạn.

{{#quiz ../quizzes/ch04-04-slices.toml}}

## Tóm Tắt

- **Slice** là các tham chiếu Fat Pointer trỏ tới một phân đoạn liên tiếp trong tập hợp dữ liệu (như `String`, mảng, `Vec`) mà không tốn chi phí sao chép bộ nhớ (zero-cost abstraction).
- **String slice (`&str`)** là cầu nối an toàn, liên kết chặt chẽ phần dữ liệu được trỏ tới với dữ liệu gốc, được **Borrow Checker** bảo vệ nghiêm ngặt để đảm bảo an toàn bộ nhớ tuyệt đối tại thời điểm biên dịch.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods
