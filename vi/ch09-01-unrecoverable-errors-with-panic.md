## Lỗi không thể phục hồi với `panic!`

Đôi khi những điều tồi tệ xảy ra trong mã của bạn và bạn không thể làm gì được
với nó. Trong những trường hợp này, Rust có macro `panic!`. Có hai cách để gây ra
một panic trong thực tế: bằng cách thực hiện một hành động khiến mã của chúng ta panic (chẳng hạn như
truy cập mảng vượt quá giới hạn) hoặc bằng cách gọi trực tiếp macro `panic!`.
Trong cả hai trường hợp, chúng ta đều gây ra một panic trong chương trình của mình. Theo mặc định, các panic này sẽ
in một thông báo lỗi, giải phóng ngăn xếp (unwind), dọn dẹp dữ liệu trên ngăn xếp và thoát. Thông qua một
biến môi trường, bạn cũng có thể yêu cầu Rust hiển thị ngăn xếp cuộc gọi (call stack) khi một
panic xảy ra để giúp việc truy vết nguồn gốc của panic dễ dàng hơn.

> ### Giải phóng ngăn xếp (Unwinding) hoặc Dừng đột ngột (Aborting) khi có Panic
>
> Theo mặc định, khi một panic xảy ra, chương trình bắt đầu _giải phóng ngăn xếp_ (unwinding), có nghĩa là
> Rust đi ngược lên ngăn xếp và dọn dẹp dữ liệu từ mỗi hàm mà nó
> gặp phải. Tuy nhiên, việc đi ngược lại và dọn dẹp là một khối lượng công việc lớn. Rust,
> do đó, cho phép bạn chọn phương án thay thế là _dừng đột ngột_ (aborting) ngay lập tức,
> hành động này sẽ kết thúc chương trình mà không cần dọn dẹp.
>
> Bộ nhớ mà chương trình đang sử dụng sau đó sẽ cần được hệ điều hành dọn dẹp. Nếu trong dự án của mình, bạn cần làm cho tệp nhị phân kết quả
> nhỏ nhất có thể, bạn có thể chuyển từ giải phóng ngăn xếp sang dừng đột ngột khi xảy ra panic bằng cách
> thêm `panic = 'abort'` vào các phần `[profile]` thích hợp trong
> file _Cargo.toml_ của bạn. Ví dụ, nếu bạn muốn dừng đột ngột khi panic ở chế độ release,
> hãy thêm phần này:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Hãy thử gọi `panic!` trong một chương trình đơn giản:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Khi bạn chạy chương trình, bạn sẽ thấy nội dung tương tự như thế này:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Lời gọi đến `panic!` gây ra thông báo lỗi nằm trong hai dòng cuối cùng.
Dòng đầu tiên hiển thị thông báo panic của chúng ta và vị trí trong mã nguồn nơi
panic xảy ra: _src/main.rs:2:5_ cho biết đó là dòng thứ hai,
ký tự thứ năm trong file _src/main.rs_ của chúng ta.

Trong trường hợp này, dòng được chỉ định là một phần mã của chúng ta, và nếu chúng ta đi đến
dòng đó, chúng ta sẽ thấy lời gọi macro `panic!`. Trong các trường hợp khác, lời gọi `panic!` có thể
nằm trong mã mà mã của chúng ta gọi đến, và tên file cùng số dòng được báo cáo bởi
thông báo lỗi sẽ là mã của người khác nơi macro `panic!` được
gọi, chứ không phải dòng mã của chúng ta mà cuối cùng đã dẫn đến lời gọi `panic!`.

<!-- Old heading. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Chúng ta có thể sử dụng truy vết ngược (backtrace) của các hàm mà lời gọi `panic!` bắt nguồn từ đó để tìm ra
phần mã của chúng ta đang gây ra sự cố. Để hiểu cách sử dụng
truy vết ngược của `panic!`, hãy xem xét một ví dụ khác và xem nó như thế nào khi
một lời gọi `panic!` đến từ một thư viện do một bug trong mã của chúng ta thay vì
từ việc mã của chúng ta gọi trực tiếp macro. Listing 9-1 có một đoạn mã
cố gắng truy cập một chỉ số trong một vector vượt quá phạm vi của các chỉ số hợp lệ.

<Listing number="9-1" file-name="src/main.rs" caption="Cố gắng truy cập một phần tử vượt quá giới hạn của một vector, điều này sẽ gây ra một lời gọi đến `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đang cố gắng truy cập phần tử thứ 100 của vector (nằm ở
chỉ số 99 vì việc đánh chỉ số bắt đầu từ không), nhưng vector chỉ có ba
phần tử. Trong tình huống này, Rust sẽ panic. Việc sử dụng `[]` được cho là sẽ trả về
một phần tử, nhưng nếu bạn truyền một chỉ số không hợp lệ, không có phần tử nào mà Rust
có thể trả về ở đây là chính xác cả.

Trong C, việc cố gắng đọc vượt quá giới hạn của một cấu trúc dữ liệu là hành vi không xác định (undefined behavior). Bạn có thể nhận được bất cứ thứ gì tại vị trí trong bộ nhớ
tương ứng với phần tử đó trong cấu trúc dữ liệu, mặc dù bộ nhớ đó
không thuộc về cấu trúc đó. Đây được gọi là _đọc quá giới hạn bộ đệm_ (buffer overread) và có thể
dẫn đến các lỗ hổng bảo mật nếu kẻ tấn công có thể thao túng chỉ số
theo cách để đọc dữ liệu mà chúng không được phép truy cập vốn được lưu trữ sau
cấu trúc dữ liệu đó.

Để bảo vệ chương trình của bạn khỏi loại lỗ hổng này, nếu bạn cố gắng đọc một
phần tử tại một chỉ số không tồn tại, Rust sẽ dừng thực thi và từ chối
tiếp tục. Hãy thử và xem kết quả:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Lỗi này chỉ vào dòng 4 trong file _main.rs_ của chúng ta, nơi chúng ta cố gắng truy cập chỉ số
`99` của vector trong `v`.

Dòng `note:` cho chúng ta biết rằng chúng ta có thể đặt biến môi trường `RUST_BACKTRACE`
để có được một truy vết ngược về chính xác những gì đã xảy ra gây ra lỗi. Một
_truy vết ngược_ (backtrace) là danh sách tất cả các hàm đã được gọi để đi đến
điểm này. Truy vết ngược trong Rust hoạt động giống như trong các ngôn ngữ khác: chìa khóa để
đọc truy vết ngược là bắt đầu từ trên cùng và đọc cho đến khi bạn thấy các file mà
bạn đã viết. Đó là nơi vấn đề bắt nguồn. Các dòng phía trên điểm đó
là mã mà mã của bạn đã gọi; các dòng phía dưới là mã đã gọi
mã của bạn. Những dòng trước và sau này có thể bao gồm mã cốt lõi của Rust, mã thư viện tiêu chuẩn,
hoặc các crate mà bạn đang sử dụng. Hãy thử lấy một truy vết ngược bằng cách
đặt biến môi trường `RUST_BACKTRACE` thành bất kỳ giá trị nào ngoại trừ `0`.
Listing 9-2 hiển thị kết quả tương tự như những gì bạn sẽ thấy.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="Truy vết ngược được tạo ra bởi một lời gọi đến `panic!` được hiển thị khi biến môi trường `RUST_BACKTRACE` được thiết lập">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

Đó là một kết quả rất dài! Kết quả chính xác bạn thấy có thể khác nhau tùy thuộc
vào hệ điều hành và phiên bản Rust của bạn. Để có được các truy vết ngược với
thông tin này, các biểu tượng debug (debug symbols) phải được bật. Các biểu tượng debug được bật theo
mặc định khi sử dụng `cargo build` hoặc `cargo run` mà không có cờ `--release`,
như chúng ta đang làm ở đây.

Trong kết quả ở Listing 9-2, dòng 6 của truy vết ngược chỉ vào dòng trong
dự án của chúng ta đang gây ra sự cố: dòng 4 của _src/main.rs_. Nếu chúng ta không muốn
chương trình của mình panic, chúng ta nên bắt đầu cuộc điều tra tại vị trí được chỉ ra
bởi dòng đầu tiên đề cập đến một file mà chúng ta đã viết. Trong Listing 9-1, nơi chúng ta
cố tình viết mã gây panic, cách để khắc phục panic là không
yêu cầu một phần tử vượt quá phạm vi của các chỉ số vector. Khi mã của bạn
panic trong tương lai, bạn sẽ cần tìm ra hành động nào mã đang thực hiện
với các giá trị nào để gây ra panic và mã nên làm gì thay thế.

Chúng ta sẽ quay lại với `panic!` và khi nào chúng ta nên và không nên sử dụng `panic!` để
xử lý các điều kiện lỗi trong phần [“Nên `panic!` hay không nên
`panic!`”][to-panic-or-not-to-panic]<!-- ignore --> sau này trong
chương này. Tiếp theo, chúng ta sẽ xem xét cách phục hồi từ một lỗi bằng cách sử dụng `Result`.

{{#quiz ../quizzes/ch09-01-panic.toml}}

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
