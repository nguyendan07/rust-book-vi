## Tóm Tắt Về Quyền Sở Hữu

Chương này đã giới thiệu rất nhiều khái niệm mới như quyền sở hữu, vay mượn, và slice. Nếu bạn chưa quen thuộc với lập trình hệ thống, chương này cũng giới thiệu các khái niệm mới như cấp phát bộ nhớ, stack so với heap, con trỏ, và hành vi chưa xác định. Trước khi chúng ta chuyển sang phần còn lại của Rust, trước tiên hãy dừng lại và hít thở một chút. Chúng ta sẽ ôn tập và thực hành với các khái niệm chính từ chương này.

### Quyền Sở Hữu vs Thu Gom Rác

Để đặt quyền sở hữu vào ngữ cảnh, chúng ta nên nói về **thu gom rác** (garbage collection). Hầu hết các ngôn ngữ lập trình sử dụng bộ thu gom rác để quản lý bộ nhớ, chẳng hạn như trong Python, Javascript, Java, và Go. Một bộ thu gom rác hoạt động tại thời gian chạy song song với một chương trình đang chạy (ít nhất là với một bộ thu gom theo vết - tracing collector). Bộ thu gom quét qua bộ nhớ để tìm dữ liệu không còn được sử dụng &mdash; nghĩa là, chương trình đang chạy không còn có thể tiếp cận dữ liệu đó từ một biến cục bộ của hàm. Sau đó bộ thu gom sẽ thu hồi bộ nhớ không sử dụng để dùng sau này.

Lợi ích chính của bộ thu gom rác là nó tránh được hành vi chưa xác định (chẳng hạn như sử dụng bộ nhớ đã được giải phóng), điều có thể xảy ra trong C hoặc C++. Thu gom rác cũng tránh được nhu cầu về một hệ thống kiểu phức tạp để kiểm tra hành vi chưa xác định, giống như trong Rust. Tuy nhiên, có một vài nhược điểm đối với thu gom rác. Một nhược điểm rõ ràng là hiệu suất, vì thu gom rác phát sinh chi phí nhỏ thường xuyên (đối với đếm tham chiếu - reference counting, như trong Python và Swift) hoặc chi phí lớn không thường xuyên (đối với theo vết - tracing, như trong tất cả các ngôn ngữ dùng GC khác).

Nhưng một nhược điểm khác ít rõ ràng hơn là **thu gom rác có thể không đoán trước được**. Để minh họa quan điểm này, giả sử chúng ta đang triển khai một kiểu `Document` đại diện cho một danh sách các từ có thể thay đổi được (mutable). Chúng ta có thể triển khai `Document` trong một ngôn ngữ có thu gom rác như Python theo cách này:

```python
class Document:
    def __init__(self, words: List[str]):
        """Create a new document"""
        self.words = words

    def add_word(self, word: str):
        """Add a word to the document"""
        self.words.append(word)

    def get_words(self) -> List[str]:
        """Get a list of all the words in the document"""
        return self.words
```

Đây là một cách chúng ta có thể sử dụng lớp `Document` này để tạo một tài liệu `d`, sao chép nó vào một tài liệu mới `d2`, và sau đó thay đổi `d2`.

```python
words = ["Hello"]
d = Document(words)

d2 = Document(d.get_words())
d2.add_word("world")
```

Hãy xem xét hai câu hỏi chính về ví dụ này:

1. **Khi nào mảng words bị thu hồi?**
   Chương trình này đã tạo ra ba con trỏ tới cùng một mảng. Các biến `words`, `d`, và `d2` đều chứa một con trỏ tới mảng words được cấp phát trên heap. Do đó Python sẽ chỉ thu hồi mảng words khi tất cả ba biến đều ra khỏi phạm vi. Nói chung, thường rất khó để đoán trước nơi dữ liệu sẽ được thu gom rác chỉ bằng cách đọc mã nguồn.

2. **Nội dung của tài liệu `d` là gì?**
   Bởi vì `d2` chứa một con trỏ tới cùng mảng words như `d`, nên `d2.add_word("world")` cũng thay đổi tài liệu `d`. Do đó trong ví dụ này, các từ trong `d` là `["Hello", "world"]`. Điều này xảy ra vì `d.get_words()` trả về một tham chiếu khả biến tới mảng words trong `d`. Các tham chiếu khả biến ngầm định, lan tràn có thể dễ dàng dẫn đến các lỗi không đoán trước được khi các cấu trúc dữ liệu để rò rỉ nội bộ của chúng[^ownership-originally]. Ở đây, hành vi thay đổi `d2` làm thay đổi `d` có lẽ không phải là hành vi được mong đợi.

Vấn đề này không phải là duy nhất đối với Python &mdash; bạn có thể gặp hành vi tương tự trong C#, Java, Javascript, v.v. Trên thực tế, hầu hết các ngôn ngữ lập trình thực sự đều có khái niệm con trỏ. Vấn đề chỉ là cách ngôn ngữ để lộ con trỏ cho lập trình viên như thế nào. Thu gom rác làm cho việc nhìn thấy biến nào trỏ tới dữ liệu nào trở nên khó khăn. Ví dụ, không rõ ràng rằng `d.get_words()` đã tạo ra một con trỏ tới dữ liệu bên trong `d`.

Ngược lại, mô hình quyền sở hữu của Rust đặt các con trỏ lên vị trí trung tâm. Chúng ta có thể thấy điều đó bằng cách dịch kiểu `Document` thành một cấu trúc dữ liệu Rust. Thông thường chúng ta sẽ sử dụng một `struct`, nhưng chúng ta chưa đề cập đến chúng, vì vậy chúng ta sẽ chỉ sử dụng một bí danh kiểu (type alias):

```rust
type Document = Vec<String>;

fn new_document(words: Vec<String>) -> Document {
    words
}

fn add_word(this: &mut Document, word: String) {
    this.push(word);
}

fn get_words(this: &Document) -> &[String] {
    this.as_slice()
}
```

API Rust này khác với API Python ở một số điểm chính:

-   Hàm `new_document` tiêu thụ quyền sở hữu của vector đầu vào `words`. Điều đó có nghĩa là `Document` _sở hữu_ vector từ ngữ đó. Vector từ ngữ sẽ được thu hồi một cách có thể đoán trước khi `Document` sở hữu nó ra khỏi phạm vi.

-   Hàm `add_word` yêu cầu một tham chiếu khả biến `&mut Document` để có thể thay đổi một tài liệu. Nó cũng tiêu thụ quyền sở hữu của `word` đầu vào, nghĩa là không ai khác có thể thay đổi các từ riêng lẻ của tài liệu.

-   Hàm `get_words` trả về một tham chiếu bất biến tường minh tới các chuỗi bên trong tài liệu. Cách duy nhất để tạo một tài liệu mới từ vector từ ngữ này là sao chép sâu (deep-copy) nội dung của nó, như thế này:

```rust,ignore
fn main() {
    let words = vec!["hello".to_string()];
    let d = new_document(words);

    // .to_vec() converts &[String] to Vec<String> by cloning each string
    let words_copy = get_words(&d).to_vec();
    let mut d2 = new_document(words_copy);
    add_word(&mut d2, "world".to_string());

    // The modification to `d2` does not affect `d`
    assert!(!get_words(&d).contains(&"world".into()));
}
```

Điểm mấu chốt của ví dụ này là để nói rằng: nếu Rust không phải là ngôn ngữ đầu tiên của bạn, thì bạn đã có kinh nghiệm làm việc với bộ nhớ và con trỏ rồi! Rust chỉ làm cho những khái niệm đó trở nên tường minh. Điều này có lợi ích kép là (1) cải thiện hiệu suất thời gian chạy bằng cách tránh thu gom rác, và (2) cải thiện khả năng dự đoán bằng cách ngăn chặn việc vô tình "rò rỉ" dữ liệu.

### Các Khái Niệm Về Quyền Sở Hữu

Tiếp theo, hãy ôn tập các khái niệm về quyền sở hữu. Phần ôn tập này sẽ nhanh thôi &mdash; mục tiêu là nhắc nhở bạn về các khái niệm liên quan. Nếu bạn nhận ra mình đã quên hoặc không hiểu một khái niệm nào đó, chúng tôi sẽ liên kết bạn đến các chương liên quan mà bạn có thể xem lại.

#### Quyền Sở Hữu tại Thời Gian Chạy

Chúng ta sẽ bắt đầu bằng cách xem lại cách Rust sử dụng bộ nhớ tại thời gian chạy:

-   Rust cấp phát các biến cục bộ trong các khung stack (stack frames), được cấp phát khi một hàm được gọi và được thu hồi khi lời gọi kết thúc.
-   Các biến cục bộ có thể chứa dữ liệu (như số, boolean, tuple, v.v.) hoặc các con trỏ.
-   Các con trỏ có thể được tạo thông qua các box (con trỏ sở hữu dữ liệu trên heap) hoặc các tham chiếu (con trỏ không sở hữu).

Sơ đồ này minh họa mỗi khái niệm trông như thế nào tại thời gian chạy:

```aquascope,interpreter,horizontal
fn main() {
  let mut a_num = 0;
  inner(&mut a_num);`[]`
}

fn inner(x: &mut i32) {
  let another_num = 1;
  let a_stack_ref = &another_num;

  let a_box = Box::new(2);
  let a_box_stack_ref = &a_box;
  let a_box_heap_ref = &*a_box;`[]`

  *x += 5;
}
```

Hãy xem lại sơ đồ này và đảm bảo bạn hiểu từng phần. Ví dụ, bạn nên có thể trả lời:

-   Tại sao `a_box_stack_ref` trỏ tới stack, trong khi `a_box_heap_ref` trỏ tới heap?
-   Tại sao giá trị `2` không còn ở trên heap tại L2?
-   Tại sao `a_num` có giá trị `5` tại L2?

Nếu bạn muốn ôn lại về box, hãy đọc lại [Chương 4.1][ch04-01]. Nếu bạn muốn ôn lại về tham chiếu, hãy đọc lại [Chương 4.2][ch04-02]. Nếu bạn muốn xem các nghiên cứu điển hình liên quan đến box và tham chiếu, hãy đọc lại [Chương 4.3][ch04-03].

Slice là một loại tham chiếu đặc biệt tham chiếu đến một chuỗi dữ liệu liên tiếp trong bộ nhớ. Sơ đồ này minh họa cách một slice tham chiếu đến một chuỗi con các ký tự trong một chuỗi (string):

```aquascope,interpreter
fn main() {
  let s = String::from("abcdefg");
  let s_slice = &s[2..5];`[]`
}
```

Nếu bạn muốn ôn lại về slice, hãy đọc lại [Chương 4.4][ch04-04].

#### Quyền Sở Hữu tại Thời Gian Biên Dịch

Rust theo dõi các quyền hạn @Perm{read} (đọc), @Perm{write} (ghi), và @Perm{own} (sở hữu) trên mỗi biến. Rust yêu cầu một biến phải có các quyền hạn thích hợp để thực hiện một thao tác nhất định. Là một ví dụ cơ bản, nếu một biến không được khai báo là `let mut`, thì nó thiếu quyền @Perm{write} và không thể bị thay đổi:

```aquascope,permissions,stepper,boundaries,shouldFail
fn main() {
  let n = 0;
  n += 1;
}
```

Quyền hạn của một biến có thể bị thay đổi nếu nó bị **di chuyển** (moved) hoặc **vay mượn** (borrowed). Một sự di chuyển của một biến với kiểu không sao chép được (như `Box<T>` hoặc `String`) yêu cầu các quyền @Perm{read}@Perm{own}, và sự di chuyển sẽ loại bỏ tất cả các quyền trên biến đó. Quy tắc đó ngăn chặn việc sử dụng các biến đã bị di chuyển:

```aquascope,permissions,stepper,boundaries,shouldFail
fn main() {
  let s = String::from("Hello world");
  consume_a_string(s);
  println!("{s}"); // can't read `s` after moving it
}

fn consume_a_string(_s: String) {
  // om nom nom
}
```

Nếu bạn muốn ôn lại cách di chuyển hoạt động, hãy đọc lại [Chương 4.1][ch04-01].

Việc vay mượn một biến (tạo một tham chiếu tới nó) tạm thời loại bỏ một số quyền của biến đó. Một lần mượn bất biến tạo ra một tham chiếu bất biến, và cũng vô hiệu hóa việc dữ liệu được mượn bị thay đổi hoặc di chuyển. Ví dụ, in một tham chiếu bất biến là được phép:

```aquascope,permissions,stepper,boundaries
#fn main() {
let mut s = String::from("Hello");
let s_ref = &s;
println!("{s_ref}");
println!("{s}");
#}
```

Nhưng thay đổi một tham chiếu bất biến là không được phép:

```aquascope,permissions,stepper,boundaries,shouldFail
#fn main() {
let mut s = String::from("Hello");
let s_ref = &s;`(focus,paths:*s_ref)`
s_ref.push_str(" world");
println!("{s}");
#}
```

Và thay đổi dữ liệu đang được mượn bất biến là không được phép:

```aquascope,permissions,stepper,boundaries,shouldFail
#fn main() {
let mut s = String::from("Hello");`(focus)`
let s_ref = &s;`(focus,rxpaths:s$)`
s.push_str(" world");
println!("{s_ref}");
#}
```

Và di chuyển dữ liệu ra khỏi tham chiếu là không được phép:

```aquascope,permissions,stepper,boundaries,shouldFail
#fn main() {
let mut s = String::from("Hello");
let s_ref = &s;`(focus,paths:*s_ref)`
let s2 = *s_ref;
println!("{s}");
#}
```

Một lần mượn khả biến tạo ra một tham chiếu khả biến, điều này vô hiệu hóa việc dữ liệu được mượn bị đọc, ghi, hoặc di chuyển. Ví dụ, thay đổi một tham chiếu khả biến là được phép:

```aquascope,permissions,stepper,boundaries
#fn main() {
let mut s = String::from("Hello");
let s_ref = &mut s;
s_ref.push_str(" world");
println!("{s}");
#}
```

Nhưng truy cập dữ liệu đang được mượn khả biến là không được phép:

```aquascope,permissions,stepper,boundaries,shouldFail
#fn main() {
let mut s = String::from("Hello");
let s_ref = &mut s;`(focus,rxpaths:s$)`
println!("{s}");
s_ref.push_str(" world");
#}
```

Nếu bạn muốn ôn lại về quyền hạn và tham chiếu, hãy đọc lại [Chương 4.2][ch04-02].

#### Kết Nối Quyền Sở Hữu giữa Thời Gian Biên Dịch và Thời Gian Chạy

Các quyền hạn của Rust được thiết kế để ngăn chặn hành vi chưa xác định. Ví dụ, một loại hành vi chưa xác định là **sử dụng sau khi giải phóng** (use-after-free) nơi bộ nhớ đã giải phóng được đọc hoặc ghi. Các lần mượn bất biến loại bỏ quyền @Perm{write} để tránh sử dụng sau khi giải phóng, như trong trường hợp này:

```aquascope,interpreter,shouldFail,horizontal
#fn main() {
let mut v = vec![1, 2, 3];
let n = &v[0];`[]`
v.push(4);`[]`
println!("{n}");`[]`
#}
```

Một loại hành vi chưa xác định khác là **giải phóng kép** (double-free) nơi bộ nhớ bị giải phóng hai lần. Việc giải tham chiếu các tham chiếu tới dữ liệu không sao chép được không có quyền @Perm{own} để tránh giải phóng kép, như trong trường hợp này:

```aquascope,interpreter,shouldFail,horizontal
#fn main() {
let v = vec![1, 2, 3];
let v_ref: &Vec<i32> = &v;
let v2 = *v_ref;`[]`
drop(v2);`[]`
drop(v);`[]`
#}
```

Nếu bạn muốn ôn lại về hành vi chưa xác định, hãy đọc lại [Chương 4.1][ch04-01] và [Chương 4.3][ch04-03].

### Phần Còn Lại Của Quyền Sở Hữu

Khi chúng tôi giới thiệu các tính năng bổ sung như struct, enum, và trait, những tính năng đó sẽ có các tương tác cụ thể với quyền sở hữu. Chương này cung cấp nền tảng thiết yếu để hiểu những tương tác đó &mdash; các khái niệm về bộ nhớ, con trỏ, hành vi chưa xác định, và quyền hạn sẽ giúp chúng ta nói về các phần nâng cao hơn của Rust trong các chương tiếp theo.

Và đừng quên làm các bài kiểm tra nếu bạn muốn kiểm tra sự hiểu biết của mình!

{{#quiz ../quizzes/ch04-05-ownership-recap.toml}}

[^ownership-originally]: Trên thực tế, phát minh ban đầu về các kiểu quyền sở hữu hoàn toàn không phải là về an toàn bộ nhớ. Nó là về việc ngăn chặn rò rỉ các tham chiếu khả biến tới nội bộ cấu trúc dữ liệu trong các ngôn ngữ giống Java. Nếu bạn tò mò muốn tìm hiểu thêm về lịch sử của các kiểu quyền sở hữu, hãy xem bài báo ["Ownership Types for Flexible Alias Protection"](https://dl.acm.org/doi/abs/10.1145/286936.286947) (Clarke và cộng sự, 1998).

[ch04-01]: ch04-01-what-is-ownership.html
[ch04-02]: ch04-02-references-and-borrowing.html
[ch04-03]: ch04-03-fixing-ownership-errors.html
[ch04-04]: ch04-04-slices.html
