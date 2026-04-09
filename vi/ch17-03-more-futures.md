## Làm việc với số lượng Futures bất kỳ

Khi chúng ta chuyển từ việc sử dụng hai future sang ba ở phần trước, chúng ta
cũng đã phải chuyển từ việc sử dụng `join` sang sử dụng `join3`. Sẽ thật phiền phức
nếu phải gọi một hàm khác nhau mỗi khi chúng ta thay đổi số lượng future mà chúng ta
muốn join. May mắn thay, chúng ta có một dạng macro của `join` mà chúng ta có thể truyền
vào một số lượng đối số tùy ý. Nó cũng tự xử lý việc await chính các future đó.
Do đó, chúng ta có thể viết lại mã từ Liệt kê 17-13 để sử dụng `join!` thay vì
`join3`, như trong Liệt kê 17-14.

<Listing number="17-14" caption="Sử dụng `join!` để đợi nhiều future" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:here}}
```

</Listing>

Đây chắc chắn là một cải tiến so với việc hoán đổi giữa `join` và
`join3` và `join4` v.v.! Tuy nhiên, ngay cả dạng macro này cũng chỉ hoạt động
khi chúng ta biết trước số lượng future. Tuy nhiên, trong mã Rust thực tế,
việc đẩy các future vào một bộ sưu tập và sau đó đợi một số hoặc
tất cả chúng hoàn thành là một mô hình phổ biến.

Để kiểm tra tất cả các future trong một bộ sưu tập nào đó, chúng ta sẽ cần lặp lại và
join trên _tất cả_ chúng. Hàm `trpl::join_all` chấp nhận bất kỳ kiểu nào
triển khai trait `Iterator`, điều mà bạn đã học trong [Trait Iterator
và Phương thức `next`][iterator-trait]<!-- ignore --> Chương 13, vì vậy
nó có vẻ chính là thứ chúng ta cần. Hãy thử đặt các future của chúng ta vào một vector và
thay thế `join!` bằng `join_all` như trong Liệt kê 17-15.

<Listing  number="17-15" caption="Lưu trữ các future ẩn danh trong một vector và gọi `join_all`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:here}}
```

</Listing>

Thật không may, mã này không biên dịch được. Thay vào đó, chúng ta nhận được lỗi này:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo build
copy just the compiler error
-->

```text
error[E0308]: mismatched types
  --> src/main.rs:45:37
   |
10 |         let tx1_fut = async move {
   |                       ---------- the expected `async` block
...
24 |         let rx_fut = async {
   |                      ----- the found `async` block
...
45 |         let futures = vec![tx1_fut, rx_fut, tx_fut];
   |                                     ^^^^^^ expected `async` block, found a different `async` block
   |
   = note: expected `async` block `{async block@src/main.rs:10:23: 10:33}`
              found `async` block `{async block@src/main.rs:24:22: 24:27}`
   = note: no two async blocks, even if identical, have the same type
   = help: consider pinning your async block and casting it to a trait object
```

Điều này có thể gây ngạc nhiên. Xét cho cùng, không có khối async nào trả về bất cứ thứ gì,
vì vậy mỗi khối tạo ra một `Future<Output = ()>`. Tuy nhiên, hãy nhớ rằng `Future` là một trait,
và trình biên dịch tạo ra một enum duy nhất cho mỗi khối async. Bạn
không thể đặt hai struct viết tay khác nhau vào một `Vec`, và quy tắc tương tự
cũng áp dụng cho các enum khác nhau do trình biên dịch tạo ra.

Để làm cho điều này hoạt động, chúng ta cần sử dụng _đối tượng trait_ (trait objects),
giống như chúng ta đã làm trong [“Trả về lỗi từ hàm run”][dyn]<!-- ignore --> ở Chương 12.
(Chúng ta sẽ tìm hiểu chi tiết về các đối tượng trait trong Chương 18.) Việc sử dụng các đối tượng trait
cho phép chúng ta coi mỗi future ẩn danh được tạo ra bởi các kiểu này là cùng một kiểu,
bởi vì tất cả chúng đều triển khai trait `Future`.

> Ghi chú: Trong [Sử dụng một Enum để Lưu trữ nhiều Giá trị][enum-alt]<!-- ignore --> ở
> Chương 8, chúng ta đã thảo luận về một cách khác để bao gồm nhiều kiểu trong một `Vec`:
> sử dụng một enum để đại diện cho mỗi kiểu có thể xuất hiện trong vector. Tuy nhiên, chúng ta
> không thể làm điều đó ở đây. Một mặt, chúng ta không có cách nào để đặt tên cho các kiểu khác nhau,
> bởi vì chúng là ẩn danh. Mặt khác, lý do chúng ta tìm đến một
> vector và `join_all` ngay từ đầu là để có thể làm việc với một bộ sưu tập động
> các future mà chúng ta chỉ quan tâm rằng chúng có cùng kiểu đầu ra.

Chúng ta bắt đầu bằng cách bao bọc mỗi future trong `vec!` vào một `Box::new`, như được hiển thị
trong Liệt kê 17-16.

<Listing number="17-16" caption="Sử dụng `Box::new` để căn chỉnh các kiểu của các future trong một `Vec`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Thật không may, mã này vẫn không biên dịch được. Thực tế, chúng ta nhận được cùng một lỗi
cơ bản như trước cho cả hai lệnh gọi `Box::new` thứ hai và thứ ba, cũng như
các lỗi mới đề cập đến trait `Unpin`. Chúng ta sẽ quay lại các lỗi `Unpin`
trong giây lát. Trước tiên, hãy sửa các lỗi kiểu trên các lệnh gọi `Box::new` bằng cách
chú thích rõ ràng kiểu của biến `futures` (xem Liệt kê 17-17).

<Listing number="17-17" caption="Sửa phần còn lại của các lỗi không khớp kiểu bằng cách sử dụng khai báo kiểu rõ ràng" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:here}}
```

</Listing>

Khai báo kiểu này hơi phức tạp, vì vậy chúng ta hãy cùng xem qua nó:

1. Kiểu sâu nhất là chính future đó. Chúng ta lưu ý rõ ràng rằng đầu ra
   của future là kiểu unit `()` bằng cách viết `Future<Output = ()>`.
2. Sau đó, chúng ta chú thích trait bằng `dyn` để đánh dấu nó là động.
3. Toàn bộ tham chiếu trait được bao bọc trong một `Box`.
4. Cuối cùng, chúng ta tuyên bố rõ ràng rằng `futures` là một `Vec` chứa các
   mục này.

Điều đó đã tạo ra một sự khác biệt lớn. Bây giờ khi chúng ta chạy trình biên dịch, chúng ta chỉ nhận được
các lỗi đề cập đến `Unpin`. Mặc dù có ba lỗi trong số chúng, nhưng nội dung của chúng
rất giống nhau.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-17
cargo build
# copy *only* the errors
# fix the paths
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
   --> src/main.rs:49:24
    |
49  |         trpl::join_all(futures).await;
    |         -------------- ^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
    |         |
    |         required by a bound introduced by this call
    |
    = note: consider using the `pin!` macro
            consider using `Box::pin` if you need to access the pinned value outside of the current scope
    = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `join_all`
   --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:105:14
    |
102 | pub fn join_all<I>(iter: I) -> JoinAll<I::Item>
    |        -------- required by a bound in this function
...
105 |     I::Item: Future,
    |              ^^^^^^ required by this bound in `join_all`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:9
   |
49 |         trpl::join_all(futures).await;
   |         ^^^^^^^^^^^^^^^^^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:33
   |
49 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `async_await` (bin "async_await") due to 3 previous errors
```

Đó là một khối lượng thông tin _lớn_ cần tiêu hóa, vì vậy hãy cùng phân tích nó. Phần đầu tiên của thông báo
nói với chúng ta rằng khối async đầu tiên (`src/main.rs:8:23: 20:10`) không
triển khai trait `Unpin` và gợi ý sử dụng `pin!` hoặc `Box::pin` để giải quyết
nó. Phần sau trong chương này, chúng ta sẽ tìm hiểu kỹ hơn một vài chi tiết về `Pin` và
`Unpin`. Tuy nhiên, tại thời điểm này, chúng ta có thể làm theo lời khuyên của trình biên dịch để thoát khỏi
bế tắc. Trong Liệt kê 17-18, chúng ta bắt đầu bằng cách import `Pin` từ `std::pin`. Tiếp theo chúng ta
cập nhật chú thích kiểu cho `futures`, với một `Pin` bao bọc mỗi `Box`.
Cuối cùng, chúng ta sử dụng `Box::pin` để ghim (pin) chính các future đó.

<Listing number="17-18" caption="Sử dụng `Pin` và `Box::pin` để làm cho kiểu của `Vec` được kiểm tra đúng" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Nếu chúng ta biên dịch và chạy đoạn mã này, cuối cùng chúng ta sẽ nhận được đầu ra mà chúng ta hy vọng:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'messages'
received 'the'
received 'for'
received 'future'
received 'you'
```

Phù!

Có thêm một vài điều để khám phá ở đây. Một mặt, việc sử dụng `Pin<Box<T>>` làm tăng một
lượng nhỏ chi phí overhead từ việc đặt các future này trên heap với `Box`—và
chúng ta chỉ làm điều đó để làm cho các kiểu khớp với nhau. Suy cho cùng, chúng ta không thực sự _cần_
việc cấp phát heap: các future này là cục bộ cho hàm cụ thể này.
Như đã lưu ý trước đây, bản thân `Pin` là một kiểu bao bọc, vì vậy chúng ta có thể hưởng lợi từ việc
có một kiểu duy nhất trong `Vec`—lý do ban đầu chúng ta tìm đến
`Box`—mà không cần thực hiện cấp phát heap. Chúng ta có thể sử dụng `Pin` trực tiếp với mỗi
future, sử dụng macro `std::pin::pin`.

Tuy nhiên, chúng ta vẫn phải nêu rõ kiểu của tham chiếu đã ghim;
nếu không, Rust sẽ vẫn không biết để diễn giải những thứ này như các đối tượng trait động,
đó là những gì chúng ta cần chúng trở thành trong `Vec`. Do đó, chúng ta thêm `pin` vào
danh sách các import từ `std::pin`. Sau đó, chúng ta có thể `pin!` mỗi future khi chúng ta định nghĩa
nó và định nghĩa `futures` là một `Vec` chứa các tham chiếu mutable đã được ghim đến
kiểu future động, như trong Liệt kê 17-19.

<Listing number="17-19" caption="Sử dụng `Pin` trực tiếp với macro `pin!` để tránh các việc cấp phát heap không cần thiết" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:here}}
```

</Listing>

Chúng ta đã đi xa đến mức này bằng cách bỏ qua sự thật rằng chúng ta có thể có các kiểu `Output`
khác nhau. Ví dụ, trong Liệt kê 17-20, future ẩn danh cho `a` triển khai
`Future<Output = u32>`, future ẩn danh cho `b` triển khai `Future<Output =
&str>`, và future ẩn danh cho `c` triển khai `Future<Output = bool>`.

<Listing number="17-20" caption="Ba future với các kiểu khác biệt" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:here}}
```

</Listing>

Chúng ta có thể sử dụng `trpl::join!` để await chúng, bởi vì nó cho phép chúng ta truyền vào nhiều
kiểu future và tạo ra một tuple của những kiểu đó. Chúng ta _không thể_ sử dụng
`trpl::join_all`, bởi vì nó yêu cầu tất cả các future được truyền vào phải có cùng một kiểu.
Hãy nhớ rằng, chính lỗi đó là nguyên nhân khiến chúng ta bắt đầu cuộc phiêu lưu này với
`Pin`!

Đây là một sự đánh đổi cơ bản: chúng ta có thể xử lý một số lượng động các
future với `join_all`, miễn là chúng đều có cùng kiểu, hoặc chúng ta có thể xử lý
một số lượng future cố định với các hàm `join` hoặc macro `join!`,
ngay cả khi chúng có các kiểu khác nhau. Đây cũng chính là kịch bản mà chúng ta sẽ gặp phải khi
làm việc với bất kỳ kiểu nào khác trong Rust. Futures không có gì đặc biệt, mặc dù chúng ta
có một số cú pháp đẹp để làm việc với chúng, và đó là một điều tốt.

### Đua các Futures

Khi chúng ta “join” các future bằng nhóm các hàm và macro `join`, chúng ta
yêu cầu _tất cả_ chúng phải hoàn thành trước khi chúng ta tiếp tục. Tuy nhiên, đôi khi, chúng ta chỉ
cần _một số_ future từ một tập hợp hoàn thành trước khi chúng ta tiếp tục—hơi giống với
việc cho một future chạy đua với một cái khác.

Trong Liệt kê 17-21, chúng ta một lần nữa sử dụng `trpl::race` để chạy hai future, `slow` và
`fast`, chống lại nhau.

<Listing number="17-21" caption="Sử dụng `race` để nhận kết quả của bất kỳ future nào kết thúc trước" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:here}}
```

</Listing>

Mỗi future in một thông báo khi nó bắt đầu chạy, tạm dừng trong một khoảng thời gian
bằng cách gọi và await `sleep`, và sau đó in một thông báo khác khi nó
kết thúc. Sau đó chúng ta truyền cả `slow` và `fast` cho `trpl::race` và đợi một
trong số chúng hoàn thành. (Kết quả ở đây không quá ngạc nhiên: `fast` thắng.) Không giống như
khi chúng ta sử dụng `race` trước đây trong [“Chương trình Async đầu tiên của chúng ta”][async-program]<!--
ignore -->, chúng ta chỉ bỏ qua thực thể `Either` mà nó trả về ở đây, vì tất cả
các hành vi thú vị đều diễn ra trong thân của các khối async.

Lưu ý rằng nếu bạn đảo ngược thứ tự các đối số cho `race`, thứ tự của
thông báo “started” sẽ thay đổi, mặc dù future `fast` luôn hoàn thành
trước. Đó là bởi vì việc triển khai hàm `race` cụ thể này là không
công bằng. Nó luôn chạy các future được truyền vào dưới dạng đối số theo thứ tự
mà chúng được truyền. Các bản triển khai khác _là_ công bằng và sẽ chọn ngẫu nhiên
future nào để thăm dò (poll) trước. Tuy nhiên, bất kể việc triển khai race
chúng ta đang sử dụng có công bằng hay không, _một_ trong các future sẽ chạy cho đến điểm
`await` đầu tiên trong thân của nó trước khi một tác vụ khác có thể bắt đầu.

Hãy nhớ lại từ [Chương trình Async đầu tiên của chúng ta][async-program]<!-- ignore --> rằng tại mỗi
điểm await, Rust cho runtime một cơ hội để tạm dừng tác vụ và chuyển sang
tác vụ khác nếu future đang được await chưa sẵn sàng. Điều ngược lại cũng đúng:
Rust _chỉ_ tạm dừng các khối async và trao lại quyền kiểm soát cho một runtime tại một điểm await.
Mọi thứ giữa các điểm await đều là đồng bộ.

Điều đó có nghĩa là nếu bạn thực hiện một đống công việc trong một khối async mà không có điểm await,
future đó sẽ chặn bất kỳ future nào khác tiến triển. Đôi khi bạn có thể
nghe thấy điều này được gọi là một future làm _đói_ (starving) các future khác. Trong một số trường hợp,
điều đó có thể không phải là vấn đề lớn. Tuy nhiên, nếu bạn đang thực hiện một số loại thiết lập tốn kém
hoặc công việc chạy lâu, hoặc nếu bạn có một future sẽ tiếp tục thực hiện một tác vụ
cụ thể nào đó vô thời hạn, bạn sẽ cần suy nghĩ về việc khi nào và ở đâu để trao lại quyền
kiểm soát cho runtime.

Cùng một lý lẽ đó, nếu bạn có các thao tác chặn chạy lâu, async có thể là một
công cụ hữu ích để cung cấp các cách giúp các phần khác nhau của chương trình liên quan đến nhau.

Nhưng làm _thế nào_ để bạn trao lại quyền kiểm soát cho runtime trong những trường hợp đó?

<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Nhường quyền Kiểm soát cho Runtime

Hãy mô phỏng một thao tác chạy lâu. Liệt kê 17-22 giới thiệu một hàm `slow`.

<Listing number="17-22" caption="Sử dụng `thread::sleep` để mô phỏng các thao tác chậm" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:slow}}
```

</Listing>

Mã này sử dụng `std::thread::sleep` thay vì `trpl::sleep` để việc gọi
`slow` sẽ chặn luồng hiện tại trong một số mili giây. Chúng ta có thể sử dụng
`slow` để đại diện cho các thao tác trong thế giới thực vừa chạy lâu vừa
chặn.

Trong Liệt kê 17-23, chúng ta sử dụng `slow` để mô phỏng việc thực hiện loại công việc ràng buộc bởi CPU này trong
một cặp future.

<Listing number="17-23" caption="Sử dụng `thread::sleep` để mô phỏng các thao tác chậm" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:slow-futures}}
```

</Listing>

Để bắt đầu, mỗi future chỉ trao lại quyền kiểm soát cho runtime _sau khi_ thực hiện
một đống các thao tác chậm. Nếu bạn chạy mã này, bạn sẽ thấy đầu ra này:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Giống như ví dụ trước của chúng ta, `race` vẫn hoàn thành ngay khi `a` xong.
Tuy nhiên, không có sự đan xen giữa hai future. Future `a` thực hiện tất cả
công việc của nó cho đến khi lệnh gọi `trpl::sleep` được await, sau đó future `b` thực hiện
tất cả công việc của nó cho đến khi lệnh gọi `trpl::sleep` của chính nó được await, và cuối cùng future `a`
hoàn thành. Để cho phép cả hai future tiến triển giữa các tác vụ chậm của chúng,
chúng ta cần các điểm await để chúng ta có thể trao lại quyền kiểm soát cho runtime. Điều đó
có nghĩa là chúng ta cần thứ gì đó mà chúng ta có thể await!

Chúng ta đã có thể thấy loại hình bàn giao này diễn ra trong Liệt kê 17-23: nếu chúng ta
xóa `trpl::sleep` ở cuối future `a`, nó sẽ hoàn thành mà future `b` không
được chạy _chút nào_. Hãy thử sử dụng hàm `sleep` như một khởi điểm để
cho phép các thao tác luân phiên tiến triển, như được hiển thị trong Liệt kê 17-24.

<Listing number="17-24" caption="Sử dụng `sleep` để cho phép các thao tác luân phiên tiến triển" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

Trong Liệt kê 17-24, chúng ta thêm các lời gọi `trpl::sleep` với các điểm await giữa mỗi lệnh gọi
đến `slow`. Bây giờ công việc của hai future được đan xen:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-24
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

Future `a` vẫn chạy một lát trước khi bàn giao quyền kiểm soát cho `b`, vì
nó gọi `slow` trước khi gọi `trpl::sleep`, nhưng sau đó các future
hoán đổi qua lại mỗi khi một trong số chúng chạm tới một điểm await. Trong trường hợp này,
chúng ta đã thực hiện điều đó sau mỗi lần gọi đến `slow`, nhưng chúng ta có thể chia nhỏ công việc
theo bất kỳ cách nào có ý nghĩa nhất đối với chúng ta.

Tuy nhiên, chúng ta thực sự không muốn _ngủ_ ở đây: chúng ta muốn tiến hành nhanh nhất
có thể. Chúng ta chỉ cần trả lại quyền kiểm soát cho runtime. Chúng ta có thể làm điều đó
trực tiếp, bằng cách sử dụng hàm `yield_now`. Trong Liệt kê 17-25, chúng ta thay thế tất cả những
lệnh gọi `sleep` đó bằng `yield_now`.

<Listing number="17-25" caption="Sử dụng `yield_now` để cho phép các thao tác luân phiên tiến triển" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:yields}}
```

</Listing>

Mã này vừa rõ ràng hơn về mục đích thực tế vừa có thể nhanh hơn đáng kể
so với việc sử dụng `sleep`, bởi vì các bộ đếm thời gian (timers) như bộ đếm được sử dụng bởi `sleep` thường
có các giới hạn về độ chi tiết của chúng. Ví dụ, phiên bản `sleep` mà chúng ta đang sử dụng,
sẽ luôn ngủ ít nhất một mili giây, ngay cả khi chúng ta truyền cho nó một
`Duration` một nano giây. Nhắc lại là máy tính hiện đại rất _nhanh_: chúng có thể làm
rất nhiều việc trong một mili giây!

Bạn có thể tự mình thấy điều này bằng cách thiết lập một bài kiểm tra hiệu năng (benchmark) nhỏ, chẳng hạn như
bài kiểm tra trong Liệt kê 17-26. (Đây không phải là một cách đặc biệt nghiêm ngặt để thực hiện kiểm tra
hiệu năng, nhưng nó đủ để cho thấy sự khác biệt ở đây.)

<Listing number="17-26" caption="So sánh hiệu suất của `sleep` và `yield_now`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-26/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta bỏ qua tất cả việc in trạng thái, truyền một `Duration` một nano giây cho
`trpl::sleep`, và để mỗi future chạy một mình, không có sự chuyển đổi giữa các
future. Sau đó chúng ta chạy 1.000 lần lặp và xem future sử dụng
`trpl::sleep` mất bao lâu so với future sử dụng `trpl::yield_now`.

Phiên bản với `yield_now` nhanh hơn _rất nhiều_!

Điều này có nghĩa là async có thể hữu ích ngay cả đối với các tác vụ ràng buộc bởi tính toán, tùy thuộc vào
những gì khác mà chương trình của bạn đang làm, bởi vì nó cung cấp một công cụ hữu ích để
cấu trúc các mối quan hệ giữa các phần khác nhau của chương trình. Đây là một
dạng của _đa nhiệm cộng tác_ (cooperative multitasking), nơi mỗi future có quyền quyết định
khi nào nó chuyển giao quyền kiểm soát thông qua các điểm await. Do đó, mỗi future cũng có
trách nhiệm tránh việc chặn quá lâu. Trong một số hệ điều hành nhúng dựa trên Rust,
đây là loại đa nhiệm _duy nhất_!

Tất nhiên, trong mã thực tế, bạn thường sẽ không luân phiên các lệnh gọi hàm với các điểm await trên
mọi dòng đơn lẻ. Mặc dù việc nhường quyền kiểm soát theo cách này tương đối rẻ, nhưng nó không miễn phí.
Trong nhiều trường hợp, việc cố gắng chia nhỏ một tác vụ ràng buộc bởi tính toán có thể làm cho nó chậm hơn
đáng kể, vì vậy đôi khi sẽ tốt hơn cho hiệu suất _tổng thể_ nếu để một thao tác chặn trong giây lát.
Luôn luôn đo lường để xem các nút thắt cổ chai về hiệu suất thực tế của mã bạn là gì. Tuy nhiên,
động lực bên dưới là điều quan trọng cần ghi nhớ nếu bạn _đang_ thấy nhiều công việc diễn ra
tuần tự mà bạn mong đợi chúng diễn ra đồng thời!

### Xây dựng các Trừu tượng Async của riêng chúng ta

Chúng ta cũng có thể kết hợp các future lại với nhau để tạo ra các mô hình mới. Ví dụ, chúng ta có thể
xây dựng một hàm `timeout` với các khối xây dựng async mà chúng ta đã có. Khi
chúng ta hoàn thành, kết quả sẽ là một khối xây dựng khác mà chúng ta có thể sử dụng để tạo ra
thêm nhiều trừu tượng async hơn nữa.

Liệt kê 17-27 cho thấy cách chúng ta mong đợi `timeout` này hoạt động với một future chậm.

<Listing number="17-27" caption="Sử dụng `timeout` giả định của chúng ta để chạy một thao tác chậm với giới hạn thời gian" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-27/src/main.rs:here}}
```

</Listing>

Hãy triển khai điều này! Để bắt đầu, hãy nghĩ về API cho `timeout`:

- Bản thân nó cần phải là một hàm async để chúng ta có thể await nó.
- Tham số đầu tiên của nó nên là một future để chạy. Chúng ta có thể làm cho nó generic để cho phép
  nó hoạt động với bất kỳ future nào.
- Tham số thứ hai của nó sẽ là thời gian chờ tối đa. Nếu chúng ta sử dụng một `Duration`,
  điều đó sẽ giúp dễ dàng truyền nó dọc theo `trpl::sleep`.
- Nó nên trả về một `Result`. Nếu future hoàn thành thành công,
  `Result` sẽ là `Ok` với giá trị được tạo ra bởi future. Nếu thời gian chờ
  hết trước, `Result` sẽ là `Err` với khoảng thời gian mà timeout đã chờ.

Liệt kê 17-28 hiển thị khai báo này.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-28" caption="Định nghĩa chữ ký của `timeout`" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-28/src/main.rs:declaration}}
```

</Listing>

Điều đó thỏa mãn các mục tiêu của chúng ta về các kiểu dữ liệu. Bây giờ hãy nghĩ về _hành vi_ chúng ta
cần: chúng ta muốn cho future được truyền vào chạy đua với khoảng thời gian. Chúng ta có thể sử dụng
`trpl::sleep` để tạo ra một future đếm giờ từ khoảng thời gian đó, và sử dụng `trpl::race` để
chạy bộ đếm giờ đó với future mà người gọi truyền vào.

Chúng ta cũng biết rằng `race` không công bằng, nó thăm dò (poll) các đối số theo thứ tự mà
chúng được truyền vào. Do đó, chúng ta truyền `future_to_try` cho `race` trước để nó
có cơ hội hoàn thành ngay cả khi `max_time` là một khoảng thời gian rất ngắn. Nếu
`future_to_try` kết thúc trước, `race` sẽ trả về `Left` với đầu ra từ
`future_to_try`. Nếu `timer` kết thúc trước, `race` sẽ trả về `Right` với
đầu ra của timer là `()`.

Trong Liệt kê 17-29, chúng ta match trên kết quả của việc await `trpl::race`.

<Listing number="17-29" caption="Định nghĩa `timeout` với `race` và `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-29/src/main.rs:implementation}}
```

</Listing>

Nếu `future_to_try` thành công và chúng ta nhận được `Left(output)`, chúng ta trả về
`Ok(output)`. Nếu timer ngủ hết hạn thay vào đó và chúng ta nhận được `Right(())`, chúng ta
bỏ qua `()` bằng `_` và trả về `Err(max_time)` thay thế.

Với điều đó, chúng ta đã có một `timeout` đang hoạt động được xây dựng từ hai trình trợ giúp async khác. Nếu
chúng ta chạy mã của mình, nó sẽ in ra chế độ lỗi sau khi hết thời gian chờ:

```text
Failed after 2 seconds
```

Bởi vì các future kết hợp với các future khác, bạn có thể xây dựng các công cụ thực sự mạnh mẽ
bằng cách sử dụng các khối xây dựng async nhỏ hơn. Ví dụ, bạn có thể sử dụng chính cách tiếp cận này
để kết hợp timeouts với việc thử lại (retries), và lần lượt sử dụng chúng với các thao tác như
lời gọi mạng (một trong những ví dụ từ đầu chương).

Trong thực tế, bạn thường sẽ làm việc trực tiếp với `async` và `await`, và
thứ hai là với các hàm và macro như `join`, `join_all`, `race`, v.v. Bạn
chỉ cần tìm đến `pin` thỉnh thoảng để sử dụng các future với các API đó.

Chúng ta đã thấy một số cách để làm việc với nhiều future tại cùng một
thời điểm. Tiếp theo, chúng ta sẽ xem cách chúng ta có thể làm việc với nhiều future trong một
chuỗi theo thời gian với _streams_. Tuy nhiên, trước hết đây là một vài điều khác bạn có thể muốn
xem xét:

- Chúng ta đã sử dụng một `Vec` với `join_all` để đợi tất cả các future trong một nhóm nào đó
  hoàn thành. Thay vào đó, làm thế nào bạn có thể sử dụng một `Vec` để xử lý một nhóm các future
  theo thứ tự? Các sự đánh đổi của việc làm đó là gì?

- Hãy xem kiểu `futures::stream::FuturesUnordered` từ crate `futures`.
  Việc sử dụng nó sẽ khác gì so với việc sử dụng một `Vec`? (Đừng lo lắng về
  thực tế là nó đến từ phần `stream` của crate; nó hoạt động tốt với
  bất kỳ bộ sưu tập future nào.)

{{#quiz ../quizzes/async-03-more-futures.toml}}

[dyn]: ch12-03-improving-error-handling-and-modularity.html
[enum-alt]: ch08-01-vectors.html#using-an-enum-to-store-multiple-types
[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
