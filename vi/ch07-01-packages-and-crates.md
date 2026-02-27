## Package và Crate

Các phần đầu tiên của hệ thống module mà chúng ta sẽ tìm hiểu là package và crate.

Một _crate_ là lượng mã nhỏ nhất mà trình biên dịch Rust xem xét tại một thời điểm. Ngay cả khi bạn chạy `rustc` thay vì `cargo` và truyền một file mã nguồn duy nhất
(như chúng ta đã làm từ “Viết và Chạy một Chương trình Rust” trong
Chương 1), trình biên dịch vẫn coi file đó là một crate. Các crate có thể chứa
các module, và các module có thể được định nghĩa trong các file khác được biên dịch cùng với
crate, như chúng ta sẽ thấy trong các phần sắp tới.

Một crate có thể ở một trong hai dạng: binary crate hoặc library crate.
_Binary crates_ là các chương trình bạn có thể biên dịch thành một tệp thực thi mà bạn có thể chạy,
chẳng hạn như một chương trình dòng lệnh hoặc một máy chủ. Mỗi binary crate phải có một hàm gọi là
`main` để định nghĩa những gì xảy ra khi tệp thực thi chạy. Tất cả các crate chúng ta đã
tạo cho đến nay đều là binary crate.

_Library crates_ không có hàm `main`, và chúng không biên dịch thành một tệp thực thi.
Thay vào đó, chúng định nghĩa các chức năng nhằm mục đích chia sẻ với
nhiều dự án. Ví dụ, crate `rand` mà chúng ta đã sử dụng trong [Chương
2][rand]<!-- ignore --> cung cấp chức năng tạo số ngẫu nhiên.
Hầu hết những người dùng Rust (Rustaceans) khi nói “crate” là họ ám chỉ library crate, và họ
sử dụng “crate” thay thế cho khái niệm lập trình chung là “thư viện” (library).

_Crate root_ là một file nguồn mà trình biên dịch Rust bắt đầu từ đó và tạo nên
module gốc của crate của bạn (chúng ta sẽ giải thích chi tiết về module trong [“Định nghĩa
Module để Kiểm soát Phạm vi và Tính Riêng tư”][modules]<!-- ignore -->).

Một _package_ là một nhóm gồm một hoặc nhiều crate cung cấp một tập hợp các
chức năng. Một package chứa một file _Cargo.toml_ mô tả cách
xây dựng các crate đó. Cargo thực chất là một package chứa binary crate
cho công cụ dòng lệnh mà bạn đang sử dụng để xây dựng mã nguồn của mình.
Package Cargo cũng chứa một library crate mà binary crate phụ thuộc vào. Các
dự án khác có thể phụ thuộc vào library crate của Cargo để sử dụng cùng một logic mà
công cụ dòng lệnh Cargo sử dụng.

Một package có thể chứa bao nhiêu binary crate tùy ý, nhưng tối đa chỉ có một
library crate. Một package phải chứa ít nhất một crate, cho dù đó là
library hay binary crate.

Hãy cùng xem qua những gì xảy ra khi chúng ta tạo một package. Đầu tiên chúng ta nhập
lệnh `cargo new my-project`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Sau khi chạy `cargo new my-project`, chúng ta sử dụng `ls` để xem những gì Cargo tạo ra. Trong
thư mục dự án, có một file _Cargo.toml_, cho chúng ta một package.
Cũng có một thư mục _src_ chứa _main.rs_. Mở _Cargo.toml_ trong
trình soạn thảo văn bản của bạn, và lưu ý rằng không có đề cập nào đến _src/main.rs_. Cargo tuân theo một
quy ước rằng _src/main.rs_ là crate root của một binary crate có cùng
tên với package. Tương tự, Cargo biết rằng nếu thư mục package
chứa _src/lib.rs_, thì package đó chứa một library crate có cùng tên
với package, và _src/lib.rs_ là crate root của nó. Cargo truyền các file crate root
cho `rustc` để xây dựng thư viện hoặc tệp thực thi.

Ở đây, chúng ta có một package chỉ chứa _src/main.rs_, nghĩa là nó chỉ
chứa một binary crate tên là `my-project`. Nếu một package chứa _src/main.rs_
và _src/lib.rs_, nó có hai crate: một binary và một library, cả hai đều có cùng
tên với package. Một package có thể có nhiều binary crate bằng cách đặt
các file trong thư mục _src/bin_: mỗi file sẽ là một binary crate riêng biệt.

{{#quiz ../quizzes/ch07-01-packages-and-crates.toml}}

[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
