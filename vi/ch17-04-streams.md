## Streams: Các Future theo chuỗi (Futures in Sequence)

<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

Cho đến nay trong chương này, chúng ta chủ yếu tập trung vào các future riêng lẻ. Một
ngoại lệ lớn là kênh (channel) async mà chúng ta đã sử dụng. Hãy nhớ lại cách chúng ta đã sử dụng bộ nhận (receiver) cho
kênh async của mình ở phần trước trong chương này tại phần [“Truyền
tin nhắn”][17-02-messages]<!-- ignore -->. Phương thức `recv` async
tạo ra một chuỗi các mục theo thời gian. Đây là một thực thể của một mô hình tổng quát hơn
được gọi là một _stream_.

Chúng ta đã thấy một chuỗi các mục trong Chương 13, khi chúng ta xem xét
trait `Iterator` trong phần [Trait Iterator và Phương thức `next`][iterator-trait]<!-- ignore
-->, nhưng có hai điểm khác biệt giữa iterators và bộ nhận kênh async. Điểm khác biệt đầu tiên là thời gian: iterators là đồng bộ, trong khi
bộ nhận kênh là bất đồng bộ. Điểm thứ hai là API. Khi làm việc
trực tiếp với `Iterator`, chúng ta gọi phương thức `next` đồng bộ của nó. Với
stream `trpl::Receiver` nói riêng, chúng ta đã gọi một phương thức `recv` bất đồng bộ
thay thế. Ngoài ra, các API này cảm giác rất giống nhau, và sự tương đồng đó
không phải là ngẫu nhiên. Một stream giống như một dạng lặp bất đồng bộ. Trong khi
`trpl::Receiver` cụ thể là đợi để nhận tin nhắn, thì
API stream đa năng rộng hơn nhiều: nó cung cấp mục tiếp theo theo
cách mà `Iterator` làm, nhưng theo cách bất đồng bộ.

Sự tương đồng giữa iterators và streams trong Rust có nghĩa là chúng ta thực sự có thể
tạo ra một stream từ bất kỳ iterator nào. Tương tự như với một iterator, chúng ta có thể làm việc với một
stream bằng cách gọi phương thức `next` của nó và sau đó await kết quả đầu ra, như trong Liệt kê
17-30.

<Listing number="17-30" caption="Tạo một stream từ một iterator và in các giá trị của nó" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-30/src/main.rs:stream}}
```

</Listing>

Chúng ta bắt đầu với một mảng các số, chúng ta chuyển đổi nó thành một iterator và sau đó gọi
`map` trên nó để nhân đôi tất cả các giá trị. Sau đó, chúng ta chuyển đổi iterator thành một stream
bằng cách sử dụng hàm `trpl::stream_from_iter`. Tiếp theo, chúng ta lặp qua các mục trong
stream khi chúng đến bằng vòng lặp `while let`.

Thật không may, khi chúng ta cố gắng chạy mã, nó không biên dịch được, mà thay vào đó nó
báo cáo rằng không có phương thức `next` nào khả dụng:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-30
cargo build
copy only the error output
-->

```console
error[E0599]: no method named `next` found for struct `Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = note: the full type name has been written to 'file:///projects/async-await/target/debug/deps/async_await-575db3dd3197d257.long-type-14490787947592691573.txt'
   = note: consider using `--verbose` to print the full type name to the console
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Như kết quả đầu ra này giải thích, lý do cho lỗi trình biên dịch là chúng ta cần có
trait phù hợp trong phạm vi (scope) để có thể sử dụng phương thức `next`. Với những thảo luận của chúng ta
cho đến nay, bạn có thể mong đợi một cách hợp lý rằng trait đó là `Stream`, nhưng thực tế nó là
`StreamExt`. Là viết tắt của _extension_ (mở rộng), `Ext` là một mô hình phổ biến trong
cộng đồng Rust để mở rộng một trait này với một trait khác.

Chúng tôi sẽ giải thích các trait `Stream` và `StreamExt` chi tiết hơn một chút ở
cuối chương, nhưng bây giờ tất cả những gì bạn cần biết là trait `Stream`
định nghĩa một giao diện cấp thấp kết hợp hiệu quả các trait `Iterator` và
`Future`. `StreamExt` cung cấp một tập hợp các API cấp cao hơn trên nền tảng của
`Stream`, bao gồm phương thức `next` cũng như các phương thức tiện ích khác tương tự
như những phương thức được cung cấp bởi trait `Iterator`. `Stream` và `StreamExt` vẫn chưa phải là
một phần của thư viện tiêu chuẩn của Rust, nhưng hầu hết các crate trong hệ sinh thái đều sử dụng cùng
một định nghĩa.

Cách khắc phục lỗi trình biên dịch là thêm một câu lệnh `use` cho `trpl::StreamExt`,
như trong Liệt kê 17-31.

<Listing number="17-31" caption="Sử dụng thành công một iterator làm cơ sở cho một stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-31/src/main.rs:all}}
```

</Listing>

Với tất cả các phần đó được ghép lại với nhau, mã này hoạt động theo cách chúng ta muốn! Hơn
thế nữa, bây giờ khi chúng ta đã có `StreamExt` trong phạm vi, chúng ta có thể sử dụng tất cả các phương thức
tiện ích của nó, giống như với iterators. Ví dụ, trong Liệt kê 17-32, chúng ta sử dụng
phương thức `filter` để lọc bỏ mọi thứ trừ các bội số của ba và năm.

<Listing number="17-32" caption="Lọc một stream bằng phương thức `StreamExt::filter`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-32/src/main.rs:all}}
```

</Listing>

Tất nhiên, điều này không thú vị lắm, vì chúng ta có thể làm điều tương tự với
iterators bình thường và không cần bất kỳ async nào cả. Hãy xem
chúng ta có thể làm gì mà _thực sự_ là duy nhất đối với streams.

### Kết hợp các Stream

Nhiều khái niệm được biểu diễn một cách tự nhiên dưới dạng streams: các mục trở nên có sẵn trong
một hàng đợi (queue), các khối dữ liệu được lấy dần dần từ hệ thống tệp khi
toàn bộ tập dữ liệu quá lớn so với bộ nhớ của máy tính, hoặc dữ liệu đến qua
mạng theo thời gian. Bởi vì streams là các future, chúng ta có thể sử dụng chúng với bất kỳ loại
future nào khác và kết hợp chúng theo những cách thú vị. Ví dụ, chúng ta có thể gom
nhóm các sự kiện (batch up) để tránh kích hoạt quá nhiều lệnh gọi mạng, đặt thời gian chờ (timeout) cho các chuỗi
thao tác chạy lâu, hoặc tiết lưu (throttle) các sự kiện giao diện người dùng để tránh thực hiện
các công việc vô ích.

Hãy bắt đầu bằng cách xây dựng một stream tin nhắn nhỏ để thay thế cho một stream
dữ liệu mà chúng ta có thể thấy từ một WebSocket hoặc một giao thức giao tiếp thời gian thực
khác, như được hiển thị trong Liệt kê 17-33.

<Listing number="17-33" caption="Sử dụng bộ nhận `rx` như một `ReceiverStream`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-33/src/main.rs:all}}
```

</Listing>

Đầu tiên, chúng ta tạo một hàm gọi là `get_messages` trả về `impl Stream<Item
= String>`. Đối với phần triển khai của nó, chúng ta tạo một kênh async, lặp qua
10 chữ cái đầu tiên của bảng chữ cái tiếng Anh và gửi chúng qua kênh.

Chúng ta cũng sử dụng một kiểu dữ liệu mới: `ReceiverStream`, cái giúp chuyển đổi bộ nhận `rx` từ
`trpl::channel` thành một `Stream` với phương thức `next`. Quay lại `main`, chúng ta sử dụng
vòng lặp `while let` để in tất cả các tin nhắn từ stream.

Khi chúng ta chạy mã này, chúng ta nhận được chính xác các kết quả mà chúng ta mong đợi:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Message: 'a'
Message: 'b'
Message: 'c'
Message: 'd'
Message: 'e'
Message: 'f'
Message: 'g'
Message: 'h'
Message: 'i'
Message: 'j'
```

Một lần nữa, chúng ta có thể làm điều này với API `Receiver` thông thường hoặc thậm chí là API `Iterator`
thông thường, vì vậy hãy thêm một tính năng yêu cầu streams: thêm một
timeout áp dụng cho mọi mục trong stream, và một độ trễ cho các mục mà chúng ta
phát ra, như được hiển thị trong Liệt kê 17-34.

<Listing number="17-34" caption="Sử dụng phương thức `StreamExt::timeout` để đặt giới hạn thời gian cho các mục trong một stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-34/src/main.rs:timeout}}
```

</Listing>

Chúng ta bắt đầu bằng cách thêm một timeout vào stream bằng phương thức `timeout`, phương thức
đến từ trait `StreamExt`. Sau đó, chúng ta cập nhật thân của vòng lặp `while let`,
bởi vì stream bây giờ trả về một `Result`. Biến thể `Ok` cho biết một
tin nhắn đã đến đúng giờ; biến thể `Err` cho biết rằng timeout đã hết
trước khi bất kỳ tin nhắn nào đến. Chúng ta `match` trên kết quả đó và in ra
tin nhắn khi chúng ta nhận được thành công hoặc in ra một thông báo về timeout.
Cuối cùng, lưu ý rằng chúng ta ghim (pin) các tin nhắn sau khi áp dụng timeout cho chúng,
bởi vì trình trợ giúp timeout tạo ra một stream cần được ghim để có thể
được thăm dò (poll).

Tuy nhiên, vì không có độ trễ giữa các tin nhắn, timeout này không
làm thay đổi hành vi của chương trình. Hãy thêm một độ trễ thay đổi cho các tin nhắn
mà chúng ta gửi, như được hiển thị trong Liệt kê 17-35.

<Listing number="17-35" caption="Gửi tin nhắn qua `tx` với một độ trễ async mà không làm cho `get_messages` thành một hàm async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-35/src/main.rs:messages}}
```

</Listing>

Trong `get_messages`, chúng ta sử dụng phương thức iterator `enumerate` với mảng `messages`
để chúng ta có thể lấy chỉ số (index) của mỗi mục chúng ta đang gửi cùng với chính
mục đó. Sau đó, chúng ta áp dụng một độ trễ 100 mili giây cho các mục có chỉ số chẵn và
độ trễ 300 mili giây cho các mục có chỉ số lẻ để mô phỏng các độ trễ khác nhau mà chúng ta
có thể thấy từ một stream tin nhắn trong thế giới thực. Vì timeout của chúng ta là
200 mili giây, điều này sẽ ảnh hưởng đến một nửa số tin nhắn.

Để ngủ giữa các tin nhắn trong hàm `get_messages` mà không gây chặn, chúng ta
cần sử dụng async. Tuy nhiên, chúng ta không thể biến chính `get_messages` thành một hàm
async, vì khi đó chúng ta sẽ trả về một `Future<Output = Stream<Item = String>>`
thay vì một `Stream<Item = String>>`. Người gọi sẽ phải await
chính `get_messages` để có quyền truy cập vào stream. Nhưng hãy nhớ rằng: mọi thứ trong một
future cụ thể diễn ra tuyến tính; tính đồng thời xảy ra _giữa_ các future. Việc await
`get_messages` sẽ yêu cầu nó gửi tất cả các tin nhắn, bao gồm cả
độ trễ khi ngủ giữa mỗi tin nhắn, trước khi trả về stream bộ nhận. Kết quả là,
timeout sẽ trở nên vô dụng. Sẽ không có độ trễ nào trong chính stream;
tất cả chúng sẽ xảy ra trước khi stream thực sự khả dụng.

Thay vào đó, chúng ta để `get_messages` là một hàm thông thường trả về một stream,
và chúng ta tạo ra một task (spawn a task) để xử lý các lời gọi `sleep` async.

> Ghi chú: Việc gọi `spawn_task` theo cách này hoạt động vì chúng ta đã thiết lập
> runtime của mình; nếu không, nó sẽ gây ra lỗi panic. Các bản triển khai khác chọn
> những sự đánh đổi khác nhau: chúng có thể tạo ra một runtime mới và tránh lỗi panic nhưng
> cuối cùng lại có một chút chi phí overhead dư thừa, hoặc đơn giản là không cung cấp một
> cách độc lập để tạo ra các task mà không tham chiếu đến một runtime. Hãy đảm bảo bạn
> biết sự đánh đổi nào mà runtime của bạn đã chọn và viết mã cho phù hợp!

Bây giờ mã của chúng ta có một kết quả thú vị hơn nhiều. Xen kẽ giữa mỗi cặp
tin nhắn là một lỗi `Problem: Elapsed(())`.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Message: 'a'
Problem: Elapsed(())
Message: 'b'
Message: 'c'
Problem: Elapsed(())
Message: 'd'
Message: 'e'
Problem: Elapsed(())
Message: 'f'
Message: 'g'
Problem: Elapsed(())
Message: 'h'
Message: 'i'
Problem: Elapsed(())
Message: 'j'
```

Timeout không ngăn cản các tin nhắn đến cuối cùng. Chúng ta vẫn nhận được
tất cả các tin nhắn ban đầu, vì kênh của chúng ta là _không giới hạn_ (unbounded): nó có thể chứa bao
nhiêu tin nhắn tùy thích trong bộ nhớ. Nếu tin nhắn không đến trước
timeout, trình xử lý stream của chúng ta sẽ giải quyết vấn đề đó, nhưng khi nó thăm dò lại stream
một lần nữa, tin nhắn bây giờ có thể đã đến.

Bạn có thể có được các hành vi khác nhau nếu cần bằng cách sử dụng các loại kênh khác hoặc
các loại stream khác nói chung. Hãy xem một trong số đó trong thực tế bằng cách
kết hợp một stream của các khoảng thời gian với stream tin nhắn này.

### Trộn các Stream

Đầu tiên, hãy tạo một stream khác, cái sẽ phát ra một mục sau mỗi mili giây nếu
chúng ta để nó chạy trực tiếp. Để đơn giản, chúng ta có thể sử dụng hàm `sleep` để gửi
một tin nhắn với một độ trễ và kết hợp nó với cùng một cách tiếp cận mà chúng ta đã sử dụng trong
`get_messages` để tạo ra một stream từ một kênh. Điểm khác biệt là lần này,
chúng ta sẽ gửi lại số lượng các khoảng thời gian đã trôi qua, vì vậy kiểu trả về
sẽ là `impl Stream<Item = u32>`, và chúng ta có thể gọi hàm là
`get_intervals` (xem Liệt kê 17-36).

<Listing number="17-36" caption="Tạo một stream với một bộ đếm sẽ được phát ra mỗi mili giây một lần" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-36/src/main.rs:intervals}}
```

</Listing>

Chúng ta bắt đầu bằng cách định nghĩa một `count` trong task. (Chúng ta cũng có thể định nghĩa nó bên ngoài
task, nhưng sẽ rõ ràng hơn nếu giới hạn phạm vi của bất kỳ biến nào cho trước.) Sau đó, chúng ta
tạo một vòng lặp vô hạn. Mỗi lần lặp của vòng lặp sẽ ngủ một cách bất đồng bộ trong
một mili giây, tăng bộ đếm và sau đó gửi nó qua kênh.
Bởi vì tất cả những điều này được bao bọc trong task được tạo ra bởi `spawn_task`, tất cả
nó—bao gồm cả vòng lặp vô hạn—sẽ được dọn dẹp cùng với runtime.

Loại vòng lặp vô hạn này, cái chỉ kết thúc khi toàn bộ runtime bị phá hủy,
khá phổ biến trong async Rust: nhiều chương trình cần tiếp tục chạy
vô thời hạn. Với async, điều này không chặn bất cứ thứ gì khác, miễn là có
ít nhất một điểm await trong mỗi lần lặp qua vòng lặp.

Bây giờ, quay lại khối async của hàm main, chúng ta có thể cố gắng trộn các
stream `messages` và `intervals`, như được hiển thị trong Liệt kê 17-37.

<Listing number="17-37" caption="Cố gắng trộn các stream `messages` và `intervals`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-37/src/main.rs:main}}
```

</Listing>

Chúng ta bắt đầu bằng cách gọi `get_intervals`. Sau đó, chúng ta trộn các stream `messages` và
`intervals` bằng phương thức `merge`, cái kết hợp nhiều streams
thành một stream tạo ra các mục từ bất kỳ nguồn stream nào ngay khi
các mục đó có sẵn, mà không áp đặt bất kỳ thứ tự cụ thể nào. Cuối cùng, chúng ta
lặp qua stream đã trộn đó thay vì qua `messages`.

Tại thời điểm này, cả `messages` và `intervals` đều không cần phải được ghim hoặc mutable,
vì cả hai sẽ được kết hợp thành một stream `merged` duy nhất. Tuy nhiên, lời gọi
`merge` này không biên dịch được! (Lời gọi `next` trong vòng lặp `while let` cũng không,
nhưng chúng ta sẽ quay lại vấn đề đó sau.) Điều này là do hai streams có
các kiểu dữ liệu khác nhau. Stream `messages` có kiểu `Timeout<impl Stream<Item =
String>>`, trong đó `Timeout` là kiểu triển khai `Stream` cho một lời gọi
`timeout`. Stream `intervals` có kiểu `impl Stream<Item = u32>`. Để trộn
hai streams này, chúng ta cần chuyển đổi một trong số chúng để khớp với cái còn lại. Chúng ta sẽ
làm lại stream intervals, vì messages đã ở định dạng cơ bản mà chúng ta
muốn và phải xử lý các lỗi timeout (xem Liệt kê 17-38).

<!-- We cannot directly test this one, because it never stops. -->

<Listing number="17-38" caption="Căn chỉnh kiểu dữ liệu của stream `intervals` với kiểu của stream `messages`" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-38/src/main.rs:main}}
```

</Listing>

Đầu tiên, chúng ta có thể sử dụng phương thức trợ giúp `map` để chuyển đổi các `intervals` thành một
chuỗi. Thứ hai, chúng ta cần khớp với `Timeout` từ `messages`. Tuy nhiên, vì chúng ta không thực sự
_muốn_ một timeout cho `intervals`, chúng ta chỉ có thể tạo ra một timeout
dài hơn các khoảng thời gian khác mà chúng ta đang sử dụng. Ở đây, chúng ta tạo một
timeout 10 giây với `Duration::from_secs(10)`. Cuối cùng, chúng ta cần làm cho
`stream` là mutable, để các lời gọi `next` của vòng lặp `while let` có thể lặp qua
stream, và ghim nó để đảm bảo an toàn khi làm như vậy. Điều đó đưa chúng ta _gần như_
đến nơi chúng ta cần. Mọi thứ đều được kiểm tra kiểu dữ liệu. Tuy nhiên, nếu bạn chạy cái này,
sẽ có hai vấn đề. Đầu tiên, nó sẽ không bao giờ dừng lại! Bạn sẽ cần phải dừng nó bằng
<span class="keystroke">ctrl-c</span>. Thứ hai, các tin nhắn từ bảng chữ cái tiếng Anh
sẽ bị vùi lấp giữa tất cả các tin nhắn của bộ đếm interval:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the tasks running differently rather than
changes in the compiler -->

```text
--snip--
Interval: 38
Interval: 39
Interval: 40
Message: 'a'
Interval: 41
Interval: 42
Interval: 43
--snip--
```

Liệt kê 17-39 cho thấy một cách để giải quyết hai vấn đề cuối cùng này.

<Listing number="17-39" caption="Sử dụng `throttle` và `take` để quản lý các streams đã trộn" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-39/src/main.rs:throttle}}
```

</Listing>

Đầu tiên, chúng ta sử dụng phương thức `throttle` trên stream `intervals` để nó không
lấn át stream `messages`. _Tiết lưu_ (throttling) là một cách để giới hạn tốc độ
mà một hàm sẽ được gọi—hoặc, trong trường hợp này, tần suất stream sẽ được
thăm dò (poll). Mỗi 100 mili giây một lần là đủ, vì đó là tốc độ xấp xỉ mà
các tin nhắn của chúng ta đến.

Để giới hạn số lượng mục mà chúng ta sẽ chấp nhận từ một stream, chúng ta áp dụng phương thức
`take` cho stream `merged`, vì chúng ta muốn giới hạn đầu ra cuối cùng, chứ không
chỉ một stream hay stream kia.

Bây giờ khi chúng ta chạy chương trình, nó dừng lại sau khi lấy 20 mục từ stream,
và các intervals không lấn át các tin nhắn. Chúng ta cũng không nhận được `Interval:
100` hay `Interval: 200` v.v., mà thay vào đó nhận được `Interval: 1`, `Interval: 2`,
v.v.—ngay cả khi chúng ta có một stream nguồn _có thể_ tạo ra một sự kiện mỗi
mili giây. Đó là bởi vì lời gọi `throttle` tạo ra một stream mới bao bọc
stream ban đầu để stream ban đầu chỉ được thăm dò ở tốc độ tiết lưu, chứ không
phải tốc độ “tự nhiên” của chính nó. Chúng ta không có một đống tin nhắn interval chưa xử lý
mà chúng ta đang chọn bỏ qua. Thay vào đó, chúng ta thậm chí không bao giờ tạo ra những tin nhắn
interval đó ngay từ đầu! Đây chính là “tính lười biếng” (laziness) vốn có của các futures trong Rust
đang hoạt động, cho phép chúng ta lựa chọn các đặc tính hiệu suất của mình.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Interval: 1
Message: 'a'
Interval: 2
Interval: 3
Problem: Elapsed(())
Interval: 4
Message: 'b'
Interval: 5
Message: 'c'
Interval: 6
Interval: 7
Problem: Elapsed(())
Interval: 8
Message: 'd'
Interval: 9
Message: 'e'
Interval: 10
Interval: 11
Problem: Elapsed(())
Interval: 12
```

Có một điều cuối cùng chúng ta cần xử lý: các lỗi! Với cả hai
streams dựa trên kênh này, các lời gọi `send` có thể thất bại khi phía bên kia của
kênh đóng lại—và đó chỉ là vấn đề về cách runtime thực thi các futures
tạo nên stream. Cho đến tận bây giờ, chúng ta đã bỏ qua khả năng này bằng cách gọi
`unwrap`, nhưng trong một ứng dụng được viết tốt, chúng ta nên xử lý lỗi một cách rõ ràng, ít nhất là
bằng cách kết thúc vòng lặp để không cố gửi thêm bất kỳ tin nhắn nào nữa. Liệt kê
17-40 cho thấy một chiến lược xử lý lỗi đơn giản: in ra vấn đề và sau đó `break` khỏi các
vòng lặp.

<Listing number="17-40" caption="Xử lý lỗi và tắt các vòng lặp">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-40/src/main.rs:errors}}
```

</Listing>

Như thường lệ, cách đúng đắn để xử lý lỗi gửi tin nhắn sẽ khác nhau; chỉ cần đảm bảo
bạn có một chiến lược.

Bây giờ chúng ta đã thấy rất nhiều ví dụ async trong thực tế, hãy lùi lại một bước và tìm hiểu
một vài chi tiết về cách `Future`, `Stream`, và các trait then chốt khác mà
Rust sử dụng để làm cho async hoạt động.

{{#quiz ../quizzes/async-04-streams.toml}}

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
