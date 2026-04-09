## Triển khai một Mẫu thiết kế Hướng đối tượng

_Mẫu thiết kế trạng thái_ (state pattern) là một mẫu thiết kế hướng đối tượng. Điểm cốt lõi của
mẫu thiết kế này là chúng ta định nghĩa một tập hợp các trạng thái mà một giá trị có thể có ở bên trong. Các
trạng thái được đại diện bởi một tập hợp các _đối tượng trạng thái_ (state objects), và hành vi của giá trị
thay đổi dựa trên trạng thái của nó. Chúng ta sẽ làm việc thông qua một ví dụ về một struct bài viết blog
có một trường để giữ trạng thái của nó, đó sẽ là một đối tượng trạng thái
từ tập hợp “nháp” (draft), “xem xét” (review), hoặc “đã xuất bản” (published).

Các đối tượng trạng thái chia sẻ chức năng: trong Rust, tất nhiên, chúng ta sử dụng các struct và
trait thay vì các đối tượng và tính kế thừa. Mỗi đối tượng trạng thái chịu trách nhiệm
cho hành vi của chính nó và quản lý việc khi nào nó nên chuyển sang một
trạng thái khác. Giá trị nắm giữ một đối tượng trạng thái không biết gì về các hành vi
khác nhau của các trạng thái hoặc khi nào cần chuyển đổi giữa các trạng thái.

Lợi thế của việc sử dụng mẫu thiết kế trạng thái là, khi các yêu cầu nghiệp vụ
của chương trình thay đổi, chúng ta sẽ không cần phải thay đổi mã của
giá trị đang giữ trạng thái hoặc mã sử dụng giá trị đó. Chúng ta sẽ chỉ cần
cập nhật mã bên trong một trong các đối tượng trạng thái để thay đổi các quy tắc của nó hoặc có lẽ
thêm nhiều đối tượng trạng thái hơn.

Đầu tiên, chúng ta sẽ triển khai mẫu thiết kế trạng thái theo cách hướng đối tượng
truyền thống hơn, sau đó chúng ta sẽ sử dụng một cách tiếp cận tự nhiên hơn một chút trong
Rust. Hãy cùng đi sâu vào việc triển khai từng bước một quy trình làm việc của bài viết blog bằng cách sử dụng
mẫu thiết kế trạng thái.

Chức năng cuối cùng sẽ trông như thế này:

1. Một bài viết blog bắt đầu như một bản nháp trống.
2. Khi bản nháp hoàn tất, một yêu cầu xem xét bài viết sẽ được thực hiện.
3. Khi bài viết được phê duyệt, nó sẽ được xuất bản.
4. Chỉ những bài viết blog đã xuất bản mới trả về nội dung để in, vì vậy các bài viết chưa được phê duyệt không thể
   vô tình bị xuất bản.

Bất kỳ thay đổi nào khác được cố gắng thực hiện trên một bài viết đều không có tác dụng. Ví dụ, nếu chúng ta
cố gắng phê duyệt một bài viết blog bản nháp trước khi chúng ta yêu cầu xem xét, bài viết đó
vẫn phải là một bản nháp chưa được xuất bản.

Liệt kê 18-11 hiển thị quy trình làm việc này dưới dạng mã: đây là một ví dụ về cách sử dụng
API mà chúng ta sẽ triển khai trong một thư viện crate có tên là `blog`. Mã này sẽ chưa biên dịch được
vì chúng ta chưa triển khai crate `blog`.

<Listing number="18-11" file-name="src/main.rs" caption="Mã minh họa hành vi mong muốn mà chúng ta muốn crate `blog` của mình có">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Chúng ta muốn cho phép người dùng tạo một bài viết blog nháp mới bằng `Post::new`. Chúng ta
muốn cho phép thêm văn bản vào bài viết blog. Nếu chúng ta cố gắng lấy nội dung của bài viết
ngay lập tức, trước khi được phê duyệt, chúng ta không nên nhận được bất kỳ văn bản nào vì
bài viết vẫn là một bản nháp. Chúng ta đã thêm `assert_eq!` vào mã nhằm mục đích
minh họa. Một bài kiểm tra đơn vị (unit test) tuyệt vời cho việc này là khẳng định rằng một bài viết blog nháp
trả về một chuỗi trống từ phương thức `content`, nhưng chúng ta sẽ không
viết các bài kiểm tra cho ví dụ này.

Tiếp theo, chúng ta muốn cho phép yêu cầu xem xét bài viết, và chúng ta muốn
`content` trả về một chuỗi trống trong khi chờ xem xét. Khi bài viết
nhận được sự phê duyệt, nó nên được xuất bản, nghĩa là văn bản của bài viết sẽ
được trả về khi `content` được gọi.

Lưu ý rằng kiểu duy nhất mà chúng ta đang tương tác từ crate là kiểu `Post`.
Kiểu này sẽ sử dụng mẫu thiết kế trạng thái và sẽ giữ một giá trị sẽ là
một trong ba đối tượng trạng thái đại diện cho các trạng thái khác nhau mà một bài viết có thể
ở trong—nháp, xem xét, hoặc đã xuất bản. Việc thay đổi từ trạng thái này sang trạng thái khác sẽ được
quản lý nội bộ bên trong kiểu `Post`. Các trạng thái thay đổi để phản hồi lại các
phương thức được gọi bởi người dùng thư viện của chúng ta trên thực thể `Post`, nhưng họ không
phải trực tiếp quản lý các thay đổi trạng thái. Ngoài ra, người dùng không thể mắc sai lầm với
các trạng thái, chẳng hạn như xuất bản một bài viết trước khi nó được xem xét.

### Định nghĩa `Post` và tạo một thực thể mới trong trạng thái Nháp

Hãy bắt đầu triển khai thư viện! Chúng ta biết mình cần một
struct `Post` công khai để giữ một số nội dung, vì vậy chúng ta sẽ bắt đầu với
định nghĩa của struct và một hàm `new` công khai liên quan để tạo một
thực thể của `Post`, như được hiển thị trong Liệt kê 18-12. Chúng ta cũng sẽ tạo một
trait `State` riêng tư để định nghĩa hành vi mà tất cả các đối tượng trạng thái cho một `Post`
phải có.

Sau đó, `Post` sẽ giữ một trait object của `Box<dyn State>` bên trong một `Option<T>`
trong một trường riêng tư tên là `state` để giữ đối tượng trạng thái. Bạn sẽ thấy lý do tại sao
`Option<T>` là cần thiết trong giây lát.

<Listing number="18-12" file-name="src/lib.rs" caption="Định nghĩa một struct `Post` và một hàm `new` tạo một thực thể `Post` mới, một trait `State`, và một struct `Draft` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

Trait `State` định nghĩa hành vi được chia sẻ bởi các trạng thái bài viết khác nhau. Các
đối tượng trạng thái là `Draft`, `PendingReview`, và `Published`, và tất cả chúng sẽ
triển khai trait `State`. Hiện tại, trait này không có bất kỳ phương thức nào, và
chúng ta sẽ bắt đầu bằng cách chỉ định nghĩa trạng thái `Draft` vì đó là trạng thái chúng ta
muốn một bài viết bắt đầu.

Khi chúng ta tạo một `Post` mới, chúng ta đặt trường `state` của nó thành một giá trị `Some`
giữ một `Box`. `Box` này trỏ đến một thực thể mới của struct `Draft`. Điều này
đảm bảo rằng bất cứ khi nào chúng ta tạo một thực thể mới của `Post`, nó sẽ bắt đầu như một
bản nháp. Vì trường `state` của `Post` là riêng tư, không có cách nào để tạo
một `Post` ở bất kỳ trạng thái nào khác! Trong hàm `Post::new`, chúng ta đặt trường `content`
thành một `String` mới, trống.

### Lưu trữ văn bản của nội dung bài viết

Chúng ta đã thấy trong Liệt kê 18-11 rằng chúng ta muốn có thể gọi một phương thức tên là
`add_text` và truyền cho nó một `&str` sau đó được thêm vào làm nội dung văn bản của
bài viết blog. Chúng ta triển khai điều này như một phương thức, thay vì để lộ trường `content`
dưới dạng `pub`, để sau này chúng ta có thể triển khai một phương thức sẽ kiểm soát cách
dữ liệu của trường `content` được đọc. Phương thức `add_text` khá
đơn giản, vì vậy hãy thêm phần triển khai trong Liệt kê 18-13 vào khối `impl Post`.

<Listing number="18-13" file-name="src/lib.rs" caption="Triển khai phương thức `add_text` để thêm văn bản vào `content` của bài viết">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

Phương thức `add_text` nhận một tham chiếu có thể thay đổi (mutable reference) đến `self` vì chúng ta đang
thay đổi thực thể `Post` mà chúng ta đang gọi `add_text` trên đó. Sau đó, chúng ta gọi
`push_str` trên `String` trong `content` và truyền đối số `text` để thêm vào
`content` đã lưu. Hành vi này không phụ thuộc vào trạng thái của bài viết,
vì vậy nó không phải là một phần của mẫu thiết kế trạng thái. Phương thức `add_text` không tương tác
với trường `state` chút nào, nhưng nó là một phần của hành vi mà chúng ta muốn
hỗ trợ.

### Đảm bảo nội dung của một bài viết nháp là trống

Ngay cả sau khi chúng ta đã gọi `add_text` và thêm một số nội dung vào bài viết của mình, chúng ta vẫn
muốn phương thức `content` trả về một lát chuỗi (string slice) trống vì bài viết
vẫn đang ở trạng thái nháp, như được hiển thị trên dòng 7 của Liệt kê 18-11. Hiện tại, hãy
triển khai phương thức `content` với thứ đơn giản nhất sẽ đáp ứng
yêu cầu này: luôn trả về một lát chuỗi trống. Chúng ta sẽ thay đổi điều này sau
khi chúng ta triển khai khả năng thay đổi trạng thái của bài viết để nó có thể được xuất bản.
Cho đến nay, các bài viết chỉ có thể ở trạng thái nháp, vì vậy nội dung bài viết phải luôn
trống. Liệt kê 18-14 hiển thị triển khai giữ chỗ này.

<Listing number="18-14" file-name="src/lib.rs" caption="Thêm một triển khai giữ chỗ cho phương thức `content` trên `Post` luôn trả về một lát chuỗi trống">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Với phương thức `content` được thêm vào này, mọi thứ trong Liệt kê 18-11 cho đến dòng 7
hoạt động như mong muốn.

<!-- Old link, do not remove -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>

### Yêu cầu xem xét làm thay đổi trạng thái của bài viết

Tiếp theo, chúng ta cần thêm chức năng để yêu cầu xem xét một bài viết, điều này sẽ
thay đổi trạng thái của nó từ `Draft` sang `PendingReview`. Liệt kê 18-15 hiển thị mã này.

<Listing number="18-15" file-name="src/lib.rs" caption="Triển khai các phương thức `request_review` trên `Post` và trait `State` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Chúng ta cung cấp cho `Post` một phương thức công khai tên là `request_review` sẽ nhận một tham chiếu
có thể thay đổi đến `self`. Sau đó, chúng ta gọi một phương thức `request_review` nội bộ trên
trạng thái hiện tại của `Post`, và phương thức `request_review` thứ hai này tiêu thụ
trạng thái hiện tại và trả về một trạng thái mới.

Chúng ta thêm phương thức `request_review` vào trait `State`; tất cả các kiểu
triển khai trait giờ đây sẽ cần triển khai phương thức `request_review`.
Lưu ý rằng thay vì có `self`, `&self`, hoặc `&mut self` làm tham số đầu tiên
của phương thức, chúng ta có `self: Box<Self>`. Cú pháp này có nghĩa là phương thức
chỉ hợp lệ khi được gọi trên một `Box` giữ kiểu đó. Cú pháp này chiếm quyền
sở hữu `Box<Self>`, làm mất hiệu lực trạng thái cũ để giá trị trạng thái của
`Post` có thể chuyển đổi thành một trạng thái mới.

Để tiêu thụ trạng thái cũ, phương thức `request_review` cần chiếm quyền sở hữu
giá trị trạng thái. Đây là lúc `Option` trong trường `state` của `Post`
phát huy tác dụng: chúng ta gọi phương thức `take` để lấy giá trị `Some` ra khỏi trường
`state` và để lại một `None` tại vị trí đó vì Rust không cho phép chúng ta có
các trường không có giá trị trong struct. Điều này cho phép chúng ta di chuyển giá trị `state` ra khỏi
`Post` thay vì mượn nó. Sau đó, chúng ta sẽ đặt giá trị `state` của bài viết thành
kết quả của thao tác này.

Chúng ta cần đặt `state` thành `None` tạm thời thay vì đặt nó trực tiếp
bằng mã như `self.state = self.state.request_review();` để có quyền sở hữu
giá trị `state`. Điều này đảm bảo `Post` không thể sử dụng giá trị `state` cũ sau khi
chúng ta đã chuyển đổi nó thành một trạng thái mới.

Phương thức `request_review` trên `Draft` trả về một thực thể mới, được đóng trong box của một struct
`PendingReview` mới, đại diện cho trạng thái khi một bài viết đang chờ
xem xét. Struct `PendingReview` cũng triển khai phương thức `request_review`
nhưng không thực hiện bất kỳ chuyển đổi nào. Thay vào đó, nó trả về chính nó vì khi chúng ta
yêu cầu xem xét một bài viết đã ở trạng thái `PendingReview`, nó nên giữ nguyên
ở trạng thái `PendingReview`.

Bây giờ chúng ta có thể bắt đầu thấy những lợi thế của mẫu thiết kế trạng thái: phương thức
`request_review` trên `Post` là giống nhau bất kể giá trị `state` của nó là gì. Mỗi
trạng thái chịu trách nhiệm cho các quy tắc riêng của nó.

Chúng ta sẽ để phương thức `content` trên `Post` như cũ, trả về một lát chuỗi
trống. Bây giờ chúng ta có thể có một `Post` ở trạng thái `PendingReview` cũng như ở
trạng thái `Draft`, nhưng chúng ta muốn hành vi tương tự ở trạng thái `PendingReview`.
Liệt kê 18-11 giờ đây hoạt động đến dòng 10!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

### Thêm `approve` để thay đổi hành vi của `content`

Phương thức `approve` sẽ tương tự như phương thức `request_review`: nó sẽ
đặt `state` thành giá trị mà trạng thái hiện tại nói rằng nó nên có khi
trạng thái đó được phê duyệt, như được hiển thị trong Liệt kê 18-16:

<Listing number="18-16" file-name="src/lib.rs" caption="Triển khai phương thức `approve` trên `Post` và trait `State` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Chúng ta thêm phương thức `approve` vào trait `State` và thêm một struct mới
triển khai `State`, đó là trạng thái `Published`.

Tương tự như cách `request_review` trên `PendingReview` hoạt động, nếu chúng ta gọi
phương thức `approve` trên một `Draft`, nó sẽ không có tác dụng vì `approve` sẽ
trả về `self`. Khi chúng ta gọi `approve` trên `PendingReview`, nó trả về một thực thể
mới, được đóng trong box của struct `Published`. Struct `Published` triển khai
trait `State`, và đối với cả phương thức `request_review` và phương thức
`approve`, nó trả về chính nó, vì bài viết nên giữ nguyên ở trạng thái `Published`
trong những trường hợp đó.

Bây giờ chúng ta cần cập nhật phương thức `content` trên `Post`. Chúng ta muốn giá trị
trả về từ `content` phụ thuộc vào trạng thái hiện tại của `Post`, vì vậy chúng ta sẽ
để `Post` ủy quyền cho một phương thức `content` được định nghĩa trên `state` của nó,
như được hiển thị trong Liệt kê 18-17:

<Listing number="18-17" file-name="src/lib.rs" caption="Cập nhật phương thức `content` trên `Post` để ủy quyền cho một phương thức `content` trên `State` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Vì mục tiêu là giữ tất cả các quy tắc này bên trong các struct triển khai
`State`, chúng ta gọi một phương thức `content` trên giá trị trong `state` và truyền thực thể bài viết
(nghĩa là `self`) làm đối số. Sau đó, chúng ta trả về giá trị được
trả về từ việc sử dụng phương thức `content` trên giá trị `state`.

Chúng ta gọi phương thức `as_ref` trên `Option` vì chúng ta muốn một tham chiếu đến
giá trị bên trong `Option` thay vì quyền sở hữu giá trị đó. Vì `state`
là một `Option<Box<dyn State>>`, khi chúng ta gọi `as_ref`, một `Option<&Box<dyn State>>`
được trả về. Nếu chúng ta không gọi `as_ref`, chúng ta sẽ gặp lỗi vì
chúng ta không thể di chuyển `state` ra khỏi `&self` đang được mượn của tham số hàm.

Sau đó, chúng ta gọi phương thức `unwrap`, mà chúng ta biết sẽ không bao giờ gây hoảng loạn (panic), vì chúng ta
biết các phương thức trên `Post` đảm bảo rằng `state` sẽ luôn chứa một giá trị `Some`
khi các phương thức đó hoàn tất. Đây là một trong những trường hợp chúng ta đã nói đến trong
[“Các trường hợp mà bạn có nhiều thông tin hơn trình biên dịch”][more-info-than-rustc]<!-- ignore --> trong Chương 9 khi chúng ta biết rằng một
giá trị `None` là không bao giờ khả thi, mặc dù trình biên dịch không thể
hiểu được điều đó.

Tại thời điểm này, khi chúng ta gọi `content` trên `&Box<dyn State>`, ép kiểu giải tham chiếu (deref coercion)
sẽ có hiệu lực trên `&` và `Box` để phương thức `content` cuối cùng
sẽ được gọi trên kiểu triển khai trait `State`. Điều đó có nghĩa là
chúng ta cần thêm `content` vào định nghĩa trait `State`, và đó là nơi
chúng ta sẽ đặt logic cho nội dung nào cần trả về tùy thuộc vào trạng thái nào chúng ta
đang có, như được hiển thị trong Liệt kê 18-18:

<Listing number="18-18" file-name="src/lib.rs" caption="Thêm phương thức `content` vào trait `State` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Chúng ta thêm một triển khai mặc định cho phương thức `content` trả về một lát chuỗi
trống. Điều đó có nghĩa là chúng ta không cần triển khai `content` trên các struct `Draft`
và `PendingReview`. Struct `Published` sẽ ghi đè phương thức `content`
và trả về giá trị trong `post.content`.

Lưu ý rằng chúng ta cần các chú thích thời gian sống (lifetime annotations) trên phương thức này, như chúng ta đã thảo luận trong
Chương 10. Chúng ta đang nhận một tham chiếu đến một `post` làm đối số và trả về một
tham chiếu đến một phần của `post` đó, vì vậy thời gian sống của tham chiếu được trả về
có liên quan đến thời gian sống của đối số `post`.

Và chúng ta đã hoàn tất—tất cả Liệt kê 18-11 giờ đây đã hoạt động! Chúng ta đã triển khai mẫu thiết kế
trạng thái với các quy tắc của quy trình làm việc bài viết blog. Logic liên quan đến các
quy tắc nằm trong các đối tượng trạng thái thay vì nằm rải rác khắp `Post`.

> ### Tại sao không phải là một Enum?
>
> Bạn có thể đã tự hỏi tại sao chúng ta không sử dụng một `enum` với các trạng thái
> bài viết khả thi khác nhau dưới dạng các biến thể (variants). Đó chắc chắn là một giải pháp khả thi; hãy
> thử nó và so sánh kết quả cuối cùng để xem bạn thích cái nào hơn! Một nhược điểm của
> việc sử dụng một enum là mọi nơi kiểm tra giá trị của enum sẽ cần
> một biểu thức `match` hoặc tương tự để xử lý mọi biến thể có thể. Điều này có thể
> trở nên lặp đi lặp lại hơn so với giải pháp trait object này.

### Đánh đổi của Mẫu thiết kế Trạng thái

Chúng ta đã chỉ ra rằng Rust có khả năng triển khai mẫu thiết kế trạng thái hướng đối tượng
để đóng gói các loại hành vi khác nhau mà một bài viết nên có trong
mỗi trạng thái. Các phương thức trên `Post` không biết gì về các hành vi khác nhau.
Cách chúng ta tổ chức mã, chúng ta chỉ phải nhìn vào một nơi để biết các
cách khác nhau mà một bài viết đã xuất bản có thể hành xử: việc triển khai trait
`State` trên struct `Published`.

Nếu chúng ta tạo ra một triển khai thay thế không sử dụng mẫu thiết kế trạng thái,
thay vào đó chúng ta có thể sử dụng các biểu thức `match` trong các phương thức trên `Post` hoặc
thậm chí trong mã `main` để kiểm tra trạng thái của bài viết và thay đổi hành vi
ở những nơi đó. Điều đó có nghĩa là chúng ta sẽ phải nhìn vào nhiều nơi để
hiểu tất cả các ý nghĩa của một bài viết đang ở trạng thái đã xuất bản! Điều này
sẽ chỉ tăng lên khi chúng ta thêm nhiều trạng thái hơn: mỗi biểu thức `match` đó
sẽ cần thêm một nhánh (arm) nữa.

Với mẫu thiết kế trạng thái, các phương thức `Post` và những nơi chúng ta sử dụng `Post` không
cần các biểu thức `match`, và để thêm một trạng thái mới, chúng ta chỉ cần thêm một
struct mới và triển khai các phương thức trait trên struct duy nhất đó.

Triển khai sử dụng mẫu thiết kế trạng thái dễ dàng mở rộng để thêm nhiều
chức năng hơn. Để thấy sự đơn giản của việc duy trì mã sử dụng mẫu thiết kế
trạng thái, hãy thử một vài gợi ý sau:

- Thêm một phương thức `reject` để thay đổi trạng thái của bài viết từ `PendingReview` quay lại
  `Draft`.
- Yêu cầu hai lần gọi `approve` trước khi trạng thái có thể được chuyển sang `Published`.
- Chỉ cho phép người dùng thêm nội dung văn bản khi bài viết đang ở trạng thái `Draft`.
  Gợi ý: để đối tượng trạng thái chịu trách nhiệm về những gì có thể thay đổi về
  nội dung nhưng không chịu trách nhiệm sửa đổi `Post`.

Một nhược điểm của mẫu thiết kế trạng thái là, vì các trạng thái triển khai
các chuyển đổi giữa các trạng thái, một số trạng thái bị liên kết chặt chẽ (coupled) với nhau. Nếu chúng ta
thêm một trạng thái khác giữa `PendingReview` và `Published`, chẳng hạn như `Scheduled`,
chúng ta sẽ phải thay đổi mã trong `PendingReview` để chuyển sang
`Scheduled` thay thế. Sẽ ít việc hơn nếu `PendingReview` không cần
thay đổi khi thêm một trạng thái mới, nhưng điều đó có nghĩa là phải chuyển sang
một mẫu thiết kế khác.

Một nhược điểm khác là chúng ta đã lặp lại một số logic. Để loại bỏ một số
sự lặp lại, chúng ta có thể thử tạo các triển khai mặc định cho các phương thức
`request_review` và `approve` trên trait `State` để trả về `self`;
tuy nhiên, điều này sẽ không hoạt động: khi sử dụng `State` như một trait object, trait
không biết chính xác `self` cụ thể sẽ là gì, vì vậy kiểu trả về không được
biết tại thời điểm biên dịch. (Đây là một trong những quy tắc tương thích dyn được đề cập
trước đó.)

Sự lặp lại khác bao gồm các triển khai tương tự của các phương thức `request_review`
và `approve` trên `Post`. Cả hai phương thức đều sử dụng `Option::take` với trường
`state` của `Post`, và nếu `state` là `Some`, chúng ủy quyền cho triển khai của giá trị được bọc
của cùng một phương thức đó và đặt giá trị mới của trường `state` thành kết quả.
Nếu chúng ta có nhiều phương thức trên `Post` tuân theo mẫu này, chúng ta có thể cân nhắc
định nghĩa một macro để loại bỏ sự lặp lại (xem [“Macros”][macros]<!-- ignore --> trong Chương 20).

Bằng cách triển khai mẫu thiết kế trạng thái chính xác như nó được định nghĩa cho các ngôn ngữ
hướng đối tượng, chúng ta không tận dụng hết các thế mạnh của Rust như chúng ta có thể.
Hãy xem xét một số thay đổi chúng ta có thể thực hiện đối với crate `blog` để có thể biến các
trạng thái và chuyển đổi không hợp lệ thành lỗi tại thời điểm biên dịch.

#### Mã hóa Trạng thái và Hành vi dưới dạng các Kiểu dữ liệu

Chúng tôi sẽ chỉ cho bạn cách suy nghĩ lại về mẫu thiết kế trạng thái để có được một tập hợp các
đánh đổi khác nhau. Thay vì đóng gói hoàn toàn các trạng thái và chuyển đổi để
mã bên ngoài không biết gì về chúng, chúng ta sẽ mã hóa các trạng thái thành các kiểu dữ liệu khác nhau.
Do đó, hệ thống kiểm tra kiểu của Rust sẽ ngăn chặn các nỗ lực sử dụng các bài viết nháp
ở những nơi chỉ cho phép các bài viết đã xuất bản bằng cách đưa ra lỗi trình biên dịch.

Hãy xem xét phần đầu tiên của `main` trong Liệt kê 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Chúng ta vẫn cho phép tạo các bài viết mới ở trạng thái nháp bằng `Post::new`
và khả năng thêm văn bản vào nội dung của bài viết. Nhưng thay vì có một phương thức
`content` trên một bài viết nháp trả về một chuỗi trống, chúng ta sẽ làm cho
các bài viết nháp không có phương thức `content` chút nào. Bằng cách đó, nếu chúng ta cố gắng lấy
nội dung của một bài viết nháp, chúng ta sẽ nhận được lỗi trình biên dịch cho biết phương thức đó
không tồn tại. Kết quả là, chúng ta sẽ không thể vô tình
hiển thị nội dung bài viết nháp trong môi trường thực tế (production) vì mã đó thậm chí sẽ không biên dịch được.
Liệt kê 18-19 hiển thị định nghĩa của một struct `Post` và một struct `DraftPost`,
cũng như các phương thức trên mỗi struct.

<Listing number="18-19" file-name="src/lib.rs" caption="Một `Post` với phương thức `content` và `DraftPost` không có phương thức `content` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Cả hai struct `Post` và `DraftPost` đều có một trường `content` riêng tư để
lưu trữ văn bản bài viết blog. Các struct này không còn trường `state` nữa vì
chúng ta đang di chuyển việc mã hóa trạng thái sang các kiểu của các struct. Struct `Post`
sẽ đại diện cho một bài viết đã xuất bản, và nó có một phương thức `content`
trả về `content`.

Chúng ta vẫn có một hàm `Post::new`, nhưng thay vì trả về một thực thể của
`Post`, nó trả về một thực thể của `DraftPost`. Bởi vì `content` là riêng tư
và không có bất kỳ hàm nào trả về `Post`, nên hiện tại không thể tạo
một thực thể của `Post`.

Struct `DraftPost` có một phương thức `add_text`, vì vậy chúng ta có thể thêm văn bản vào
`content` như trước, nhưng lưu ý rằng `DraftPost` không có phương thức `content`
được định nghĩa! Vì vậy, bây giờ chương trình đảm bảo tất cả các bài viết đều bắt đầu như các bài viết nháp, và các
bài viết nháp không có sẵn nội dung để hiển thị. Bất kỳ nỗ lực nào nhằm vượt qua
các ràng buộc này sẽ dẫn đến lỗi trình biên dịch.

#### Triển khai Chuyển đổi dưới dạng các Phép biến đổi thành các Kiểu dữ liệu khác nhau

Vậy làm thế nào để chúng ta có được một bài viết đã xuất bản? Chúng ta muốn thực thi quy tắc rằng một bài viết
nháp phải được xem xét và phê duyệt trước khi nó có thể được xuất bản. Một bài viết ở
trạng thái chờ xem xét vẫn không nên hiển thị bất kỳ nội dung nào. Hãy triển khai
các ràng buộc này bằng cách thêm một struct khác, `PendingReviewPost`, định nghĩa
phương thức `request_review` trên `DraftPost` để trả về một `PendingReviewPost` và
định nghĩa một phương thức `approve` trên `PendingReviewPost` để trả về một `Post`, như
được hiển thị trong Liệt kê 18-20.

<Listing number="18-20" file-name="src/lib.rs" caption="Một `PendingReviewPost` được tạo ra bằng cách gọi `request_review` trên `DraftPost` và một phương thức `approve` biến một `PendingReviewPost` thành một `Post` đã xuất bản">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

Các phương thức `request_review` và `approve` chiếm quyền sở hữu `self`, do đó
tiêu thụ các thực thể `DraftPost` và `PendingReviewPost` và biến đổi
chúng thành một `PendingReviewPost` và một `Post` đã xuất bản, tương ứng. Bằng cách này,
chúng ta sẽ không còn bất kỳ thực thể `DraftPost` nào sau khi chúng ta đã gọi
`request_review` trên chúng, và cứ tiếp tục như vậy. Struct `PendingReviewPost` không
có phương thức `content` được định nghĩa trên nó, vì vậy việc cố gắng đọc nội dung của nó
sẽ dẫn đến lỗi trình biên dịch, giống như với `DraftPost`. Bởi vì cách duy nhất để có một
thực thể `Post` đã xuất bản có phương thức `content` được định nghĩa là gọi
phương thức `approve` trên một `PendingReviewPost`, và cách duy nhất để có một
`PendingReviewPost` là gọi phương thức `request_review` trên một `DraftPost`,
chúng ta hiện đã mã hóa quy trình làm việc của bài viết blog vào hệ thống kiểu.

Nhưng chúng ta cũng phải thực hiện một số thay đổi nhỏ đối với `main`. Các phương thức `request_review` và
`approve` trả về các thực thể mới thay vì sửa đổi struct mà chúng được
gọi trên đó, vì vậy chúng ta cần thêm nhiều phép gán bóng (shadowing assignments) `let post =` để lưu
các thực thể được trả về. Chúng ta cũng không thể có các khẳng định về việc nội dung của các bài viết nháp và
chờ xem xét là các chuỗi trống, và chúng ta cũng không cần chúng: chúng ta không thể
biên dịch mã cố gắng sử dụng nội dung của các bài viết ở những trạng thái đó được nữa.
Mã được cập nhật trong `main` được hiển thị trong Liệt kê 18-21.

<Listing number="18-21" file-name="src/main.rs" caption="Các sửa đổi đối với `main` để sử dụng triển khai mới của quy trình làm việc bài viết blog">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

Những thay đổi chúng ta cần thực hiện đối với `main` để gán lại `post` có nghĩa là
triển khai này không hoàn toàn tuân theo mẫu thiết kế trạng thái hướng đối tượng nữa:
các phép biến đổi giữa các trạng thái không còn được đóng gói hoàn toàn
bên trong triển khai `Post`. Tuy nhiên, lợi ích của chúng ta là các trạng thái không hợp lệ
giờ đây là không thể xảy ra nhờ hệ thống kiểu và việc kiểm tra kiểu diễn ra tại
thời điểm biên dịch! Điều này đảm bảo rằng một số lỗi nhất định, chẳng hạn như hiển thị nội dung của
một bài viết chưa xuất bản, sẽ được phát hiện trước khi chúng được đưa vào thực tế.

Hãy thử các nhiệm vụ được gợi ý ở đầu phần này trên crate `blog` như nó
đang ở sau Liệt kê 18-21 để xem bạn nghĩ gì về thiết kế của phiên bản mã này.
Lưu ý rằng một số nhiệm vụ có thể đã được hoàn thành trong thiết kế này.

Chúng ta đã thấy rằng mặc dù Rust có khả năng triển khai các mẫu thiết kế hướng đối tượng,
nhưng các mẫu khác, chẳng hạn như mã hóa trạng thái vào hệ thống kiểu,
cũng có sẵn trong Rust. Các mẫu này có những đánh đổi khác nhau. Mặc dù
bạn có thể rất quen thuộc với các mẫu hướng đối tượng, nhưng việc suy nghĩ lại về
vấn đề để tận dụng các tính năng của Rust có thể mang lại lợi ích, chẳng hạn như
ngăn chặn một số lỗi tại thời điểm biên dịch. Các mẫu hướng đối tượng không phải lúc nào cũng là
giải pháp tốt nhất trong Rust do một số tính năng nhất định, như quyền sở hữu, mà các ngôn ngữ
hướng đối tượng không có.

## Tóm tắt

Bất kể bạn có nghĩ Rust là một ngôn ngữ hướng đối tượng sau khi
đọc chương này hay không, bây giờ bạn đã biết rằng bạn có thể sử dụng các trait object để có được một số
tính năng hướng đối tượng trong Rust. Điều phối động có thể mang lại cho mã của bạn sự
linh hoạt để đổi lấy một chút hiệu năng thời gian chạy. Bạn có thể sử dụng sự
linh hoạt này để triển khai các mẫu hướng đối tượng có thể giúp ích cho khả năng bảo trì
của mã. Rust cũng có các tính năng khác, như quyền sở hữu, mà các ngôn ngữ
hướng đối tượng không có. Một mẫu hướng đối tượng không phải lúc nào cũng là
cách tốt nhất để tận dụng các thế mạnh của Rust, nhưng nó là một tùy chọn có sẵn.

Tiếp theo, chúng ta sẽ xem xét các mẫu (patterns), vốn là một tính năng khác của Rust cho phép
rất nhiều sự linh hoạt. Chúng ta đã xem xét chúng một cách ngắn gọn trong suốt cuốn sách nhưng
chưa thấy hết khả năng của chúng. Đi thôi!

{{#quiz ../quizzes/ch17-03-oo-design-patterns.toml}}

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
