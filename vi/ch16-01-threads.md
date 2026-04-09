## Sử dụng các luồng để chạy mã đồng thời

Trong hầu hết các hệ điều hành hiện nay, mã của một chương trình đang thực thi được chạy trong một
_tiến trình_ (process), và hệ điều hành sẽ quản lý nhiều tiến trình cùng một lúc.
Bên trong một chương trình, bạn cũng có thể có các phần độc lập chạy đồng thời.
Các tính năng chạy những phần độc lập này được gọi là các _luồng_ (threads). Ví dụ,
một máy chủ web có thể có nhiều luồng để nó có thể phản hồi
nhiều hơn một yêu cầu cùng một lúc.

Việc chia nhỏ tính toán trong chương trình của bạn thành nhiều luồng để chạy nhiều
tác vụ cùng lúc có thể cải thiện hiệu suất, nhưng nó cũng làm tăng thêm sự phức tạp.
Bởi vì các luồng có thể chạy đồng thời, nên không có sự đảm bảo vốn có nào về
thứ tự mà các phần mã của bạn trên các luồng khác nhau sẽ chạy. Điều này có thể dẫn đến
các vấn đề, chẳng hạn như:

- Tình trạng tranh đua (Race conditions), trong đó các luồng đang truy cập dữ liệu hoặc tài nguyên theo một
  thứ tự không nhất quán
- Bế tắc (Deadlocks), trong đó hai luồng đang chờ đợi lẫn nhau, ngăn cản cả hai
  luồng tiếp tục
- Các lỗi chỉ xảy ra trong những tình huống nhất định và khó tái hiện cũng như sửa chữa
  một cách đáng tin cậy

Rust cố gắng giảm thiểu các tác động tiêu cực của việc sử dụng các luồng, nhưng
việc lập trình trong ngữ cảnh đa luồng vẫn cần sự suy nghĩ cẩn thận và yêu cầu
một cấu trúc mã khác với các chương trình chạy trong một
luồng duy nhất.

Các ngôn ngữ lập trình triển khai các luồng theo một vài cách khác nhau, và nhiều
hệ điều hành cung cấp một API mà ngôn ngữ có thể gọi để tạo các luồng mới.
Thư viện tiêu chuẩn của Rust sử dụng mô hình triển khai luồng _1:1_, theo đó một
chương trình sử dụng một luồng hệ điều hành cho mỗi một luồng ngôn ngữ. Có những
crate triển khai các mô hình luồng khác có những sự đánh đổi khác nhau đối với
mô hình 1:1. (Hệ thống async của Rust, mà chúng ta sẽ thấy trong chương tiếp theo,
cũng cung cấp một cách tiếp cận khác cho concurrency.)

### Tạo một luồng mới với `spawn`

Để tạo một luồng mới, chúng ta gọi hàm `thread::spawn` và truyền cho nó một
closure (chúng ta đã nói về closure trong Chương 13) chứa mã chúng ta muốn
chạy trong luồng mới. Ví dụ trong Listing 16-1 in ra một số văn bản từ một luồng chính
và văn bản khác từ một luồng mới:

<Listing number="16-1" file-name="src/main.rs" caption="Tạo một luồng mới để in một thứ trong khi luồng chính in thứ khác">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Lưu ý rằng khi luồng chính của một chương trình Rust hoàn thành, tất cả các luồng được tạo (spawned)
đều bị tắt, cho dù chúng đã chạy xong hay chưa. Đầu ra từ chương trình
này có thể hơi khác nhau mỗi lần chạy, nhưng nó sẽ trông tương tự như
sau đây:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Các lệnh gọi `thread::sleep` buộc một luồng phải dừng thực thi trong một khoảng thời gian
ngắn, cho phép một luồng khác chạy. Các luồng có thể sẽ thay
phiên nhau, nhưng điều đó không được đảm bảo: nó phụ thuộc vào cách hệ điều hành của bạn
lập trình cho các luồng. Trong lần chạy này, luồng chính đã in trước, mặc dù
câu lệnh in từ luồng được tạo xuất hiện trước trong mã. Và thậm chí
mặc dù chúng ta đã bảo luồng được tạo in cho đến khi `i` là `9`, nó chỉ đạt tới `5`
trước khi luồng chính tắt.

Nếu bạn chạy mã này và chỉ thấy đầu ra từ luồng chính, hoặc không thấy bất kỳ
sự chồng chéo nào, hãy thử tăng các số trong phạm vi để tạo ra nhiều cơ hội hơn
cho hệ điều hành chuyển đổi giữa các luồng.

### Chờ tất cả các luồng kết thúc bằng cách sử dụng các handle `join`

Mã trong Listing 16-1 không chỉ dừng luồng được tạo sớm trong hầu hết
thời gian do luồng chính kết thúc, mà còn vì không có sự đảm bảo về
thứ tự các luồng chạy, chúng ta cũng không thể đảm bảo rằng luồng được tạo
sẽ được chạy chút nào!

Chúng ta có thể khắc phục vấn đề luồng được tạo không chạy hoặc kết thúc sớm
bằng cách lưu giá trị trả về của `thread::spawn` vào một biến. Kiểu trả về của
`thread::spawn` là `JoinHandle<T>`. Một `JoinHandle<T>` là một giá trị được sở hữu mà,
khi chúng ta gọi phương thức `join` trên nó, sẽ đợi cho luồng của nó kết thúc.
Listing 16-2 cho thấy cách sử dụng `JoinHandle<T>` của luồng chúng ta đã tạo trong
Listing 16-1 và cách gọi `join` để đảm bảo luồng được tạo kết thúc
trước khi `main` thoát.

<Listing number="16-2" file-name="src/main.rs" caption="Lưu một `JoinHandle<T>` từ `thread::spawn` để đảm bảo luồng được chạy đến khi hoàn tất">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Việc gọi `join` trên handle sẽ chặn (blocks) luồng hiện đang chạy cho đến khi
luồng được đại diện bởi handle đó kết thúc. _Chặn_ một luồng có nghĩa là
luồng đó bị ngăn cản thực hiện công việc hoặc thoát. Bởi vì chúng ta đã đặt lời gọi
`join` sau vòng lặp `for` của luồng chính, việc chạy Listing 16-2 sẽ
tạo ra đầu ra tương tự như thế này:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Hai luồng tiếp tục xen kẽ nhau, nhưng luồng chính chờ đợi vì
lời gọi `handle.join()` và không kết thúc cho đến khi luồng được tạo hoàn thành.

Nhưng hãy xem điều gì xảy ra khi thay vào đó chúng ta di chuyển `handle.join()` ra trước
vòng lặp `for` trong `main`, như thế này:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

Luồng chính sẽ đợi luồng được tạo kết thúc và sau đó mới chạy vòng lặp
`for` của nó, vì vậy đầu ra sẽ không còn xen kẽ nữa, như được hiển thị ở đây:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Các chi tiết nhỏ, chẳng hạn như nơi `join` được gọi, có thể ảnh hưởng đến việc liệu các
luồng của bạn có chạy cùng lúc hay không.

### Sử dụng closure `move` với các luồng

Chúng ta sẽ thường xuyên sử dụng từ khóa `move` với các closure được truyền cho `thread::spawn`
bởi vì closure sau đó sẽ nắm quyền sở hữu các giá trị mà nó sử dụng từ
môi trường, do đó chuyển quyền sở hữu của những giá trị đó từ luồng này sang
luồng khác. Trong phần [“Bắt giữ môi trường với Closure”][capture]<!-- ignore -->
trong Chương 13, chúng ta đã thảo luận về `move` trong ngữ cảnh của các closure. Bây giờ, chúng ta sẽ
tập trung nhiều hơn vào sự tương tác giữa `move` và `thread::spawn`.

Lưu ý trong Listing 16-1 rằng closure chúng ta truyền cho `thread::spawn` không nhận
đối số nào: chúng ta không sử dụng bất kỳ dữ liệu nào từ luồng chính trong mã của luồng
được tạo. Để sử dụng dữ liệu từ luồng chính trong luồng được tạo,
closure của luồng được tạo phải bắt giữ (capture) các giá trị nó cần. Listing 16-3 cho thấy
một nỗ lực tạo một vector trong luồng chính và sử dụng nó trong luồng
được tạo. Tuy nhiên, điều này sẽ chưa hoạt động, như bạn sẽ thấy trong giây lát.

<Listing number="16-3" file-name="src/main.rs" caption="Cố gắng sử dụng một vector được tạo bởi luồng chính trong một luồng khác">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

Closure sử dụng `v`, vì vậy nó sẽ bắt giữ `v` và biến nó thành một phần của môi trường
của closure. Bởi vì `thread::spawn` chạy closure này trong một luồng mới, chúng ta
đáng lẽ phải có thể truy cập `v` bên trong luồng mới đó. Nhưng khi chúng ta biên dịch
ví dụ này, chúng ta nhận được lỗi sau:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust _suy luận_ cách bắt giữ `v`, và vì `println!` chỉ cần một tham chiếu
đến `v`, closure cố gắng mượn `v`. Tuy nhiên, có một vấn đề: Rust không thể
biết luồng được tạo sẽ chạy trong bao lâu, vì vậy nó không biết liệu
tham chiếu đến `v` có luôn hợp lệ hay không.

Listing 16-4 cung cấp một kịch bản có nhiều khả năng có một tham chiếu đến `v`
sẽ không hợp lệ:

<Listing number="16-4" file-name="src/main.rs" caption="Một luồng với một closure cố gắng bắt giữ một tham chiếu đến `v` từ một luồng chính mà thực hiện drop `v` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Nếu Rust cho phép chúng ta chạy mã này, có khả năng luồng được tạo
sẽ ngay lập tức được đưa vào chạy ngầm mà không chạy chút nào.
Luồng được tạo có một tham chiếu đến `v` bên trong, nhưng luồng chính ngay lập tức
hủy (drops) `v`, sử dụng hàm `drop` chúng ta đã thảo luận trong Chương 15. Sau đó, khi
luồng được tạo bắt đầu thực thi, `v` không còn hợp lệ nữa, vì vậy một tham chiếu đến nó
cũng không hợp lệ. Ôi không!

Để sửa lỗi trình biên dịch trong Listing 16-3, chúng ta có thể sử dụng lời khuyên của
thông báo lỗi:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Bằng cách thêm từ khóa `move` trước closure, chúng ta buộc closure phải lấy
quyền sở hữu các giá trị mà nó đang sử dụng thay vì để Rust suy luận rằng nó
nên mượn các giá trị đó. Sự sửa đổi đối với Listing 16-3 được hiển thị trong Listing
16-5 sẽ biên dịch và chạy như chúng ta mong muốn.

<Listing number="16-5" file-name="src/main.rs" caption="Sử dụng từ khóa `move` để buộc một closure lấy quyền sở hữu các giá trị nó sử dụng">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Chúng ta có thể bị cám dỗ để thử điều tương tự nhằm sửa mã trong Listing 16-4 nơi
luồng chính đã gọi `drop` bằng cách sử dụng một closure `move`. Tuy nhiên, cách sửa này sẽ
không hoạt động vì những gì Listing 16-4 đang cố gắng thực hiện bị cấm vì một
lý do khác. Nếu chúng ta thêm `move` vào closure, chúng ta sẽ di chuyển `v` vào trong
môi trường của closure, và chúng ta không còn có thể gọi `drop` trên nó trong luồng
chính nữa. Thay vào đó chúng ta sẽ nhận được lỗi trình biên dịch này:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Các quy tắc sở hữu của Rust đã cứu chúng ta một lần nữa! Chúng ta đã nhận được lỗi từ mã trong
Listing 16-3 vì Rust đã thận trọng và chỉ mượn `v` cho
luồng, điều đó có nghĩa là luồng chính về mặt lý thuyết có thể làm mất hiệu lực tham chiếu của
luồng được tạo. Bằng cách bảo Rust di chuyển quyền sở hữu `v` sang luồng được
tạo, chúng ta đang đảm bảo với Rust rằng luồng chính sẽ không sử dụng `v` nữa.
Nếu chúng ta thay đổi Listing 16-4 theo cùng một cách, thì chúng ta đang vi phạm các quy tắc
sở hữu khi chúng ta cố gắng sử dụng `v` trong luồng chính. Từ khóa `move` ghi đè
mặc định mượn thận trọng của Rust; nó không cho phép chúng ta vi phạm các
quy tắc sở hữu.

Bây giờ chúng ta đã tìm hiểu luồng là gì và các phương thức được cung cấp bởi API
luồng, hãy xem xét một số tình huống mà chúng ta có thể sử dụng các luồng.

{{#quiz ../quizzes/ch16-01-threads.toml}}

[capture]: ch13-01-closures.html#capturing-the-environment-with-closures
