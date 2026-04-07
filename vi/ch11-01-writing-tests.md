## Cách Viết Các Bài Kiểm Thử (Test)

Các bài kiểm thử là các hàm Rust dùng để xác minh rằng mã nguồn đang hoạt động theo
cách mong đợi. Thân của các hàm thường được thực hiện với ba
hành động sau:

- Thiết lập bất kỳ dữ liệu hoặc trạng thái cần thiết nào.
- Chạy mã bạn muốn kiểm thử.
- Khẳng định (Assert) rằng kết quả đúng như những gì bạn mong đợi.

Hãy cùng xem các tính năng mà Rust cung cấp đặc biệt cho việc viết test để
thực hiện các hành động này, bao gồm thuộc tính (attribute) `test`, một vài macro, và
thuộc tính `should_panic`.

### Giải phẫu một hàm Test

Ở mức đơn giản nhất, một test trong Rust là một hàm được đánh dấu bằng thuộc tính
`test`. Các thuộc tính là siêu dữ liệu (metadata) về các đoạn mã Rust; một ví dụ là
thuộc tính `derive` mà chúng ta đã sử dụng với các struct trong Chương 5. Để chuyển một hàm
thành một hàm test, hãy thêm `#[test]` vào dòng trước `fn`. Khi bạn chạy các
test của mình bằng lệnh `cargo test`, Rust sẽ xây dựng một bản thực thi test runner
để chạy các hàm được đánh dấu và báo cáo về việc mỗi hàm test
vượt qua hay thất bại.

Bất cứ khi nào chúng ta tạo một dự án thư viện mới với Cargo, một module test với một hàm
test bên trong sẽ được tự động tạo cho chúng ta. Module này cung cấp cho bạn một
mẫu để viết các test để bạn không phải tra cứu cấu trúc và
cú pháp chính xác mỗi khi bắt đầu một dự án mới. Bạn có thể thêm bao nhiêu
hàm test bổ sung và bao nhiêu module test tùy thích!

Chúng ta sẽ khám phá một số khía cạnh về cách các test hoạt động bằng cách thử nghiệm với
test mẫu trước khi chúng ta thực sự test bất kỳ mã nào. Sau đó chúng ta sẽ viết một số
test thực tế gọi một số mã mà chúng ta đã viết và khẳng định rằng
hành vi của nó là chính xác.

Hãy tạo một dự án thư viện mới tên là `adder` để cộng hai số:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

Nội dung của tệp _src/lib.rs_ trong thư viện `adder` của bạn sẽ trông giống như
Liệt kê 11-1.

<Listing number="11-1" file-name="src/lib.rs" caption="Mã được tạo tự động bởi `cargo new` house">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

Tệp bắt đầu với một hàm `add` ví dụ, để chúng ta có cái gì đó
để test.

Hiện tại, hãy chỉ tập trung vào hàm `it_works`. Lưu ý chú thích `#[test]`
: thuộc tính này chỉ ra đây là một hàm test, vì vậy test
runner biết để xử lý hàm này như một test. Chúng ta cũng có thể có các hàm không phải
test trong module `tests` để giúp thiết lập các kịch bản chung hoặc thực hiện các
thao tác chung, vì vậy chúng ta luôn cần chỉ định hàm nào là test.

Thân hàm ví dụ sử dụng macro `assert_eq!` để khẳng định rằng `result`,
cái chứa kết quả của việc gọi `add` với 2 và 2, bằng 4. Khẳng định
này đóng vai trò như một ví dụ về định dạng cho một test điển hình. Hãy chạy nó
để thấy rằng test này vượt qua.

Lệnh `cargo test` chạy tất cả các test trong dự án của chúng ta, như được hiển thị trong Liệt kê
11-2.

<Listing number="11-2" caption="Đầu ra từ việc chạy test được tạo tự động">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo đã biên dịch và chạy test. Chúng ta thấy dòng `running 1 test`. Dòng tiếp
theo hiển thị tên của hàm test được tạo, gọi là `tests::it_works`,
và kết quả của việc chạy test đó là `ok`. Tóm tắt tổng thể `test
result: ok.` có nghĩa là tất cả các test đã vượt qua, và phần ghi `1
passed; 0 failed` tổng hợp số lượng test đã vượt qua hoặc thất bại.

Có thể đánh dấu một test là bị bỏ qua (ignored) để nó không chạy trong một
trường hợp cụ thể; chúng ta sẽ đề cập đến điều đó trong phần [“Bỏ qua một số test trừ khi được
yêu cầu cụ thể”][ignoring]<!-- ignore --> sau này trong chương này. Bởi vì chúng ta
chưa làm điều đó ở đây, bản tóm tắt hiển thị `0 ignored`. Chúng ta cũng có thể truyền một
đối số cho lệnh `cargo test` để chỉ chạy các test có tên khớp với một
chuỗi; điều này được gọi là _lọc_ (filtering) và chúng ta sẽ đề cập đến điều đó trong phần [“Chạy một
tập hợp con các test theo tên”][subset]<!-- ignore -->. Ở đây chúng ta chưa lọc các
test đang chạy, vì vậy phần cuối của bản tóm tắt hiển thị `0 filtered out`.

Thống kê `0 measured` dành cho các bài test hiệu năng (benchmark tests) dùng để đo lường hiệu suất.
Các bài test benchmark, tại thời điểm viết bài này, chỉ có sẵn trong Rust nightly. Xem
[tài liệu về benchmark tests][bench] để tìm hiểu thêm.

Chúng ta có thể truyền một đối số cho lệnh `cargo test` để chỉ chạy các test có
tên khớp với một chuỗi; điều này được gọi là _lọc_ và chúng ta sẽ đề cập đến điều đó trong
phần [“Chạy một tập hợp con các test theo tên”][subset]<!-- ignore -->. Ở đây chúng
ta không lọc các test đang chạy, vì vậy phần cuối của bản tóm tắt hiển thị `0
filtered out`.

Phần tiếp theo của đầu ra test bắt đầu bằng `Doc-tests adder` là dành cho
kết quả của bất kỳ test tài liệu (documentation tests) nào. Chúng ta chưa có bất kỳ test tài liệu nào,
nhưng Rust có thể biên dịch bất kỳ ví dụ mã nào xuất hiện trong tài liệu API của chúng ta.
Tính năng này giúp giữ cho tài liệu và mã của bạn luôn đồng bộ! Chúng ta sẽ thảo luận về cách
viết test tài liệu trong phần [“Các bình luận tài liệu dưới dạng
Test”][doc-comments]<!-- ignore --> của Chương 14. Hiện tại, chúng ta sẽ
bỏ qua đầu ra `Doc-tests`.

Hãy bắt đầu tùy chỉnh test theo nhu cầu của chúng ta. Đầu tiên, hãy đổi tên của
hàm `it_works` thành một tên khác, chẳng hạn như `exploration`, như sau:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Sau đó chạy lại `cargo test`. Đầu ra bây giờ hiển thị `exploration` thay vì
`it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Bây giờ chúng ta sẽ thêm một test khác, nhưng lần này chúng ta sẽ tạo một test bị thất bại! Các
test thất bại khi một cái gì đó trong hàm test gây ra panic. Mỗi test được chạy trong một
luồng mới, và khi luồng chính thấy rằng một luồng test đã chết,
test đó được đánh dấu là thất bại. Trong Chương 9, chúng ta đã nói về cách đơn giản nhất để gây ra
panic là gọi macro `panic!`. Nhập test mới dưới dạng một hàm tên là
`another`, để tệp _src/lib.rs_ của bạn trông giống như Liệt kê 11-3.

<Listing number="11-3" file-name="src/lib.rs" caption="Thêm một test thứ hai sẽ thất bại vì chúng ta gọi macro `panic!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

Chạy lại các test bằng `cargo test`. Đầu ra sẽ trông giống như Liệt kê 11-4,
cho thấy rằng test `exploration` của chúng ta đã vượt qua và `another` đã thất bại.

<Listing number="11-4" caption="Kết quả test khi một test vượt qua và một test thất bại">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

Thay vì `ok`, dòng `test tests::another` hiển thị `FAILED`. Hai
phần mới xuất hiện giữa các kết quả riêng lẻ và bản tóm tắt: phần đầu tiên
hiển thị lý do chi tiết cho mỗi lần thất bại của test. Trong trường hợp này, chúng ta nhận được
thông tin chi tiết rằng `another` thất bại vì nó `panicked at 'Make this test fail'` trên
dòng 17 trong tệp _src/lib.rs_. Phần tiếp theo liệt kê chỉ tên của tất cả
các test bị thất bại, điều này hữu ích khi có nhiều test và nhiều đầu ra
test thất bại chi tiết. Chúng ta có thể sử dụng tên của một test bị thất bại để chạy chỉ
test đó nhằm debug nó dễ dàng hơn; chúng ta sẽ nói nhiều hơn về các cách chạy test trong
phần [“Kiểm soát cách chạy các Test”][controlling-how-tests-are-run]<!-- ignore
-->.

Dòng tóm tắt hiển thị ở cuối: nhìn chung, kết quả test của chúng ta là `FAILED`. Chúng
ta đã có một test vượt qua và một test thất bại.

Bây giờ bạn đã thấy kết quả test trông như thế nào trong các tình huống khác nhau,
hãy cùng xem một số macro khác ngoài `panic!` hữu ích trong các test.

### Kiểm tra kết quả với Macro `assert!`

Macro `assert!`, được cung cấp bởi thư viện chuẩn, hữu ích khi bạn muốn
đảm bảo rằng một điều kiện nào đó trong một test được đánh giá là `true`. Chúng ta đưa cho
macro `assert!` một đối số được đánh giá thành một giá trị Boolean. Nếu giá trị là
`true`, không có gì xảy ra và test vượt qua. Nếu giá trị là `false`, macro
`assert!` gọi `panic!` để làm cho test thất bại. Sử dụng macro `assert!`
giúp chúng ta kiểm tra xem mã của mình có hoạt động theo cách chúng ta dự định hay không.

Trong Chương 5, Liệt kê 5-15, chúng ta đã sử dụng một struct `Rectangle` và một phương thức
`can_hold`, chúng được lặp lại ở đây trong Liệt kê 11-5. Hãy đặt mã này vào
tệp _src/lib.rs_, sau đó viết một số test cho nó bằng macro `assert!`.

<Listing number="11-5" file-name="src/lib.rs" caption="Struct `Rectangle` và phương thức `can_hold` của nó từ Chương 5">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

Phương thức `can_hold` trả về một giá trị Boolean, điều này có nghĩa là nó là một trường hợp sử dụng hoàn hảo
cho macro `assert!`. Trong Liệt kê 11-6, chúng ta viết một test thực thi phương thức
`can_hold` bằng cách tạo một instance `Rectangle` có chiều rộng là 8 và
chiều cao là 7 và khẳng định rằng nó có thể chứa một instance `Rectangle` khác
có chiều rộng là 5 và chiều cao là 1.

<Listing number="11-6" file-name="src/lib.rs" caption="Một test cho `can_hold` kiểm tra xem một hình chữ nhật lớn hơn có thực sự có thể chứa một hình chữ nhật nhỏ hơn hay không">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

Lưu ý dòng `use super::*;` bên trong module `tests`. Module `tests` là
một module thông thường tuân theo các quy tắc hiển thị thông thường mà chúng ta đã đề cập trong Chương
7 trong phần [“Đường dẫn để tham chiếu đến một Item trong
cây Module”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->.
Bởi vì module `tests` là một module con, chúng ta cần đưa
mã đang được test trong module cha vào phạm vi của module con. Chúng ta sử dụng
một glob ở đây, vì vậy bất cứ thứ gì chúng ta định nghĩa trong module cha đều có sẵn cho
module `tests` này.

Chúng ta đã đặt tên cho test là `larger_can_hold_smaller`, và chúng ta đã tạo hai
instance `Rectangle` mà chúng ta cần. Sau đó chúng ta gọi macro `assert!` và
truyền cho nó kết quả của việc gọi `larger.can_hold(&smaller)`. Biểu thức này
được cho là sẽ trả về `true`, vì vậy test của chúng ta sẽ vượt qua. Hãy cùng tìm hiểu!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Nó thực sự vượt qua! Hãy thêm một test khác, lần này khẳng định rằng một
hình chữ nhật nhỏ hơn không thể chứa một hình chữ nhật lớn hơn:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Bởi vì kết quả đúng của hàm `can_hold` trong trường hợp này là `false`,
chúng ta cần phủ định kết quả đó trước khi truyền nó vào macro `assert!`. Kết
quả là, test của chúng ta sẽ vượt qua nếu `can_hold` trả về `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Hai test đã vượt qua! Bây giờ hãy xem điều gì xảy ra với kết quả test của chúng ta khi chúng ta
đưa một lỗi vào mã nguồn. Chúng ta sẽ thay đổi việc triển khai phương thức `can_hold`
bằng cách thay thế dấu lớn hơn bằng dấu nhỏ hơn khi nó
so sánh các chiều rộng:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Chạy các test bây giờ sẽ tạo ra kết quả sau:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Các test của chúng ta đã bắt được lỗi! Bởi vì `larger.width` là `8` và `smaller.width` là
`5`, việc so sánh các chiều rộng trong `can_hold` bây giờ trả về `false`: 8 không
nhỏ hơn 5.

### Kiểm tra sự bằng nhau với các Macro `assert_eq!` và `assert_ne!`

Một cách phổ biến để xác minh chức năng là kiểm tra sự bằng nhau giữa kết quả
của mã đang được test và giá trị bạn mong đợi mã đó trả về. Bạn có thể
làm điều này bằng cách sử dụng macro `assert!` và truyền cho nó một biểu thức sử dụng
toán tử `==`. Tuy nhiên, đây là một kiểu test phổ biến đến mức thư viện chuẩn
cung cấp một cặp macro—`assert_eq!` và `assert_ne!`—để thực hiện việc kiểm tra
này thuận tiện hơn. Các macro này so sánh hai đối số để xem chúng bằng nhau hoặc
không bằng nhau, tương ứng. Chúng cũng sẽ in hai giá trị nếu khẳng định
thất bại, điều này giúp dễ dàng thấy _tại sao_ test thất bại; ngược lại,
macro `assert!` chỉ chỉ ra rằng nó nhận được giá trị `false` cho
biểu thức `==`, mà không in các giá trị dẫn đến giá trị `false`.

Trong Liệt kê 11-7, chúng ta viết một hàm tên là `add_two` cộng
`2` vào tham số của nó, sau đó chúng ta test hàm này bằng macro `assert_eq!`.

<Listing number="11-7" file-name="src/lib.rs" caption="Test hàm `add_two` bằng macro `assert_eq!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Hãy kiểm tra xem nó có vượt qua không!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Chúng ta tạo một biến tên là `result` chứa kết quả của việc gọi
`add_two(2)`. Sau đó chúng ta truyền `result` và `4` làm các đối số cho `assert_eq!`.
Dòng đầu ra cho test này là `test tests::it_adds_two ... ok`, và văn bản `ok`
cho biết rằng test của chúng ta đã vượt qua!

Hãy đưa một lỗi vào mã của chúng ta để xem `assert_eq!` trông như thế nào khi nó
thất bại. Thay đổi việc triển khai hàm `add_two` để thay vào đó cộng thêm `3`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Chạy lại các test:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Test của chúng ta đã bắt được lỗi! Test `it_adds_two` đã thất bại, và thông báo cho chúng ta
biết rằng khẳng định bị thất bại là ``assertion `left == right` failed`` và các giá trị
`left` và `right` là gì. Thông báo này giúp chúng ta bắt đầu debug: đối số
`left`, nơi chúng ta có kết quả của việc gọi `add_two(2)`, là `5` nhưng
đối số `right` là `4`. Bạn có thể tưởng tượng rằng điều này sẽ
đặc biệt hữu ích khi chúng ta có nhiều test đang diễn ra.

Lưu ý rằng trong một số ngôn ngữ và khung kiểm thử, các tham số cho các hàm khẳng định
bằng nhau được gọi là `expected` và `actual`, và thứ tự mà
chúng ta chỉ định các đối số là quan trọng. Tuy nhiên, trong Rust, chúng được gọi là `left` và
`right`, và thứ tự mà chúng ta chỉ định giá trị chúng ta mong đợi và giá trị mà
mã tạo ra không quan trọng. Chúng ta có thể viết khẳng định trong test này là
`assert_eq!(add_two(2), result)`, điều này sẽ dẫn đến cùng một thông báo thất bại
hiển thị `` assertion failed: `(left == right)` ``.

Macro `assert_ne!` sẽ vượt qua nếu hai giá trị chúng ta đưa cho nó không bằng nhau và
thất bại nếu chúng bằng nhau. Macro này hữu ích nhất cho các trường hợp khi chúng ta không chắc chắn
một giá trị _sẽ_ là gì, nhưng chúng ta biết giá trị đó chắc chắn _không nên_ là gì.
Ví dụ, nếu chúng ta đang test một hàm được đảm bảo sẽ thay đổi đầu vào
của nó theo một cách nào đó, nhưng cách thức mà đầu vào được thay đổi phụ thuộc vào
ngày trong tuần mà chúng ta chạy các test, thì điều tốt nhất nên khẳng định có thể là
đầu ra của hàm không bằng đầu vào.

Bên dưới bề mặt, các macro `assert_eq!` và `assert_ne!` sử dụng các toán tử
`==` và `!=`, tương ứng. Khi các khẳng định thất bại, các macro này in
các đối số của chúng bằng cách sử dụng định dạng debug, điều này có nghĩa là các giá trị
được so sánh phải triển khai các trait `PartialEq` và `Debug`. Tất cả các kiểu nguyên thủy
và hầu hết các kiểu thư viện chuẩn đều triển khai các trait này. Đối với các struct và
enum mà bạn tự định nghĩa, bạn sẽ cần triển khai `PartialEq` để khẳng định sự bằng nhau
của các kiểu đó. Bạn cũng sẽ cần triển khai `Debug` để in các giá trị khi
khẳng định thất bại. Bởi vì cả hai trait đều là các trait có thể dẫn xuất, như đã đề cập trong
Liệt kê 5-12 trong Chương 5, việc này thường đơn giản như thêm
chú thích `#[derive(PartialEq, Debug)]` vào định nghĩa struct hoặc enum của bạn. Xem
Phụ lục C, [“Các Trait có thể dẫn xuất,”][derivable-traits]<!-- ignore --> để biết thêm
chi tiết về các trait này và các trait có thể dẫn xuất khác.

### Thêm các thông báo thất bại tùy chỉnh

Bạn cũng có thể thêm một thông báo tùy chỉnh để in cùng với thông báo thất bại dưới dạng các
đối số tùy chọn cho các macro `assert!`, `assert_eq!`, và `assert_ne!`. Bất kỳ
đối số nào được chỉ định sau các đối số bắt buộc đều được chuyển tiếp đến macro
`format!` (được thảo luận trong [“Nối chuỗi với Toán tử `+` hoặc
Macro `format!`”][concatenation-with-the--operator-or-the-format-macro]<!--
ignore --> ở Chương 8), vì vậy bạn có thể truyền một chuỗi định dạng chứa các
trình giữ chỗ `{}` và các giá trị để đưa vào các trình giữ chỗ đó. Các thông báo tùy chỉnh hữu ích
cho việc ghi lại ý nghĩa của một khẳng định; khi một test thất bại, bạn sẽ có một ý tưởng tốt hơn
về vấn đề nằm ở đâu trong mã.

Ví dụ, giả sử chúng ta có một hàm chào hỏi mọi người bằng tên và chúng ta
muốn test rằng cái tên chúng ta truyền vào hàm xuất hiện trong đầu ra:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Các yêu cầu cho chương trình này vẫn chưa được thống nhất, và chúng ta
khá chắc chắn rằng văn bản `Hello` ở đầu lời chào sẽ thay đổi. Chúng ta
quyết định không muốn phải cập nhật test khi các yêu cầu thay đổi,
vì vậy thay vì kiểm tra sự bằng nhau chính xác với giá trị được trả về từ
hàm `greeting`, chúng ta sẽ chỉ khẳng định rằng đầu ra chứa văn bản của
tham số đầu vào.

Bây giờ hãy đưa một lỗi vào mã này bằng cách thay đổi `greeting` để loại trừ
`name` nhằm xem lỗi test mặc định trông như thế nào:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Chạy test này tạo ra kết quả sau:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Kết quả này chỉ cho biết rằng khẳng định đã thất bại và khẳng định đó nằm ở dòng nào.
Một thông báo thất bại hữu ích hơn sẽ in giá trị từ
hàm `greeting`. Hãy thêm một thông báo thất bại tùy chỉnh bao gồm một chuỗi định dạng
với một trình giữ chỗ được điền bằng giá trị thực tế mà chúng ta nhận được từ
hàm `greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Bây giờ khi chúng ta chạy test, chúng ta sẽ nhận được một thông báo lỗi đầy đủ thông tin hơn:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Chúng ta có thể thấy giá trị mà chúng ta thực sự nhận được trong đầu ra test, điều này sẽ giúp chúng ta
debug những gì đã xảy ra thay vì những gì chúng ta mong đợi xảy ra.

### Kiểm tra Panic với `should_panic`

Ngoài việc kiểm tra các giá trị trả về, việc kiểm tra xem mã của chúng ta có xử lý
các điều kiện lỗi như chúng ta mong đợi hay không cũng rất quan trọng. Ví dụ, hãy xem xét kiểu `Guess`
mà chúng ta đã tạo trong Chương 9, Liệt kê 9-13. Mã khác sử dụng `Guess`
phụ thuộc vào sự đảm bảo rằng các instance `Guess` sẽ chỉ chứa các giá trị
từ 1 đến 100. Chúng ta có thể viết một test đảm bảo rằng việc cố gắng tạo một
instance `Guess` với giá trị nằm ngoài phạm vi đó sẽ gây ra panic.

Chúng ta thực hiện việc này bằng cách thêm thuộc tính `should_panic` vào hàm test của mình.
Test sẽ vượt qua nếu mã bên trong hàm gây ra panic; test sẽ thất bại nếu mã
bên trong hàm không gây ra panic.

Liệt kê 11-8 hiển thị một test kiểm tra xem các điều kiện lỗi của `Guess::new`
có xảy ra khi chúng ta mong đợi hay không.

<Listing number="11-8" file-name="src/lib.rs" caption="Test rằng một điều kiện sẽ gây ra một `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

Chúng ta đặt thuộc tính `#[should_panic]` sau thuộc tính `#[test]` và
trước hàm test mà nó áp dụng. Hãy xem kết quả khi test này
vượt qua:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Trông ổn đấy! Bây giờ hãy đưa một lỗi vào mã của chúng ta bằng cách xóa điều kiện
rằng hàm `new` sẽ panic nếu giá trị lớn hơn 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Khi chúng ta chạy test trong Liệt kê 11-8, nó sẽ thất bại:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Chúng ta không nhận được một thông báo hữu ích lắm trong trường hợp này, nhưng khi chúng ta nhìn vào
hàm test, chúng ta thấy rằng nó được chú thích bằng `#[should_panic]`. Sự thất bại mà chúng ta
nhận được có nghĩa là mã trong hàm test đã không gây ra panic.

Các test sử dụng `should_panic` có thể không chính xác. Một test `should_panic` sẽ
vượt qua ngay cả khi test đó panic vì một lý do khác với lý do chúng ta
mong đợi. Để làm cho các test `should_panic` chính xác hơn, chúng ta có thể thêm một tham số
`expected` tùy chọn vào thuộc tính `should_panic`. Test harness sẽ
đảm bảo rằng thông báo thất bại chứa văn bản được cung cấp. Ví dụ,
hãy xem xét mã đã sửa đổi cho `Guess` trong Liệt kê 11-9 nơi hàm `new`
gây ra panic với các thông báo khác nhau tùy thuộc vào việc giá trị quá nhỏ hoặc quá lớn.

<Listing number="11-9" file-name="src/lib.rs" caption="Test một `panic!` với thông báo panic chứa một chuỗi con được chỉ định">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Test này sẽ vượt qua vì giá trị chúng ta đặt trong tham số `expected` của thuộc tính
`should_panic` là một chuỗi con của thông báo mà hàm `Guess::new`
gây ra panic. Chúng ta có thể đã chỉ định toàn bộ thông báo panic mà chúng ta
mong đợi, trong trường hợp này sẽ là `Guess value must be less than or equal to
100, got 200`. Những gì bạn chọn để chỉ định phụ thuộc vào việc bao nhiêu phần của thông báo
panic là duy nhất hoặc động và mức độ chính xác mà bạn muốn test của mình đạt được. Trong
trường hợp này, một chuỗi con của thông báo panic là đủ để đảm bảo rằng mã trong
hàm test thực thi trường hợp `else if value > 100`.

Để xem điều gì xảy ra khi một test `should_panic` với thông báo `expected`
thất bại, hãy một lần nữa đưa một lỗi vào mã của chúng ta bằng cách hoán đổi thân của
các khối `if value < 1` và `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Lần này khi chúng ta chạy test `should_panic`, nó sẽ thất bại:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Thông báo thất bại cho biết rằng test này thực sự đã panic như chúng ta mong đợi,
nhưng thông báo panic không bao gồm chuỗi mong đợi `less than or equal
to 100`. Thông báo panic mà chúng ta nhận được trong trường hợp này là `Guess value must
be greater than or equal to 1, got 200.` Bây giờ chúng ta có thể bắt đầu tìm ra lỗi
của mình nằm ở đâu!

### Sử dụng `Result<T, E>` trong các Test

Các test của chúng ta từ đầu đến giờ đều panic khi chúng thất bại. Chúng ta cũng có thể viết các test sử dụng
`Result<T, E>`! Đây là test từ Liệt kê 11-1, được viết lại để sử dụng `Result<T,
E>` và trả về một `Err` thay vì gây ra panic:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

Hàm `it_works` bây giờ có kiểu trả về `Result<(), String>`. Trong
thân hàm, thay vì gọi macro `assert_eq!`, chúng ta trả về
`Ok(())` khi test vượt qua và một `Err` với một `String` bên trong khi test
thất bại.

Viết các test để chúng trả về một `Result<T, E>` cho phép bạn sử dụng toán tử dấu hỏi
chấm trong thân của các test, đây có thể là một cách thuận tiện để viết
các test nên thất bại nếu bất kỳ thao tác nào bên trong chúng trả về một biến thể `Err`.

Bạn không thể sử dụng chú thích `#[should_panic]` trên các test sử dụng `Result<T,
E>`. Để khẳng định rằng một thao tác trả về một biến thể `Err`, _đừng_ sử dụng
toán tử dấu hỏi chấm trên giá trị `Result<T, E>`. Thay vào đó, hãy sử dụng
`assert!(value.is_err())`.

Bây giờ bạn đã biết một vài cách để viết test, hãy cùng xem điều gì đang xảy ra
khi chúng ta chạy các test và khám phá các tùy chọn khác nhau mà chúng ta có thể sử dụng với `cargo
test`.

{{#quiz ../quizzes/ch11-01-writing-tests.toml}}

[concatenation-with-the--operator-or-the-format-macro]: ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
