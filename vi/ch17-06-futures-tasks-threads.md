## Tổng kết: Futures, Tasks và Threads

Như chúng ta đã thấy trong [Chương 16][ch16]<!-- ignore -->, các luồng (threads) cung cấp một hướng tiếp cận tới
tính đồng thời. Chúng ta đã thấy một hướng tiếp cận khác trong chương này: sử dụng async với
futures và streams. Nếu bạn đang tự hỏi khi nào nên chọn phương pháp này hơn phương pháp kia,
câu trả lời là: nó tùy thuộc vào từng trường hợp! Và trong nhiều trường hợp, sự lựa chọn không phải là threads _hay_
async mà là threads _và_ async.

Nhiều hệ điều hành đã cung cấp các mô hình đồng thời dựa trên luồng trong
nhiều thập kỷ qua, và kết quả là nhiều ngôn ngữ lập trình hỗ trợ chúng. Tuy nhiên,
những mô hình này không phải là không có những sự đánh đổi. Trên nhiều hệ điều hành, chúng
sử dụng khá nhiều bộ nhớ cho mỗi luồng, và chúng đi kèm với một số chi phí (overhead) để
khởi động và tắt. Các luồng cũng chỉ là một lựa chọn khi
hệ điều hành và phần cứng của bạn hỗ trợ chúng. Không giống như các máy tính để bàn và di động
phổ biến, một số hệ thống nhúng hoàn toàn không có hệ điều hành (OS), vì vậy chúng cũng không
có các luồng.

Mô hình async cung cấp một tập hợp các sự đánh đổi khác biệt—và cuối cùng là bổ trợ—.
Trong mô hình async, các thao tác đồng thời không yêu cầu các luồng riêng của chúng.
Thay vào đó, chúng có thể chạy trên các tác vụ (tasks), như khi chúng ta sử dụng `trpl::spawn_task` để
bắt đầu công việc từ một hàm đồng bộ trong phần streams. Một task là
tương tự như một luồng, nhưng thay vì được quản lý bởi hệ điều hành, nó được
quản lý bởi mã ở cấp độ thư viện: runtime.

Trong phần trước, chúng ta thấy rằng chúng ta có thể xây dựng một stream bằng cách sử dụng một async
channel và tạo ra (spawn) một async task chúng ta có thể gọi từ mã đồng bộ. Chúng ta có thể
làm điều tương tự y hệt với một luồng. Trong Liệt kê 17-40, chúng ta đã sử dụng
`trpl::spawn_task` và `trpl::sleep`. Trong Liệt kê 17-41, chúng ta thay thế chúng bằng
các API `thread::spawn` và `thread::sleep` từ thư viện tiêu chuẩn trong
hàm `get_intervals`.

<Listing number="17-41" caption="Sử dụng các API `std::thread` thay vì các API `trpl` async cho hàm `get_intervals`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-41/src/main.rs:threads}}
```

</Listing>

Nếu bạn chạy mã này, kết quả đầu ra giống hệt với Liệt kê 17-40. Và
hãy lưu ý rằng có rất ít thay đổi ở đây từ góc độ của mã gọi. Hơn
thế nữa, ngay cả khi một trong các hàm của chúng ta đã tạo ra một async task trên runtime và
hàm kia tạo ra một luồng hệ điều hành, các streams kết quả vẫn không bị ảnh hưởng bởi
những sự khác biệt đó.

Bất chấp những điểm tương đồng của chúng, hai cách tiếp cận này hoạt động rất khác nhau,
mặc dù chúng ta có thể gặp khó khăn trong việc đo lường nó trong ví dụ rất đơn giản này. Chúng ta
có thể tạo ra hàng triệu async tasks trên bất kỳ máy tính cá nhân hiện đại nào. Nếu chúng ta cố gắng
làm điều đó với các luồng, chúng ta sẽ thực sự bị hết bộ nhớ!

Tuy nhiên, có một lý do tại sao các API này lại giống nhau đến vậy. Các luồng hoạt động như một ranh giới
cho các tập hợp các thao tác đồng bộ; tính đồng thời là có thể _giữa_ các luồng.
Các tác vụ hoạt động như một ranh giới cho các tập hợp các thao tác _bất đồng bộ_; tính đồng thời là
có thể cả _giữa_ và _trong_ các tác vụ, bởi vì một tác vụ có thể chuyển đổi giữa các
futures trong thân của nó. Cuối cùng, futures là đơn vị đồng thời chi tiết nhất của Rust,
và mỗi future có thể đại diện cho một cây của các futures khác.
Runtime—cụ thể là executor (bộ thực thi) của nó—quản lý các tasks, và các tasks quản lý các futures. Về
mặt đó, các tasks tương tự như các luồng nhẹ, do runtime quản lý với
các khả năng bổ sung đến từ việc được quản lý bởi một runtime thay vì bởi hệ điều hành.

Điều này không có nghĩa là async tasks luôn tốt hơn threads (hoặc ngược
lại). Tính đồng thời với threads theo một số cách là một mô hình lập trình đơn giản hơn
so với tính đồng thời với `async`. Đó có thể là một thế mạnh hoặc một điểm yếu. Các luồng
phần nào mang tính “kích hoạt và quên đi” (fire and forget); chúng không có khái niệm tương đương bản địa với một future, vì vậy chúng
đơn giản là chạy cho đến khi hoàn thành mà không bị ngắt quãng ngoại trừ bởi chính hệ điều hành.
Nghĩa là, chúng không có hỗ trợ tích hợp cho _tính đồng thời nội trong tác vụ_ (intratask concurrency)
theo cách các futures có. Các luồng trong Rust cũng không có cơ chế để
hủy bỏ (cancellation)—một chủ đề chúng ta chưa đề cập rõ ràng trong chương này nhưng đã được
ngầm định bởi thực tế là bất cứ khi nào chúng ta kết thúc một future, trạng thái của nó sẽ được dọn dẹp
một cách chính xác.

Những hạn chế này cũng làm cho các luồng khó kết hợp hơn các futures. Sẽ khó
hơn nhiều, ví dụ, để sử dụng các luồng để xây dựng các trình trợ giúp như các phương thức
`timeout` và `throttle` chúng ta đã xây dựng trước đó trong chương này. Thực tế là
các futures là các cấu trúc dữ liệu phong phú hơn có nghĩa là chúng có thể được kết hợp với nhau một cách
tự nhiên hơn, như chúng ta đã thấy.

Khi đó, các tác vụ cho chúng ta sự kiểm soát _bổ sung_ đối với các futures, cho phép chúng ta chọn
nơi nào và cách nào để nhóm chúng lại. Và hóa ra các luồng và các tác vụ thường làm việc
rất tốt cùng nhau, bởi vì các tác vụ có thể (ít nhất là trong một số runtimes) được di chuyển
qua lại giữa các luồng. Trong thực tế, đằng sau hậu trường, runtime mà chúng ta đã
sử dụng—bao gồm cả các hàm `spawn_blocking` và `spawn_task`—mặc định là đa luồng!
Nhiều runtimes sử dụng một phương pháp gọi là _đánh cắp công việc_ (work stealing) để
di chuyển các tác vụ qua lại giữa các luồng một cách thầm lặng, dựa trên việc các luồng đang được
sử dụng như thế nào vào lúc đó, để cải thiện hiệu suất tổng thể của hệ thống.
Cách tiếp cận đó thực sự yêu cầu các luồng _và_ các tác vụ, và do đó là các futures.

Khi suy nghĩ về việc sử dụng phương pháp nào khi nào, hãy xem xét các quy tắc kinh nghiệm (rules of thumb) sau:

- Nếu công việc là _rất có khả năng song song hóa_ (very parallelizable), chẳng hạn như xử lý một đống dữ liệu mà ở đó
  mỗi phần có thể được xử lý riêng biệt, các luồng là lựa chọn tốt hơn.
- Nếu công việc là _rất đồng thời_ (very concurrent), chẳng hạn như xử lý tin nhắn từ một đống các
  nguồn khác nhau có thể đến vào những khoảng thời gian khác nhau hoặc tốc độ khác nhau,
  async là lựa chọn tốt hơn.

Và nếu bạn cần cả tính song song và tính đồng thời, bạn không cần phải chọn
giữa luồng và async. Bạn có thể sử dụng chúng cùng nhau một cách tự do, để mỗi cái
đóng vai trò mà nó giỏi nhất. Ví dụ, Liệt kê 17-42 cho thấy một ví dụ khá phổ biến
của loại kết hợp này trong mã Rust thực tế.

<Listing number="17-42" caption="Gửi tin nhắn với mã gây chặn trong một luồng và await các tin nhắn trong một khối async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-42/src/main.rs:all}}
```

</Listing>

Chúng ta bắt đầu bằng cách tạo một kênh async, sau đó tạo ra một luồng (spawn a thread) lấy
quyền sở hữu phía người gửi (sender side) của kênh. Trong luồng đó, chúng ta gửi các
số từ 1 đến 10, ngủ một giây giữa mỗi lần. Cuối cùng, chúng ta chạy một
future được tạo ra bằng một khối async được truyền vào `trpl::run` giống như chúng ta đã làm
suốt chương này. Trong future đó, chúng ta await những tin nhắn đó, giống như trong
các ví dụ truyền tin nhắn khác mà chúng ta đã thấy.

Để quay lại kịch bản mà chúng ta đã mở đầu chương, hãy tưởng tượng việc chạy một bộ các
tác vụ mã hóa video sử dụng một luồng chuyên dụng (bởi vì mã hóa video là
compute-bound) nhưng thông báo cho giao diện người dùng rằng những thao tác đó đã hoàn thành bằng một
kênh async. Có vô số ví dụ về những loại kết hợp này trong
các trường hợp sử dụng thực tế.

## Tóm tắt

Đây không phải là lần cuối cùng bạn thấy tính đồng thời trong cuốn sách này. Dự án trong
[Chương 21][ch21] sẽ áp dụng các khái niệm này trong một tình huống thực tế hơn
so với các ví dụ đơn giản hơn được thảo luận ở đây và so sánh việc giải quyết vấn đề với luồng so với tác vụ trực tiếp hơn.

Dù bạn chọn cách tiếp cận nào trong số này, Rust cung cấp cho bạn các công cụ cần thiết để viết mã đồng thời
an toàn và nhanh chóng—cho dù dành cho một máy chủ web thông lượng cao hay một hệ điều hành nhúng.

Tiếp theo, chúng ta sẽ nói về những cách viết mã chuẩn mực (idiomatic ways) để mô hình hóa các vấn đề và cấu trúc các giải pháp
khi các chương trình Rust của bạn lớn dần lên. Ngoài ra, chúng ta sẽ thảo luận về cách các thành ngữ (idioms) của Rust
liên quan đến những thứ bạn có thể đã quen thuộc từ lập trình hướng đối tượng.

[ch16]: http://localhost:3000/ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
