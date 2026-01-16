## Sửa Các Lỗi Về Quyền Sở Hữu

Học cách sửa một lỗi về quyền sở hữu (ownership error) là một kỹ năng cốt lõi trong Rust. Khi bộ kiểm tra mượn (borrow checker) từ chối code của bạn, bạn nên phản ứng như thế nào? Trong phần này, chúng ta sẽ thảo luận về một vài trường hợp nghiên cứu của các lỗi quyền sở hữu phổ biến. Mỗi trường hợp nghiên cứu sẽ trình bày một hàm bị trình biên dịch từ chối. Sau đó chúng tôi sẽ giải thích tại sao Rust từ chối hàm đó, và chỉ ra một vài cách để sửa nó.

Một chủ đề chung sẽ là việc hiểu xem một hàm _thực sự_ an toàn hay không an toàn. Rust sẽ luôn từ chối một chương trình không an toàn[^safe-subset]. Nhưng đôi khi, Rust cũng sẽ từ chối một chương trình an toàn. Các trường hợp nghiên cứu này sẽ chỉ ra cách phản ứng với các lỗi trong cả hai tình huống.

<!-- The last two sections have shown how a Rust program can be **unsafe** if it triggers undefined behavior. The ownership guarantee is that Rust will reject all unsafe programs. However, Rust will also reject *some* safe programs. Fixing an ownership error will depend on whether your program is *actually* safe or unsafe. -->

### Sửa Một Chương Trình Không An Toàn: Trả Về Một Tham Chiếu Đến Stack

Trường hợp nghiên cứu đầu tiên của chúng ta là về việc trả về một tham chiếu đến stack, giống như chúng ta đã thảo luận ở phần trước trong ["Dữ Liệu Phải Sống Lâu Hơn Tất Cả Các Tham Chiếu Đến Nó"](ch04-02-references-and-borrowing.html#data-must-outlive-all-of-its-references). Đây là hàm mà chúng ta đã xem xét:

```rust,ignore,does_not_compile
fn return_a_string() -> &String {
    let s = String::from("Hello world");
    &s
}
```

Khi suy nghĩ về cách sửa hàm này, chúng ta cần đặt câu hỏi: **tại sao chương trình này không an toàn?** Ở đây, vấn đề nằm ở vòng đời (lifetime) của dữ liệu được tham chiếu. Nếu bạn muốn truyền đi một tham chiếu đến một chuỗi, bạn phải đảm bảo rằng chuỗi bên dưới sống đủ lâu.

Tùy thuộc vào tình huống, đây là bốn cách bạn có thể kéo dài vòng đời của chuỗi. Một là di chuyển quyền sở hữu của chuỗi ra khỏi hàm, thay đổi `&String` thành `String`:

```rust
fn return_a_string() -> String {
    let s = String::from("Hello world");
    s
}
```

Một khả năng khác là trả về một chuỗi literal (chuỗi ký tự tĩnh), thứ sống mãi mãi (được chỉ định bởi `'static`). Giải pháp này áp dụng nếu chúng ta không bao giờ có ý định thay đổi chuỗi, và khi đó việc cấp phát heap là không cần thiết:

```rust
fn return_a_string() -> &'static str {
    "Hello world"
}
```

Một khả năng khác là hoãn việc kiểm tra mượn (borrow-checking) đến thời gian chạy (runtime) bằng cách sử dụng bộ thu gom rác (garbage collection). Ví dụ, bạn có thể sử dụng một [con trỏ đếm tham chiếu (reference-counted pointer)][rc]:

```rust
use std::rc::Rc;
fn return_a_string() -> Rc<String> {
    let s = Rc::new(String::from("Hello world"));
    Rc::clone(&s)
}
```

Chúng ta sẽ thảo luận về đếm tham chiếu kỹ hơn trong Chương 15.4 ["`Rc<T>`, Con Trỏ Thông Minh Đếm Tham Chiếu"](ch15-04-rc.html). Tóm lại, `Rc::clone` chỉ clone một con trỏ tới `s` và không phải chính dữ liệu đó. Tại runtime, `Rc` kiểm tra khi nào `Rc` cuối cùng trỏ đến dữ liệu đã bị drop, và sau đó giải phóng dữ liệu.

Lại một khả năng nữa là để cho người gọi cung cấp một "khe chứa" (slot) để đặt chuỗi vào bằng cách sử dụng một tham chiếu khả biến:

```rust
fn return_a_string(output: &mut String) {
    output.replace_range(.., "Hello world");
}
```

Với chiến lược này, người gọi chịu trách nhiệm tạo không gian cho chuỗi. Phong cách này có thể dài dòng, nhưng nó cũng có thể hiệu quả về bộ nhớ hơn nếu người gọi cần kiểm soát cẩn thận khi nào việc cấp phát xảy ra.

Chiến lược nào phù hợp nhất sẽ phụ thuộc vào ứng dụng của bạn. Nhưng ý tưởng chính là nhận ra vấn đề gốc rễ nằm dưới lỗi quyền sở hữu ở bề mặt. Chuỗi của tôi nên sống bao lâu? Ai nên chịu trách nhiệm giải phóng nó? Một khi bạn có câu trả lời rõ ràng cho những câu hỏi đó, thì vấn đề chỉ là thay đổi API của bạn cho phù hợp.

### Sửa Một Chương Trình Không An Toàn: Không Đủ Quyền Hạn

Một vấn đề phổ biến khác là cố gắng thay đổi dữ liệu chỉ đọc, hoặc cố gắng drop dữ liệu đằng sau một tham chiếu. Ví dụ, giả sử chúng ta cố gắng viết một hàm `stringify_name_with_title`. Hàm này được cho là tạo ra tên đầy đủ của một người từ một vector các phần của tên, bao gồm thêm một danh xưng.

```aquascope,permissions,stepper,boundaries,shouldFail
fn stringify_name_with_title(name: &Vec<String>) -> String {
    name.push(String::from("Esq."));`{}`
    let full = name.join(" ");
    full
}

// ideally: ["Ferris", "Jr."] => "Ferris Jr. Esq."
```

Chương trình này bị bộ kiểm tra mượn từ chối bởi vì `name` là một tham chiếu bất biến, nhưng `name.push(..)` yêu cầu quyền @Perm{write} (ghi). Chương trình này không an toàn bởi vì `push` có thể làm mất hiệu lực các tham chiếu khác tới `name` bên ngoài `stringify_name_with_title`, như thế này:

```aquascope,interpreter,shouldFail,horizontal
#fn stringify_name_with_title(name: &Vec<String>) -> String {
#    name.push(String::from("Esq."));
#    let full = name.join(" ");
#    full
#}
fn main() {
    let name = vec![String::from("Ferris")];
    let first = &name[0];`[]`
    stringify_name_with_title(&name);`[]`
    println!("{}", first);`[]`
}
```

Trong ví dụ này, một tham chiếu `first` tới `name[0]` được tạo ra trước khi gọi `stringify_name_with_title`. Hàm `name.push(..)` tái cấp phát nội dung của `name`, điều này làm mất hiệu lực `first`, khiến cho `println` đọc bộ nhớ đã bị giải phóng.

Vậy làm thế nào chúng ta sửa API này? Một giải pháp đơn giản là thay đổi kiểu của name từ `&Vec<String>` thành `&mut Vec<String>`:

```rust,ignore
fn stringify_name_with_title(name: &mut Vec<String>) -> String {
    name.push(String::from("Esq."));
    let full = name.join(" ");
    full
}
```

Nhưng đây không phải là một giải pháp tốt! **Các hàm không nên thay đổi đầu vào của chúng nếu người gọi không mong đợi điều đó.** Một người gọi `stringify_name_with_title` có lẽ không mong đợi vector của họ bị sửa đổi bởi hàm này. Một hàm khác như `add_title_to_name` có thể được mong đợi sẽ thay đổi đầu vào của nó, nhưng không phải hàm của chúng ta.

Một lựa chọn khác là lấy quyền sở hữu của name, bằng cách thay đổi `&Vec<String>` thành `Vec<String>`:

```rust,ignore
fn stringify_name_with_title(mut name: Vec<String>) -> String {
    name.push(String::from("Esq."));
    let full = name.join(" ");
    full
}
```

Nhưng đây cũng không phải là một giải pháp tốt! **Rất hiếm khi các hàm Rust lấy quyền sở hữu của các cấu trúc dữ liệu sở hữu heap như `Vec` và `String`.** Phiên bản này của `stringify_name_with_title` sẽ làm cho đầu vào `name` không thể dùng được nữa, điều này rất phiền toái cho người gọi như chúng ta đã thảo luận ở phần đầu của ["Tham Chiếu và Mượn"](ch04-02-references-and-borrowing.html).

Vì vậy lựa chọn `&Vec` thực sự là một lựa chọn tốt, cái mà chúng ta _không_ muốn thay đổi. Thay vào đó, chúng ta có thể thay đổi phần thân của hàm. Có nhiều cách sửa có thể khác nhau về lượng bộ nhớ chúng sử dụng. Một khả năng là clone đầu vào `name`:

```rust,ignore
fn stringify_name_with_title(name: &Vec<String>) -> String {
    let mut name_clone = name.clone();
    name_clone.push(String::from("Esq."));
    let full = name_clone.join(" ");
    full
}
```

Bằng cách clone `name`, chúng ta được phép thay đổi bản sao cục bộ của vector. Tuy nhiên, việc clone sao chép mọi chuỗi trong đầu vào. Chúng ta có thể tránh các sao chép không cần thiết bằng cách thêm hậu tố vào sau:

```rust,ignore
fn stringify_name_with_title(name: &Vec<String>) -> String {
    let mut full = name.join(" ");
    full.push_str(" Esq.");
    full
}
```

Giải pháp này hoạt động vì [`slice::join`] đã sao chép dữ liệu trong `name` vào chuỗi `full`.

Nói chung, viết các hàm Rust là một sự cân bằng cẩn thận của việc yêu cầu mức độ quyền hạn _đúng_. Đối với ví dụ này, cách đúng chuẩn (idiomatic) nhất là chỉ mong đợi quyền đọc trên `name`.

{{#quiz ../quizzes/ch04-03-fixing-ownership-errors-sec1-idioms.toml}}

### Sửa Một Chương Trình Không An Toàn: Aliasing và Thay Đổi Một Cấu Trúc Dữ Liệu

Một thao tác không an toàn khác là sử dụng một tham chiếu đến dữ liệu heap mà dữ liệu đó bị giải phóng bởi một alias khác. Ví dụ, đây là một hàm lấy một tham chiếu đến chuỗi lớn nhất trong một vector, và sau đó sử dụng nó trong khi thay đổi vector đó:

```aquascope,permissions,stepper,boundaries,shouldFail
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {`(focus,paths:*dst)`
    let largest: &String =
      dst.iter().max_by_key(|s| s.len()).unwrap();`(focus,paths:*dst)`
    for s in src {
        if s.len() > largest.len() {
            dst.push(s.clone());`{}`
        }
    }
}
```

> _Lưu ý:_ ví dụ này sử dụng [iterators] và [closures] để tìm một tham chiếu đến chuỗi lớn nhất một cách ngắn gọn. Chúng ta sẽ thảo luận về các tính năng đó trong các chương sau, và bây giờ chúng ta sẽ cung cấp một cảm nhận trực quan về cách các tính năng hoạt động ở đây.

Chương trình này bị bộ kiểm tra mượn từ chối bởi vì `let largest = ..` loại bỏ quyền @Perm{write} trên `dst`. Tuy nhiên, `dst.push(..)` yêu cầu quyền @Perm{write}. Một lần nữa, chúng ta nên hỏi: **tại sao chương trình này không an toàn?** Bởi vì `dst.push(..)` có thể giải phóng nội dung của `dst`, làm mất hiệu lực tham chiếu `largest`.

Để sửa chương trình, hiểu biết then chốt là chúng ta cần rút ngắn vòng đời của `largest` để không chồng chéo với `dst.push(..)`. Một khả năng là clone `largest`:

```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest: String = dst.iter().max_by_key(|s| s.len()).unwrap().clone();
    for s in src {
        if s.len() > largest.len() {
            dst.push(s.clone());
        }
    }
}
```

Tuy nhiên, điều này có thể gây ảnh hưởng hiệu năng vì việc cấp phát và sao chép dữ liệu chuỗi.

Một khả năng khác là thực hiện tất cả các so sánh độ dài trước, và sau đó thay đổi `dst` sau:

```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest: &String = dst.iter().max_by_key(|s| s.len()).unwrap();
    let to_add: Vec<String> =
        src.iter().filter(|s| s.len() > largest.len()).cloned().collect();
    dst.extend(to_add);
}
```

Tuy nhiên, điều này cũng gây ảnh hưởng hiệu năng vì việc cấp phát vector `to_add`.

Một khả năng cuối cùng là sao chép ra độ dài của `largest`, vì chúng ta thực sự không cần nội dung của `largest`, chỉ cần độ dài của nó.
Giải pháp này được cho là đúng chuẩn nhất và hiệu năng cao nhất:

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

Tất cả các giải pháp này đều có chung một ý tưởng chính: rút ngắn vòng đời của các lần mượn trên `dst` để không chồng chéo với một sự thay đổi tới `dst`.

### Sửa Một Chương Trình Không An Toàn: Sao Chép so với Di Chuyển Ra Khỏi Một Tập Hợp

Một sự nhầm lẫn phổ biến cho người học Rust xảy ra khi sao chép dữ liệu ra khỏi một tập hợp, như một vector. Ví dụ, đây là một chương trình an toàn sao chép một số ra khỏi một vector:

```aquascope,permissions,stepper,boundaries
#fn main() {
let v: Vec<i32> = vec![0, 1, 2];
let n_ref: &i32 = &v[0];`(focus,paths:*n_ref)`
let n: i32 = *n_ref;`{}`
#}
```

Thao tác giải tham chiếu `*n_ref` chỉ mong đợi quyền @Perm{read}, quyền mà đường dẫn (path) `*n_ref` có. Nhưng điều gì xảy ra nếu chúng ta thay đổi kiểu của các phần tử trong vector từ `i32` sang `String`? Khi đó hóa ra chúng ta không còn có các quyền hạn cần thiết nữa:

```aquascope,permissions,stepper,boundaries,shouldFail
#fn main() {
let v: Vec<String> =
  vec![String::from("Hello world")];
let s_ref: &String = &v[0];`(focus,paths:*s_ref)`
let s: String = *s_ref;`[]``{}`
#}
```

Chương trình đầu tiên sẽ biên dịch, nhưng chương trình thứ hai sẽ không biên dịch. Rust đưa ra thông báo lỗi sau:

```text
error[E0507]: cannot move out of `*s_ref` which is behind a shared reference
 --> test.rs:4:9
  |
4 | let s = *s_ref;
  |         ^^^^^^
  |         |
  |         move occurs because `*s_ref` has type `String`, which does not implement the `Copy` trait
```

Vấn đề là vector `v` sở hữu chuỗi "Hello world". Khi chúng ta giải tham chiếu `s_ref`, việc đó cố gắng lấy quyền sở hữu chuỗi từ vector. Nhưng các tham chiếu là các con trỏ không sở hữu &mdash; chúng ta không thể lấy quyền sở hữu _thông qua_ một tham chiếu. Do đó Rust phàn nàn rằng chúng ta "cannot move out of \[...\] a shared reference" (không thể di chuyển ra khỏi một tham chiếu chia sẻ).

Nhưng tại sao điều này không an toàn? Chúng ta có thể minh họa vấn đề bằng cách mô phỏng chương trình bị từ chối:

```aquascope,interpreter,shouldFail,horizontal
#fn main() {
let v: Vec<String> =
  vec![String::from("Hello world")];
let s_ref: &String = &v[0];`(focus,paths:*s_ref)`
let s: String = *s_ref;`[]``{}`

// These drops are normally implicit, but we've added them for clarity.
drop(s);`[]`
drop(v);`[]`
#}
```

Điều xảy ra ở đây là một **double-free (giải phóng kép).** Sau khi thực thi `let s = *s_ref`, cả `v` và `s` đều nghĩ chúng sở hữu "Hello world". Sau khi `s` bị drop, "Hello world" bị giải phóng. Sau đó `v` bị drop, và hành vi không xác định xảy ra khi chuỗi được giải phóng lần thứ hai.

> _Lưu ý:_ sau khi thực thi `s = *s_ref`, chúng ta thậm chí không cần sử dụng `v` hoặc `s` để gây ra hành vi không xác định thông qua double-free. Ngay khi chúng ta di chuyển chuỗi ra khỏi `s_ref`, hành vi không xác định sẽ xảy ra một khi các phần tử bị drop.

Tuy nhiên, hành vi không xác định này không xảy ra khi vector chứa các phần tử `i32`. Sự khác biệt là việc sao chép một `String` sao chép một con trỏ tới dữ liệu heap. Sao chép một `i32` thì không.
Theo thuật ngữ kỹ thuật, Rust nói rằng kiểu `i32` thực hiện `Copy` trait, trong khi `String` không thực hiện `Copy` (chúng ta sẽ thảo luận về trait trong một chương sau).

Tóm lại, **nếu một giá trị không sở hữu dữ liệu heap, thì nó có thể được sao chép mà không cần di chuyển (move).** Ví dụ:

-   Một `i32` **không** sở hữu dữ liệu heap, nên nó **có thể** được sao chép mà không cần di chuyển.
-   Một `String` **có** sở hữu dữ liệu heap, nên nó **không thể** được sao chép mà không cần di chuyển.
-   Một `&String` **không** sở hữu dữ liệu heap, nên nó **có thể** được sao chép mà không cần di chuyển.

> _Lưu ý:_ Một ngoại lệ cho quy tắc này là các tham chiếu khả biến. Ví dụ, `&mut i32` không phải là một kiểu copyable (có thể sao chép). Vì vậy nếu bạn làm điều gì đó như:
>
> ```rust,ignore
> let mut n = 0;
> let a = &mut n;
> let b = a;
> ```
>
> Thì `a` không thể được sử dụng sau khi được gán cho `b`. Điều đó ngăn cản hai tham chiếu khả biến tới cùng một dữ liệu được sử dụng cùng một lúc.

Vậy nếu chúng ta có một vector của các kiểu không `Copy` như `String`, thì làm thế nào chúng ta lấy quyền truy cập an toàn tới một phần tử của vector? Đây là một vài cách khác nhau để làm điều đó một cách an toàn. Đầu tiên, bạn có thể tránh lấy quyền sở hữu của chuỗi và chỉ sử dụng một tham chiếu bất biến:

```rust,ignore
# fn main() {
let v: Vec<String> = vec![String::from("Hello world")];
let s_ref: &String = &v[0];
println!("{s_ref}!");
# }
```

Thứ hai, bạn có thể clone dữ liệu nếu bạn muốn lấy quyền sở hữu của chuỗi trong khi để yên vector:

```rust,ignore
# fn main() {
let v: Vec<String> = vec![String::from("Hello world")];
let mut s: String = v[0].clone();
s.push('!');
println!("{s}");
# }
```

Cuối cùng, bạn có thể sử dụng một phương thức như [`Vec::remove`] để di chuyển chuỗi ra khỏi vector:

```rust,ignore
# fn main() {
let mut v: Vec<String> = vec![String::from("Hello world")];
let mut s: String = v.remove(0);
s.push('!');
println!("{s}");
assert!(v.len() == 0);
# }
```

### Sửa Một Chương Trình An Toàn: Thay Đổi Các Trường Tuple Khác Nhau

Các ví dụ trên là các trường hợp mà một chương trình không an toàn. Rust cũng có thể từ chối các chương trình an toàn. Một vấn đề phổ biến là Rust cố gắng theo dõi các quyền hạn ở mức độ chi tiết (fine-grained level). Tuy nhiên, Rust có thể gộp hai place (vị trí) khác nhau thành cùng một place.

Đầu tiên hãy xem một ví dụ về theo dõi quyền hạn chi tiết mà vượt qua được bộ kiểm tra mượn. Chương trình này cho thấy cách bạn có thể mượn một trường của một tuple, và ghi vào một trường khác của cùng tuple đó:

```aquascope,permissions,stepper,boundaries
#fn main() {
let mut name = (
    String::from("Ferris"),
    String::from("Rustacean")
);`(focus,paths:name)`
let first = &name.0;`(focus,paths:name)`
name.1.push_str(", Esq.");`{}`
println!("{first} {}", name.1);
#}
```

Câu lệnh `let first = &name.0` mượn `name.0`. Việc mượn này loại bỏ quyền @Perm{write}@Perm{own} từ `name.0`. Nó cũng loại bỏ quyền @Perm{write}@Perm{own} từ `name`. (Ví dụ, người ta không thể truyền `name` cho một hàm nhận đầu vào là một giá trị kiểu `(String, String)`.) Nhưng `name.1` vẫn giữ lại quyền @Perm{write}, nên việc thực hiện `name.1.push_str(...)` là một thao tác hợp lệ.

Tuy nhiên, Rust có thể mất dấu chính xác những place nào được mượn. Ví dụ, giả sử chúng ta cấu trúc lại biểu thức `&name.0` vào trong một hàm `get_first`. Chú ý cách mà sau khi gọi `get_first(&name)`, Rust bây giờ loại bỏ quyền @Perm{write} trên `name.1`:

```aquascope,permissions,stepper,boundaries,shouldFail
fn get_first(name: &(String, String)) -> &String {
    &name.0
}

fn main() {
    let mut name = (
        String::from("Ferris"),
        String::from("Rustacean")
    );
    let first = get_first(&name);`(focus,paths:name)`
    name.1.push_str(", Esq.");`{}`
    println!("{first} {}", name.1);
}
```

Bây giờ chúng ta không thể thực hiện `name.1.push_str(..)`! Rust sẽ trả về lỗi này:

```text
error[E0502]: cannot borrow `name.1` as mutable because it is also borrowed as immutable
  --> test.rs:11:5
   |
10 |     let first = get_first(&name);
   |                           ----- immutable borrow occurs here
11 |     name.1.push_str(", Esq.");
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^ mutable borrow occurs here
12 |     println!("{first} {}", name.1);
   |                ----- immutable borrow later used here
```

Điều đó thật lạ, vì chương trình vẫn an toàn trước khi chúng ta chỉnh sửa nó. Chỉnh sửa chúng ta đã làm không thay đổi ý nghĩa hành vi runtime. Vậy tại sao việc chúng ta đặt `&name.0` vào trong một hàm lại quan trọng?

Vấn đề là Rust không nhìn vào phần cài đặt của `get_first` khi quyết định `get_first(&name)` nên mượn cái gì. Rust chỉ nhìn vào chữ ký kiểu (type signature), cái mà chỉ nói rằng "một `String` nào đó trong đầu vào được mượn". Rust sau đó quyết định một cách bảo thủ rằng cả `name.0` và `name.1` đều bị mượn, và loại bỏ quyền ghi và sở hữu trên cả hai.

Hãy nhớ, ý tưởng chính là **chương trình ở trên là an toàn.** Nó không có hành vi không xác định! Một phiên bản tương lai của Rust có thể đủ thông minh để cho phép nó biên dịch, nhưng cho hôm nay, nó bị từ chối. Vậy chúng ta nên làm việc xung quanh bộ kiểm tra mượn như thế nào hôm nay? Một khả năng là inline (viết trực tiếp) biểu thức `&name.0`, giống như trong chương trình gốc. Một khả năng khác là hoãn việc kiểm tra mượn đến runtime với [cells], cái mà chúng ta sẽ thảo luận trong các chương tương lai.

### Sửa Một Chương Trình An Toàn: Thay Đổi Các Phần Tử Mảng Khác Nhau

Một loại vấn đề tương tự nảy sinh khi chúng ta mượn các phần tử của một mảng. Ví dụ, quan sát những place nào bị mượn khi chúng ta lấy một tham chiếu khả biến tới một mảng:

```aquascope,permissions,stepper,boundaries
#fn main() {
let mut a = [0, 1, 2, 3];
let x = &mut a[1];`(focus,paths:a[_])`
*x += 1;`(focus,paths:a[_])`
println!("{a:?}");
#}
```

Bộ kiểm tra mượn của Rust không chứa các place khác nhau cho `a[0]`, `a[1]`, và vân vân. Nó sử dụng một place đơn lẻ `a[_]` đại diện cho _tất cả_ các chỉ mục (indexes) của `a`. Rust làm điều này bởi vì nó không phải lúc nào cũng xác định được giá trị của một chỉ mục. Ví dụ, hãy tưởng tượng một kịch bản phức tạp hơn như thế này:

```rust,ignore
let idx = a_complex_function();
let x = &mut a[idx];
```

Giá trị của `idx` là gì? Rust sẽ không đoán, nên nó giả định `idx` có thể là bất cứ cái gì. Ví dụ, giả sử chúng ta cố gắng đọc từ một chỉ mục mảng trong khi ghi vào một chỉ mục khác:

```aquascope,permissions,boundaries,stepper,shouldFail
#fn main() {
let mut a = [0, 1, 2, 3];
let x = &mut a[1];`(focus,paths:a[_])`
let y = &a[2];`{}`
*x += *y;
#}
```

Tuy nhiên, Rust sẽ từ chối chương trình này bởi vì `a` đã trao quyền đọc của nó cho `x`. Thông báo lỗi của trình biên dịch cũng nói điều tương tự:

```text
error[E0502]: cannot borrow `a[_]` as immutable because it is also borrowed as mutable
 --> test.rs:4:9
  |
3 | let x = &mut a[1];
  |         --------- mutable borrow occurs here
4 | let y = &a[2];
  |         ^^^^^ immutable borrow occurs here
5 | *x += *y;
  | -------- mutable borrow later used here
```

<!-- However, Rust will reject this program because `a` gave its read permission to `x`. -->

Một lần nữa, **chương trình này là an toàn.** Đối với những trường hợp như thế này, Rust thường cung cấp một hàm trong thư viện chuẩn có thể làm việc xung quanh bộ kiểm tra mượn. Ví dụ, chúng ta có thể sử dụng [`slice::split_at_mut`][split_at_mut]:

```rust,ignore
# fn main() {
let mut a = [0, 1, 2, 3];
let (a_l, a_r) = a.split_at_mut(2);
let x = &mut a_l[1];
let y = &a_r[0];
*x += *y;
# }
```

Bạn có thể tự hỏi, nhưng `split_at_mut` được cài đặt như thế nào? Trong một số thư viện Rust, đặc biệt là các kiểu cốt lõi như `Vec` hoặc `slice`, bạn sẽ thường tìm thấy các **khối `unsafe`**. Các khối `unsafe` cho phép sử dụng các con trỏ "thô" (raw pointers), thứ không được kiểm tra độ an toàn bởi bộ kiểm tra mượn. Ví dụ, chúng ta có thể sử dụng một khối unsafe để hoàn thành tác vụ của mình:

```rust,ignore
# fn main() {
let mut a = [0, 1, 2, 3];
let x = &mut a[1] as *mut i32;
let y = &a[2] as *const i32;
unsafe { *x += *y; } // DO NOT DO THIS unless you know what you're doing!
# }
```

Code unsafe đôi khi là cần thiết để làm việc xung quanh các hạn chế của bộ kiểm tra mượn. Như một chiến lược chung, giả sử bộ kiểm tra mượn từ chối một chương trình mà bạn nghĩ là thực sự an toàn. Khi đó bạn nên tìm kiếm các hàm thư viện chuẩn (như `split_at_mut`) có chứa các khối `unsafe` giải quyết vấn đề của bạn. Chúng ta sẽ thảo luận về code unsafe xa hơn trong [Chương 20][unsafe]. Hiện tại, chỉ cần nhận thức rằng code unsafe là cách Rust cài đặt một số mẫu hình (patterns) nhất định mà nếu không có nó thì không thể thực hiện được.

{{#quiz ../quizzes/ch04-03-fixing-ownership-errors-sec2-safety.toml}}

### Tóm Tắt

Khi sửa một lỗi quyền sở hữu, bạn nên tự hỏi mình: chương trình của tôi có thực sự không an toàn không? Nếu có, thì bạn cần hiểu nguyên nhân gốc rễ của sự không an toàn. Nếu không, thì bạn cần hiểu các hạn chế của bộ kiểm tra mượn để làm việc xung quanh chúng.

[rc]: https://doc.rust-lang.org/std/rc/index.html
[cells]: https://doc.rust-lang.org/std/cell/index.html
[split_at_mut]: https://doc.rust-lang.org/std/primitive.slice.html#method.split_at_mut
[unsafe]: ch19-01-unsafe-rust.html
[`Vec::remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.remove
[`slice::join`]: https://doc.rust-lang.org/std/primitive.slice.html#method.join
[iterators]: ch13-02-iterators.html
[closures]: ch13-01-closures.html

[^safe-subset]: Sự đảm bảo này áp dụng cho các chương trình được viết trong "tập con an toàn" (safe subset) của Rust. Nếu bạn sử dụng code `unsafe` hoặc gọi các thành phần không an toàn (như gọi một thư viện C), thì bạn phải cẩn thận hơn để tránh hành vi không xác định.
