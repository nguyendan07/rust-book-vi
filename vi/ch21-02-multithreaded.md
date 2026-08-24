## Chuyển Web Server Đơn Luồng Thành Web Server Đa Luồng

Hiện tại, server sẽ xử lý lần lượt từng request, nghĩa là nó sẽ không xử lý kết
nối thứ hai cho đến khi kết nối đầu tiên được xử lý xong. Nếu server nhận được ngày
càng nhiều request, việc thực thi tuần tự (serial execution) này sẽ ngày càng kém
tối ưu. Nếu server nhận được một request mất nhiều thời gian để xử lý, các request
tiếp theo sẽ phải đợi cho đến khi request kéo dài đó kết thúc, ngay cả khi các request
mới có thể được xử lý nhanh chóng. Chúng ta sẽ cần khắc phục điều này, nhưng trước
tiên chúng ta sẽ xem xét vấn đề đang diễn ra trong thực tế.

### Mô Phỏng Một Request Chậm Trong Triển Khai Server Hiện Tại

Chúng ta sẽ xem xét cách một request xử lý chậm có thể ảnh hưởng đến các request
khác được gửi tới bản triển khai server hiện tại của chúng ta. Listing 21-10 triển
khai việc xử lý một request đến _/sleep_ với một phản hồi chậm được mô phỏng sẽ khiến
server sleep (ngủ) trong năm giây trước khi phản hồi.

<Listing number="21-10" file-name="src/main.rs" caption="Mô phỏng một request chậm bằng cách sleep trong 5 giây">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Chúng ta đã chuyển từ `if` sang `match` vì giờ đây chúng ta có ba trường hợp. Chúng
ta cần khớp rõ ràng trên một slice của `request_line` để khớp mẫu (pattern match) với
các giá trị chuỗi ký tự (string literal); `match` không tự động tạo tham chiếu và
giải tham chiếu (dereferencing) như phương thức so sánh bằng.

Nhánh (arm) đầu tiên giống như khối `if` trong Listing 21-9. Nhánh thứ hai khớp với
request đến _/sleep_. Khi nhận được request đó, server sẽ sleep trong năm giây trước
khi hiển thị trang HTML thành công. Nhánh thứ ba giống như khối `else` trong Listing
21-9.

Bạn có thể thấy server của chúng ta sơ khai đến mức nào: các thư viện thực tế sẽ
xử lý việc nhận diện nhiều request theo cách ít rườm rà hơn nhiều!

Khởi động server bằng `cargo run`. Sau đó mở hai cửa sổ trình duyệt: một cho
_http://127.0.0.1:7878/_ và một cho _http://127.0.0.1:7878/sleep_. Nếu bạn nhập URI
_/_ một vài lần, như trước đây, bạn sẽ thấy nó phản hồi nhanh chóng. Nhưng nếu bạn
nhập _/sleep_ rồi sau đó tải _/_, bạn sẽ thấy rằng _/_ phải đợi cho đến khi `sleep`
đã ngủ đủ năm giây trước khi tải.

Có nhiều kỹ thuật chúng ta có thể sử dụng để tránh việc các request bị ùn tắc phía
sau một request chậm, bao gồm cả việc sử dụng async như chúng ta đã làm trong Chương
17; kỹ thuật mà chúng ta sẽ triển khai là thread pool (nhóm luồng).

### Cải Thiện Thông Lượng Với Thread Pool

Một _thread pool_ (nhóm luồng) là một nhóm các luồng (thread) đã được tạo sẵn, đang
chờ đợi và sẵn sàng xử lý một tác vụ. Khi chương trình nhận được một tác vụ mới, nó
sẽ gán một trong các luồng trong pool cho tác vụ đó, và luồng đó sẽ xử lý tác vụ.
Các luồng còn lại trong pool luôn sẵn sàng xử lý bất kỳ tác vụ nào khác xuất hiện
trong khi luồng đầu tiên đang xử lý. Khi luồng đầu tiên xử lý xong tác vụ của nó, nó
sẽ được đưa trở lại nhóm các luồng rảnh rỗi (idle threads), sẵn sàng xử lý một tác vụ
mới. Thread pool cho phép bạn xử lý các kết nối đồng thời (concurrently), làm tăng
thông lượng (throughput) của server.

Chúng ta sẽ giới hạn số lượng luồng trong pool ở một con số nhỏ để bảo vệ chúng ta
khỏi các cuộc tấn công DoS (Từ chối dịch vụ); nếu chúng ta để chương trình tạo một
luồng mới cho mỗi request đến, ai đó gửi 10 triệu request đến server của chúng ta có
thể gây ra hỗn loạn bằng cách sử dụng hết tài nguyên của server và làm tê liệt quá
trình xử lý request.

Do đó, thay vì tạo số lượng luồng không giới hạn, chúng ta sẽ có một số lượng luồng
cố định chờ đợi trong pool. Các request đến sẽ được gửi đến pool để xử lý. Pool sẽ
duy trì một hàng đợi (queue) chứa các request gửi đến. Mỗi luồng trong pool sẽ lấy
một request ra khỏi hàng đợi này, xử lý request, và sau đó yêu cầu hàng đợi cung cấp
một request khác. Với thiết kế này, chúng ta có thể xử lý đồng thời tối đa *`N`*
request, trong đó *`N`* là số lượng luồng. Nếu mỗi luồng đang phản hồi một request kéo
dài, các request tiếp theo vẫn có thể bị ùn lại trong hàng đợi, nhưng chúng ta đã
tăng số lượng request kéo dài mà chúng ta có thể xử lý trước khi chạm đến điểm giới
hạn đó.

Kỹ thuật này chỉ là một trong nhiều cách để cải thiện thông lượng của một web server.
Các tùy chọn khác bạn có thể khám phá là mô hình fork/join, mô hình async I/O đơn
luồng và mô hình async I/O đa luồng. Nếu bạn quan tâm đến chủ đề này, bạn có thể đọc
thêm về các giải pháp khác và thử triển khai chúng; với một ngôn ngữ cấp thấp như
Rust, tất cả các tùy chọn này đều khả thi.

Trước khi bắt đầu triển khai một thread pool, hãy nói về việc sử dụng pool trông sẽ
như thế nào. Khi bạn đang cố gắng thiết kế mã, viết giao diện phía client (client
interface) trước có thể giúp định hướng thiết kế của bạn. Hãy viết API của mã để nó
được cấu trúc theo cách bạn muốn gọi; sau đó triển khai chức năng bên trong cấu trúc
đó thay vì triển khai chức năng trước rồi mới thiết kế public API.

Tương tự như cách chúng ta đã sử dụng phát triển hướng kiểm thử (test-driven
development) trong dự án ở Chương 12, ở đây chúng ta sẽ sử dụng phát triển hướng
trình biên dịch (compiler-driven development). Chúng ta sẽ viết mã gọi các hàm mình
muốn, và sau đó xem xét các lỗi từ trình biên dịch để xác định xem nên thay đổi điều
gì tiếp theo nhằm làm cho mã hoạt động. Tuy nhiên, trước khi làm điều đó, chúng ta
sẽ khám phá kỹ thuật mà chúng ta sẽ không sử dụng làm điểm khởi đầu.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Tạo Một Luồng Cho Mỗi Request

Đầu tiên, hãy khám phá xem mã của chúng ta sẽ trông như thế nào nếu nó thực sự tạo
một luồng mới cho mỗi kết nối. Như đã đề cập trước đó, đây không phải là kế hoạch
cuối cùng của chúng ta do các vấn đề tiềm ẩn khi tạo ra số lượng luồng không giới hạn,
nhưng nó là một điểm khởi đầu để có được một server đa luồng hoạt động trước. Sau
đó, chúng ta sẽ thêm thread pool như một sự cải tiến, và việc đối chiếu giữa hai giải
pháp sẽ dễ dàng hơn. Listing 21-11 cho thấy những thay đổi cần thực hiện đối với
`main` để tạo một luồng mới xử lý từng stream bên trong vòng lặp `for`.

<Listing number="21-11" file-name="src/main.rs" caption="Tạo một luồng mới cho mỗi stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Như bạn đã học trong Chương 16, `thread::spawn` sẽ tạo một luồng mới và sau đó chạy
mã trong closure ở luồng mới. Nếu bạn chạy mã này và tải _/sleep_ trong trình duyệt của
mình, sau đó tải _/_ trong hai tab trình duyệt khác, bạn thực sự sẽ thấy rằng các
request đến _/_ không cần phải đợi _/sleep_ hoàn thành. Tuy nhiên, như chúng tôi đã đề
cập, điều này cuối cùng sẽ làm quá tải hệ thống vì bạn sẽ tạo ra các luồng mới mà
không có bất kỳ giới hạn nào.

Bạn cũng có thể nhớ lại từ Chương 17 rằng đây chính xác là tình huống mà async và
await thực sự tỏa sáng! Hãy ghi nhớ điều đó khi chúng ta xây dựng thread pool và suy
nghĩ về việc mọi thứ sẽ khác nhau hay giống nhau như thế nào với async.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Tạo Một Số Lượng Hữu Hạn Các Luồng

Chúng ta muốn thread pool của mình hoạt động theo cách tương tự, quen thuộc để việc
chuyển đổi từ các luồng sang thread pool không đòi hỏi những thay đổi lớn đối với mã
sử dụng API của chúng ta. Listing 21-12 hiển thị giao diện giả định cho một struct
`ThreadPool` mà chúng ta muốn sử dụng thay cho `thread::spawn`.

<Listing number="21-12" file-name="src/main.rs" caption="Giao diện `ThreadPool` lý tưởng của chúng ta">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Chúng ta sử dụng `ThreadPool::new` để tạo một thread pool mới với số lượng luồng có
thể cấu hình được, trong trường hợp này là 4. Sau đó, trong vòng lặp `for`,
`pool.execute` có giao diện tương tự như `thread::spawn` ở chỗ nó nhận một closure mà
pool nên chạy cho mỗi stream. Chúng ta cần triển khai `pool.execute` sao cho nó nhận
closure và giao cho một luồng trong pool chạy. Mã này chưa thể biên dịch được, nhưng
chúng ta sẽ thử để trình biên dịch có thể hướng dẫn chúng ta cách sửa nó.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Xây Dựng `ThreadPool` Bằng Cách Phát Triển Hướng Trình Biên Dịch

Thực hiện các thay đổi trong Listing 21-12 vào _src/main.rs_, và sau đó hãy sử dụng
các lỗi trình biên dịch từ `cargo check` để thúc đẩy quá trình phát triển của chúng
ta. Dưới đây là lỗi đầu tiên chúng ta nhận được:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Tuyệt vời! Lỗi này cho chúng ta biết rằng chúng ta cần một kiểu hoặc module
`ThreadPool`, vì vậy chúng ta sẽ xây dựng một cái ngay bây giờ. Bản triển khai
`ThreadPool` của chúng ta sẽ độc lập với loại công việc mà web server của chúng ta
đang làm. Vì vậy, hãy chuyển crate `hello` từ một binary crate sang một library crate
để chứa bản triển khai `ThreadPool` của chúng ta. Sau khi chuyển sang library crate,
chúng ta cũng có thể sử dụng thư viện thread pool riêng biệt này cho bất kỳ công
việc nào chúng ta muốn thực hiện bằng thread pool, chứ không chỉ để phục vụ các web
request.

Tạo một tệp _src/lib.rs_ chứa nội dung sau, đây là định nghĩa đơn giản nhất của struct
`ThreadPool` mà chúng ta có thể có vào lúc này:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

Sau đó chỉnh sửa tệp _src/main.rs_ để đưa `ThreadPool` vào phạm vi từ library crate
bằng cách thêm mã sau vào đầu _src/main.rs_:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Mã này vẫn chưa hoạt động, nhưng hãy kiểm tra lại để nhận lỗi tiếp theo mà chúng ta
cần giải quyết:

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Lỗi này chỉ ra rằng tiếp theo chúng ta cần tạo một hàm liên kết (associated
function) có tên là `new` cho `ThreadPool`. Chúng ta cũng biết rằng `new` cần có một
tham số có thể nhận `4` làm đối số và sẽ trả về một instance `ThreadPool`. Hãy triển
khai hàm `new` đơn giản nhất có các đặc điểm đó:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

Chúng ta đã chọn `usize` làm kiểu của tham số `size` vì chúng ta biết rằng số lượng
luồng là một số âm thì không có ý nghĩa gì. Chúng ta cũng biết mình sẽ sử dụng `4`
này làm số lượng phần tử trong một tập hợp các luồng, đó chính là mục đích của kiểu
`usize`, như đã thảo luận trong phần [“Các kiểu số nguyên”][integer-types]<!-- ignore
--> ở Chương 3.

Hãy kiểm tra lại mã:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Bây giờ lỗi xảy ra vì chúng ta không có phương thức `execute` trên `ThreadPool`. Nhớ
lại từ phần [“Tạo Một Số Lượng Hữu Hạn Các
Luồng”](#creating-a-finite-number-of-threads)<!-- ignore --> rằng chúng ta đã quyết
định thread pool của mình nên có một giao diện tương tự như `thread::spawn`. Ngoài ra,
chúng ta sẽ triển khai hàm `execute` sao cho nó nhận closure được truyền vào và giao
nó cho một luồng đang rảnh rỗi trong pool để chạy.

Chúng ta sẽ định nghĩa phương thức `execute` trên `ThreadPool` để nhận một closure
làm tham số. Nhớ lại từ [“Di chuyển các giá trị đã capture ra khỏi closure và các
trait `Fn`”][fn-traits]<!-- ignore --> trong Chương 13 rằng chúng ta có thể nhận các
closure làm tham số với ba trait khác nhau: `Fn`, `FnMut`, và `FnOnce`. Chúng ta cần
quyết định loại closure nào sẽ sử dụng ở đây. Chúng ta biết cuối cùng mình sẽ làm điều
gì đó tương tự như bản triển khai `thread::spawn` của thư viện chuẩn, vì vậy chúng ta
có thể xem xét những ràng buộc (bounds) mà chữ ký của `thread::spawn` đặt trên tham số
của nó. Tài liệu cho chúng ta thấy như sau:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Tham số kiểu `F` là tham số chúng ta quan tâm ở đây; tham số kiểu `T` liên quan đến
giá trị trả về, và chúng ta không quan tâm đến điều đó. Chúng ta có thể thấy rằng
`spawn` sử dụng `FnOnce` làm trait bound trên `F`. Đây có thể cũng là điều chúng ta
muốn, bởi vì cuối cùng chúng ta sẽ truyền đối số nhận được trong `execute` cho
`spawn`. Chúng ta có thể tự tin hơn nữa rằng `FnOnce` là trait chúng ta muốn sử dụng
vì luồng chạy request sẽ chỉ thực thi closure của request đó một lần, điều này khớp
với chữ `Once` trong `FnOnce`.

Tham số kiểu `F` cũng có trait bound `Send` và lifetime bound `'static`, những ràng
buộc này rất hữu ích trong tình huống của chúng ta: chúng ta cần `Send` để truyền
closure từ luồng này sang luồng khác và `'static` vì chúng ta không biết luồng sẽ mất
bao lâu để thực thi. Hãy tạo một phương thức `execute` trên `ThreadPool` nhận một
tham số generic kiểu `F` với các ràng buộc này:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Chúng ta vẫn sử dụng `()` sau `FnOnce` vì `FnOnce` này đại diện cho một closure không
nhận tham số nào và trả về kiểu unit `()`. Giống như định nghĩa hàm, kiểu trả về có
thể được bỏ qua khỏi chữ ký, nhưng ngay cả khi không có tham số, chúng ta vẫn cần
dấu ngoặc đơn.

Một lần nữa, đây là bản triển khai đơn giản nhất của phương thức `execute`: nó không
làm gì cả, nhưng chúng ta chỉ đang cố gắng làm cho mã của mình biên dịch được. Hãy
kiểm tra lại:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Nó đã biên dịch được! Nhưng lưu ý rằng nếu bạn thử chạy `cargo run` và thực hiện một
yêu cầu trong trình duyệt, bạn sẽ thấy các lỗi trong trình duyệt mà chúng ta đã thấy
ở đầu chương. Thư viện của chúng ta thực sự vẫn chưa gọi closure được truyền vào
`execute`!

> Ghi chú: Một câu nói bạn có thể nghe về các ngôn ngữ có trình biên dịch nghiêm
> ngặt, chẳng hạn như Haskell và Rust, là “nếu mã biên dịch được thì nó hoạt động”.
> Nhưng câu nói này không đúng trong mọi trường hợp. Dự án của chúng ta biên dịch
> được, nhưng nó hoàn toàn không làm gì cả! Nếu chúng ta đang xây dựng một dự án hoàn
> chỉnh, thực tế, đây sẽ là thời điểm thích hợp để bắt đầu viết unit test để kiểm tra
> xem mã vừa biên dịch được _vừa_ có hành vi mà chúng ta mong muốn.

Hãy cân nhắc: điều gì sẽ khác ở đây nếu chúng ta thực thi một _future_ thay vì một
closure?

#### Xác Thực Số Lượng Luồng Trong `new`

Chúng ta chưa làm gì với các tham số truyền vào `new` và `execute`. Hãy triển khai
phần thân của các hàm này với hành vi mà chúng ta muốn. Để bắt đầu, hãy nghĩ về `new`.
Trước đó, chúng ta đã chọn một kiểu số không dấu cho tham số `size` vì một pool có
số lượng luồng âm là vô nghĩa. Tuy nhiên, một pool có không luồng cũng là vô nghĩa,
thế nhưng 0 lại là một giá trị `usize` hoàn toàn hợp lệ. Chúng ta sẽ thêm mã để kiểm
tra xem `size` có lớn hơn 0 hay không trước khi trả về một instance `ThreadPool` và
làm cho chương trình panic nếu nó nhận giá trị 0 bằng cách sử dụng macro `assert!`,
như được hiển thị trong Listing 21-13.

<Listing number="21-13" file-name="src/lib.rs" caption="Triển khai `ThreadPool::new` để panic nếu `size` bằng 0">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Chúng ta cũng đã thêm một số tài liệu cho `ThreadPool` bằng các doc comment. Lưu ý
rằng chúng ta đã tuân theo các thực hành tài liệu tốt bằng cách thêm một phần nêu rõ
các tình huống mà hàm của chúng ta có thể panic, như đã thảo luận trong Chương 14.
Hãy thử chạy `cargo doc --open` và nhấp vào struct `ThreadPool` để xem tài liệu được
tạo cho `new` trông như thế nào!

Thay vì thêm macro `assert!` như chúng ta đã làm ở đây, chúng ta có thể đổi `new`
thành `build` và trả về một `Result` giống như chúng ta đã làm với `Config::build`
trong dự án I/O ở Listing 12-9. Nhưng chúng ta đã quyết định trong trường hợp này rằng
việc cố gắng tạo một thread pool không có luồng nào phải là một lỗi không thể phục
hồi (unrecoverable error). Nếu bạn muốn thử thách bản thân, hãy thử viết một hàm có
tên là `build` với chữ ký sau để so sánh với hàm `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Tạo Không Gian Để Lưu Trữ Các Luồng

Bây giờ chúng ta đã có cách để biết mình có một số lượng luồng hợp lệ để lưu trữ trong
pool, chúng ta có thể tạo các luồng đó và lưu trữ chúng trong struct `ThreadPool`
trước khi trả về struct. Nhưng làm thế nào để chúng ta “lưu trữ” một luồng? Hãy xem
lại chữ ký của `thread::spawn` một lần nữa:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Hàm `spawn` trả về một `JoinHandle<T>`, trong đó `T` là kiểu mà closure trả về. Hãy
thử sử dụng `JoinHandle` và xem điều gì xảy ra. Trong trường hợp của chúng ta, các
closure mà chúng ta truyền vào thread pool sẽ xử lý kết nối và không trả về bất kỳ
thứ gì, vì vậy `T` sẽ là kiểu unit `()`.

Mã trong Listing 21-14 sẽ biên dịch nhưng vẫn chưa tạo bất kỳ luồng nào. Chúng ta đã
thay đổi định nghĩa của `ThreadPool` để chứa một vector các instance
`thread::JoinHandle<()>`, khởi tạo vector với dung lượng (capacity) là `size`, thiết
lập một vòng lặp `for` sẽ chạy một số mã để tạo các luồng, và trả về một instance
`ThreadPool` chứa chúng.

<Listing number="21-14" file-name="src/lib.rs" caption="Tạo một vector cho `ThreadPool` để giữ các luồng">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Chúng ta đã đưa `std::thread` vào phạm vi trong library crate vì chúng ta đang sử
dụng `thread::JoinHandle` làm kiểu của các phần tử trong vector của `ThreadPool`.

Khi nhận được một kích thước hợp lệ, `ThreadPool` của chúng ta tạo một vector mới có
thể chứa `size` phần tử. Hàm `with_capacity` thực hiện cùng tác vụ với `Vec::new`
nhưng có một điểm khác biệt quan trọng: nó cấp phát trước không gian trong vector.
Bởi vì chúng ta biết mình cần lưu trữ `size` phần tử trong vector, việc cấp phát
trước này hiệu quả hơn một chút so với việc sử dụng `Vec::new`, vốn sẽ tự thay đổi
kích thước khi các phần tử được chèn vào.

Khi bạn chạy lại `cargo check`, nó sẽ thành công.

#### Một Struct `Worker` Chịu Trách Nhiệm Gửi Mã Từ `ThreadPool` Đến Một Luồng

Chúng ta đã để lại một comment trong vòng lặp `for` ở Listing 21-14 liên quan đến
việc tạo các luồng. Ở đây, chúng ta sẽ xem xét cách chúng ta thực sự tạo các luồng.
Thư viện chuẩn cung cấp `thread::spawn` như một cách để tạo các luồng, và
`thread::spawn` mong muốn nhận được một số mã mà luồng nên chạy ngay khi luồng được
tạo. Tuy nhiên, trong trường hợp của chúng ta, chúng ta muốn tạo các luồng và để chúng
_chờ đợi_ mã mà chúng ta sẽ gửi sau. Bản triển khai các luồng của thư viện chuẩn
không bao gồm bất kỳ cách nào để làm điều đó; chúng ta phải tự triển khai nó một cách
thủ công.

Chúng ta sẽ triển khai hành vi này bằng cách giới thiệu một cấu trúc dữ liệu mới nằm
giữa `ThreadPool` và các luồng để quản lý hành vi mới này. Chúng ta sẽ gọi cấu trúc
dữ liệu này là _Worker_ (công nhân), đây là một thuật ngữ phổ biến trong các triển
khai pooling. `Worker` nhận mã cần chạy và chạy mã đó trong luồng của Worker.

Hãy nghĩ về những người làm việc trong bếp tại một nhà hàng: các nhân viên chờ đợi
cho đến khi có đơn đặt hàng từ khách hàng, và sau đó họ chịu trách nhiệm nhận các đơn
hàng đó và hoàn thành chúng.

Thay vì lưu trữ một vector chứa các instance `JoinHandle<()>` trong thread pool,
chúng ta sẽ lưu trữ các instance của struct `Worker`. Mỗi `Worker` sẽ lưu trữ một
instance `JoinHandle<()>` đơn lẻ. Sau đó, chúng ta sẽ triển khai một phương thức trên
`Worker` nhận một closure chứa mã để chạy và gửi nó đến luồng đang chạy để thực thi.
Chúng ta cũng sẽ cung cấp cho mỗi `Worker` một `id` để có thể phân biệt giữa các
instance `Worker` khác nhau trong pool khi ghi log hoặc debug.

Dưới đây là quy trình mới sẽ diễn ra khi chúng ta tạo một `ThreadPool`. Chúng ta sẽ
triển khai mã gửi closure đến luồng sau khi đã thiết lập `Worker` theo cách này:

1. Định nghĩa một struct `Worker` chứa một `id` và một `JoinHandle<()>`.
2. Thay đổi `ThreadPool` để chứa một vector các instance `Worker`.
3. Định nghĩa một hàm `Worker::new` nhận một số `id` và trả về một instance `Worker`
   chứa `id` đó và một luồng được tạo với một closure rỗng.
4. Trong `ThreadPool::new`, sử dụng biến đếm của vòng lặp `for` để tạo một `id`, tạo
   một `Worker` mới với `id` đó, và lưu trữ worker trong vector.

Nếu bạn muốn thử sức, hãy thử tự mình thực hiện những thay đổi này trước khi xem mã
trong Listing 21-15.

Sẵn sàng chưa? Dưới đây là Listing 21-15 với một cách để thực hiện các sửa đổi nói
trên.

<Listing number="21-15" file-name="src/lib.rs" caption="Sửa đổi `ThreadPool` để chứa các instance `Worker` thay vì giữ trực tiếp các luồng">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Chúng ta đã đổi tên trường trên `ThreadPool` từ `threads` thành `workers` vì giờ đây
nó chứa các instance `Worker` thay vì các instance `JoinHandle<()>`. Chúng ta sử
dụng biến đếm trong vòng lặp `for` làm đối số cho `Worker::new`, và chúng ta lưu trữ
từng `Worker` mới trong vector có tên là `workers`.

Mã bên ngoài (như server của chúng ta trong _src/main.rs_) không cần biết chi tiết
triển khai liên quan đến việc sử dụng struct `Worker` bên trong `ThreadPool`, vì vậy
chúng ta để struct `Worker` và hàm `new` của nó ở chế độ private. Hàm `Worker::new`
sử dụng `id` mà chúng ta cung cấp và lưu trữ một instance `JoinHandle<()>` được tạo
bằng cách sinh ra một luồng mới sử dụng một closure rỗng.

> Ghi chú: Nếu hệ điều hành không thể tạo luồng vì không đủ tài nguyên hệ thống,
> `thread::spawn` sẽ panic. Điều đó sẽ khiến toàn bộ server của chúng ta bị panic, mặc
> dù việc tạo một số luồng có thể đã thành công. Vì sự đơn giản, hành vi này có thể
> chấp nhận được, nhưng trong một bản triển khai thread pool thực tế cho môi trường
> production, bạn có thể sẽ muốn sử dụng [`std::thread::Builder`][builder]<!-- ignore
> --> và phương thức [`spawn`][builder-spawn]<!-- ignore --> của nó trả về `Result`
> thay thế.

Mã này sẽ biên dịch và sẽ lưu trữ số lượng các instance `Worker` mà chúng ta đã chỉ
định làm đối số cho `ThreadPool::new`. Nhưng chúng ta _vẫn_ chưa xử lý closure nhận
được trong `execute`. Hãy xem cách thực hiện điều đó tiếp theo.

#### Gửi Request Đến Các Luồng Thông Qua Channel

Vấn đề tiếp theo chúng ta sẽ giải quyết là các closure được cung cấp cho
`thread::spawn` hoàn toàn không làm gì cả. Hiện tại, chúng ta nhận được closure muốn
thực thi trong phương thức `execute`. Nhưng chúng ta cần cung cấp cho `thread::spawn`
một closure để chạy khi chúng ta tạo từng `Worker` trong quá trình khởi tạo
`ThreadPool`.

Chúng ta muốn các struct `Worker` mà chúng ta vừa tạo lấy mã cần chạy từ một hàng đợi
được giữ trong `ThreadPool` và gửi mã đó đến luồng của nó để chạy.

Các channel mà chúng ta đã tìm hiểu trong Chương 16—một cách đơn giản để giao tiếp
giữa hai luồng—sẽ là sự lựa chọn hoàn hảo cho trường hợp sử dụng này. Chúng ta sẽ sử
dụng một channel để hoạt động như một hàng đợi các công việc (job), và `execute` sẽ
gửi một job từ `ThreadPool` đến các instance `Worker`, và worker sẽ gửi job đó đến
luồng của nó. Dưới đây là kế hoạch:

1. `ThreadPool` sẽ tạo một channel và giữ bên gửi (sender).
2. Mỗi `Worker` sẽ giữ bên nhận (receiver).
3. Chúng ta sẽ tạo một struct `Job` mới chứa các closure mà chúng ta muốn gửi qua
   channel.
4. Phương thức `execute` sẽ gửi job mà nó muốn thực thi thông qua sender.
5. Trong luồng của mình, `Worker` sẽ lặp qua receiver của nó và thực thi các closure
   của bất kỳ job nào mà nó nhận được.

Hãy bắt đầu bằng cách tạo một channel trong `ThreadPool::new` và giữ sender trong
instance `ThreadPool`, như được hiển thị trong Listing 21-16. Struct `Job` hiện tại
chưa chứa bất kỳ thứ gì nhưng sẽ là kiểu của mục mà chúng ta gửi qua channel.

<Listing number="21-16" file-name="src/lib.rs" caption="Sửa đổi `ThreadPool` để lưu trữ sender của một channel truyền các instance `Job`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

Trong `ThreadPool::new`, chúng ta tạo channel mới và để pool giữ sender. Mã này sẽ
biên dịch thành công.

Hãy thử truyền một receiver của channel vào từng `Worker` khi thread pool tạo
channel. Chúng ta biết mình muốn sử dụng receiver trong luồng mà các instance `Worker`
sinh ra, vì vậy chúng ta sẽ tham chiếu tham số `receiver` trong closure. Mã trong
Listing 21-17 vẫn chưa thể biên dịch được.

<Listing number="21-17" file-name="src/lib.rs" caption="Truyền receiver đến từng `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

Chúng ta đã thực hiện một số thay đổi nhỏ và trực tiếp: chúng ta truyền receiver vào
`Worker::new`, và sau đó chúng ta sử dụng nó bên trong closure.

Khi chúng ta thử kiểm tra mã này, chúng ta nhận được lỗi sau:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

Mã đang cố gắng truyền `receiver` cho nhiều instance `Worker`. Điều này sẽ không hoạt
động, như bạn nhớ lại từ Chương 16: bản triển khai channel mà Rust cung cấp là đa
_nhà sản xuất_ (multiple producer), đơn _người tiêu dùng_ (single consumer). Điều này
có nghĩa là chúng ta không thể chỉ đơn giản là clone đầu nhận (consuming end) của
channel để sửa mã này. Chúng ta cũng không muốn gửi một thông điệp nhiều lần tới nhiều
người nhận; chúng ta muốn có một danh sách các thông điệp với nhiều instance `Worker`
sao cho mỗi thông điệp chỉ được xử lý một lần.

Ngoài ra, việc lấy một job ra khỏi hàng đợi channel liên quan đến việc thay đổi
(mutate) `receiver`, vì vậy các luồng cần một cách an toàn để chia sẻ và sửa đổi
`receiver`; nếu không, chúng ta có thể gặp phải race conditions (tình trạng tranh đua,
như đã đề cập trong Chương 16).

Hãy nhớ lại các con trỏ thông minh an toàn theo luồng (thread-safe smart pointers)
đã được thảo luận trong Chương 16: để chia sẻ quyền sở hữu (ownership) qua nhiều luồng
và cho phép các luồng thay đổi giá trị, chúng ta cần sử dụng `Arc<Mutex<T>>`. Kiểu
`Arc` sẽ cho phép nhiều instance `Worker` sở hữu receiver, và `Mutex` sẽ đảm bảo rằng
chỉ có một `Worker` lấy job từ receiver tại một thời điểm. Listing 21-18 cho thấy
những thay đổi chúng ta cần thực hiện.

<Listing number="21-18" file-name="src/lib.rs" caption="Chia sẻ receiver giữa các instance `Worker` bằng cách sử dụng `Arc` và `Mutex`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

Trong `ThreadPool::new`, chúng ta đặt receiver vào trong một `Arc` và một `Mutex`.
Đối với mỗi `Worker` mới, chúng ta clone `Arc` để tăng số lượng tham chiếu (reference
count) để các instance `Worker` có thể chia sẻ quyền sở hữu receiver.

Với những thay đổi này, mã đã biên dịch được! Chúng ta đang tiến gần đến đích rồi!

#### Triển Khai Phương Thức `execute`

Cuối cùng hãy triển khai phương thức `execute` trên `ThreadPool`. Chúng ta cũng sẽ đổi
`Job` từ một struct thành một bí danh kiểu (type alias) cho một trait object chứa kiểu
closure mà `execute` nhận được. Như đã thảo luận trong phần [“Tạo tên đồng nghĩa cho
kiểu với bí danh kiểu”][creating-type-synonyms-with-type-aliases]<!-- ignore --> ở
Chương 20, bí danh kiểu cho phép chúng ta làm ngắn các kiểu dài để dễ sử dụng hơn. Hãy
xem Listing 21-19.

<Listing number="21-19" file-name="src/lib.rs" caption="Tạo bí danh kiểu `Job` cho một `Box` chứa từng closure và sau đó gửi job qua channel">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

Sau khi tạo một instance `Job` mới bằng cách sử dụng closure nhận được trong
`execute`, chúng ta gửi job đó qua đầu gửi của channel. Chúng ta gọi `unwrap` trên
`send` cho trường hợp việc gửi bị thất bại. Điều này có thể xảy ra nếu, chẳng hạn,
chúng ta dừng tất cả các luồng không cho thực thi, nghĩa là đầu nhận đã ngừng nhận các
thông điệp mới. Tại thời điểm này, chúng ta không thể dừng các luồng của mình thực thi:
các luồng của chúng ta tiếp tục thực thi chừng nào pool còn tồn tại. Lý do chúng ta sử
dụng `unwrap` là vì chúng ta biết trường hợp thất bại sẽ không xảy ra, nhưng trình
biên dịch thì không biết điều đó.

Nhưng chúng ta vẫn chưa hoàn thành hoàn toàn! Trong `Worker`, closure được truyền vào
`thread::spawn` vẫn chỉ _tham chiếu_ đến đầu nhận của channel. Thay vào đó, chúng ta
cần closure lặp vô tận, liên tục yêu cầu đầu nhận của channel cung cấp một job và chạy
job đó khi nhận được. Hãy thực hiện thay đổi hiển thị trong Listing 21-20 đối với
`Worker::new`.

<Listing number="21-20" file-name="src/lib.rs" caption="Nhận và thực thi các job trong luồng của instance `Worker`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Ở đây, trước tiên chúng ta gọi `lock` trên `receiver` để giành quyền sở hữu mutex
(acquire the mutex), và sau đó chúng ta gọi `unwrap` để panic đối với bất kỳ lỗi nào.
Việc giành khóa (acquiring a lock) có thể thất bại nếu mutex ở trạng thái _bị nhiễm
độc_ (poisoned), điều này có thể xảy ra nếu một luồng khác bị panic trong khi đang giữ
khóa thay vì giải phóng khóa. Trong tình huống này, việc gọi `unwrap` để luồng này
panic là hành động chính xác cần thực hiện. Bạn có thể thoải mái đổi `unwrap` này thành
`expect` với thông báo lỗi có ý nghĩa với bạn.

Nếu chúng ta có được khóa trên mutex, chúng ta gọi `recv` để nhận một `Job` từ
channel. Lệnh `unwrap` cuối cùng cũng vượt qua mọi lỗi ở đây, lỗi này có thể xảy ra
nếu luồng giữ sender đã bị tắt, tương tự như cách phương thức `send` trả về `Err` nếu
receiver bị tắt.

Lệnh gọi tới `recv` là chặn (blocks), vì vậy nếu chưa có job nào, luồng hiện tại sẽ
chờ cho đến khi có một job khả dụng. `Mutex<T>` đảm bảo rằng chỉ có một luồng `Worker`
tại một thời điểm cố gắng yêu cầu một job.

Thread pool của chúng ta bây giờ đã ở trạng thái hoạt động! Hãy chạy `cargo run` và
thực hiện một số request:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Thành công rồi! Bây giờ chúng ta có một thread pool thực thi các kết nối bất đồng bộ.
Không bao giờ có nhiều hơn bốn luồng được tạo ra, vì vậy hệ thống của chúng ta sẽ
không bị quá tải nếu server nhận được nhiều request. Nếu chúng ta thực hiện một
request đến _/sleep_, server sẽ có thể phục vụ các request khác bằng cách để một
luồng khác chạy chúng.

> Ghi chú: Nếu bạn mở _/sleep_ trong nhiều cửa sổ trình duyệt cùng một lúc, chúng có
> thể tải từng cửa sổ một theo các khoảng thời gian năm giây. Một số trình duyệt web
> thực thi tuần tự nhiều phiên bản của cùng một request vì lý do bộ nhớ đệm (caching).
> Giới hạn này không phải do web server của chúng ta gây ra.

Đây là thời điểm thích hợp để dừng lại và xem xét mã trong các Listing 21-18, 21-19 và
21-20 sẽ khác nhau như thế nào nếu chúng ta sử dụng future thay vì một closure cho công
việc cần hoàn thành. Những kiểu nào sẽ thay đổi? Chữ ký của các phương thức sẽ khác
nhau như thế nào, nếu có? Những phần nào của mã sẽ giữ nguyên?

Sau khi tìm hiểu về vòng lặp `while let` trong Chương 17 và 18, bạn có thể thắc mắc
tại sao chúng ta không viết mã luồng worker như hiển thị trong Listing 21-21.

<Listing number="21-21" file-name="src/lib.rs" caption="Một cách triển khai thay thế của `Worker::new` sử dụng `while let`">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

Mã này biên dịch và chạy được nhưng không mang lại hành vi phân luồng mong muốn: một
request chậm vẫn sẽ khiến các request khác phải đợi để được xử lý. Lý do có phần tinh
tế: struct `Mutex` không có phương thức `unlock` công khai (public) vì quyền sở hữu
khóa dựa trên lifetime (vòng đời) của `MutexGuard<T>` bên trong
`LockResult<MutexGuard<T>>` mà phương thức `lock` trả về. Tại thời điểm biên dịch,
borrow checker có thể thực thi quy tắc rằng tài nguyên được bảo vệ bởi `Mutex` không
thể được truy cập trừ khi chúng ta giữ khóa. Tuy nhiên, việc triển khai này cũng có
thể dẫn đến việc khóa bị giữ lâu hơn dự định nếu chúng ta không chú ý đến lifetime
của `MutexGuard<T>`.

Mã trong Listing 21-20 sử dụng `let job = receiver.lock().unwrap().recv().unwrap();`
hoạt động được là vì với `let`, bất kỳ giá trị tạm thời nào được sử dụng trong biểu
thức ở phía bên phải dấu bằng đều ngay lập tức bị drop khi câu lệnh `let` kết thúc.
Tuy nhiên, `while let` (và cả `if let` cùng `match`) không drop các giá trị tạm thời
cho đến khi kết thúc khối mã liên quan. Trong Listing 21-21, khóa vẫn được giữ trong
suốt thời gian diễn ra lệnh gọi `job()`, nghĩa là các instance `Worker` khác không thể
nhận job.

[creating-type-synonyms-with-type-aliases]: ch20-03-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]: ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
