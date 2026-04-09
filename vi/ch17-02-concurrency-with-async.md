## Áp dụng concurrency với Async

<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

Trong phần này, chúng ta sẽ áp dụng async vào một số thử thách đồng thời tương tự
mà chúng ta đã giải quyết với các luồng (threads) trong chương 16. Vì chúng ta đã nói về nhiều
ý tưởng then chốt ở đó, trong phần này chúng ta sẽ tập trung vào những điểm khác biệt giữa
threads và futures.

Trong nhiều trường hợp, các API để làm việc với tính đồng thời bằng async rất
giống với các API sử dụng luồng. Trong các trường hợp khác, chúng lại khá
khác biệt. Ngay cả khi các API _trông_ giống nhau giữa luồng và async, chúng
thường có hành vi khác nhau—và chúng hầu như luôn có các đặc tính hiệu suất khác nhau.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Tạo một Tác vụ Mới với `spawn_task`

Thao tác đầu tiên chúng ta đã giải quyết trong [Tạo một Luồng mới với
Spawn][thread-spawn]<!-- ignore --> là đếm số trên hai luồng riêng biệt.
Hãy làm điều tương tự bằng cách sử dụng async. Crate `trpl` cung cấp một hàm `spawn_task`
trông rất giống với API `thread::spawn`, và một hàm `sleep`
là phiên bản async của API `thread::sleep`. Chúng ta có thể sử dụng chúng cùng nhau
để triển khai ví dụ đếm số, như được hiển thị trong Liệt kê 17-6.

<Listing number="17-6" caption="Tạo một tác vụ mới để in một thứ trong khi tác vụ chính in một thứ khác" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Để bắt đầu, chúng ta thiết lập hàm `main` với `trpl::run` để
hàm cấp cao nhất của chúng ta có thể là async.

> Ghi chú: Từ thời điểm này trở đi trong chương, mọi ví dụ sẽ bao gồm đoạn mã
> bao bọc tương tự với `trpl::run` trong `main`, vì vậy chúng tôi sẽ thường bỏ qua nó
> giống như cách chúng tôi làm với `main`. Đừng quên đưa nó vào mã của bạn!

Sau đó, chúng ta viết hai vòng lặp bên trong khối đó, mỗi vòng lặp chứa một lệnh gọi `trpl::sleep`,
lệnh này sẽ đợi nửa giây (500 mili giây) trước khi gửi tin nhắn
tiếp theo. Chúng ta đặt một vòng lặp vào thân của một `trpl::spawn_task` và vòng lặp còn lại vào một
vòng lặp `for` ở cấp cao nhất. Chúng ta cũng thêm một `await` sau các lệnh gọi `sleep`.

Mã này hoạt động tương tự như bản triển khai dựa trên luồng—bao gồm cả việc
bạn có thể thấy các tin nhắn xuất hiện theo một thứ tự khác trong terminal của chính bạn
khi bạn chạy nó:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Phiên bản này dừng lại ngay khi vòng lặp `for` trong thân của khối async chính
kết thúc, bởi vì tác vụ được tạo bởi `spawn_task` bị tắt khi hàm `main`
kết thúc. Nếu bạn muốn nó chạy cho đến khi tác vụ hoàn thành, bạn
sẽ cần sử dụng một join handle để đợi tác vụ đầu tiên hoàn thành. Với
các luồng, chúng ta đã sử dụng phương thức `join` để “chặn” cho đến khi luồng chạy xong.
Trong Liệt kê 17-7, chúng ta có thể sử dụng `await` để làm điều tương tự, bởi vì bản thân
task handle là một future. Kiểu `Output` của nó là một `Result`, vì vậy chúng ta cũng unwrap nó
sau khi await nó.

<Listing number="17-7" caption="Sử dụng `await` với một join handle để chạy một tác vụ cho đến khi hoàn thành" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Phiên bản cập nhật này chạy cho đến khi _cả hai_ vòng lặp kết thúc.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Cho đến nay, có vẻ như async và các luồng mang lại cho chúng ta cùng một kết quả cơ bản, chỉ là
với cú pháp khác nhau: sử dụng `await` thay vì gọi `join` trên join
handle, và await các lệnh gọi `sleep`.

Điểm khác biệt lớn hơn là chúng ta không cần phải tạo một luồng hệ điều hành
khác để làm việc này. Thực tế, chúng ta thậm chí không cần tạo một tác vụ (task) ở đây. Bởi vì
các khối async được biên dịch thành các future ẩn danh, chúng ta có thể đặt mỗi vòng lặp trong một khối async
và để runtime chạy cả hai cho đến khi hoàn thành bằng hàm `trpl::join`.

Trong phần [Chờ tất cả các luồng kết thúc bằng `join`
Handles][join-handles]<!-- ignore -->, chúng ta đã chỉ ra cách sử dụng phương thức `join` trên
kiểu `JoinHandle` được trả về khi bạn gọi `std::thread::spawn`. Hàm
`trpl::join` cũng tương tự, nhưng dành cho các future. Khi bạn cung cấp cho nó hai future,
nó tạo ra một future mới duy nhất có đầu ra là một tuple chứa đầu ra của
mỗi future bạn đã truyền vào khi _cả hai_ đều hoàn thành. Do đó, trong Liệt kê 17-8, chúng ta
sử dụng `trpl::join` để đợi cả `fut1` và `fut2` hoàn thành. Chúng ta _không_ await
`fut1` và `fut2` mà thay vào đó là future mới được tạo ra bởi `trpl::join`. Chúng ta bỏ qua
đầu ra, vì nó chỉ là một tuple chứa hai giá trị unit.

<Listing number="17-8" caption="Sử dụng `trpl::join` để await hai future ẩn danh" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Khi chúng ta chạy mã này, chúng ta thấy cả hai future đều chạy cho đến khi hoàn thành:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Bây giờ, bạn sẽ thấy thứ tự giống hệt nhau mỗi lần chạy, điều này rất khác so với
những gì chúng ta đã thấy với các luồng. Đó là bởi vì hàm `trpl::join` mang tính _công bằng_ (fair),
nghĩa là nó kiểm tra từng future thường xuyên như nhau, luân phiên giữa chúng và không bao giờ
để một cái chạy trước nếu cái kia đã sẵn sàng. Với các luồng, hệ điều hành
quyết định luồng nào sẽ được kiểm tra và để nó chạy trong bao lâu. Với async Rust,
runtime quyết định tác vụ nào sẽ được kiểm tra. (Trong thực tế, các chi tiết trở nên phức tạp
vì một async runtime có thể sử dụng các luồng hệ điều hành ở bên dưới như một phần trong
cách nó quản lý tính đồng thời, vì vậy việc đảm bảo tính công bằng có thể là một công việc tốn kém hơn
cho một runtime—nhưng nó vẫn có thể thực hiện được!) Các runtime không bắt buộc phải đảm bảo
tính công bằng cho bất kỳ thao tác nào cho trước, và chúng thường cung cấp các API khác nhau để cho phép bạn
chọn có muốn tính công bằng hay không.

Hãy thử một số biến thể này của việc await các future và xem chúng làm gì:

- Xóa khối async bao quanh một hoặc cả hai vòng lặp.
- Await mỗi khối async ngay sau khi định nghĩa nó.
- Chỉ bao bọc vòng lặp đầu tiên trong một khối async, và await future kết quả
  sau thân của vòng lặp thứ hai.

Để thêm một thử thách, hãy xem liệu bạn có thể đoán được kết quả sẽ như thế nào trong
mỗi trường hợp _trước khi_ chạy mã hay không!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>

### Đếm số trên hai Tác vụ sử dụng Truyền tin nhắn

Việc chia sẻ dữ liệu giữa các future cũng sẽ rất quen thuộc: chúng ta sẽ lại sử dụng truyền tin nhắn
(message passing), nhưng lần này là với các phiên bản async của các kiểu dữ liệu và hàm. Chúng ta sẽ đi theo
một lộ trình hơi khác so với những gì chúng ta đã làm trong [Sử dụng Truyền tin nhắn để Chuyển dữ liệu
giữa các Luồng][message-passing-threads]<!-- ignore --> để minh họa một số
khác biệt chính giữa tính đồng thời dựa trên luồng và dựa trên future. Trong
Liệt kê 17-9, chúng ta sẽ bắt đầu chỉ với một khối async đơn lẻ—_không_ tạo một
tác vụ riêng biệt như cách chúng ta đã tạo một luồng riêng biệt.

<Listing number="17-9" caption="Tạo một kênh (channel) async và gán hai nửa cho `tx` và `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Ở đây, chúng ta sử dụng `trpl::channel`, một phiên bản async của API kênh nhiều người gửi,
một người nhận (multiple-producer, single-consumer) mà chúng ta đã sử dụng với các luồng trong Chương 16. Phiên bản
async của API này chỉ khác một chút so với phiên bản dựa trên luồng: nó
sử dụng một bộ nhận `rx` có thể thay đổi (mutable) thay vì bất biến (immutable), và phương thức `recv` của nó
tạo ra một future mà chúng ta cần await thay vì tạo ra giá trị trực tiếp. Bây giờ
chúng ta có thể gửi tin nhắn từ người gửi đến người nhận. Lưu ý rằng chúng ta không cần
phải tạo một luồng riêng biệt hoặc thậm chí là một tác vụ; chúng ta chỉ cần await lời gọi `rx.recv`.

Phương thức `Receiver::recv` đồng bộ trong `std::mpsc::channel` sẽ chặn (block) cho đến khi
nó nhận được một tin nhắn. Phương thức `trpl::Receiver::recv` thì không, bởi vì nó
là async. Thay vì chặn, nó trao lại quyền kiểm soát cho runtime cho đến khi hoặc là
nhận được một tin nhắn hoặc phía gửi của kênh đóng lại. Ngược lại, chúng ta không
await lời gọi `send`, bởi vì nó không chặn. Nó không cần thiết,
bởi vì kênh mà chúng ta đang gửi vào là không giới hạn (unbounded).

> Ghi chú: Bởi vì tất cả mã async này chạy trong một khối async trong một lệnh gọi
> `trpl::run`, mọi thứ bên trong nó đều có thể tránh bị chặn. Tuy nhiên, mã _bên ngoài_ nó
> sẽ bị chặn cho đến khi hàm `run` trả về. Đó là toàn bộ mục đích của hàm
> `trpl::run`: nó cho phép bạn _chọn_ nơi để chặn trên một tập hợp mã async nào đó,
> và do đó là nơi chuyển đổi giữa mã đồng bộ và bất đồng bộ. Trong hầu hết các async
> runtime, `run` thực sự được đặt tên là `block_on` chính vì lý do này.

Lưu ý hai điều về ví dụ này. Thứ nhất, tin nhắn sẽ đến ngay lập tức.
Thứ hai, mặc dù chúng ta sử dụng một future ở đây, nhưng vẫn chưa có tính đồng thời. Mọi thứ
trong liệt kê diễn ra theo tuần tự, giống như khi không có future
nào liên quan.

Hãy giải quyết phần đầu tiên bằng cách gửi một loạt tin nhắn và ngủ ở giữa
chúng, như được hiển thị trong Liệt kê 17-10.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Gửi và nhận nhiều tin nhắn qua kênh async và ngủ bằng một `await` giữa mỗi tin nhắn" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Ngoài việc gửi tin nhắn, chúng ta cũng cần nhận chúng. Trong trường hợp này,
bởi vì chúng ta biết có bao nhiêu tin nhắn sẽ đến, chúng ta có thể làm điều đó thủ công bằng cách
gọi `rx.recv().await` bốn lần. Tuy nhiên, trong thế giới thực, chúng ta thường sẽ
phải đợi một số lượng tin nhắn _không xác định_, vì vậy chúng ta cần tiếp tục đợi
cho đến khi xác định được rằng không còn tin nhắn nào nữa.

Trong Liệt kê 16-10, chúng ta đã sử dụng vòng lặp `for` để xử lý tất cả các mục nhận được từ một
kênh đồng bộ. Tuy nhiên, Rust vẫn chưa có cách nào để viết một vòng lặp `for` trên một
chuỗi các mục _bất đồng bộ_, vì vậy chúng ta cần sử dụng một vòng lặp mà chúng ta chưa từng
thấy trước đây: vòng lặp điều kiện `while let`. Đây là phiên bản vòng lặp của cấu trúc
`if let` mà chúng ta đã thấy trong phần [Luồng điều khiển súc tích với `if let`
và `let else`][if-let]<!-- ignore -->. Vòng lặp sẽ tiếp tục thực thi chừng nào
mẫu (pattern) mà nó chỉ định tiếp tục khớp với giá trị.

Lời gọi `rx.recv` tạo ra một future, mà chúng ta await. Runtime sẽ tạm dừng
future cho đến khi nó sẵn sàng. Khi một tin nhắn đến, future sẽ phân giải thành
`Some(message)` mỗi khi có tin nhắn đến. Khi kênh đóng lại,
bất kể _bất kỳ_ tin nhắn nào đã đến hay chưa, future thay vào đó sẽ phân giải thành
`None` để cho biết rằng không còn giá trị nào nữa và do đó chúng ta nên ngừng thăm dò (polling)—nghĩa là
ngừng await.

Vòng lặp `while let` kết hợp tất cả những điều này lại với nhau. Nếu kết quả của việc gọi
`rx.recv().await` là `Some(message)`, chúng ta có quyền truy cập vào tin nhắn và có thể
sử dụng nó trong thân vòng lặp, giống như cách chúng ta có thể làm với `if let`. Nếu kết quả là
`None`, vòng lặp kết thúc. Mỗi khi vòng lặp hoàn thành, nó lại chạm tới điểm await
một lần nữa, vì vậy runtime sẽ tạm dừng nó cho đến khi có một tin nhắn khác đến.

Mã hiện tại đã gửi và nhận thành công tất cả các tin nhắn. Thật không may,
vẫn còn một vài vấn đề. Một mặt, các tin nhắn không đến sau các khoảng thời gian nửa giây.
Chúng đến tất cả cùng một lúc, 2 giây (2.000 mili giây) sau khi chúng ta bắt đầu chương trình. Mặt khác,
chương trình này không bao giờ thoát! Thay vào đó, nó đợi mãi mãi các tin nhắn mới.
Bạn sẽ cần phải tắt nó bằng cách sử dụng <span class="keystroke">ctrl-c</span>.

Hãy bắt đầu bằng cách tìm hiểu tại sao các tin nhắn lại đến cùng một lúc sau toàn bộ thời gian trễ,
thay vì đến với các khoảng trễ giữa mỗi tin nhắn. Trong một khối async cho trước,
thứ tự xuất hiện của các từ khóa `await` trong mã cũng là thứ tự mà chúng
được thực thi khi chương trình chạy.

Chỉ có một khối async trong Liệt kê 17-10, vì vậy mọi thứ trong đó chạy
tuyến tính. Vẫn không có tính đồng thời. Tất cả các lệnh gọi `tx.send` đều xảy ra,
xen kẽ với tất cả các lệnh gọi `trpl::sleep` và các điểm await liên quan của chúng.
Chỉ khi đó, vòng lặp `while let` mới được thực hiện bất kỳ điểm `await` nào
trên các lệnh gọi `recv`.

Để có được hành vi chúng ta muốn, nơi thời gian trễ xảy ra giữa mỗi tin nhắn,
chúng ta cần đặt các thao tác `tx` và `rx` vào các khối async của riêng chúng, như được hiển thị trong
Liệt kê 17-11. Sau đó, runtime có thể thực thi từng khối một cách riêng biệt bằng cách sử dụng
`trpl::join`, giống như trong ví dụ đếm số. Một lần nữa, chúng ta await kết quả
của việc gọi `trpl::join`, chứ không phải các future riêng lẻ. Nếu chúng ta await
các future riêng lẻ theo trình tự, chúng ta sẽ chỉ kết thúc lại trong một luồng
tuần tự—chính xác là những gì chúng ta đang cố gắng _không_ làm.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="Tách biệt `send` và `recv` thành các khối `async` riêng của chúng và await các future cho các khối đó" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Với mã cập nhật trong Liệt kê 17-11, các tin nhắn được in ra theo
khoảng thời gian 500 mili giây, thay vì tất cả cùng lúc sau 2 giây.

Tuy nhiên, chương trình vẫn không bao giờ thoát, vì cách mà vòng lặp `while let`
tương tác với `trpl::join`:

- Future được trả về từ `trpl::join` chỉ hoàn thành khi _cả hai_ future
  được truyền cho nó đã hoàn thành.
- Future `tx` hoàn thành sau khi nó kết thúc việc ngủ sau khi gửi tin nhắn cuối cùng
  trong `vals`.
- Future `rx` sẽ không hoàn thành cho đến khi vòng lặp `while let` kết thúc.
- Vòng lặp `while let` sẽ không kết thúc cho đến khi await `rx.recv` tạo ra `None`.
- Await `rx.recv` sẽ trả về `None` chỉ khi đầu kia của kênh được đóng lại.
- Kênh sẽ chỉ đóng nếu chúng ta gọi `rx.close` hoặc khi phía người gửi,
  `tx`, bị hủy (dropped).
- Chúng ta không gọi `rx.close` ở bất kỳ đâu, và `tx` sẽ không bị hủy cho đến khi
  khối async ngoài cùng được truyền vào `trpl::run` kết thúc.
- Khối này không thể kết thúc vì nó bị chặn trên `trpl::join` hoàn thành,
  điều này đưa chúng ta quay lại đầu danh sách này.

Chúng ta có thể đóng `rx` thủ công bằng cách gọi `rx.close` ở đâu đó, nhưng điều đó
không có nhiều ý nghĩa. Việc dừng lại sau khi xử lý một số tin nhắn tùy ý nào đó sẽ
làm cho chương trình tắt, nhưng chúng ta có thể bỏ lỡ các tin nhắn. Chúng ta cần một cách khác
để đảm bảo rằng `tx` bị hủy _trước khi_ kết thúc hàm.

Hiện tại, khối async nơi chúng ta gửi tin nhắn chỉ mượn (borrow) `tx` bởi vì
việc gửi tin nhắn không yêu cầu quyền sở hữu, nhưng nếu chúng ta có thể chuyển (move) `tx` vào
khối async đó, nó sẽ bị hủy khi khối đó kết thúc. Trong phần của Chương 13
[Chụp tham chiếu hoặc Chuyển quyền sở hữu][capture-or-move]<!-- ignore -->, bạn đã
học cách sử dụng từ khóa `move` với closure, và như đã thảo luận trong phần của
Chương 16 [Sử dụng Closure `move` với Luồng][move-threads]<!-- ignore -->,
chúng ta thường cần chuyển dữ liệu vào các closure khi làm việc với các luồng. Cùng một động lực
cơ bản đó cũng áp dụng cho các khối async, vì vậy từ khóa `move` hoạt động với
các khối async giống như cách nó hoạt động với các closure.

Trong Liệt kê 17-12, chúng ta thay đổi khối được sử dụng để gửi tin nhắn từ `async` sang
`async move`. Khi chúng ta chạy phiên bản này của mã, nó sẽ tắt một cách duyên dáng
sau khi tin nhắn cuối cùng được gửi và nhận.

<Listing number="17-12" caption="Bản sửa đổi mã từ Liệt kê 17-11 tắt chính xác khi hoàn tất" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Kênh async này cũng là một kênh nhiều người gửi, vì vậy chúng ta có thể gọi `clone`
trên `tx` nếu chúng ta muốn gửi tin nhắn từ nhiều future, như được hiển thị trong Liệt kê
17-13.

<Listing number="17-13" caption="Sử dụng nhiều người gửi với các khối async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Đầu tiên, chúng ta clone `tx`, tạo ra `tx1` bên ngoài khối async đầu tiên. Chúng ta chuyển
`tx1` vào khối đó giống như chúng ta đã làm trước đây với `tx`. Sau đó, lát sau, chúng ta chuyển
`tx` ban đầu vào một khối async _mới_, nơi chúng ta gửi thêm các tin nhắn với một độ trễ
hơi chậm hơn. Chúng ta tình cờ đặt khối async mới này sau khối async
để nhận tin nhắn, nhưng nó cũng có thể đặt trước đó. Điều quan trọng là thứ tự
mà các future được await, chứ không phải thứ tự mà chúng được tạo ra.

Cả hai khối async để gửi tin nhắn đều cần phải là các khối `async move`
để cả `tx` và `tx1` đều bị hủy khi các khối đó kết thúc. Nếu không, chúng ta sẽ
kết thúc lại trong chính vòng lặp vô hạn mà chúng ta đã bắt đầu. Cuối cùng, chúng ta chuyển từ
`trpl::join` sang `trpl::join3` để xử lý future bổ sung.

Bây giờ chúng ta thấy tất cả các tin nhắn từ cả hai future gửi, và bởi vì các
future gửi sử dụng các độ trễ hơi khác nhau sau khi gửi, các tin nhắn cũng
được nhận ở những khoảng thời gian khác nhau đó.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Đây là một khởi đầu tốt, nhưng nó giới hạn chúng ta chỉ trong một số ít các future: hai với
`join`, hoặc ba với `join3`. Hãy xem cách chúng ta có thể làm việc với nhiều future hơn.

{{#quiz ../quizzes/async-02-concurrency-with-async.toml}}

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish-using-join-handles
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
