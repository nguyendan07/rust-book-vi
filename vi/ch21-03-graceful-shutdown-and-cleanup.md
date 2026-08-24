## Dừng Hoạt Động Nhịp Nhàng (Graceful Shutdown) và Dọn Dẹp

Mã trong Listing 21-20 đang phản hồi các request một cách bất đồng bộ thông qua
việc sử dụng thread pool, đúng như chúng ta mong muốn. Chúng ta nhận được một số
cảnh báo về các trường `workers`, `id`, và `thread` mà chúng ta không sử dụng một cách
trực tiếp, điều này nhắc nhở chúng ta rằng mình chưa dọn dẹp bất kỳ thứ gì. Khi chúng
ta sử dụng phương pháp kém tinh tế là nhấn <kbd>ctrl</kbd>-<kbd>c</kbd> để dừng
luồng chính, tất cả các luồng khác cũng bị dừng ngay lập tức, ngay cả khi chúng đang
trong quá trình phục vụ một request.

Tiếp theo, chúng ta sẽ triển khai trait `Drop` để gọi `join` trên từng luồng trong
pool nhằm đảm bảo chúng có thể hoàn thành các request đang xử lý trước khi đóng. Sau
đó, chúng ta sẽ triển khai một cách để báo cho các luồng biết rằng chúng nên ngừng
nhận các request mới và tắt đi. Để thấy mã này hoạt động trong thực tế, chúng ta sẽ
sửa đổi server của mình để chỉ chấp nhận hai request trước khi tắt thread pool một
cách nhịp nhàng (gracefully).

Một điều cần lưu ý trong quá trình thực hiện: không có điều nào trong số này ảnh
hưởng đến các phần mã xử lý việc thực thi các closure, vì vậy mọi thứ ở đây sẽ hoàn
toàn giống nhau nếu chúng ta sử dụng một thread pool cho một async runtime.

### Triển Khai Trait `Drop` Cho `ThreadPool`

Hãy bắt đầu với việc triển khai `Drop` cho thread pool của chúng ta. Khi pool bị
drop, tất cả các luồng của chúng ta nên join để đảm bảo chúng hoàn thành công việc
của mình. Listing 21-22 hiển thị thử nghiệm đầu tiên về việc triển khai `Drop`; mã
này vẫn chưa hoạt động được.

<Listing number="21-22" file-name="src/lib.rs" caption="Join từng luồng khi thread pool đi ra ngoài phạm vi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Đầu tiên, chúng ta lặp qua từng phần tử trong `workers` của thread pool. Chúng ta
sử dụng `&mut` cho việc này vì `self` là một tham chiếu khả biến (mutable reference),
và chúng ta cũng cần có khả năng thay đổi `worker`. Đối với mỗi worker, chúng ta in
một thông báo cho biết instance `Worker` cụ thể này đang tắt, và sau đó chúng ta gọi
`join` trên luồng của instance `Worker` đó. Nếu lệnh gọi `join` thất bại, chúng ta
sử dụng `unwrap` để làm cho Rust panic và chuyển sang trạng thái tắt không an toàn
(ungraceful shutdown).

Dưới đây là lỗi chúng ta nhận được khi biên dịch mã này:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Lỗi cho chúng ta biết rằng chúng ta không thể gọi `join` vì chúng ta chỉ có một
mutable borrow của từng `worker` và `join` nhận quyền sở hữu (ownership) của đối số
của nó. Để giải quyết vấn đề này, chúng ta cần di chuyển luồng ra khỏi instance
`Worker` sở hữu `thread` để `join` có thể tiêu thụ (consume) luồng đó. Một cách để
làm điều này là áp dụng cách tiếp cận tương tự như chúng ta đã làm trong Listing
18-15. Nếu `Worker` giữ một `Option<thread::JoinHandle<()>>`, chúng ta có thể gọi
phương thức `take` trên `Option` để di chuyển giá trị ra khỏi biến thể `Some` và để
lại biến thể `None` vào vị trí của nó. Nói cách khác, một `Worker` đang chạy sẽ có
biến thể `Some` trong `thread`, và khi chúng ta muốn dọn dẹp một `Worker`, chúng ta
sẽ thay thế `Some` bằng `None` để `Worker` không còn luồng nào để chạy nữa.

Tuy nhiên, thời điểm _duy nhất_ điều này xuất hiện là khi drop `Worker`. Đổi lại,
chúng ta sẽ phải xử lý `Option<thread::JoinHandle<()>>` ở bất kỳ nơi nào chúng ta
truy cập `worker.thread`. Mã Rust chuẩn mực (idiomatic Rust) sử dụng `Option` khá
nhiều, nhưng khi bạn thấy mình đang bọc một thứ mà bạn biết chắc sẽ luôn tồn tại vào
`Option` như một giải pháp tình thế (workaround) như thế này, bạn nên tìm kiếm các
phương pháp tiếp cận thay thế. Chúng có thể làm cho mã của bạn sạch hơn và ít bị lỗi
hơn.

Trong trường hợp này, tồn tại một giải pháp thay thế tốt hơn: phương thức
`Vec::drain`. Nó chấp nhận một tham số phạm vi (range) để chỉ định những phần tử nào
cần xóa khỏi `Vec`, và trả về một iterator của những phần tử đó. Truyền cú pháp
phạm vi `..` sẽ xóa _mọi_ giá trị khỏi `Vec`.

Vì vậy, chúng ta cần cập nhật bản triển khai `drop` của `ThreadPool` như sau:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Điều này giải quyết được lỗi trình biên dịch và không yêu cầu bất kỳ thay đổi nào
khác đối với mã của chúng ta.

### Báo Hiệu Cho Các Luồng Dừng Lắng Nghe Các Job

Với tất cả những thay đổi đã thực hiện, mã của chúng ta biên dịch mà không có bất
kỳ cảnh báo nào. Tuy nhiên, tin xấu là mã này vẫn chưa hoạt động theo cách chúng ta
mong muốn. Mấu chốt nằm ở logic trong các closure được chạy bởi các luồng của các
instance `Worker`: tại thời điểm này, chúng ta gọi `join`, nhưng điều đó sẽ không tắt
các luồng vì chúng `loop` (lặp) vô tận để tìm kiếm job. Nếu chúng ta cố gắng drop
`ThreadPool` với bản triển khai `drop` hiện tại, luồng chính sẽ bị chặn (block) mãi
mãi để đợi luồng đầu tiên hoàn thành.

Để khắc phục vấn đề này, chúng ta sẽ cần thay đổi trong bản triển khai `drop` của
`ThreadPool` và sau đó thay đổi trong vòng lặp của `Worker`.

Trước tiên, chúng ta sẽ thay đổi bản triển khai `drop` của `ThreadPool` để drop
`sender` một cách rõ ràng (explicitly) trước khi đợi các luồng hoàn thành. Listing
21-23 hiển thị các thay đổi đối với `ThreadPool` để drop `sender` một cách rõ ràng.
Không giống như với luồng, ở đây chúng ta _thực sự_ cần sử dụng `Option` để có thể di
chuyển `sender` ra khỏi `ThreadPool` bằng `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="Drop `sender` một cách rõ ràng trước khi join các luồng `Worker`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Việc drop `sender` sẽ đóng channel, báo hiệu rằng sẽ không có thêm thông điệp nào
được gửi nữa. Khi điều đó xảy ra, tất cả các lệnh gọi tới `recv` mà các instance
`Worker` thực hiện trong vòng lặp vô hạn sẽ trả về một lỗi. Trong Listing 21-24,
chúng ta thay đổi vòng lặp của `Worker` để thoát khỏi vòng lặp một cách nhẹ nhàng trong
trường hợp đó, điều này có nghĩa là các luồng sẽ kết thúc khi bản triển khai `drop`
của `ThreadPool` gọi `join` trên chúng.

<Listing number="21-24" file-name="src/lib.rs" caption="Thoát khỏi vòng lặp một cách rõ ràng khi `recv` trả về một lỗi">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Để thấy mã này hoạt động, hãy sửa đổi `main` để chỉ chấp nhận hai request trước khi
tắt server một cách nhịp nhàng, như được hiển thị trong Listing 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Tắt server sau khi phục vụ hai request bằng cách thoát khỏi vòng lặp">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Bạn sẽ không muốn một web server trong thế giới thực tắt sau khi chỉ phục vụ hai
request. Mã này chỉ nhằm chứng minh rằng quá trình graceful shutdown và cleanup đang
hoạt động bình thường.

Phương thức `take` được định nghĩa trong trait `Iterator` và giới hạn số lần lặp tối
đa ở hai phần tử đầu tiên. `ThreadPool` sẽ đi ra ngoài phạm vi ở cuối hàm `main`, và
bản triển khai `drop` sẽ chạy.

Khởi động server bằng `cargo run`, và thực hiện ba request. Request thứ ba sẽ bị
lỗi, và trong terminal của bạn sẽ thấy đầu ra tương tự như thế này:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Bạn có thể thấy thứ tự các ID của `Worker` và các thông báo được in ra có chút khác
biệt. Chúng ta có thể thấy cách mã này hoạt động từ các thông báo: các instance
`Worker` 0 và 3 đã nhận hai request đầu tiên. Server đã ngừng chấp nhận kết nối sau
kết nối thứ hai, và bản triển khai `Drop` trên `ThreadPool` bắt đầu thực thi thậm chí
trước khi `Worker` 3 bắt đầu job của nó. Việc drop `sender` sẽ ngắt kết nối tất cả các
instance `Worker` và báo cho chúng tắt. Mỗi instance `Worker` in một thông báo khi
chúng ngắt kết nối, và sau đó thread pool gọi `join` để đợi từng luồng `Worker` hoàn
thành.

Hãy chú ý một khía cạnh thú vị của lần thực thi cụ thể này: `ThreadPool` đã drop
`sender`, và trước khi bất kỳ `Worker` nào nhận được lỗi, chúng ta đã thử join
`Worker` 0. `Worker` 0 vẫn chưa nhận được lỗi từ `recv`, vì vậy luồng chính bị chặn
để đợi `Worker` 0 hoàn thành. Trong thời gian đó, `Worker` 3 nhận được một job và sau
đó tất cả các luồng đều nhận được lỗi. Khi `Worker` 0 hoàn thành, luồng chính đợi
các instance `Worker` còn lại hoàn thành. Tại thời điểm đó, tất cả chúng đều đã thoát
khỏi vòng lặp và dừng lại.

Chúc mừng! Bây giờ chúng ta đã hoàn thành dự án của mình; chúng ta có một web
server cơ bản sử dụng thread pool để phản hồi bất đồng bộ. Chúng ta có thể thực hiện
graceful shutdown server, giúp dọn dẹp tất cả các luồng trong pool.

Dưới đây là toàn bộ mã để tham khảo:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Chúng ta có thể làm nhiều hơn ở đây! Nếu bạn muốn tiếp tục nâng cao dự án này, dưới
đây là một số ý tưởng:

- Thêm nhiều tài liệu hơn cho `ThreadPool` và các phương thức công khai của nó.
- Thêm các bài kiểm thử (tests) cho chức năng của thư viện.
- Thay đổi các lệnh gọi tới `unwrap` bằng cách xử lý lỗi mạnh mẽ hơn.
- Sử dụng `ThreadPool` để thực hiện một số tác vụ khác ngoài việc phục vụ các web
  request.
- Tìm một crate thread pool trên [crates.io](https://crates.io/) và triển khai một web
  server tương tự bằng cách sử dụng crate đó thay thế. Sau đó so sánh API và độ mạnh mẽ
  (robustness) của nó với thread pool mà chúng ta đã triển khai.

## Tổng kết

Làm tốt lắm! Bạn đã đi đến phần cuối của cuốn sách! Chúng tôi muốn cảm ơn bạn đã
tham gia cùng chúng tôi trong chuyến hành trình khám phá Rust này. Bây giờ bạn đã sẵn
sàng để tự triển khai các dự án Rust của riêng mình và giúp đỡ các dự án của những
người khác. Hãy nhớ rằng luôn có một cộng đồng chào đón gồm các Rustacean khác, những
người sẵn lòng giúp đỡ bạn giải quyết bất kỳ thách thức nào bạn gặp phải trên hành
trình Rust của mình.
