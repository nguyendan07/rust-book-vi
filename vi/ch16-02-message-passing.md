## Sử dụng truyền thông điệp để chuyển dữ liệu giữa các luồng
(Using Message Passing to Transfer Data Between Threads)

Một cách tiếp cận ngày càng phổ biến để đảm bảo concurrency an toàn là _truyền thông
điệp_ (message passing), nơi các luồng hoặc actor giao tiếp bằng cách gửi cho nhau các thông điệp
chứa dữ liệu. Đây là ý tưởng trong một câu khẩu hiệu từ [tài liệu ngôn ngữ Go](https://golang.org/doc/effective_go.html#concurrency):
“Đừng giao tiếp bằng cách chia sẻ bộ nhớ; thay vào đó, hãy chia sẻ bộ nhớ bằng cách giao tiếp.”

Để thực hiện concurrency gửi thông điệp, thư viện tiêu chuẩn của Rust cung cấp một
triển khai của các kênh (channels). Một _kênh_ là một khái niệm lập trình chung mà
qua đó dữ liệu được gửi từ luồng này sang luồng khác.

Bạn có thể tưởng tượng một kênh trong lập trình giống như một kênh nước có hướng,
chẳng hạn như một con suối hoặc một con sông. Nếu bạn đặt một thứ gì đó như một con vịt cao su
vào một con sông, nó sẽ trôi xuôi dòng đến cuối đường thủy.

Một kênh có hai nửa: một bộ truyền (transmitter) và một bộ nhận (receiver). Nửa bộ truyền là
vị trí thượng nguồn nơi bạn đặt con vịt cao su vào sông, và nửa
bộ nhận là nơi con vịt cao su kết thúc ở hạ lưu. Một phần mã của
bạn gọi các phương thức trên bộ truyền với dữ liệu bạn muốn gửi, và
một phần khác kiểm tra đầu nhận để tìm các thông điệp đang đến. Một kênh được gọi là
bị _đóng_ (closed) nếu một trong hai nửa bộ truyền hoặc bộ nhận bị hủy (dropped).

Tại đây, chúng ta sẽ xây dựng một chương trình có một luồng để tạo ra các giá trị và
gửi chúng xuống một kênh, và một luồng khác sẽ nhận các giá trị đó và
in chúng ra. Chúng ta sẽ gửi các giá trị đơn giản giữa các luồng bằng cách sử dụng một kênh
để minh họa tính năng này. Khi bạn đã quen với kỹ thuật này, bạn có thể
sử dụng các kênh cho bất kỳ luồng nào cần giao tiếp với nhau, chẳng hạn như
một hệ thống trò chuyện hoặc một hệ thống nơi nhiều luồng thực hiện các phần của một phép tính và
gửi các phần đó đến một luồng để tổng hợp kết quả.

Đầu tiên, trong Listing 16-6, chúng ta sẽ tạo một kênh nhưng chưa làm gì với nó.
Lưu ý rằng điều này sẽ chưa biên dịch được vì Rust không thể biết loại giá trị nào chúng ta
muốn gửi qua kênh.

<Listing number="16-6" file-name="src/main.rs" caption="Tạo một kênh và gán hai nửa cho `tx` và `rx` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

Chúng ta tạo một kênh mới bằng hàm `mpsc::channel`; `mpsc` là viết tắt của
_nhiều nhà sản xuất, một người tiêu dùng_ (multiple producer, single consumer). Tóm lại, cách thư viện tiêu chuẩn của Rust
triển khai các kênh có nghĩa là một kênh có thể có nhiều đầu _gửi_
tạo ra các giá trị nhưng chỉ có một đầu _nhận_ tiêu thụ các giá trị đó. Hãy tưởng tượng
nhiều dòng suối cùng chảy vào một con sông lớn: mọi thứ được gửi xuống bất kỳ
dòng suối nào cũng sẽ kết thúc ở một con sông duy nhất. Chúng ta sẽ bắt đầu với một nhà sản xuất
duy nhất vào lúc này, nhưng chúng ta sẽ thêm nhiều nhà sản xuất khi ví dụ này
hoạt động.

Hàm `mpsc::channel` trả về một tuple, phần tử đầu tiên trong đó là
đầu gửi—bộ truyền—và phần tử thứ hai là đầu nhận—
bộ nhận. Các từ viết tắt `tx` và `rx` theo truyền thống được sử dụng trong nhiều
lĩnh vực tương ứng cho _transmitter_ (bộ truyền) và _receiver_ (bộ nhận), vì vậy chúng ta đặt tên các biến của mình
như vậy để chỉ ra mỗi đầu. Chúng ta đang sử dụng một câu lệnh `let` với một pattern thực hiện
giải cấu trúc (destructures) các tuple; chúng ta sẽ thảo luận về việc sử dụng các pattern trong câu lệnh `let`
và giải cấu trúc trong Chương 19. Hiện tại, hãy biết rằng việc sử dụng câu lệnh `let` theo cách này
là một cách tiếp cận thuận tiện để trích xuất các phần của tuple được trả về bởi
`mpsc::channel`.

Hãy di chuyển đầu truyền vào một luồng được tạo và để nó gửi một
chuỗi để luồng được tạo giao tiếp với luồng chính, như được hiển thị trong
Listing 16-7. Điều này giống như việc đặt một con vịt cao su lên thượng nguồn con sông hoặc
gửi một tin nhắn trò chuyện từ luồng này sang luồng khác.

<Listing number="16-7" file-name="src/main.rs" caption='Di chuyển `tx` sang một luồng được tạo và gửi `"hi"` '>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Một lần nữa, chúng ta sử dụng `thread::spawn` để tạo một luồng mới và sau đó sử dụng `move`
để di chuyển `tx` vào closure để luồng được tạo sở hữu `tx`. Luồng được
tạo cần sở hữu bộ truyền để có thể gửi thông điệp qua
kênh.

Bộ truyền có một phương thức `send` nhận giá trị chúng ta muốn gửi. Phương thức
`send` trả về một kiểu `Result<T, E>`, vì vậy nếu bộ nhận đã
bị hủy và không còn nơi nào để gửi giá trị, thao tác gửi sẽ
trả về một lỗi. Trong ví dụ này, chúng ta đang gọi `unwrap` để gây hoảng loạn (panic) trong trường hợp có
lỗi. Nhưng trong một ứng dụng thực tế, chúng ta sẽ xử lý nó một cách thích hợp: hãy quay lại
Chương 9 để xem lại các chiến lược xử lý lỗi thích hợp.

Trong Listing 16-8, chúng ta sẽ lấy giá trị từ bộ nhận trong luồng chính. Điều này
giống như việc lấy con vịt cao su ra khỏi nước ở cuối con sông hoặc
nhận một tin nhắn trò chuyện.

<Listing number="16-8" file-name="src/main.rs" caption='Nhận giá trị `"hi"` trong luồng chính và in nó ra'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

Bộ nhận có hai phương thức hữu ích: `recv` và `try_recv`. Chúng ta đang sử dụng `recv`,
viết tắt của _receive_ (nhận), nó sẽ chặn việc thực thi của luồng chính và đợi
cho đến khi một giá trị được gửi xuống kênh. Khi một giá trị được gửi, `recv` sẽ
trả về nó trong một `Result<T, E>`. Khi bộ truyền đóng lại, `recv` sẽ trả về
một lỗi để báo hiệu rằng sẽ không còn giá trị nào đến nữa.

Phương thức `try_recv` không chặn, mà thay vào đó sẽ trả về một `Result<T, E>`
ngay lập tức: một giá trị `Ok` chứa thông điệp nếu có sẵn và một giá trị `Err`
nếu lần này không có bất kỳ thông điệp nào. Sử dụng `try_recv` rất hữu ích nếu
luồng này có công việc khác phải làm trong khi chờ đợi thông điệp: chúng ta có thể viết một
vòng lặp gọi `try_recv` thường xuyên, xử lý thông điệp nếu có,
và ngược lại thì làm công việc khác trong một lúc cho đến khi kiểm tra
lại.

Chúng ta đã sử dụng `recv` trong ví dụ này cho đơn giản; chúng ta không có bất kỳ công việc nào khác
cho luồng chính làm ngoài việc chờ đợi thông điệp, vì vậy việc chặn luồng
chính là phù hợp.

Khi chúng ta chạy mã trong Listing 16-8, chúng ta sẽ thấy giá trị được in từ luồng
chính:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Hoàn hảo!

### Các kênh và sự chuyển giao quyền sở hữu

Các quy tắc sở hữu đóng một vai trò quan trọng trong việc gửi thông điệp vì chúng giúp bạn
viết mã đồng thời an toàn. Ngăn chặn các lỗi trong lập trình đồng thời là
lợi thế của việc suy nghĩ về quyền sở hữu xuyên suốt các chương trình Rust của bạn. Hãy làm
một thử nghiệm để xem các kênh và quyền sở hữu hoạt động cùng nhau như thế nào để ngăn chặn
các vấn đề: chúng ta sẽ thử sử dụng một giá trị `val` trong luồng được tạo _sau khi_ chúng ta đã
gửi nó xuống kênh. Hãy thử biên dịch mã trong Listing 16-9 để xem tại sao
mã này không được phép.

<Listing number="16-9" file-name="src/main.rs" caption="Cố gắng sử dụng `val` sau khi chúng ta đã gửi nó xuống kênh">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Ở đây, chúng ta cố gắng in `val` sau khi chúng ta đã gửi nó xuống kênh qua `tx.send`.
Cho phép điều này sẽ là một ý tưởng tồi: một khi giá trị đã được gửi đến một luồng
khác, luồng đó có thể sửa đổi hoặc hủy nó trước khi chúng ta cố gắng sử dụng lại giá trị
đó. Tiềm ẩn khả năng, những sửa đổi của luồng khác có thể gây ra lỗi hoặc
kết quả không mong muốn do dữ liệu không nhất quán hoặc không tồn tại. Tuy nhiên, Rust đưa
ra một lỗi nếu chúng ta cố gắng biên dịch mã trong Listing 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Sai lầm về concurrency của chúng ta đã gây ra một lỗi thời điểm biên dịch. Hàm `send`
lấy quyền sở hữu tham số của nó, và khi giá trị được di chuyển, bộ nhận
lấy quyền sở hữu nó. Điều này ngăn chúng ta vô tình sử dụng lại giá trị
sau khi gửi nó; hệ thống sở hữu kiểm tra xem mọi thứ có ổn không.

### Gửi nhiều giá trị và thấy bộ nhận đang chờ

Mã trong Listing 16-8 đã biên dịch và chạy, nhưng nó không cho chúng ta thấy rõ ràng rằng
hai luồng riêng biệt đang nói chuyện với nhau qua kênh. Trong Listing
16-10 chúng ta đã thực hiện một số sửa đổi sẽ chứng minh mã trong Listing 16-8 đang
chạy đồng thời: luồng được tạo bây giờ sẽ gửi nhiều thông điệp và
tạm dừng một giây giữa mỗi thông điệp.

<Listing number="16-10" file-name="src/main.rs" caption="Gửi nhiều thông điệp và tạm dừng giữa mỗi thông điệp">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Lần này, luồng được tạo có một vector các chuỗi mà chúng ta muốn gửi đến
luồng chính. Chúng ta lặp qua chúng, gửi từng cái riêng lẻ, và tạm dừng
giữa mỗi lần bằng cách gọi hàm `thread::sleep` với một giá trị `Duration` là
một giây.

Trong luồng chính, chúng ta không gọi hàm `recv` một cách rõ ràng nữa:
thay vào đó, chúng ta coi `rx` như một iterator. Đối với mỗi giá trị nhận được, chúng ta
in nó ra. Khi kênh bị đóng, việc lặp sẽ kết thúc.

Khi chạy mã trong Listing 16-10, bạn sẽ thấy đầu ra sau đây
với một giây tạm dừng giữa mỗi dòng:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Bởi vì chúng ta không có bất kỳ mã nào tạm dừng hoặc trì hoãn trong vòng lặp `for` ở
luồng chính, chúng ta có thể thấy rằng luồng chính đang chờ để nhận các giá trị từ
luồng được tạo.

### Tạo nhiều nhà sản xuất bằng cách nhân bản bộ truyền

Trước đó chúng ta đã đề cập rằng `mpsc` là viết tắt của _nhiều nhà sản xuất,
một người tiêu dùng_. Hãy đưa `mpsc` vào sử dụng và mở rộng mã trong Listing 16-10
để tạo nhiều luồng mà tất cả đều gửi giá trị đến cùng một bộ nhận. Chúng ta có thể làm
như vậy bằng cách nhân bản (cloning) bộ truyền, như được hiển thị trong Listing 16-11.

<Listing number="16-11" file-name="src/main.rs" caption="Gửi nhiều thông điệp từ nhiều nhà sản xuất">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Lần này, trước khi chúng ta tạo luồng đầu tiên, chúng ta gọi `clone` trên
bộ truyền. Điều này sẽ cung cấp cho chúng ta một bộ truyền mới mà chúng ta có thể truyền cho
luồng được tạo đầu tiên. Chúng ta truyền bộ truyền ban đầu cho luồng được tạo thứ hai.
Điều này cung cấp cho chúng ta hai luồng, mỗi luồng gửi các thông điệp khác nhau đến một bộ nhận duy nhất.

Khi bạn chạy mã, đầu ra của bạn sẽ trông giống như thế này:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Bạn có thể thấy các giá trị theo một thứ tự khác, tùy thuộc vào hệ thống của bạn. Đây là
điều làm cho concurrency trở nên thú vị cũng như khó khăn. Nếu bạn thử nghiệm với
`thread::sleep`, cung cấp cho nó các giá trị khác nhau trong các luồng khác nhau, mỗi lần chạy
sẽ mang tính bất định (nondeterministic) hơn và tạo ra đầu ra khác nhau mỗi lần.

Bây giờ chúng ta đã xem xét cách thức hoạt động của các kênh, hãy xem xét một phương pháp khác của
concurrency.

{{#quiz ../quizzes/ch16-02-message-passing.toml}}
