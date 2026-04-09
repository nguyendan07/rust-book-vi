## Futures và Cú pháp Async

Các yếu tố then chốt của lập trình bất đồng bộ trong Rust là _futures_ và các từ khóa
`async` và `await` của Rust.

Một _future_ là một giá trị có thể chưa sẵn sàng ngay bây giờ nhưng sẽ sẵn sàng vào một thời điểm
nào đó trong tương lai. (Khái niệm này cũng xuất hiện trong nhiều ngôn ngữ khác, đôi khi
dưới các tên gọi khác như _task_ hoặc _promise_.) Rust cung cấp trait `Future` như một khối xây dựng
để các thao tác bất đồng bộ khác nhau có thể được triển khai với các cấu trúc dữ liệu khác nhau
nhưng với một giao diện chung. Trong Rust, futures là các kiểu dữ liệu triển khai trait `Future`.
Mỗi future nắm giữ thông tin riêng về tiến trình đã đạt được và ý nghĩa của việc "sẵn sàng" (ready).

Bạn có thể áp dụng từ khóa `async` cho các khối code (blocks) và các hàm (functions) để chỉ định rằng chúng
có thể bị ngắt quãng và tiếp tục lại. Bên trong một khối async hoặc hàm async, bạn có thể
sử dụng từ khóa `await` để _chờ một future_ (nghĩa là đợi cho đến khi nó sẵn sàng).
Bất kỳ điểm nào bạn sử dụng await cho một future bên trong một khối hoặc hàm async đều là một điểm tiềm năng để
khối hoặc hàm async đó tạm dừng và tiếp tục lại. Quá trình kiểm tra một future để xem
giá trị của nó đã có sẵn chưa được gọi là _thăm dò_ (polling).

Một số ngôn ngữ khác, chẳng hạn như C# và JavaScript, cũng sử dụng các từ khóa `async` và `await`
cho lập trình bất đồng bộ. Nếu bạn đã quen thuộc với những ngôn ngữ đó, bạn có thể nhận thấy một số
khác biệt đáng kể trong cách Rust thực hiện, bao gồm cả cách nó xử lý cú pháp.
Điều đó là có lý do chính đáng, như chúng ta sẽ thấy!

Khi viết async Rust, chúng ta sử dụng các từ khóa `async` và `await` hầu hết thời gian. Rust biên dịch
chúng thành mã tương đương sử dụng trait `Future`, giống như cách nó biên dịch các vòng lặp `for`
thành mã tương đương sử dụng trait `Iterator`. Tuy nhiên, vì Rust cung cấp trait `Future`,
bạn cũng có thể triển khai nó cho các kiểu dữ liệu của riêng mình khi cần thiết. Nhiều hàm
chúng ta sẽ thấy trong suốt chương này trả về các kiểu dữ liệu với triển khai riêng của chúng về `Future`.
Chúng ta sẽ quay lại định nghĩa của trait này ở cuối chương và tìm hiểu kỹ hơn về cách
nó hoạt động, nhưng bấy nhiêu chi tiết này là đủ để chúng ta tiếp tục.

Tất cả những điều này có vẻ hơi trừu tượng, vì vậy hãy viết chương trình async đầu tiên của chúng ta: một
trình thu thập dữ liệu web (web scraper) nhỏ. Chúng ta sẽ truyền vào hai URL từ dòng lệnh, lấy cả hai
một cách đồng thời và trả về kết quả của cái nào kết thúc trước. Ví dụ này sẽ có
khá nhiều cú pháp mới, nhưng đừng lo lắng—chúng tôi sẽ giải thích mọi thứ bạn cần biết.

## Chương trình Async đầu tiên của chúng ta

Để giữ cho trọng tâm của chương này là học async thay vì loay hoay với các phần của hệ sinh thái,
chúng tôi đã tạo crate `trpl` (`trpl` là viết tắt của “The Rust Programming Language”). Nó tái xuất (re-exports)
tất cả các kiểu dữ liệu, trait và hàm bạn sẽ cần, chủ yếu từ các crate [`futures`][futures-crate]<!-- ignore --> và
[`tokio`][tokio]<!-- ignore -->. Crate `futures` là ngôi nhà chính thức cho các thử nghiệm của Rust cho mã async,
và nó thực sự là nơi trait `Future` ban đầu được thiết kế. Tokio là runtime bất đồng bộ được sử dụng rộng rãi
nhất trong Rust hiện nay, đặc biệt là cho các ứng dụng web. Có những runtime tuyệt vời khác ngoài kia,
và chúng có thể phù hợp hơn cho các mục đích của bạn. Chúng tôi sử dụng crate `tokio` bên dưới `trpl`
vì nó đã được kiểm thử kỹ lưỡng và sử dụng rộng rãi.

Trong một số trường hợp, `trpl` cũng đổi tên hoặc bao bọc các API gốc để giúp bạn tập trung vào
các chi tiết liên quan đến chương này. Nếu bạn muốn hiểu crate đó làm gì, chúng tôi khuyến khích
bạn xem [mã nguồn của nó][crate-source]<!-- ignore -->. Bạn sẽ có thể thấy mỗi phần tái xuất đến từ
crate nào, và chúng tôi đã để lại nhiều chú thích giải thích crate đó làm gì.

Tạo một dự án binary mới tên là `hello-async` và thêm crate `trpl` làm phụ thuộc:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Bây giờ chúng ta có thể sử dụng các phần khác nhau được cung cấp bởi `trpl` để viết chương trình async đầu tiên của mình.
Chúng ta sẽ xây dựng một công cụ dòng lệnh nhỏ lấy hai trang web, rút trích phần tử `<title>`
từ mỗi trang, và in ra tiêu đề của bất kỳ trang nào hoàn thành toàn bộ quá trình đó trước.

### Định nghĩa hàm page_title

Hãy bắt đầu bằng cách viết một hàm nhận một URL trang web làm tham số, thực hiện một yêu cầu tới nó,
và trả về văn bản của phần tử tiêu đề (xem Liệt kê 17-1).

<Listing number="17-1" file-name="src/main.rs" caption="Định nghĩa một hàm async để lấy phần tử title từ một trang HTML">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Đầu tiên, chúng ta định nghĩa một hàm tên là `page_title` và đánh dấu nó bằng từ khóa `async`. Sau đó, chúng ta
sử dụng hàm `trpl::get` để lấy bất kỳ URL nào được truyền vào và thêm từ khóa `await` để chờ phản hồi.
Để lấy văn bản của phản hồi, chúng ta gọi phương thức `text` của nó, và một lần nữa await nó bằng
từ khóa `await`. Cả hai bước này đều bất đồng bộ. Đối với hàm `get`, chúng ta phải đợi
máy chủ gửi lại phần đầu tiên của phản hồi, bao gồm HTTP headers, cookies, v.v.,
và có thể được gửi tách biệt với thân phản hồi (response body). Đặc biệt nếu thân phản hồi rất lớn,
có thể mất một thời gian để tất cả được gửi đến. Bởi vì chúng ta phải đợi _toàn bộ_ phản hồi
đến, phương thức `text` cũng là async.

Chúng ta phải await cả hai future này một cách rõ ràng, vì các future trong Rust là _lười_ (lazy):
chúng không làm gì cho đến khi bạn yêu cầu chúng bằng từ khóa `await`. (Thực tế, Rust sẽ hiển thị
một cảnh báo trình biên dịch nếu bạn không sử dụng một future.) Điều này có thể gợi nhớ cho bạn về cuộc thảo luận
trong Chương 13 về iterator trong phần [Xử lý một chuỗi các mục với Iterators][iterators-lazy]<!-- ignore -->.
Iterators không làm gì trừ khi bạn gọi phương thức `next` của chúng—cho dù trực tiếp hay bằng cách
sử dụng các vòng lặp `for` hoặc các phương thức như `map` sử dụng `next` bên dưới. Tương tự,
các future không làm gì trừ khi bạn yêu cầu rõ ràng. Sự lười biếng này cho phép Rust tránh chạy mã async
cho đến khi nó thực sự cần thiết.

> Ghi chú: Điều này khác với hành vi chúng ta đã thấy trong chương trước khi sử dụng
> `thread::spawn` trong [Tạo một Luồng mới với spawn][thread-spawn]<!--ignore-->, nơi closure
> chúng ta truyền cho một luồng khác bắt đầu chạy ngay lập tức. Nó cũng khác với cách nhiều
> ngôn ngữ khác tiếp cận async. Nhưng điều quan trọng là Rust có thể cung cấp các đảm bảo
> về hiệu suất của nó, giống như đối với iterator.

Một khi chúng ta có `response_text`, chúng ta có thể phân tích cú pháp nó thành một thực thể (instance) của kiểu `Html`
bằng cách sử dụng `Html::parse`. Thay vì một chuỗi thô, bây giờ chúng ta có một kiểu dữ liệu mà chúng ta
có thể sử dụng để làm việc với HTML như một cấu trúc dữ liệu phong phú hơn. Cụ thể, chúng ta có thể sử dụng phương thức
`select_first` để tìm thực thể đầu tiên của một bộ chọn CSS cho trước. Bằng cách truyền chuỗi `"title"`,
chúng ta sẽ lấy phần tử `<title>` đầu tiên trong tài liệu, nếu có. Bởi vì có thể không có
bất kỳ phần tử nào khớp, `select_first` trả về một `Option<ElementRef>`. Cuối cùng, chúng ta sử dụng
phương thức `Option::map`, cho phép chúng ta làm việc với mục trong `Option` nếu nó hiện diện,
và không làm gì nếu không có. (Chúng ta cũng có thể sử dụng biểu thức `match` ở đây, nhưng `map` thì
đúng phong cách idiomatic hơn.) Trong thân của hàm mà chúng ta cung cấp cho `map`, chúng ta gọi `inner_html` trên
`title_element` để lấy nội dung của nó, đó là một `String`. Khi mọi thứ hoàn tất, chúng ta có một `Option<String>`.

Lưu ý rằng từ khóa `await` của Rust nằm _sau_ biểu thức bạn đang await, không phải trước nó.
Nghĩa là, nó là một từ khóa _hậu tố_ (postfix). Điều này có thể khác với những gì bạn đã quen
nếu bạn đã sử dụng `async` trong các ngôn ngữ khác, nhưng trong Rust, nó làm cho các chuỗi phương thức
trông đẹp hơn nhiều khi làm việc cùng. Kết quả là, chúng ta có thể thay đổi thân của `page_title` để
nối chuỗi các lệnh gọi hàm `trpl::get` và `text` lại với nhau bằng `await` ở giữa chúng, như trong Liệt kê 17-2.

<Listing number="17-2" file-name="src/main.rs" caption="Chaining với từ khóa `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Vậy là chúng ta đã viết thành công hàm async đầu tiên của mình! Trước khi thêm một số mã vào `main`
để gọi nó, hãy nói thêm một chút về những gì chúng ta đã viết và ý nghĩa của nó.

Khi Rust thấy một khối được đánh dấu bằng từ khóa `async`, nó biên dịch khối đó thành một kiểu dữ liệu ẩn danh,
độc nhất triển khai trait `Future`. Khi Rust thấy một hàm được đánh dấu bằng `async`, nó biên dịch nó
thành một hàm không phải async mà thân của nó là một khối async. Kiểu trả về của một hàm async
là kiểu của dữ liệu ẩn danh mà trình biên dịch tạo ra cho khối async đó.

Vì vậy, viết `async fn` tương đương với việc viết một hàm trả về một _future_ của kiểu trả về đó.
Đối với trình biên dịch, một định nghĩa hàm như `async fn page_title` trong Liệt kê 17-1 tương đương
với một hàm không phải async được định nghĩa như thế này:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Hãy cùng xem qua từng phần của phiên bản đã được chuyển đổi:

- Nó sử dụng cú pháp `impl Trait` mà chúng ta đã thảo luận trong Chương 10 trong phần
  [“Traits làm Tham số”][impl-trait]<!-- ignore -->.
- Trait được trả về là một `Future` với một kiểu liên kết (associated type) là `Output`. Lưu ý rằng
  kiểu `Output` là `Option<String>`, giống như kiểu trả về ban đầu từ phiên bản `async fn`
  của `page_title`.
- Tất cả mã được gọi trong thân của hàm gốc được bao bọc trong một khối `async move`. Nhớ rằng các khối
  là các biểu thức. Toàn bộ khối này là biểu thức được trả về từ hàm.
- Khối async này tạo ra một giá trị với kiểu `Option<String>`, như vừa mô tả. Giá trị đó khớp với
  kiểu `Output` trong kiểu trả về. Điều này giống hệt như các khối khác bạn đã thấy.
- Thân hàm mới là một khối `async move` vì cách nó sử dụng tham số `url`. (Chúng ta sẽ nói nhiều hơn
  về `async` so với `async move` ở phần sau của chương.)

Bây giờ chúng ta có thể gọi `page_title` trong `main`.

## Xác định tiêu đề của một trang đơn lẻ

Để bắt đầu, chúng ta sẽ chỉ lấy tiêu đề của một trang duy nhất. Trong Liệt kê 17-3, chúng ta làm theo
cùng một khuôn mẫu mà chúng ta đã sử dụng trong Chương 12 để nhận các đối số dòng lệnh trong phần
[Chấp nhận các đối số dòng lệnh][cli-args]<!-- ignore -->. Sau đó, chúng ta truyền URL đầu tiên vào `page_title`
và await kết quả. Bởi vì giá trị được tạo ra bởi future là một `Option<String>`, chúng ta sử dụng một
biểu thức `match` để in các thông báo khác nhau tùy thuộc vào việc trang đó có `<title>` hay không.

<Listing number="17-3" file-name="src/main.rs" caption="Gọi hàm `page_title` từ `main` với một đối số do người dùng cung cấp">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Thật không may, mã này không biên dịch được. Nơi duy nhất chúng ta có thể sử dụng từ khóa `await`
là trong các hàm async hoặc các khối async, và Rust sẽ không cho phép chúng ta đánh dấu hàm `main`
đặc biệt là `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

Lý do `main` không thể được đánh dấu là `async` là vì mã async cần một _runtime_ (môi trường thực thi):
một crate Rust quản lý các chi tiết của việc thực thi mã bất đồng bộ. Hàm `main` của một chương trình
có thể _khởi tạo_ một runtime, nhưng bản thân nó không phải là một runtime. (Chúng ta sẽ thấy thêm về
lý do tại sao lại như vậy trong một lát.) Mọi chương trình Rust thực thi mã async đều có ít nhất một nơi
nó thiết lập một runtime và thực thi các future.

Hầu hết các ngôn ngữ hỗ trợ async đều đi kèm với một runtime, nhưng Rust thì không. Thay vào đó,
có nhiều async runtime khác nhau hiện có, mỗi cái trong số đó có những sự đánh đổi khác nhau
phù hợp với trường hợp sử dụng mà nó hướng tới. Ví dụ, một máy chủ web thông lượng cao với nhiều lõi CPU
và lượng RAM lớn có nhu cầu rất khác so với một vi điều khiển (microcontroller) với một lõi đơn,
lượng RAM nhỏ và không có khả năng cấp phát heap. Các crate cung cấp các runtime đó cũng thường
cung cấp các phiên bản async của các chức năng phổ biến như tệp hoặc mạng I/O.

Ở đây, và trong suốt phần còn lại của chương này, chúng ta sẽ sử dụng hàm `run` từ crate `trpl`,
hàm này nhận một future làm đối số và chạy nó cho đến khi hoàn thành. Đằng sau hậu trường,
việc gọi `run` sẽ thiết lập một runtime được sử dụng để chạy future được truyền vào.
Một khi future hoàn thành, `run` trả về bất kỳ giá trị nào mà future đó tạo ra.

Chúng ta có thể truyền future được trả về bởi `page_title` trực tiếp cho `run`, và một khi nó hoàn thành,
chúng ta có thể match trên kết quả `Option<String>`, như chúng ta đã cố gắng làm trong Liệt kê 17-3.
Tuy nhiên, đối với hầu hết các ví dụ trong chương này (và hầu hết mã async trong thế giới thực),
chúng ta sẽ thực hiện nhiều hơn một lời gọi hàm async, vì vậy thay vào đó chúng ta sẽ truyền một khối `async`
và await một cách rõ ràng kết quả của lời gọi `page_title`, như trong Liệt kê 17-4.

<Listing number="17-4" caption="Await một khối async với `trpl::run`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Khi chúng ta chạy mã này, chúng ta nhận được hành vi mà chúng ta mong đợi ban đầu:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run https://www.rust-lang.org
# copy the output here
-->

```console
$ cargo run -- https://www.rust-lang.org
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Phù—cuối cùng chúng ta đã có một đoạn mã async hoạt động! Nhưng trước khi thêm mã để cho hai trang web
đua với nhau, chúng ta hãy quay lại xem các future hoạt động như thế nào.

Mỗi _điểm await_ (await point)—nghĩa là mọi nơi mã sử dụng từ khóa `await`—đại diện cho một nơi
mà quyền kiểm soát được trao lại cho runtime. Để làm được điều đó, Rust cần theo dõi trạng thái
liên quan trong khối async để runtime có thể bắt đầu một công việc khác và sau đó quay lại
khi đã sẵn sàng để thử tiến hành cái đầu tiên một lần nữa. Đây là một máy trạng thái (state machine) ẩn,
giống như thể bạn đã viết một enum như thế này để lưu trạng thái hiện tại tại mỗi điểm await:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Tuy nhiên, việc viết mã để chuyển đổi giữa các trạng thái bằng tay sẽ rất tẻ nhạt và dễ mắc lỗi,
đặc biệt là khi bạn cần thêm nhiều chức năng và nhiều trạng thái hơn cho mã sau này. May mắn thay,
trình biên dịch Rust tự động tạo và quản lý các cấu trúc dữ liệu của máy trạng thái cho mã async.
Các quy tắc vay mượn (borrowing) và sở hữu (ownership) thông thường xung quanh các cấu trúc dữ liệu vẫn được áp dụng,
và thật hạnh phúc, trình biên dịch cũng xử lý việc kiểm tra những quy tắc đó cho chúng ta và cung cấp
các thông báo lỗi hữu ích. Chúng ta sẽ cùng xem qua một vài ví dụ về chúng ở phần sau của chương.

Cuối cùng, một thứ gì đó phải thực thi máy trạng thái này, và thứ đó chính là một runtime. (Đây là lý do tại sao
bạn có thể gặp các tham chiếu đến _executors_ (bộ thực thi) khi tìm hiểu về runtime: một executor là phần
của runtime chịu trách nhiệm thực thi mã async.)

Bây giờ bạn có thể thấy tại sao trình biên dịch đã ngăn chúng ta tạo hàm `main` là một hàm async
trước đó trong Liệt kê 17-3. Nếu `main` là một hàm async, một thứ khác sẽ cần quản lý máy trạng thái
cho bất kỳ future nào mà `main` trả về, nhưng `main` lại là điểm bắt đầu cho chương trình! Thay vào đó,
chúng ta đã gọi hàm `trpl::run` trong `main` để thiết lập một runtime và chạy future được trả về bởi khối
`async` cho đến khi nó hoàn thành.

> Ghi chú: Một số runtime cung cấp các macro để bạn _có thể_ viết một hàm `main` async.
> Các macro đó viết lại `async fn main() { ... }` thành một hàm `fn main` bình thường,
> thực hiện điều tương tự chúng ta đã làm thủ công trong Liệt kê 17-4: gọi một hàm
> chạy một future cho đến khi hoàn thành giống như cách `trpl::run` thực hiện.

Bây giờ hãy kết hợp những mảnh này lại với nhau và xem cách chúng ta có thể viết mã đồng thời.

### Cho hai URL chạy đua với nhau

Trong Liệt kê 17-5, chúng ta gọi `page_title` với hai URL khác nhau được truyền vào từ
dòng lệnh và cho chúng đua với nhau.

<Listing number="17-5" caption="" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Chúng ta bắt đầu bằng cách gọi `page_title` cho mỗi URL do người dùng cung cấp. Chúng ta lưu kết quả
dưới dạng các future `title_fut_1` và `title_fut_2`. Hãy nhớ rằng, những thứ này chưa làm gì cả,
vì các future là lười biếng và chúng ta chưa await chúng. Sau đó, chúng ta truyền các future vào `trpl::race`,
hàm này trả về một giá trị để cho biết future nào được truyền cho nó kết thúc trước.

> Ghi chú: Đằng sau hậu trường, `race` được xây dựng trên một hàm tổng quát hơn là `select`,
> hàm mà bạn sẽ gặp thường xuyên hơn trong mã Rust thực tế. Một hàm `select` có thể làm rất nhiều thứ
> mà hàm `trpl::race` không thể, nhưng nó cũng có thêm một số sự phức tạp mà chúng ta có thể bỏ qua bây giờ.

Cả hai future đều có thể "chiến thắng" một cách hợp lệ, vì vậy việc trả về một `Result` là không hợp lý.
Thay vào đó, `race` trả về một kiểu mà chúng ta chưa từng thấy trước đây, `trpl::Either`. Kiểu `Either`
phần nào giống với `Result` ở chỗ nó có hai trường hợp. Tuy nhiên, không giống như `Result`,
không có khái niệm thành công hay thất bại nào được đưa vào `Either`. Thay vào đó, nó sử dụng `Left` và `Right`
để chỉ định “cái này hoặc cái kia”:

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

Hàm `race` trả về `Left` với đầu ra từ đối số future đầu tiên nếu nó kết thúc trước, hoặc `Right`
với đầu ra của đối số future thứ hai nếu cái đó kết thúc trước. Điều này khớp với thứ tự các đối số
xuất hiện khi gọi hàm: đối số đầu tiên nằm ở bên trái của đối số thứ hai.

Chúng ta cũng cập nhật `page_title` để trả về cùng một URL đã được truyền vào. Bằng cách đó, nếu
trang trả về trước không có `<title>` mà chúng ta có thể giải quyết, chúng ta vẫn có thể
in một thông báo có ý nghĩa. Với thông tin đó sẵn có, chúng ta kết thúc bằng cách cập nhật
đầu ra `println!` của mình để cho biết cả URL nào đã hoàn thành trước và nội dung của `<title>` (nếu có)
của trang web tại URL đó.

Bây giờ bạn đã xây dựng một trình thu thập dữ liệu web nhỏ đang hoạt động! Hãy chọn một vài URL và chạy
công cụ dòng lệnh này. Bạn có thể phát hiện ra rằng một số trang web luôn nhanh hơn các trang web khác, trong khi trong
các trường hợp khác, trang nhanh hơn thay đổi theo từng lần chạy. Quan trọng hơn, bạn đã học được kiến thức cơ bản
về cách làm việc với các future, vì vậy bây giờ chúng ta có thể đi sâu hơn vào những gì chúng ta có thể làm với async.

{{#quiz ../quizzes/async-01-futures-and-syntax.toml}}

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
