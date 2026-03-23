## Lưu trữ Danh sách các Giá trị với Vector (Storing Lists of Values with Vectors)

Kiểu bộ sưu tập đầu tiên chúng ta sẽ xem xét là `Vec<T>`, còn được gọi là một _vector_.
Vector cho phép bạn lưu trữ nhiều hơn một giá trị trong một cấu trúc dữ liệu duy nhất,
đặt tất cả các giá trị cạnh nhau trong bộ nhớ. Vector chỉ có thể lưu trữ các giá trị
cùng một kiểu. Chúng hữu ích khi bạn có một danh sách các mục, chẳng hạn như
các dòng văn bản trong một file hoặc giá của các mục trong giỏ hàng.

### Tạo một Vector Mới

Để tạo một vector trống mới, chúng ta gọi hàm `Vec::new`, như được hiển thị trong
Listing 8-1.

<Listing number="8-1" caption="Tạo một vector trống mới để chứa các giá trị kiểu `i32`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta đã thêm một chú thích kiểu ở đây. Bởi vì chúng ta không chèn bất kỳ
giá trị nào vào vector này, Rust không biết chúng ta định lưu trữ loại phần tử nào.
Đây là một điểm quan trọng. Vector được triển khai bằng cách sử dụng generic;
chúng ta sẽ đề cập đến cách sử dụng generic với các kiểu của riêng bạn trong Chương 10. Hiện tại,
hãy biết rằng kiểu `Vec<T>` do thư viện tiêu chuẩn cung cấp có thể chứa bất kỳ kiểu nào.
Khi chúng ta tạo một vector để chứa một kiểu cụ thể, chúng ta có thể chỉ định kiểu đó trong
ngoặc nhọn. Trong Listing 8-1, chúng ta đã nói với Rust rằng `Vec<T>` trong `v` sẽ
chứa các phần tử kiểu `i32`.

Thường xuyên hơn, bạn sẽ tạo một `Vec<T>` với các giá trị ban đầu và Rust sẽ suy luận
kiểu giá trị bạn muốn lưu trữ, vì vậy bạn hiếm khi cần thực hiện chú thích kiểu này.
Rust cung cấp macro `vec!` một cách tiện lợi, macro này sẽ tạo ra một
vector mới chứa các giá trị bạn đưa cho nó. Listing 8-2 tạo ra một
`Vec<i32>` mới chứa các giá trị `1`, `2`, và `3`. Kiểu số nguyên là `i32`
vì đó là kiểu số nguyên mặc định, như chúng ta đã thảo luận trong phần [“Các Kiểu
Dữ liệu”][data-types]<!-- ignore --> của Chương 3.

<Listing number="8-2" caption="Tạo một vector mới chứa các giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

Bởi vì chúng ta đã đưa ra các giá trị `i32` ban đầu, Rust có thể suy luận rằng kiểu của `v`
là `Vec<i32>`, và chú thích kiểu là không cần thiết. Tiếp theo, chúng ta sẽ xem xét cách
sửa đổi một vector.

### Cập nhật một Vector

Để tạo một vector và sau đó thêm các phần tử vào nó, chúng ta có thể sử dụng phương thức `push`,
như được hiển thị trong Listing 8-3.

<Listing number="8-3" caption="Sử dụng phương thức `push` để thêm giá trị vào một vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

Giống như bất kỳ biến nào, nếu chúng ta muốn có thể thay đổi giá trị của nó, chúng ta cần
làm cho nó có thể thay đổi (mutable) bằng cách sử dụng từ khóa `mut`, như đã thảo luận trong Chương 3. Các con số
chúng ta đặt bên trong đều có kiểu `i32`, và Rust suy luận điều này từ dữ liệu, vì vậy
chúng ta không cần chú thích `Vec<i32>`.

### Đọc các Phần tử của Vector

Có hai cách để tham chiếu đến một giá trị được lưu trữ trong một vector: thông qua chỉ số (indexing) hoặc bằng
cách sử dụng phương thức `get`. Trong các ví dụ sau, chúng ta đã chú thích các kiểu của
giá trị được trả về từ các hàm này để làm rõ hơn.

Listing 8-4 hiển thị cả hai phương pháp truy cập một giá trị trong một vector, với cú pháp
chỉ số và phương thức `get`.

<Listing number="8-4" caption="Sử dụng cú pháp chỉ số và sử dụng phương thức `get` để truy cập một mục trong một vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

Lưu ý một vài chi tiết ở đây. Chúng ta sử dụng giá trị chỉ số `2` để lấy phần tử thứ ba
vì vector được đánh chỉ số theo số, bắt đầu từ số không. Sử dụng `&` và `[]`
cho chúng ta một tham chiếu đến phần tử tại giá trị chỉ số đó. Khi chúng ta sử dụng phương thức `get`
với chỉ số được truyền vào như một đối số, chúng ta nhận được một `Option<&T>` mà chúng ta có thể
sử dụng với `match`.

Rust cung cấp hai cách này để tham chiếu đến một phần tử để bạn có thể chọn cách
chương trình hoạt động khi bạn cố gắng sử dụng một giá trị chỉ số nằm ngoài phạm vi của
các phần tử hiện có. Ví dụ, hãy xem điều gì xảy ra khi chúng ta có một vector
gồm năm phần tử và sau đó chúng ta cố gắng truy cập một phần tử ở chỉ số 100 với mỗi
kỹ thuật, như được hiển thị trong Listing 8-5.

<Listing number="8-5" caption="Cố gắng truy cập phần tử ở chỉ số 100 trong một vector chứa năm phần tử">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

Khi chúng ta chạy mã này, phương thức `[]` đầu tiên sẽ khiến chương trình bị hoảng loạn (panic)
vì nó tham chiếu đến một phần tử không tồn tại. Phương pháp này tốt nhất nên được sử dụng khi bạn
muốn chương trình của mình bị dừng nếu có nỗ lực truy cập vào một phần tử vượt quá
phần cuối của vector.

Khi phương thức `get` được truyền một chỉ số nằm ngoài vector, nó trả về
`None` mà không gây ra hoảng loạn. Bạn sẽ sử dụng phương thức này nếu việc truy cập một phần tử
vượt quá phạm vi của vector thỉnh thoảng có thể xảy ra trong các điều kiện bình thường.
Mã của bạn sau đó sẽ có logic để xử lý việc có `Some(&element)` hoặc `None`, như đã thảo luận trong Chương 6. Ví dụ, chỉ số
có thể đến từ một người nhập một con số. Nếu họ vô tình nhập một
con số quá lớn và chương trình nhận được giá trị `None`, bạn có thể nói với
người dùng có bao nhiêu mục trong vector hiện tại và cho họ một cơ hội khác để
nhập một giá trị hợp lệ. Điều đó sẽ thân thiện với người dùng hơn là làm dừng chương trình
do một lỗi đánh máy!

Khi chương trình có một tham chiếu hợp lệ, bộ kiểm tra mượn (borrow checker) thực thi các
quy tắc sở hữu và mượn (đã đề cập trong Chương 4) để đảm bảo tham chiếu này
và bất kỳ tham chiếu nào khác đến nội dung của vector vẫn hợp lệ. Hãy nhớ lại
quy tắc quy định rằng bạn không thể có cả tham chiếu có thể thay đổi và bất biến trong cùng một
phạm vi. Quy tắc đó áp dụng trong Listing 8-6, nơi chúng ta giữ một tham chiếu bất biến
đến phần tử đầu tiên trong một vector và cố gắng thêm một phần tử vào cuối. Chương trình này
sẽ không hoạt động nếu chúng ta cũng cố gắng tham chiếu đến phần tử đó sau này trong
hàm.

<Listing number="8-6" caption="Cố gắng thêm một phần tử vào một vector trong khi đang giữ một tham chiếu đến một mục">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

Biên dịch mã này sẽ dẫn đến lỗi này:

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Mã nguồn trong Listing 8-6 có vẻ như nó nên hoạt động: tại sao một tham chiếu
đến phần tử đầu tiên lại quan tâm đến những thay đổi ở cuối vector? Lỗi này là
do cách vector hoạt động: vì vector đặt các giá trị cạnh nhau
trong bộ nhớ, việc thêm một phần tử mới vào cuối vector có thể yêu cầu
cấp phát bộ nhớ mới và sao chép các phần tử cũ sang không gian mới, nếu không
có đủ chỗ để đặt tất cả các phần tử cạnh nhau ở nơi vector
hiện đang được lưu trữ. Trong trường hợp đó, tham chiếu đến phần tử đầu tiên sẽ
trỏ đến bộ nhớ đã bị giải phóng. Các quy tắc mượn ngăn chương trình
rơi vào tình huống đó.

> Lưu ý: Để biết thêm về các chi tiết triển khai của kiểu `Vec<T>`, hãy xem [“The
> Rustonomicon”][nomicon].

### Duyệt qua các Giá trị trong một Vector

<!-- BEGIN INTERVENTION: e8da8773-8df2-4279-8c27-b7e9eda1dddd -->

Để truy cập lần lượt từng phần tử trong một vector, chúng ta sẽ duyệt qua tất cả các
phần tử thay vì sử dụng các chỉ số để truy cập từng cái một. Listing 8-7 cho thấy cách
sử dụng vòng lặp `for` để lấy các tham chiếu bất biến đến từng phần tử trong một vector gồm các
giá trị `i32` và in chúng ra.

<Listing number="8-7" caption="In từng phần tử trong một vector bằng cách duyệt qua các phần tử bằng vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

Để đọc con số mà `n_ref` tham chiếu đến, chúng ta phải sử dụng toán tử giải tham chiếu `*` (dereference operator) để lấy giá trị trong `n_ref` trước khi chúng ta có thể cộng 1 vào nó, như đã đề cập trong ["Giải tham chiếu một Con trỏ để Truy cập Dữ liệu của nó"][deref].

Chúng ta cũng có thể duyệt qua các tham chiếu có thể thay đổi đến từng phần tử trong một vector có thể thay đổi
để thực hiện các thay đổi đối với tất cả các phần tử. Vòng lặp `for` trong Listing 8-8
sẽ cộng `50` vào mỗi phần tử.

<Listing number="8-8" caption="Duyệt qua các tham chiếu có thể thay đổi đến các phần tử trong một vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

Để thay đổi giá trị mà tham chiếu có thể thay đổi trỏ tới, chúng ta lại sử dụng toán tử giải tham chiếu `*` để lấy giá trị trong `n_ref` trước khi chúng ta có thể sử dụng toán tử `+=`.

<!-- END INTERVENTION -->

{{#quiz ../quizzes/ch08-01-vec-sec1.toml}}

### Sử dụng Iterator một cách An toàn

Chúng ta sẽ thảo luận nhiều hơn về cách hoạt động của iterator trong Chương 13.2 ["Xử lý một Chuỗi các Mục với Iterator"](ch13-02-iterators.html).
Hiện tại, một chi tiết quan trọng là các iterator chứa một con trỏ đến dữ liệu bên trong vector. Chúng ta có thể thấy cách
iterator hoạt động bằng cách lược bỏ cú pháp rút gọn của một vòng lặp for thành các lời gọi phương thức tương ứng của [`Vec::iter`] và [`Iterator::next`]:

```aquascope,interpreter,horizontal
#fn main() {
#use std::slice::Iter;
let mut v: Vec<i32>         = vec![1, 2];
let mut iter: Iter<'_, i32> = v.iter();`[]`
let n1: &i32                = iter.next().unwrap();`[]`
let n2: &i32                = iter.next().unwrap();`[]`
let end: Option<&i32>       = iter.next();`[]`
#}
```

Quan sát thấy rằng iterator `iter` là một con trỏ di chuyển qua từng phần tử của vector. Phương thức `next` tiến
iterator lên và trả về một tham chiếu tùy chọn đến phần tử trước đó, hoặc là `Some` (mà chúng ta unwrap) hoặc `None` ở cuối vector.

Chi tiết này có liên quan đến việc sử dụng vector một cách an toàn. Ví dụ, giả sử chúng ta muốn sao đôi một vector tại chỗ (in-place), chẳng hạn như `[1, 2]` trở thành `[1, 2, 1, 2]`.
Một cách triển khai ngây thơ có thể trông như thế này, được chú thích bằng các quyền được suy luận bởi trình biên dịch:

```aquascope,permissions,stepper,boundaries,shouldFail
fn dup_in_place(v: &mut Vec<i32>) {
    for n_ref in v.iter() {`(focus,paths:*v)`
        v.push(*n_ref);`{}`
    }
}
```

Lưu ý rằng `v.iter()` loại bỏ quyền @Perm{write} khỏi `*v`. Do đó, thao tác `v.push(..)` bị thiếu quyền @Perm{write} được mong đợi. Trình biên dịch Rust sẽ từ chối chương trình này với thông báo lỗi tương ứng:

```text
error[E0502]: cannot borrow `*v` as mutable because it is also borrowed as immutable
 --> test.rs:3:9
  |
2 |     for n_ref in v.iter() {
  |                  --------
  |                  |
  |                  immutable borrow occurs here
  |                  immutable borrow later used here
3 |         v.push(*n_ref);
  |         ^^^^^^^^^^^^^^ mutable borrow occurs here
```

Như chúng ta đã thảo luận trong Chương 4, vấn đề an toàn đằng sau lỗi này là việc đọc bộ nhớ đã bị giải phóng. Ngay khi `v.push(1)` xảy ra, vector sẽ cấp phát lại nội dung của nó và làm mất hiệu lực con trỏ của iterator. Vì vậy, để sử dụng iterator một cách an toàn, Rust không cho phép bạn thêm hoặc xóa các phần tử khỏi vector trong khi đang duyệt.

<!-- TODO: add loop support and make this diagram look reasonable -->
<!-- ```aquascope,interpreter,shouldFail,horizontal
fn dup_in_place(v: &mut Vec<i32>) {`[]`
    for n_ref in v.iter() {
        v.push(*n_ref);
    }`[]`
}
fn main() {
    let mut v = vec![1, 2, 3];
    dup_in_place(&mut v);
}
``` -->

Một cách để duyệt qua một vector mà không sử dụng con trỏ là sử dụng một phạm vi (range), giống như cách chúng ta đã sử dụng cho các lát cắt chuỗi (string slices) trong [Chương 4.4](ch04-04-slices.html#range-syntax). Ví dụ, phạm vi `0 .. v.len()` là một iterator trên tất cả các chỉ số của một vector `v`, như được thấy ở đây:

```aquascope,interpreter,horizontal
#fn main() {
#use std::ops::Range;
let mut v: Vec<i32>        = vec![1, 2];
let mut iter: Range<usize> = 0 .. v.len();`[]`
let i1: usize              = iter.next().unwrap();
let n1: &i32               = &v[i1];`[]`
#}
```

### Sử dụng một Enum để Lưu trữ Nhiều Kiểu dữ liệu

Vector chỉ có thể lưu trữ các giá trị cùng một kiểu. Điều này có thể
gây bất tiện; chắc chắn có những trường hợp sử dụng cần lưu trữ một danh sách các
mục thuộc các kiểu khác nhau. May mắn thay, các biến thể của một enum được định nghĩa
dưới cùng một kiểu enum, vì vậy khi chúng ta cần một kiểu để đại diện cho các phần tử của
các kiểu khác nhau, chúng ta có thể định nghĩa và sử dụng một enum!

Ví dụ: giả sử chúng ta muốn lấy các giá trị từ một hàng trong bảng tính, trong đó
một số cột trong hàng chứa số nguyên, một số chứa số dấu phẩy động,
và một số chứa chuỗi. Chúng ta có thể định nghĩa một enum mà các biến thể của nó sẽ chứa các kiểu
giá trị khác nhau, và tất cả các biến thể enum sẽ được coi là cùng một kiểu: đó
là kiểu của chính enum đó. Sau đó, chúng ta có thể tạo một vector để chứa enum đó và do đó, cuối cùng,
chứa các kiểu khác nhau. Chúng ta đã chứng minh điều này trong Listing 8-9.

<Listing number="8-9" caption="Định nghĩa một `enum` để lưu trữ các giá trị thuộc các kiểu khác nhau trong một vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust cần biết những kiểu nào sẽ có trong vector tại thời điểm biên dịch để nó biết
chính xác cần bao nhiêu bộ nhớ trên heap để lưu trữ mỗi phần tử. Chúng ta
cũng phải rõ ràng về những kiểu nào được phép trong vector này. Nếu Rust
cho phép một vector chứa bất kỳ kiểu nào, sẽ có khả năng một hoặc nhiều
kiểu gây ra lỗi với các thao tác được thực hiện trên các phần tử của
vector. Sử dụng một enum cộng với một biểu thức `match` có nghĩa là Rust sẽ đảm bảo
tại thời điểm biên dịch rằng mọi trường hợp có thể xảy ra đều được xử lý, như đã thảo luận trong Chương 6.

Nếu bạn không biết tập hợp đầy đủ các kiểu mà một chương trình sẽ nhận được khi chạy để
lưu trữ trong một vector, kỹ thuật enum sẽ không hiệu quả. Thay vào đó, bạn có thể sử dụng một trait
object, chúng ta sẽ đề cập trong Chương 18.

Bây giờ chúng ta đã thảo luận về một số cách phổ biến nhất để sử dụng vector, hãy đảm bảo
xem lại [tài liệu API][vec-api]<!-- ignore --> cho tất cả các phương thức hữu ích
được định nghĩa trên `Vec<T>` bởi thư viện tiêu chuẩn. Ví dụ, ngoài
`push`, phương thức `pop` sẽ xóa và trả về phần tử cuối cùng.

### Giải phóng một Vector sẽ Giải phóng các Phần tử của nó

Giống như bất kỳ `struct` nào khác, một vector sẽ được giải phóng khi nó nằm ngoài phạm vi, như
được chú thích trong Listing 8-10.

<Listing number="8-10" caption="Chỉ ra nơi vector và các phần tử của nó bị giải phóng">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

Khi vector bị giải phóng (dropped), tất cả nội dung của nó cũng bị giải phóng, nghĩa là các
số nguyên mà nó chứa sẽ được dọn dẹp. Bộ kiểm tra mượn đảm bảo rằng bất kỳ
tham chiếu nào đến nội dung của một vector chỉ được sử dụng trong khi bản thân vector đó vẫn
hợp lệ.

Hãy chuyển sang kiểu bộ sưu tập tiếp theo: `String`!

{{#quiz ../quizzes/ch08-01-vec-sec2.toml}}

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: https://doc.rust-lang.org/nomicon/vec/vec.html
[vec-api]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[deref]: ch04-02-references-and-borrowing.html#dereferencing-a-pointer-accesses-its-data
[`Vec::iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter
[`Iterator::next`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#tymethod.next
