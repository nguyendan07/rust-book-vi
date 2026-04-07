## Tổ chức Test

Như đã đề cập ở đầu chương, kiểm thử (testing) là một kỷ luật phức tạp, và
những người khác nhau sử dụng các thuật ngữ và cách tổ chức khác nhau. Cộng đồng Rust
suy nghĩ về các bài test theo hai loại chính: unit test (test đơn vị) và integration
test (test tích hợp). _Unit test_ nhỏ và tập trung hơn, kiểm tra từng module một cách riêng biệt
tại một thời điểm, và có thể kiểm tra các giao diện riêng tư (private interfaces). _Integration test_ hoàn toàn
nằm bên ngoài thư viện của bạn và sử dụng mã của bạn theo cùng một cách như bất kỳ mã bên ngoài
nào khác, chỉ sử dụng giao diện công khai (public interface) và có khả năng thực thi nhiều
module trong mỗi bài test.

Viết cả hai loại test này là quan trọng để đảm bảo rằng các phần trong
thư viện của bạn đang thực hiện những gì bạn mong đợi, một cách riêng rẽ và khi kết hợp với nhau.

### Unit Test

Mục đích của unit test là để kiểm tra từng đơn vị mã một cách cô lập với
phần còn lại của mã để nhanh chóng xác định chính xác nơi mã đang hoạt động hoặc không hoạt động như
mong đợi. Bạn sẽ đặt các unit test trong thư mục _src_ trong mỗi tệp cùng với
mã mà chúng đang kiểm tra. Quy ước là tạo một module tên là `tests`
trong mỗi tệp để chứa các hàm test và đánh dấu module đó bằng
`cfg(test)`.

#### Module Tests và `#[cfg(test)]`

Chú thích `#[cfg(test)]` trên module `tests` bảo Rust chỉ biên dịch và
chạy mã test khi bạn chạy `cargo test`, chứ không phải khi bạn chạy `cargo
build`. Điều này giúp tiết kiệm thời gian biên dịch khi bạn chỉ muốn xây dựng thư viện và
tiết kiệm không gian trong thành phẩm biên dịch kết quả vì các bài test
không được bao gồm. Bạn sẽ thấy rằng vì các integration test nằm trong một
thư mục khác, chúng không cần chú thích `#[cfg(test)]`. Tuy nhiên, vì unit
test nằm trong cùng các tệp với mã nguồn, bạn sẽ sử dụng `#[cfg(test)]` để chỉ định
rằng chúng không nên được bao gồm trong kết quả biên dịch.

Hãy nhớ lại rằng khi chúng ta tạo dự án `adder` mới trong phần đầu tiên của
chương này, Cargo đã tạo mã này cho chúng ta:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Trên module `tests` được tạo tự động, thuộc tính `cfg` là viết tắt của
_configuration_ (cấu hình) và bảo Rust rằng item theo sau chỉ nên được bao gồm
khi có một tùy chọn cấu hình nhất định. Trong trường hợp này, tùy chọn cấu hình là
`test`, được cung cấp bởi Rust để biên dịch và chạy các bài test. Bằng cách sử dụng
thuộc tính `cfg`, Cargo chỉ biên dịch mã test của chúng ta nếu chúng ta chủ động chạy các bài test
với `cargo test`. Điều này bao gồm bất kỳ hàm trợ giúp nào có thể nằm trong
module này, bên cạnh các hàm được đánh dấu bằng `#[test]`.

#### Test các hàm riêng tư

Có sự tranh luận trong cộng đồng kiểm thử về việc liệu các hàm riêng tư
(private functions) có nên được kiểm tra trực tiếp hay không, và các ngôn ngữ khác gây khó khăn hoặc
không thể kiểm tra các hàm riêng tư. Bất kể bạn tuân theo hệ tư tưởng kiểm thử
nào, các quy tắc về tính riêng tư của Rust cho phép bạn kiểm tra các hàm riêng tư.
Hãy xem xét mã trong Liệt kê 11-12 với hàm riêng tư `internal_adder`.

<Listing number="11-12" file-name="src/lib.rs" caption="Kiểm tra một hàm riêng tư">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

Lưu ý rằng hàm `internal_adder` không được đánh dấu là `pub`. Các bài test chỉ
là mã Rust, và module `tests` chỉ là một module khác. Như chúng ta đã thảo luận trong
phần [“Đường dẫn để tham chiếu đến một Item trong cây Module”][paths]<!-- ignore -->,
các item trong module con có thể sử dụng các item trong module tổ tiên của chúng. Trong
bài test này, chúng ta đưa tất cả các item của module cha của module `tests` vào phạm vi bằng `use
super::*`, và sau đó bài test có thể gọi `internal_adder`. Nếu bạn không nghĩ
các hàm riêng tư nên được kiểm tra, không có gì trong Rust ép buộc bạn
phải làm như vậy.

### Integration Test

Trong Rust, integration test hoàn toàn nằm bên ngoài thư viện của bạn. Chúng sử dụng
thư viện của bạn theo cùng một cách như bất kỳ mã nào khác, điều đó có nghĩa là chúng chỉ có thể gọi
các hàm là một phần của API công khai của thư viện. Mục đích của chúng là để kiểm tra
liệu nhiều phần trong thư viện của bạn có hoạt động cùng nhau một cách chính xác hay không. Các đơn vị mã
hoạt động chính xác khi đứng một mình có thể gặp vấn đề khi được tích hợp, vì vậy
phạm vi kiểm tra (test coverage) của mã được tích hợp cũng rất quan trọng. Để tạo các integration
test, trước tiên bạn cần một thư mục _tests_.

#### Thư mục _tests_

Chúng ta tạo một thư mục _tests_ ở cấp cao nhất của thư mục dự án, bên cạnh
_src_. Cargo biết để tìm kiếm các tệp integration test trong thư mục này. Chúng ta
sau đó có thể tạo bao nhiêu tệp test tùy thích, và Cargo sẽ biên dịch mỗi
tệp như một crate riêng lẻ.

Hãy tạo một integration test. Với mã trong Liệt kê 11-12 vẫn còn trong
tệp _src/lib.rs_, hãy tạo một thư mục _tests_, và tạo một tệp mới tên là
_tests/integration_test.rs_. Cấu trúc thư mục của bạn nên trông như thế này:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Nhập mã trong Liệt kê 11-13 vào tệp _tests/integration_test.rs_.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="Một integration test của một hàm trong crate `adder` house">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

Mỗi tệp trong thư mục _tests_ là một crate riêng biệt, vì vậy chúng ta cần đưa
thư viện của mình vào phạm vi của mỗi crate test. Vì lý do đó chúng ta thêm `use
adder::add_two;` ở đầu mã, điều mà chúng ta không cần trong các unit test.

Chúng ta không cần đánh dấu bất kỳ mã nào trong _tests/integration_test.rs_ với
`#[cfg(test)]`. Cargo xử lý thư mục _tests_ một cách đặc biệt và biên dịch các tệp
trong thư mục này chỉ khi chúng ta chạy `cargo test`. Chạy `cargo test` ngay bây giờ:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Ba phần của đầu ra bao gồm unit test, integration test, và
doc test. Lưu ý rằng nếu bất kỳ bài test nào trong một phần bị thất bại, các phần tiếp theo
sẽ không được chạy. Ví dụ, nếu một unit test thất bại, sẽ không có bất kỳ đầu ra nào
cho integration test và doc test vì những bài test đó sẽ chỉ được chạy nếu tất cả unit
test đều vượt qua.

Phần đầu tiên cho các unit test giống như những gì chúng ta đã thấy: một dòng
cho mỗi unit test (một cái tên là `internal` mà chúng ta đã thêm trong Liệt kê 11-12) và
sau đó là một dòng tóm tắt cho các unit test.

Phần integration test bắt đầu với dòng `Running
tests/integration_test.rs`. Tiếp theo, có một dòng cho mỗi hàm test trong
integration test đó và một dòng tóm tắt cho kết quả của integration
test ngay trước khi phần `Doc-tests adder` bắt đầu.

Mỗi tệp integration test có phần riêng của nó, vì vậy nếu chúng ta thêm nhiều tệp hơn trong
thư mục _tests_, sẽ có nhiều phần integration test hơn.

Chúng ta vẫn có thể chạy một hàm integration test cụ thể bằng cách chỉ định tên của
hàm test đó làm đối số cho `cargo test`. Để chạy tất cả các bài test trong một
tệp integration test cụ thể, hãy sử dụng đối số `--test` của `cargo test`
theo sau là tên của tệp:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Lệnh này chỉ chạy các bài test trong tệp _tests/integration_test.rs_.

#### Các module con trong Integration Test

Khi bạn thêm nhiều integration test hơn, bạn có thể muốn tạo thêm nhiều tệp trong thư mục
_tests_ để giúp tổ chức chúng; ví dụ, bạn có thể nhóm các
hàm test theo chức năng mà chúng đang kiểm tra. Như đã đề cập trước đó, mỗi tệp
trong thư mục _tests_ được biên dịch như một crate riêng biệt của chính nó, điều này hữu ích
cho việc tạo các phạm vi riêng biệt để mô phỏng chặt chẽ hơn cách người dùng cuối sẽ
sử dụng crate của bạn. Tuy nhiên, điều này có nghĩa là các tệp trong thư mục _tests_ không
chia sẻ cùng một hành vi như các tệp trong _src_ làm, như bạn đã học ở Chương 7
liên quan đến cách chia mã thành các module và các tệp.

Hành vi khác nhau của các tệp trong thư mục _tests_ dễ nhận thấy nhất khi bạn
có một tập hợp các hàm trợ giúp để sử dụng trong nhiều tệp integration test và
bạn cố gắng làm theo các bước trong phần [“Tách các Module thành các
Tệp khác nhau”][separating-modules-into-files]<!-- ignore --> của Chương 7 để
trích xuất chúng vào một module chung. Ví dụ, nếu chúng ta tạo _tests/common.rs_
và đặt một hàm tên là `setup` trong đó, chúng ta có thể thêm một số mã vào `setup` mà
chúng ta muốn gọi từ nhiều hàm test trong nhiều tệp test:

<span class="filename">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Khi chúng ta chạy lại các bài test, chúng ta sẽ thấy một phần mới trong đầu ra test cho tệp
_common.rs_, mặc dù tệp này không chứa bất kỳ hàm test nào cũng như
chúng ta không gọi hàm `setup` từ bất kỳ đâu:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Việc `common` xuất hiện trong kết quả test với `running 0 tests` hiển thị cho
nó không phải là những gì chúng ta muốn. Chúng ta chỉ muốn chia sẻ một số mã với các tệp
integration test khác. Để tránh việc `common` xuất hiện trong đầu ra test,
thay vì tạo _tests/common.rs_, chúng ta sẽ tạo _tests/common/mod.rs_. Thư mục
dự án bây giờ trông như thế này:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Đây là quy ước đặt tên cũ hơn mà Rust cũng hiểu mà chúng ta đã đề cập
trong phần [“Các đường dẫn tệp thay thế”][alt-paths]<!-- ignore --> ở Chương 7. Đặt tên
tệp theo cách này bảo Rust không xử lý module `common` như một tệp integration test
. Khi chúng ta chuyển mã hàm `setup` vào _tests/common/mod.rs_ và
xóa tệp _tests/common.rs_, phần đó trong đầu ra test sẽ không còn
xuất hiện nữa. Các tệp trong các thư mục con của thư mục _tests_ không được biên dịch dưới dạng
các crate riêng biệt hoặc có các phần trong đầu ra test.

Sau khi chúng ta đã tạo _tests/common/mod.rs_, chúng ta có thể sử dụng nó từ bất kỳ tệp
integration test nào như một module. Đây là một ví dụ về việc gọi hàm `setup`
từ bài test `it_adds_two` trong _tests/integration_test.rs_:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Lưu ý rằng khai báo `mod common;` giống với khai báo module
mà chúng ta đã trình bày trong Liệt kê 7-21. Sau đó, trong hàm test, chúng ta có thể gọi
hàm `common::setup()`.

#### Integration Test cho các Binary Crate

Nếu dự án của chúng ta là một binary crate chỉ chứa một tệp _src/main.rs_ và
không có tệp _src/lib.rs_, chúng ta không thể tạo các integration test trong thư mục
_tests_ và đưa các hàm được định nghĩa trong tệp _src/main.rs_ vào
phạm vi bằng một câu lệnh `use`. Chỉ các library crate mới để lộ các hàm mà các
crate khác có thể sử dụng; các binary crate được thiết kế để chạy độc lập.

Đây là một trong những lý do các dự án Rust cung cấp một bản thực thi (binary) có một tệp
_src/main.rs_ đơn giản gọi logic nằm trong tệp
_src/lib.rs_. Sử dụng cấu trúc đó, các integration test _có thể_ kiểm tra
library crate bằng `use` để làm cho các chức năng quan trọng trở nên khả dụng. Nếu
chức năng quan trọng hoạt động, một lượng nhỏ mã trong tệp _src/main.rs_
cũng sẽ hoạt động, và lượng nhỏ mã đó không cần phải được kiểm tra.

## Tổng kết

Các tính năng kiểm thử của Rust cung cấp một cách để chỉ định mã nên hoạt động như thế nào
nhằm đảm bảo nó tiếp tục hoạt động như bạn mong đợi, ngay cả khi bạn thực hiện các thay đổi. Unit test
thực thi các phần khác nhau của một thư viện một cách riêng biệt và có thể kiểm tra các chi tiết
triển khai riêng tư. Integration test kiểm tra xem liệu nhiều phần của thư viện có hoạt động
cùng nhau một cách chính xác hay không, và chúng sử dụng API công khai của thư viện để kiểm tra mã
theo cùng một cách mà mã bên ngoài sẽ sử dụng nó. Mặc dù hệ thống kiểu và
các quy tắc sở hữu (ownership rules) của Rust giúp ngăn chặn một số loại lỗi, các bài test vẫn quan trọng để
giảm thiểu các lỗi logic liên quan đến cách mã của bạn được kỳ vọng sẽ hành xử.

Hãy kết hợp kiến thức bạn đã học trong chương này và trong các chương trước
để thực hiện một dự án!

{{#quiz ../quizzes/ch11-03-test-organization.toml}}

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
