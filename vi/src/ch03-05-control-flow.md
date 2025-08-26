## Luồng điều khiển

Khả năng chạy một đoạn mã tùy thuộc vào việc một điều kiện là `true` và chạy lặp đi lặp lại khi điều kiện là `true` là những khối xây dựng cơ bản trong hầu hết các ngôn ngữ lập trình. Các cấu trúc phổ biến nhất giúp bạn điều khiển luồng thực thi của mã Rust là các biểu thức `if` và các vòng lặp.

### Biểu thức `if`

Một biểu thức `if` cho phép bạn rẽ nhánh mã tùy theo điều kiện. Bạn cung cấp một điều kiện rồi nêu: “Nếu điều kiện này thỏa, chạy khối mã này. Nếu điều kiện không thỏa, đừng chạy khối mã này.”

Tạo một dự án mới tên là _branches_ trong thư mục _projects_ của bạn để khám phá biểu thức `if`. Trong tệp _src/main.rs_, nhập nội dung sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Tất cả các biểu thức `if` bắt đầu với từ khóa `if`, theo sau là một điều kiện. Trong trường hợp này, điều kiện kiểm tra xem biến `number` có giá trị nhỏ hơn 5 hay không. Chúng ta đặt khối mã cần thực thi nếu điều kiện là `true` ngay sau điều kiện, bên trong dấu ngoặc nhọn. Các khối mã gắn với điều kiện trong biểu thức `if` đôi khi được gọi là _arm_, tương tự như các arm trong biểu thức `match` mà chúng ta đã bàn trong phần [“So sánh lần đoán với số bí mật”][comparing-the-guess-to-the-secret-number]<!--
ignore --> của Chương 2.

Tùy chọn, chúng ta cũng có thể thêm một biểu thức `else`, như ở đây, để cung cấp cho chương trình một khối mã thay thế để thực thi nếu điều kiện đánh giá là `false`. Nếu bạn không cung cấp `else` và điều kiện là `false`, chương trình sẽ chỉ bỏ qua khối `if` và tiếp tục phần mã tiếp theo.

Hãy chạy đoạn mã này; bạn sẽ thấy kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Hãy thử đổi giá trị của `number` sang một giá trị khiến điều kiện `false` để xem điều gì xảy ra:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Chạy lại chương trình và xem kết quả:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Cũng đáng lưu ý là điều kiện trong đoạn mã này _bắt buộc_ phải là `bool`. Nếu điều kiện không phải là `bool`, chúng ta sẽ nhận lỗi. Ví dụ, thử chạy đoạn mã sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

Lần này điều kiện `if` đánh giá ra giá trị `3`, và Rust báo lỗi:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Lỗi cho biết Rust mong đợi `bool` nhưng nhận được một số nguyên. Không giống các ngôn ngữ như Ruby và JavaScript, Rust sẽ không tự động cố chuyển các kiểu không phải Boolean thành Boolean. Bạn phải rõ ràng và luôn cung cấp một giá trị Boolean làm điều kiện cho `if`. Nếu chúng ta muốn khối `if` chỉ chạy khi một số khác `0`, chẳng hạn, ta có thể đổi biểu thức `if` như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Chạy đoạn mã này sẽ in `number was something other than zero`.

#### Xử lý nhiều điều kiện với `else if`

Bạn có thể dùng nhiều điều kiện bằng cách kết hợp `if` và `else` trong một biểu thức `else if`. Ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Chương trình này có bốn đường đi có thể thực hiện. Sau khi chạy, bạn sẽ thấy kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Khi chương trình thực thi, nó kiểm tra từng biểu thức `if` theo thứ tự và thực thi khối đầu tiên có điều kiện đánh giá là `true`. Lưu ý rằng mặc dù 6 chia hết cho 2, chúng ta không thấy dòng `number is divisible by 2`, cũng không thấy dòng `number is not divisible by 4, 3, or 2` từ khối `else`. Đó là vì Rust chỉ thực thi khối cho điều kiện `true` đầu tiên, và khi đã tìm thấy, nó sẽ không kiểm tra các điều kiện còn lại.

Dùng quá nhiều `else if` có thể khiến mã rối rắm, nên nếu bạn có nhiều hơn một `else if`, bạn có thể muốn tái cấu trúc. Chương 6 mô tả một cấu trúc rẽ nhánh mạnh mẽ trong Rust gọi là `match` cho những trường hợp như vậy.

#### Dùng `if` trong câu lệnh `let`

Vì `if` là một biểu thức, ta có thể dùng nó ở vế phải của câu lệnh `let` để gán kết quả cho một biến, như trong Liệt kê 3-2.

<Listing number="3-2" file-name="src/main.rs" caption="Gán kết quả của một biểu thức `if` cho biến">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

Biến `number` sẽ được gán một giá trị dựa trên kết quả của biểu thức `if`. Hãy chạy đoạn mã này để xem điều gì xảy ra:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Hãy nhớ rằng các khối mã được đánh giá thành biểu thức cuối cùng trong khối, và các con số tự chúng cũng là biểu thức. Trong trường hợp này, giá trị của toàn bộ biểu thức `if` phụ thuộc vào khối nào được thực thi. Điều này có nghĩa là các giá trị có thể trở thành kết quả từ mỗi arm của `if` phải cùng kiểu; trong Liệt kê 3-2, kết quả của cả arm `if` và arm `else` đều là số nguyên `i32`. Nếu các kiểu không trùng khớp, như trong ví dụ sau, chúng ta sẽ nhận lỗi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Khi cố biên dịch đoạn mã này, chúng ta sẽ nhận lỗi. Các arm `if` và `else` có kiểu giá trị không tương thích, và Rust chỉ ra chính xác vị trí có vấn đề trong chương trình:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

Biểu thức trong khối `if` đánh giá thành một số nguyên, còn biểu thức trong khối `else` đánh giá thành một chuỗi. Điều này sẽ không hoạt động vì biến phải có một kiểu duy nhất, và Rust cần biết kiểu của biến `number` ngay tại thời điểm biên dịch, một cách dứt khoát. Biết kiểu của `number` cho phép trình biên dịch xác minh kiểu đó là hợp lệ ở mọi nơi ta dùng `number`. Rust sẽ không thể làm điều đó nếu kiểu của `number` chỉ được xác định lúc chạy; trình biên dịch sẽ phức tạp hơn và đưa ra ít đảm bảo hơn về mã nếu nó phải theo dõi nhiều kiểu giả định cho bất kỳ biến nào.

### Lặp lại với các vòng lặp

Thường thì chúng ta cần thực thi một khối mã nhiều hơn một lần. Để làm việc này, Rust cung cấp nhiều _vòng lặp_, chúng sẽ chạy qua đoạn mã bên trong thân vòng lặp đến cuối rồi ngay lập tức bắt đầu lại từ đầu. Để thử nghiệm với vòng lặp, hãy tạo một dự án mới tên _loops_.

Rust có ba loại vòng lặp: `loop`, `while`, và `for`. Hãy thử từng loại.

#### Lặp lại mã với `loop`

Từ khóa `loop` bảo Rust thực thi một khối mã lặp đi lặp lại mãi mãi hoặc cho đến khi bạn bảo nó dừng lại một cách tường minh.

Ví dụ, hãy đổi tệp _src/main.rs_ trong thư mục _loops_ của bạn thành như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Khi chạy chương trình này, chúng ta sẽ thấy `again!` được in ra liên tục cho đến khi chúng ta dừng chương trình thủ công. Hầu hết terminal hỗ trợ phím tắt <kbd>ctrl</kbd>-<kbd>c</kbd> để ngắt một chương trình bị kẹt trong vòng lặp vô tận. Hãy thử nhé:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
  Compiling loops v0.1.0 (file:///projects/loops)
   Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
    Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

Ký hiệu `^C` biểu thị nơi bạn đã nhấn <kbd>ctrl</kbd>-<kbd>c</kbd>.

Bạn có thể thấy hoặc không thấy chữ `again!` được in sau `^C`, tùy thuộc vào việc mã đang ở đâu trong vòng lặp khi nhận tín hiệu ngắt.

May mắn thay, Rust cũng cung cấp cách thoát khỏi vòng lặp bằng mã. Bạn có thể đặt từ khóa `break` bên trong vòng lặp để cho chương trình biết khi nào dừng thực thi vòng lặp. Hãy nhớ rằng chúng ta đã làm điều này trong trò chơi đoán số ở phần [“Thoát sau khi đoán đúng”][quitting-after-a-correct-guess]<!-- ignore --> của Chương 2 để thoát chương trình khi người dùng thắng trò chơi bằng cách đoán đúng số.

Chúng ta cũng đã dùng `continue` trong trò chơi đoán số, thứ mà trong vòng lặp sẽ bảo chương trình bỏ qua phần mã còn lại của vòng lặp hiện tại và chuyển sang vòng lặp kế tiếp.

#### Trả về giá trị từ vòng lặp

Một trong những cách dùng của `loop` là thử lại một thao tác mà bạn biết có thể thất bại, chẳng hạn như kiểm tra xem một thread đã hoàn thành công việc hay chưa. Bạn cũng có thể cần truyền kết quả của thao tác đó ra khỏi vòng lặp cho phần mã còn lại. Để làm điều này, bạn có thể thêm giá trị muốn trả về sau biểu thức `break` bạn dùng để dừng vòng lặp; giá trị đó sẽ được trả về từ vòng lặp để bạn dùng, như sau:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Trước vòng lặp, chúng ta khai báo một biến tên `counter` và khởi tạo nó bằng `0`. Sau đó khai báo một biến tên `result` để giữ giá trị trả về từ vòng lặp. Ở mỗi lần lặp, chúng ta cộng `1` vào `counter`, rồi kiểm tra xem `counter` có bằng `10` hay không. Khi bằng, chúng ta dùng từ khóa `break` với giá trị `counter * 2`. Sau vòng lặp, chúng ta dùng dấu chấm phẩy để kết thúc câu lệnh gán giá trị cho `result`. Cuối cùng, chúng ta in giá trị trong `result`, trong trường hợp này là `20`.

Bạn cũng có thể `return` từ bên trong một vòng lặp. Trong khi `break` chỉ thoát vòng lặp hiện tại, `return` luôn thoát khỏi hàm hiện tại.

#### Nhãn vòng lặp để phân biệt giữa nhiều vòng lặp

Nếu bạn có các vòng lặp lồng nhau, `break` và `continue` áp dụng cho vòng lặp trong cùng tại thời điểm đó. Bạn có thể tùy chọn chỉ định một _nhãn vòng lặp_ cho một vòng lặp, rồi dùng với `break` hoặc `continue` để chỉ định rằng các từ khóa đó áp dụng cho vòng lặp được gắn nhãn thay vì vòng lặp trong cùng. Nhãn vòng lặp phải bắt đầu bằng một dấu nháy đơn. Đây là một ví dụ với hai vòng lặp lồng nhau:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Vòng lặp ngoài có nhãn `'counting_up`, và nó sẽ đếm tăng từ 0 đến 2. Vòng lặp trong không có nhãn đếm giảm từ 10 xuống 9. Lệnh `break` đầu tiên không chỉ rõ nhãn sẽ chỉ thoát vòng lặp trong. Câu lệnh `break 'counting_up;` sẽ thoát vòng lặp ngoài. Đoạn mã này in ra:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

#### Vòng lặp có điều kiện với `while`

Một chương trình thường cần đánh giá một điều kiện bên trong vòng lặp. Khi điều kiện là `true`, vòng lặp chạy. Khi điều kiện không còn `true`, chương trình gọi `break`, dừng vòng lặp. Có thể hiện thực hành vi như vậy bằng cách kết hợp `loop`, `if`, `else`, và `break`; bạn có thể thử tự làm nếu muốn. Tuy nhiên, mẫu này phổ biến đến mức Rust có một cấu trúc ngôn ngữ dựng sẵn cho nó, gọi là vòng lặp `while`. Trong Liệt kê 3-3, chúng ta dùng `while` để lặp chương trình ba lần, đếm ngược mỗi lần, rồi sau vòng lặp in một thông điệp và thoát.

<Listing number="3-3" file-name="src/main.rs" caption="Dùng vòng lặp `while` để chạy mã khi một điều kiện đánh giá là `true`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Cấu trúc này loại bỏ nhiều tầng lồng nhau nếu bạn dùng `loop`, `if`, `else`, và `break`, và nó rõ ràng hơn. Chừng nào điều kiện đánh giá là `true`, mã sẽ chạy; nếu không, nó thoát vòng lặp.

#### Lặp qua một tập hợp với `for`

Bạn có thể chọn dùng `while` để lặp qua các phần tử của một tập hợp, như một mảng. Ví dụ, vòng lặp trong Liệt kê 3-4 in từng phần tử trong mảng `a`.

<Listing number="3-4" file-name="src/main.rs" caption="Lặp qua từng phần tử của một tập hợp bằng vòng lặp `while`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Ở đây, mã đếm tăng qua các phần tử trong mảng. Nó bắt đầu tại chỉ mục `0`, rồi lặp cho đến khi đạt đến chỉ mục cuối cùng trong mảng (nghĩa là khi `index < 5` không còn `true`). Chạy đoạn mã này sẽ in mọi phần tử trong mảng:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Cả năm giá trị của mảng xuất hiện trong terminal, như mong đợi. Mặc dù `index` sẽ đạt giá trị `5` tại một thời điểm, vòng lặp dừng thực thi trước khi cố lấy phần tử thứ sáu từ mảng.

Tuy nhiên, cách tiếp cận này dễ lỗi; chúng ta có thể khiến chương trình panic nếu giá trị chỉ mục hoặc điều kiện kiểm tra không đúng. Ví dụ, nếu bạn đổi định nghĩa mảng `a` thành có bốn phần tử nhưng quên cập nhật điều kiện thành `while index < 4`, mã sẽ panic. Nó cũng chậm, vì trình biên dịch thêm mã chạy lúc thực thi để kiểm tra xem chỉ mục có nằm trong phạm vi mảng ở mỗi vòng lặp hay không.

Là một cách gọn gàng hơn, bạn có thể dùng vòng lặp `for` để thực thi một đoạn mã cho từng phần tử trong tập hợp. Vòng lặp `for` trông như mã trong Liệt kê 3-5.

<Listing number="3-5" file-name="src/main.rs" caption="Lặp qua từng phần tử của một tập hợp bằng vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Khi chạy đoạn mã này, chúng ta sẽ thấy đầu ra giống Liệt kê 3-4. Quan trọng hơn, chúng ta đã tăng độ an toàn cho mã và loại bỏ khả năng lỗi có thể xảy ra do vượt quá cuối mảng hoặc không đi đủ xa và bỏ sót một số phần tử. Mã máy được tạo từ vòng lặp `for` cũng có thể hiệu quả hơn, vì không cần so sánh chỉ mục với độ dài mảng ở mỗi vòng lặp.

Dùng vòng lặp `for`, bạn sẽ không phải nhớ đổi bất kỳ đoạn mã nào khác nếu bạn thay đổi số lượng phần tử trong mảng, như bạn phải làm với cách ở Liệt kê 3-4.

Tính an toàn và ngắn gọn của vòng lặp `for` khiến nó trở thành cấu trúc vòng lặp được dùng nhiều nhất trong Rust. Ngay cả trong những tình huống bạn muốn chạy một đoạn mã một số lần nhất định, như ví dụ đếm ngược dùng vòng lặp `while` trong Liệt kê 3-3, hầu hết Rustacean sẽ dùng vòng lặp `for`. Cách làm là dùng một `Range` do thư viện tiêu chuẩn cung cấp, tạo ra tất cả các số theo thứ tự bắt đầu từ một số và kết thúc trước một số khác.

Đoạn đếm ngược sẽ trông như thế này khi dùng vòng lặp `for` và một phương thức khác mà ta chưa bàn, `rev`, để đảo ngược range:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Đoạn mã này dễ chịu hơn một chút, phải không?

## Tóm tắt

Tuyệt vời! Đây là một chương khá dài: bạn đã học về biến, kiểu dữ liệu vô hướng và tổng hợp, hàm, chú thích, biểu thức `if`, và các vòng lặp! Để luyện tập các khái niệm trong chương này, hãy thử xây dựng các chương trình sau:

- Chuyển đổi nhiệt độ giữa Fahrenheit và Celsius.
- Tạo số Fibonacci thứ *n*.
- In lời bài hát Giáng Sinh “The Twelve Days of Christmas,” tận dụng sự lặp lại trong bài.

Khi sẵn sàng tiếp tục, chúng ta sẽ nói về một khái niệm trong Rust mà _thường_ không tồn tại ở các ngôn ngữ lập trình khác: ownership.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
