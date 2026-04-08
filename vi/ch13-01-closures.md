<!-- Old heading. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## Closure: Các hàm ẩn danh có thể ghi lại môi trường của chúng

Closure trong Rust là các hàm ẩn danh mà bạn có thể lưu trong một biến hoặc truyền dưới dạng
đối số cho các hàm khác. Bạn có thể tạo closure ở một nơi và sau đó
gọi closure ở nơi khác để thực thi nó trong một ngữ cảnh khác. Không giống như
các hàm, closure có thể ghi lại (capture) các giá trị từ phạm vi mà chúng được định nghĩa.
Chúng tôi sẽ trình bày cách các tính năng này của closure cho phép tái sử dụng mã và
tùy biến hành vi.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### Ghi lại môi trường với Closure

Đầu tiên chúng ta sẽ xem xét cách sử dụng closure để ghi lại các giá trị từ
môi trường mà chúng được định nghĩa để sử dụng sau này. Đây là kịch bản: Thỉnh thoảng,
công ty áo thun của chúng ta tặng một chiếc áo độc quyền, phiên bản giới hạn cho
ai đó trong danh sách gửi thư của chúng ta như một chương trình khuyến mãi. Những người trong danh sách gửi thư có thể
tùy chọn thêm màu sắc yêu thích vào hồ sơ của họ. Nếu người được chọn cho
một chiếc áo miễn phí đã thiết lập màu sắc yêu thích, họ sẽ nhận được chiếc áo màu đó. Nếu
người đó chưa chỉ định màu sắc yêu thích, họ sẽ nhận được bất kỳ màu nào mà
công ty hiện đang có nhiều nhất.

Có nhiều cách để thực hiện điều này. Đối với ví dụ này, chúng ta sẽ sử dụng một
enum có tên là `ShirtColor` có các biến thể `Red` và `Blue` (giới hạn
số lượng màu có sẵn để cho đơn giản). Chúng ta đại diện cho kho hàng của công ty
bằng một struct `Inventory` có một trường tên là `shirts` chứa một
`Vec<ShirtColor>` đại diện cho các màu áo hiện có trong kho.
Phương thức `giveaway` được định nghĩa trên `Inventory` lấy tùy chọn màu áo
tùy chọn của người chiến thắng áo miễn phí và trả về màu áo mà
người đó sẽ nhận được. Thiết lập này được hiển thị trong Danh sách 13-1:

<Listing number="13-1" file-name="src/main.rs" caption="Tình huống tặng quà của công ty áo thun">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`store` được định nghĩa trong `main` có hai áo xanh và một áo đỏ còn lại
để phân phối cho đợt khuyến mãi phiên bản giới hạn này. Chúng ta gọi phương thức `giveaway`
cho một người dùng có sở thích về áo đỏ và một người dùng không có bất kỳ sở thích nào.

Một lần nữa, mã này có thể được thực hiện theo nhiều cách, và ở đây, để tập trung vào
closure, chúng ta đã bám sát các khái niệm bạn đã học, ngoại trừ phần thân của
phương thức `giveaway` có sử dụng một closure. Trong phương thức `giveaway`, chúng ta nhận
tùy chọn của người dùng dưới dạng một tham số kiểu `Option<ShirtColor>` và gọi phương thức
`unwrap_or_else` trên `user_preference`. Phương thức [`unwrap_or_else` trên
`Option<T>`][unwrap-or-else]<!-- ignore --> được định nghĩa bởi thư viện chuẩn.
Nó nhận một đối số: một closure không có bất kỳ đối số nào và trả về một giá trị `T`
(cùng kiểu được lưu trữ trong biến thể `Some` của `Option<T>`, trong trường hợp này là
`ShirtColor`). Nếu `Option<T>` là biến thể `Some`, `unwrap_or_else`
trả về giá trị từ bên trong `Some`. Nếu `Option<T>` là biến thể `None`,
`unwrap_or_else` gọi closure và trả về giá trị được trả về bởi closure.

Chúng ta chỉ định biểu thức closure `|| self.most_stocked()` làm đối số cho
`unwrap_or_else`. Đây là một closure không tự nhận tham số nào (nếu
closure có tham số, chúng sẽ xuất hiện giữa hai thanh dọc). Phần
thân của closure gọi `self.most_stocked()`. Chúng ta đang định nghĩa closure ở đây, và
việc thực thi `unwrap_or_else` sẽ đánh giá closure sau đó nếu kết quả là cần thiết.

Chạy mã này sẽ in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Một khía cạnh thú vị ở đây là chúng ta đã truyền một closure gọi
`self.most_stocked()` trên instance `Inventory` hiện tại. Thư viện chuẩn
không cần biết bất cứ điều gì về các kiểu `Inventory` hoặc `ShirtColor` mà
chúng ta đã định nghĩa, hoặc logic chúng ta muốn sử dụng trong kịch bản này. Closure ghi lại
một tham số chiếu bất biến (immutable reference) đến instance `Inventory` `self` và truyền nó cùng với
mã chúng ta chỉ định cho phương thức `unwrap_or_else`. Mặt khác, các hàm
không thể ghi lại môi trường của chúng theo cách này.

### Suy luận kiểu và Chú thích kiểu của Closure

Có nhiều điểm khác biệt hơn giữa hàm và closure. Closure thường
không yêu cầu bạn chú thích kiểu của các tham số hoặc giá trị trả về
như các hàm `fn`. Chú thích kiểu là bắt buộc trên các hàm vì các kiểu
là một phần của giao diện rõ ràng được hiển thị cho người dùng của bạn. Việc định nghĩa
giao diện này một cách cứng nhắc là quan trọng để đảm bảo rằng mọi người đều đồng ý về
kiểu giá trị nào mà một hàm sử dụng và trả về. Mặt khác, closure
không được sử dụng trong một giao diện công khai như thế này: chúng được lưu trữ trong các biến và
được sử dụng mà không cần đặt tên và hiển thị chúng cho người dùng thư viện của chúng ta.

Closure thường ngắn và chỉ phù hợp trong một ngữ cảnh hẹp
thay vì trong bất kỳ kịch bản tùy ý nào. Trong các ngữ cảnh hạn chế này,
trình biên dịch có thể suy luận kiểu của các tham số và kiểu trả về,
tương tự như cách nó có thể suy luận kiểu của hầu hết các biến (có những
trường hợp hiếm hoi trình biên dịch cũng cần chú thích kiểu cho closure).

Giống như với các biến, chúng ta có thể thêm chú thích kiểu nếu muốn tăng tính
rõ ràng và minh bạch dù phải đánh đổi bằng việc dài dòng hơn mức
cần thiết. Việc chú thích các kiểu cho một closure sẽ trông giống như định nghĩa
được hiển thị trong Danh sách 13-2. Trong ví dụ này, chúng ta đang định nghĩa một closure và lưu trữ nó
trong một biến thay vì định nghĩa closure tại chỗ chúng ta truyền nó như một
đối số như chúng ta đã làm trong Danh sách 13-1.

<Listing number="13-2" file-name="src/main.rs" caption="Thêm các chú thích kiểu tùy chọn cho kiểu tham số và giá trị trả về trong closure">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Với các chú thích kiểu được thêm vào, cú pháp của closure trông giống với
cú pháp của hàm hơn. Ở đây chúng ta định nghĩa một hàm cộng 1 vào tham số của nó và
một closure có cùng hành vi, để so sánh. Chúng ta đã thêm một số khoảng trắng
để căn chỉnh các phần liên quan. Điều này minh họa cách cú pháp closure
tương tự như cú pháp hàm ngoại trừ việc sử dụng các thanh dọc và lượng cú pháp
là tùy chọn:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Dòng đầu tiên hiển thị một định nghĩa hàm, và dòng thứ hai hiển thị một định nghĩa
closure được chú thích đầy đủ. Ở dòng thứ ba, chúng ta loại bỏ các chú thích kiểu
khỏi định nghĩa closure. Ở dòng thứ tư, chúng ta loại bỏ các dấu ngoặc nhọn, vốn là
tùy chọn vì thân closure chỉ có một biểu thức. Đây đều là
các định nghĩa hợp lệ sẽ tạo ra cùng một hành vi khi chúng được gọi. Các dòng
`add_one_v3` và `add_one_v4` yêu cầu các closure phải được thực thi để
có thể biên dịch vì các kiểu sẽ được suy luận từ cách sử dụng chúng. Điều này
tương tự như việc `let v = Vec::new();` cần chú thích kiểu hoặc các giá trị của
một kiểu nào đó được chèn vào `Vec` để Rust có thể suy luận ra kiểu dữ liệu.

Đối với các định nghĩa closure, trình biên dịch sẽ suy luận một kiểu cụ thể cho mỗi
tham số của chúng và cho giá trị trả về của chúng. Ví dụ, Danh sách 13-3 hiển thị
định nghĩa của một closure ngắn chỉ trả về giá trị mà nó nhận được như một
tham số. Closure này không hữu ích lắm ngoại trừ mục đích của
ví dụ này. Lưu ý rằng chúng ta chưa thêm bất kỳ chú thích kiểu nào vào định nghĩa.
Bởi vì không có chú thích kiểu, chúng ta có thể gọi closure với bất kỳ kiểu nào,
điều mà chúng ta đã làm ở đây với `String` lần đầu tiên. Nếu sau đó chúng ta cố gắng gọi
`example_closure` với một số nguyên, chúng ta sẽ nhận được lỗi.

<Listing number="13-3" file-name="src/main.rs" caption="Cố gắng gọi một closure có các kiểu được suy luận với hai kiểu khác nhau">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Trình biên dịch cho chúng ta lỗi này:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Lần đầu tiên chúng ta gọi `example_closure` với giá trị `String`, trình biên dịch
suy luận kiểu của `x` và kiểu trả về của closure là `String`. Những
kiểu đó sau đó được khóa vào closure trong `example_closure`, và chúng ta nhận được một lỗi kiểu
khi chúng ta cố gắng sử dụng một kiểu khác với cùng một closure đó lần tiếp theo.

{{#quiz ../quizzes/ch13-01-closures-sec1.toml}}

### Ghi lại các tham chiếu hoặc Di chuyển quyền sở hữu

Closure có thể ghi lại các giá trị từ môi trường của chúng theo ba cách, vốn
tương ứng trực tiếp với ba cách mà một hàm có thể nhận một tham số: mượn
bất biến (borrowing immutably), mượn khả biến (borrowing mutably), và lấy quyền sở hữu (taking ownership). Closure sẽ quyết định
sử dụng cách nào trong số này dựa trên những gì phần thân của hàm làm với các
giá trị được ghi lại.

Trong Danh sách 13-4, chúng ta định nghĩa một closure ghi lại một tham chiếu bất biến đến
vector có tên là `list` bởi vì nó chỉ cần một tham chiếu bất biến để in giá trị:

<Listing number="13-4" file-name="src/main.rs" caption="Định nghĩa và gọi một closure ghi lại một tham chiếu bất biến">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Ví dụ này cũng minh họa rằng một biến có thể liên kết với một định nghĩa closure,
và sau đó chúng ta có thể gọi closure bằng cách sử dụng tên biến và dấu ngoặc đơn như thể
tên biến đó là một tên hàm.

Bởi vì chúng ta có thể có nhiều tham chiếu bất biến đến `list` cùng một lúc,
`list` vẫn có thể truy cập được từ mã trước định nghĩa closure, sau
định nghĩa closure nhưng trước khi closure được gọi, và sau khi closure
được gọi. Mã này biên dịch, chạy và in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Tiếp theo, trong Danh sách 13-5, chúng ta thay đổi thân closure để nó thêm một phần tử vào
vector `list`. Closure bây giờ ghi lại một tham chiếu khả biến:

<Listing number="13-5" file-name="src/main.rs" caption="Định nghĩa và gọi một closure ghi lại một tham chiếu khả biến">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Mã này biên dịch, chạy và in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Lưu ý rằng không còn `println!` nào giữa định nghĩa và lời gọi của
closure `borrows_mutably` nữa: khi `borrows_mutably` được định nghĩa, nó ghi lại một
tham chiếu khả biến đến `list`. Chúng ta không sử dụng closure một lần nữa sau khi closure
được gọi, vì vậy việc mượn khả biến kết thúc. Giữa định nghĩa closure và
lời gọi closure, một việc mượn bất biến để in là không được phép vì không có
lần mượn nào khác được phép khi đang có một lần mượn khả biến. Hãy thử thêm một `println!`
vào đó để xem bạn nhận được thông báo lỗi gì!

Nếu bạn muốn ép buộc closure lấy quyền sở hữu các giá trị nó sử dụng trong
môi trường mặc dù phần thân của closure không thực sự cần
quyền sở hữu, bạn có thể sử dụng từ khóa `move` trước danh sách tham số.

Kỹ thuật này chủ yếu hữu ích khi truyền một closure sang một luồng (thread) mới để di chuyển
dữ liệu sao cho nó được sở hữu bởi luồng mới. Chúng ta sẽ thảo luận về các luồng và lý do tại sao
bạn muốn sử dụng chúng chi tiết trong Chương 16 khi nói về
đồng thời (concurrency), nhưng hiện tại, hãy khám phá ngắn gọn việc tạo một luồng mới bằng cách sử dụng một
closure cần từ khóa `move`. Danh sách 13-6 cho thấy Danh sách 13-4 được sửa đổi
để in vector trong một luồng mới thay vì trong luồng chính:

<Listing number="13-6" file-name="src/main.rs" caption="Sử dụng `move` để ép buộc closure cho luồng lấy quyền sở hữu của `list`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Chúng ta tạo một luồng mới, đưa cho luồng một closure để chạy như một đối số. Phần
thân closure in ra danh sách. Trong Danh sách 13-4, closure chỉ ghi lại
`list` bằng một tham chiếu bất biến vì đó là mức truy cập tối thiểu
đối với `list` cần thiết để in nó. Trong ví dụ này, mặc dù thân closure
vẫn chỉ cần một tham chiếu bất biến, chúng ta cần chỉ định rằng `list` nên
được di chuyển vào closure bằng cách đặt từ khóa `move` ở đầu
định nghĩa closure. Luồng mới có thể kết thúc trước khi phần còn lại của luồng chính
kết thúc, hoặc luồng chính có thể kết thúc trước. Nếu luồng chính
duy trì quyền sở hữu `list` nhưng kết thúc trước luồng mới và
giải phóng (drop) `list`, tham chiếu bất biến trong luồng sẽ không hợp lệ. Do đó,
trình biên dịch yêu cầu `list` phải được di chuyển vào closure được đưa cho luồng mới
để tham chiếu sẽ hợp lệ. Hãy thử xóa từ khóa `move` hoặc sử dụng `list`
trong luồng chính sau khi closure được định nghĩa để xem bạn nhận được lỗi biên dịch gì!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### Di chuyển các giá trị đã ghi lại ra khỏi Closure và các Trait `Fn`

Khi một closure đã ghi lại một tham chiếu hoặc ghi lại quyền sở hữu một giá trị từ
môi trường nơi closure được định nghĩa (do đó ảnh hưởng đến những gì, nếu có, được di chuyển
_vào_ closure), mã trong phần thân của closure sẽ định nghĩa những gì
xảy ra với các tham chiếu hoặc giá trị khi closure được thực thi sau đó (do đó
ảnh hưởng đến những gì, nếu có, được di chuyển _ra khỏi_ closure). Một thân closure có thể
làm bất kỳ điều nào sau đây: di chuyển một giá trị đã ghi lại ra khỏi closure, thay đổi
giá trị đã ghi lại, không di chuyển cũng không thay đổi giá trị, hoặc không ghi lại gì từ
môi trường ngay từ đầu.

Cách một closure ghi lại và xử lý các giá trị từ môi trường ảnh hưởng đến
việc closure đó triển khai trait nào, và trait là cách các hàm và struct
có thể chỉ định loại closure nào chúng có thể sử dụng. Các closure sẽ tự động
triển khai một, hai hoặc cả ba trait `Fn` này, theo cách cộng dồn,
tùy thuộc vào cách thân của closure xử lý các giá trị:

1. `FnOnce` áp dụng cho các closure có thể được gọi một lần. Tất cả các closure đều triển khai
   ít nhất trait này vì tất cả các closure đều có thể được gọi. Một closure di chuyển
   các giá trị đã ghi lại ra khỏi thân của nó sẽ chỉ triển khai `FnOnce` và không triển khai
   bất kỳ trait `Fn` nào khác, bởi vì nó chỉ có thể được gọi một lần.
2. `FnMut` áp dụng cho các closure không di chuyển các giá trị đã ghi lại ra khỏi
   thân của chúng, nhưng có thể thay đổi các giá trị đã ghi lại. Những closure này có thể
   được gọi nhiều hơn một lần.
3. `Fn` áp dụng cho các closure không di chuyển các giá trị đã ghi lại ra khỏi thân của chúng
   và không thay đổi các giá trị đã ghi lại, cũng như các closure không ghi lại
   gì từ môi trường của chúng. Những closure này có thể được gọi nhiều hơn một lần
   mà không làm thay đổi môi trường của chúng, điều này quan trọng trong các trường hợp như
   gọi một closure nhiều lần đồng thời.

Hãy nhìn vào định nghĩa của phương thức `unwrap_or_else` trên `Option<T>` mà
chúng ta đã sử dụng trong Danh sách 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Nhớ lại rằng `T` là kiểu generic đại diện cho kiểu của giá trị trong
biến thể `Some` của một `Option`. Kiểu `T` đó cũng là kiểu trả về của
hàm `unwrap_or_else`: mã gọi `unwrap_or_else` trên một
`Option<String>`, chẳng hạn, sẽ nhận được một `String`.

Tiếp theo, hãy chú ý rằng hàm `unwrap_or_else` có thêm tham số kiểu generic
`F`. Kiểu `F` là kiểu của tham số có tên `f`, chính là
closure chúng ta cung cấp khi gọi `unwrap_or_else`.

Ràng buộc trait (trait bound) được chỉ định trên kiểu generic `F` là `FnOnce() -> T`, có nghĩa
là `F` phải có thể được gọi một lần, không nhận đối số nào và trả về một `T`.
Việc sử dụng `FnOnce` trong ràng buộc trait thể hiện sự hạn chế rằng
`unwrap_or_else` sẽ chỉ gọi `f` tối đa một lần. Trong phần thân của
`unwrap_or_else`, chúng ta có thể thấy rằng nếu `Option` là `Some`, `f` sẽ không
được gọi. Nếu `Option` là `None`, `f` sẽ được gọi một lần. Bởi vì tất cả
closure đều triển khai `FnOnce`, `unwrap_or_else` chấp nhận cả ba loại
closure và linh hoạt nhất có thể.

> Ghi chú: Nếu những gì chúng ta muốn làm không yêu cầu ghi lại một giá trị từ
> môi trường, chúng ta có thể sử dụng tên của một hàm thay vì một closure. Ví dụ,
> chúng ta có thể gọi `unwrap_or_else(Vec::new)` trên một giá trị `Option<Vec<T>>`
> để nhận một vector mới, trống nếu giá trị là `None`. Trình biên dịch tự động
> triển khai bất kỳ trait `Fn` nào có thể áp dụng cho một định nghĩa hàm.

Bây giờ hãy nhìn vào phương thức thư viện chuẩn `sort_by_key` được định nghĩa trên các slice,
để xem nó khác với `unwrap_or_else` như thế nào và tại sao `sort_by_key` sử dụng
`FnMut` thay vì `FnOnce` cho ràng buộc trait. Closure nhận một đối số
dưới dạng một tham chiếu đến mục hiện tại trong slice đang được xem xét,
và trả về một giá trị kiểu `K` có thể được sắp xếp. Hàm này hữu ích
khi bạn muốn sắp xếp một slice theo một thuộc tính cụ thể của mỗi mục. Trong
Danh sách 13-7, chúng ta có một danh sách các instance `Rectangle` và chúng ta sử dụng `sort_by_key`
để sắp xếp chúng theo thuộc tính `width` của chúng từ thấp đến cao:

<Listing number="13-7" file-name="src/main.rs" caption="Sử dụng `sort_by_key` để sắp xếp các hình chữ nhật theo chiều rộng">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Mã này in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

Lý do `sort_by_key` được định nghĩa để nhận một closure `FnMut` là vì nó gọi
closure nhiều lần: một lần cho mỗi mục trong slice. Closure `|r|
r.width` không ghi lại, thay đổi hoặc di chuyển bất cứ thứ gì ra khỏi môi trường của nó, vì vậy
nó đáp ứng các yêu cầu ràng buộc trait.

Ngược lại, Danh sách 13-8 hiển thị một ví dụ về một closure chỉ triển khai
trait `FnOnce`, bởi vì nó di chuyển một giá trị ra khỏi môi trường.
Trình biên dịch sẽ không cho phép chúng ta sử dụng closure này với `sort_by_key`:

<Listing number="13-8" file-name="src/main.rs" caption="Cố gắng sử dụng một closure `FnOnce` với `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

Đây là một cách gượng ép, rắc rối (và không hoạt động) để cố gắng đếm
số lần `sort_by_key` gọi closure khi sắp xếp `list`. Mã này
cố gắng thực hiện việc đếm này bằng cách đẩy `value`—một `String` từ môi trường
của closure—vào vector `sort_operations`. Closure ghi lại `value` và
sau đó di chuyển `value` ra khỏi closure bằng cách chuyển quyền sở hữu `value` cho
vector `sort_operations`. Closure này có thể được gọi một lần; cố gắng gọi
nó lần thứ hai sẽ không hoạt động vì `value` sẽ không còn trong
môi trường để được đẩy vào `sort_operations` một lần nữa! Do đó, closure này
chỉ triển khai `FnOnce`. Khi chúng ta cố gắng biên dịch mã này, chúng ta nhận được lỗi này
rằng `value` không thể được di chuyển ra khỏi closure vì closure phải
triển khai `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Lỗi chỉ vào dòng trong thân closure di chuyển `value` ra khỏi
môi trường. Để khắc phục điều này, chúng ta cần thay đổi thân closure sao cho nó không
di chuyển các giá trị ra khỏi môi trường. Giữ một bộ đếm trong môi trường và
tăng giá trị của nó trong thân closure là một cách đơn giản hơn để
đếm số lần closure được gọi. Closure trong Danh sách 13-9
hoạt động với `sort_by_key` vì nó chỉ ghi lại một tham chiếu khả biến đến
bộ đếm `num_sort_operations` và do đó có thể được gọi nhiều hơn một lần:

<Listing number="13-9" file-name="src/main.rs" caption="Sử dụng một closure `FnMut` với `sort_by_key` là được phép">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

<!-- TODO: consider adding a section on the use<> operator -->

Tóm lại, các trait `Fn` rất quan trọng khi định nghĩa hoặc sử dụng các hàm hoặc kiểu
có sử dụng closure. Trong phần tiếp theo, chúng ta sẽ thảo luận về iterator. Nhiều
phương thức của iterator nhận các đối số là closure, vì vậy hãy ghi nhớ những chi tiết về closure này
khi chúng ta tiếp tục!

{{#quiz ../quizzes/ch13-01-closures-sec2.toml}}

[unwrap-or-else]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else
[lifetime elision]: ch10-03-lifetime-syntax.html#lifetime-elision
