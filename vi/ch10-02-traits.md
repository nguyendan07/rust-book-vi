## Traits: Định nghĩa Hành vi Chung

Một _trait_ định nghĩa chức năng mà một kiểu cụ thể có và có thể chia sẻ với
các kiểu khác. Chúng ta có thể sử dụng traits để định nghĩa hành vi chung theo một cách trừu tượng. Chúng
ta có thể sử dụng _trait bounds_ (ràng buộc trait) để chỉ định rằng một kiểu generic có thể là bất kỳ kiểu nào có
hành vi nhất định.

> Lưu ý: Traits tương tự như một tính năng thường được gọi là _interfaces_ (giao diện) trong các
> ngôn ngữ khác, mặc dù có một số khác biệt.

### Định nghĩa một Trait

Hành vi của một kiểu bao gồm các phương thức mà chúng ta có thể gọi trên kiểu đó. Các
kiểu khác nhau chia sẻ cùng một hành vi nếu chúng ta có thể gọi cùng một phương thức trên tất cả các
kiểu đó. Định nghĩa trait là một cách để nhóm các chữ ký phương thức lại với nhau để
định nghĩa một tập hợp các hành vi cần thiết nhằm hoàn thành một mục đích nào đó.

Ví dụ, giả sử chúng ta có nhiều struct chứa các loại và lượng văn bản
khác nhau: một struct `NewsArticle` chứa một bản tin được gửi từ một
địa điểm cụ thể và một `SocialPost` có tối đa 280 ký tự
cùng với siêu dữ liệu (metadata) cho biết đó là bài đăng mới, bài đăng lại, hoặc một
câu trả lời cho một bài đăng khác.

Chúng ta muốn tạo một crate thư viện tổng hợp phương tiện truyền thông tên là `aggregator` có thể
hiển thị các bản tóm tắt dữ liệu có thể được lưu trữ trong một instance `NewsArticle` hoặc
`SocialPost`. Để làm điều này, chúng ta cần một bản tóm tắt từ mỗi kiểu, và chúng ta sẽ
yêu cầu bản tóm tắt đó bằng cách gọi phương thức `summarize` trên một instance. Liệt kê
10-12 hiển thị định nghĩa của một trait `Summary` công khai thể hiện
hành vi này.

<Listing number="10-12" file-name="src/lib.rs" caption="Một trait Summary bao gồm hành vi được cung cấp bởi một phương thức summarize">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Ở đây, chúng ta khai báo một trait bằng từ khóa `trait` và sau đó là tên của trait,
trong trường hợp này là `Summary`. Chúng ta cũng khai báo trait là `pub` để các
crate phụ thuộc vào crate này cũng có thể sử dụng trait này, như chúng ta sẽ thấy trong
một vài ví dụ. Bên trong dấu ngoặc nhọn, chúng ta khai báo các chữ ký phương thức
mô tả hành vi của các kiểu triển khai trait này, trong
trường hợp này là `fn summarize(&self) -> String`.

Sau chữ ký phương thức, thay vì cung cấp phần triển khai bên trong dấu ngoặc
nhọn, chúng ta sử dụng dấu chấm phẩy. Mỗi kiểu triển khai trait này phải cung cấp
hành vi tùy chỉnh của riêng nó cho thân của phương thức. Trình biên dịch sẽ bắt buộc
rằng bất kỳ kiểu nào có trait `Summary` sẽ có phương thức `summarize`
được định nghĩa với chính xác chữ ký này.

Một trait có thể có nhiều phương thức trong thân của nó: các chữ ký phương thức được liệt kê
từng dòng một, và mỗi dòng kết thúc bằng một dấu chấm phẩy.

### Triển khai một Trait trên một Kiểu

Bây giờ chúng ta đã định nghĩa các chữ ký mong muốn của các phương thức trong trait `Summary`,
chúng ta có thể triển khai nó trên các kiểu trong bộ tổng hợp phương tiện của mình. Liệt kê 10-13 hiển thị
một bản triển khai của trait `Summary` trên struct `NewsArticle` sử dụng
tiêu đề, tác giả và địa điểm để tạo giá trị trả về của
`summarize`. Đối với struct `SocialPost`, chúng ta định nghĩa `summarize` là tên người dùng
theo sau là toàn bộ nội dung của bài đăng, giả sử nội dung bài đăng
đã được giới hạn ở 280 ký tự.

<Listing number="10-13" file-name="src/lib.rs" caption="Triển khai trait Summary trên các kiểu NewsArticle và SocialPost">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Triển khai một trait trên một kiểu tương tự như triển khai các phương thức thông thường. Sự
khác biệt là sau `impl`, chúng ta đặt tên trait mà chúng ta muốn triển khai,
sau đó sử dụng từ khóa `for`, và sau đó chỉ định tên của kiểu mà chúng ta muốn
triển khai trait cho nó. Bên trong khối `impl`, chúng ta đặt các chữ ký phương thức
mà định nghĩa trait đã quy định. Thay vì thêm dấu chấm phẩy sau mỗi
chữ ký, chúng ta sử dụng dấu ngoặc nhọn và điền vào thân phương thức với hành vi
cụ thể mà chúng ta muốn các phương thức của trait có đối với kiểu cụ thể đó.

Bây giờ thư viện đã triển khai trait `Summary` trên `NewsArticle` và
`SocialPost`, người dùng crate có thể gọi các phương thức của trait trên các instance của
`NewsArticle` và `SocialPost` theo cùng cách chúng ta gọi các phương thức thông thường. Sự
khác biệt duy nhất là người dùng phải đưa cả trait và
các kiểu vào phạm vi. Đây là một ví dụ về cách một crate binary có thể sử dụng crate thư viện
`aggregator` của chúng ta:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Mã nguồn này in ra `1 new post: horse_ebooks: of course, as you probably already
know, people`.

Các crate khác phụ thuộc vào crate `aggregator` cũng có thể đưa trait `Summary`
vào phạm vi để triển khai `Summary` trên các kiểu của riêng họ. Một hạn chế
cần lưu ý là chúng ta chỉ có thể triển khai một trait trên một kiểu nếu trait đó hoặc
kiểu đó, hoặc cả hai, là cục bộ đối với crate của chúng ta. Ví dụ, chúng ta có thể triển khai các
trait của thư viện chuẩn như `Display` trên một kiểu tùy chỉnh như `SocialPost` như một phần của
chức năng crate `aggregator` vì kiểu `SocialPost` là cục bộ đối với crate
`aggregator` của chúng ta. Chúng ta cũng có thể triển khai `Summary` trên `Vec<T>` trong
crate `aggregator` của chúng ta vì trait `Summary` là cục bộ đối với crate `aggregator`
của chúng ta.

Nhưng chúng ta không thể triển khai các trait bên ngoài trên các kiểu bên ngoài. Ví dụ, chúng ta không thể
triển khai trait `Display` trên `Vec<T>` bên trong crate `aggregator` của chúng ta vì
`Display` và `Vec<T>` đều được định nghĩa trong thư viện chuẩn và không
cục bộ đối với crate `aggregator` của chúng ta. Hạn chế này là một phần của đặc tính gọi là
_tính nhất quán_ (coherence), và cụ thể hơn là _quy tắc mồ côi_ (orphan rule), được gọi như vậy vì
kiểu cha không hiện diện. Quy tắc này đảm bảo rằng mã nguồn của người khác không thể
làm hỏng mã nguồn của bạn và ngược lại. Nếu không có quy tắc này, hai crate có thể triển khai
cùng một trait cho cùng một kiểu, và Rust sẽ không biết nên sử dụng bản triển khai
nào.

### Triển khai Mặc định

Đôi khi việc có hành vi mặc định cho một số hoặc tất cả các phương thức
trong một trait là hữu ích thay vì yêu cầu triển khai cho tất cả các phương thức trên mọi kiểu.
Sau đó, khi chúng ta triển khai trait trên một kiểu cụ thể, chúng ta có thể giữ nguyên hoặc ghi đè
hành vi mặc định của mỗi phương thức.

Trong Liệt kê 10-14, chúng ta chỉ định một chuỗi mặc định cho phương thức `summarize` của
trait `Summary` thay vì chỉ định nghĩa chữ ký phương thức, như chúng ta đã làm trong
Liệt kê 10-12.

<Listing number="10-14" file-name="src/lib.rs" caption="Định nghĩa một trait Summary với một bản triển khai mặc định của phương thức summarize">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

Để sử dụng một bản triển khai mặc định nhằm tóm tắt các instance của `NewsArticle`, chúng
ta chỉ định một khối `impl` trống với `impl Summary for NewsArticle {}`.

Mặc dù chúng ta không còn định nghĩa trực tiếp phương thức `summarize` trên `NewsArticle`
nữa, nhưng chúng ta đã cung cấp một bản triển khai mặc định và chỉ định rằng
`NewsArticle` triển khai trait `Summary`. Kết quả là, chúng ta vẫn có thể gọi
phương thức `summarize` trên một instance của `NewsArticle`, như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Mã nguồn này in ra `New article available! (Read more...)`.

Việc tạo một bản triển khai mặc định không yêu cầu chúng ta thay đổi bất cứ điều gì về
bản triển khai của `Summary` trên `SocialPost` trong Liệt kê 10-13. Lý do là
cú pháp để ghi đè một bản triển khai mặc định giống hệt với
cú pháp để triển khai một phương thức trait không có bản triển khai
mặc định.

Các bản triển khai mặc định có thể gọi các phương thức khác trong cùng một trait, ngay cả khi
những phương thức khác đó không có bản triển khai mặc định. Theo cách này, một trait có thể
cung cấp rất nhiều chức năng hữu ích và chỉ yêu cầu người triển khai chỉ định một
phần nhỏ trong số đó. Ví dụ, chúng ta có thể định nghĩa trait `Summary` có một phương thức
`summarize_author` bắt buộc phải triển khai, và sau đó định nghĩa một phương thức
`summarize` có bản triển khai mặc định gọi đến phương thức
`summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Để sử dụng phiên bản `Summary` này, chúng ta chỉ cần định nghĩa `summarize_author`
khi chúng ta triển khai trait trên một kiểu:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

Sau khi chúng ta định nghĩa `summarize_author`, chúng ta có thể gọi `summarize` trên các instance của
struct `SocialPost`, và bản triển khai mặc định của `summarize` sẽ gọi
định nghĩa `summarize_author` mà chúng ta đã cung cấp. Bởi vì chúng ta đã triển khai
`summarize_author`, trait `Summary` đã cung cấp cho chúng ta hành vi của phương thức
`summarize` mà không yêu cầu chúng ta phải viết thêm bất kỳ mã nguồn nào nữa. Đây là
kết quả trông như thế nào:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Mã nguồn này in ra `1 new post: (Read more from @horse_ebooks...)`.

Lưu ý rằng không thể gọi bản triển khai mặc định từ một bản triển khai
ghi đè của chính phương thức đó.

{{#quiz ../quizzes/ch10-02-traits-sec1.toml}}

### Trait làm Tham số

Bây giờ bạn đã biết cách định nghĩa và triển khai traits, chúng ta có thể khám phá cách sử dụng
traits để định nghĩa các hàm chấp nhận nhiều kiểu khác nhau. Chúng ta sẽ sử dụng
trait `Summary` mà chúng ta đã triển khai trên các kiểu `NewsArticle` và `SocialPost` trong
Liệt kê 10-13 để định nghĩa một hàm `notify` gọi phương thức `summarize`
trên tham số `item` của nó, tham số này thuộc một kiểu nào đó có triển khai trait
`Summary`. Để làm điều này, chúng ta sử dụng cú pháp `impl Trait`, như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Thay vì một kiểu cụ thể cho tham số `item`, chúng ta chỉ định từ khóa `impl`
và tên trait. Tham số này chấp nhận bất kỳ kiểu nào có triển khai
trait được chỉ định. Trong thân hàm `notify`, chúng ta có thể gọi bất kỳ phương thức nào trên `item`
mà đến từ trait `Summary`, chẳng hạn như `summarize`. Chúng ta có thể gọi `notify`
và truyền vào bất kỳ instance nào của `NewsArticle` hoặc `SocialPost`. Mã nguồn gọi
hàm với bất kỳ kiểu nào khác, chẳng hạn như `String` hoặc `i32`, sẽ không biên dịch được vì
những kiểu đó không triển khai `Summary`.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Cú pháp Trait Bound

Cú pháp `impl Trait` hoạt động cho các trường hợp đơn giản nhưng thực chất là cú pháp
rút gọn (syntax sugar) cho một dạng dài hơn được gọi là một _trait bound_ (ràng buộc trait); nó trông như thế này:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Dạng dài hơn này tương đương với ví dụ trong phần trước nhưng
dài dòng hơn. Chúng ta đặt các trait bound cùng với khai báo của tham số kiểu generic
sau dấu hai chấm và bên trong dấu ngoặc nhọn.

Cú pháp `impl Trait` thuận tiện và giúp mã nguồn súc tích hơn trong các trường hợp đơn giản,
trong khi cú pháp trait bound đầy đủ hơn có thể diễn đạt sự phức tạp hơn trong các trường hợp khác.
Ví dụ, chúng ta có thể có hai tham số cùng triển khai `Summary`. Thực hiện
điều đó với cú pháp `impl Trait` trông như thế này:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Sử dụng `impl Trait` là phù hợp nếu chúng ta muốn hàm này cho phép `item1` và
`item2` có các kiểu khác nhau (miễn là cả hai kiểu đều triển khai `Summary`). Tuy nhiên,
nếu chúng ta muốn bắt buộc cả hai tham số phải có cùng một kiểu, chúng ta phải sử dụng một
trait bound, như thế này:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

Kiểu generic `T` được chỉ định là kiểu của các tham số `item1` và `item2`
sẽ ràng buộc hàm sao cho kiểu cụ thể của giá trị được truyền làm đối số
cho `item1` và `item2` phải giống nhau.

#### Chỉ định Nhiều Trait Bound với Cú pháp `+`

Chúng ta cũng có thể chỉ định nhiều hơn một trait bound. Giả sử chúng ta muốn `notify` sử dụng
định dạng hiển thị cũng như `summarize` trên `item`: chúng ta chỉ định trong định nghĩa
`notify` rằng `item` phải triển khai cả `Display` và `Summary`. Chúng ta có thể làm
như vậy bằng cách sử dụng cú pháp `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

Cú pháp `+` cũng hợp lệ với các trait bound trên các kiểu generic:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

Với hai trait bound được chỉ định, thân hàm `notify` có thể gọi `summarize`
và sử dụng `{}` để định dạng `item`.

#### Trait Bound Rõ ràng hơn với các Câu lệnh `where`

Sử dụng quá nhiều trait bound có những nhược điểm của nó. Mỗi generic có các trait
bound riêng của nó, vì vậy các hàm có nhiều tham số kiểu generic có thể chứa rất nhiều
thông tin về trait bound giữa tên hàm và danh sách tham số của nó,
làm cho chữ ký hàm khó đọc. Vì lý do này, Rust có cú pháp thay thế
để chỉ định các trait bound bên trong một câu lệnh `where` sau chữ ký hàm.
Vì vậy, thay vì viết như thế này:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

chúng ta có thể sử dụng một câu lệnh `where`, như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

Chữ ký của hàm này bớt lộn xộn hơn: tên hàm, danh sách tham số,
và kiểu trả về nằm gần nhau, tương tự như một hàm không có nhiều trait
bound.

### Trả về các Kiểu có Triển khai Traits

Chúng ta cũng có thể sử dụng cú pháp `impl Trait` ở vị trí trả về để trả về một
giá trị của một kiểu nào đó có triển khai một trait, như được hiển thị ở đây:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Bằng cách sử dụng `impl Summary` cho kiểu trả về, chúng ta chỉ định rằng hàm
`returns_summarizable` trả về một kiểu nào đó có triển khai trait `Summary`
mà không cần đặt tên cho kiểu cụ thể. Trong trường hợp này, `returns_summarizable`
trả về một `SocialPost`, nhưng mã nguồn gọi hàm này không cần biết
điều đó.

Khả năng chỉ định một kiểu trả về thông qua trait mà nó triển khai đặc biệt hữu ích
trong ngữ cảnh của closure và iterator, những nội dung mà chúng ta đề cập ở
Chương 13. Closure và iterator tạo ra các kiểu mà chỉ trình biên dịch mới biết hoặc
các kiểu rất dài để chỉ định. Cú pháp `impl Trait` cho phép bạn chỉ định một cách súc tích
rằng một hàm trả về một kiểu nào đó có triển khai trait `Iterator`
mà không cần phải viết ra một kiểu rất dài.

Tuy nhiên, bạn chỉ có thể sử dụng `impl Trait` nếu bạn đang trả về một kiểu duy nhất. Ví
dụ, mã nguồn này trả về `NewsArticle` hoặc `SocialPost` với
kiểu trả về được chỉ định là `impl Summary` sẽ không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Việc trả về `NewsArticle` hoặc `SocialPost` đều không được phép do
các hạn chế xung quanh cách cú pháp `impl Trait` được triển khai trong trình biên dịch.
Chúng ta sẽ tìm hiểu cách viết một hàm có hành vi này trong phần [“Sử dụng Trait
Object cho phép các giá trị thuộc các kiểu khác
nhau”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore
--> của Chương 18.

### Sử dụng Trait Bound để Triển khai các Phương thức có Điều kiện

Bằng cách sử dụng một trait bound với một khối `impl` sử dụng các tham số kiểu generic,
chúng ta có thể triển khai các phương thức có điều kiện cho các kiểu có triển khai các trait
được chỉ định. Ví dụ, kiểu `Pair<T>` trong Liệt kê 10-15 luôn triển khai
hàm `new` để trả về một instance mới của `Pair<T>` (nhớ lại từ phần
[“Định nghĩa các phương thức”][methods]<!-- ignore --> của Chương 5 rằng `Self`
là một bí danh kiểu cho kiểu của khối `impl`, trong trường hợp này là
`Pair<T>`). Nhưng trong khối `impl` tiếp theo, `Pair<T>` chỉ triển khai phương thức
`cmp_display` nếu kiểu nội bộ `T` của nó triển khai trait `PartialOrd`
cho phép so sánh _và_ trait `Display` cho phép in ấn.

<Listing number="10-15" file-name="src/lib.rs" caption="Triển khai các phương thức có điều kiện trên một kiểu generic tùy thuộc vào các trait bound">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Chúng ta cũng có thể triển khai một trait có điều kiện cho bất kỳ kiểu nào có triển khai
một trait khác. Các bản triển khai của một trait trên bất kỳ kiểu nào thỏa mãn các
trait bound được gọi là _các bản triển khai bao quát_ (blanket implementations) và được sử dụng rộng rãi trong
thư viện chuẩn Rust. Ví dụ, thư viện chuẩn triển khai trait
`ToString` trên bất kỳ kiểu nào có triển khai trait `Display`. Khối `impl`
trong thư viện chuẩn trông tương tự như mã nguồn này:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Bởi vì thư viện chuẩn có bản triển khai bao quát này, chúng ta có thể gọi phương thức
`to_string` được định nghĩa bởi trait `ToString` trên bất kỳ kiểu nào có triển khai
trait `Display`. Ví dụ, chúng ta có thể chuyển các số nguyên thành các giá trị `String` tương ứng của
chúng như thế này vì các số nguyên có triển khai `Display`:

```rust
let s = 3.to_string();
```

Các bản triển khai bao quát xuất hiện trong tài liệu cho trait trong phần
“Implementors” (Người triển khai).

Traits và trait bounds cho phép chúng ta viết mã nguồn sử dụng các tham số kiểu generic để
giảm bớt sự trùng lặp nhưng đồng thời cũng chỉ rõ cho trình biên dịch rằng chúng ta muốn kiểu generic
đó có hành vi cụ thể. Trình biên dịch sau đó có thể sử dụng thông tin trait bound
để kiểm tra xem tất cả các kiểu cụ thể được sử dụng với mã nguồn của chúng ta có cung cấp
đúng hành vi hay không. Trong các ngôn ngữ định kiểu động, chúng ta sẽ nhận được lỗi tại
thời điểm thực thi nếu chúng ta gọi một phương thức trên một kiểu mà không định nghĩa phương thức đó. Nhưng
Rust chuyển các lỗi này sang thời điểm biên dịch để chúng ta buộc phải khắc phục các vấn đề
trước khi mã nguồn của chúng ta có thể chạy. Ngoài ra, chúng ta không phải viết mã nguồn
để kiểm tra hành vi tại thời điểm thực thi vì chúng ta đã kiểm tra tại thời điểm biên dịch.
Làm như vậy sẽ cải thiện hiệu suất mà không phải từ bỏ tính linh hoạt
của generics.

{{#quiz ../quizzes/ch10-02-traits-sec2.toml}}

[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
