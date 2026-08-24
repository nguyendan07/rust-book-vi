## Luồng Điều Khiển (Control Flow)

Khả năng thực thi một đoạn mã tùy thuộc vào việc một điều kiện có `true` hay không và khả năng thực thi lặp đi lặp lại một đoạn mã trong khi một điều kiện là `true` là những khối xây dựng cơ bản trong hầu hết các ngôn ngữ lập trình. Các cấu trúc phổ biến nhất cho phép bạn kiểm soát luồng thực thi của mã nguồn Rust là các biểu thức `if` và các vòng lặp (loops).

### Biểu Thức `if`

Một biểu thức `if` cho phép bạn rẽ nhánh mã nguồn của mình tùy thuộc vào các điều kiện. Bạn cung cấp một điều kiện và sau đó chỉ định: “Nếu điều kiện này được thỏa mãn, hãy chạy khối mã này. Nếu điều kiện không được thỏa mãn, đừng chạy khối mã này.”

Tạo một dự án mới có tên là _branches_ trong thư mục _projects_ của bạn để khám phá biểu thức `if`. Trong tệp _src/main.rs_, nhập đoạn mã sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Tất cả các biểu thức `if` đều bắt đầu bằng từ khóa `if`, theo sau là một điều kiện. Trong trường hợp này, điều kiện kiểm tra xem biến `number` có giá trị nhỏ hơn 5 hay không. Chúng ta đặt khối mã sẽ thực thi nếu điều kiện là `true` ngay sau điều kiện bên trong cặp dấu ngoặc nhọn. Các khối mã liên kết với các điều kiện trong biểu thức `if` đôi khi được gọi là các _nhánh_ (arms), tương tự như các nhánh trong biểu thức `match` mà chúng ta đã thảo luận trong phần [“So Sánh Số Dự Đoán với Số Bí Mật”][comparing-the-guess-to-the-secret-number]<!-- ignore --> ở Chương 2.

Tùy chọn, chúng ta cũng có thể bao gồm một biểu thức `else`, điều mà chúng ta đã làm ở đây, để cung cấp cho chương trình một khối mã thay thế sẽ thực thi nếu điều kiện đánh giá thành `false`. Nếu bạn không cung cấp biểu thức `else` và điều kiện là `false`, chương trình sẽ chỉ đơn giản bỏ qua khối `if` và tiếp tục chuyển sang đoạn mã tiếp theo.

Hãy chạy thử đoạn mã này; bạn sẽ thấy kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Hãy thử thay đổi giá trị của `number` thành một giá trị làm cho điều kiện trở thành `false` để xem điều gì xảy ra:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Chạy lại chương trình và xem kết quả:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Điều đáng chú ý là điều kiện trong đoạn mã này **bắt buộc phải là một kiểu `bool`**. Nếu điều kiện không phải là một `bool`, chúng ta sẽ gặp lỗi. Ví dụ, hãy thử chạy đoạn mã sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

Điều kiện `if` lần này đánh giá thành giá trị `3`, và Rust báo lỗi:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Lỗi chỉ ra rằng Rust mong đợi một kiểu `bool` nhưng lại nhận được một số nguyên. Không giống như các ngôn ngữ như Ruby hay JavaScript (và Python), Rust **sẽ không tự động chuyển đổi các kiểu không phải Boolean thành Boolean** (không có khái niệm "truthy/falsy"). Bạn phải viết tường minh và luôn cung cấp cho `if` một giá trị Boolean làm điều kiện. Ví dụ: nếu chúng ta muốn khối mã `if` chỉ chạy khi một số khác `0`, chúng ta có thể thay đổi biểu thức `if` thành như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Chạy đoạn mã này sẽ in ra dòng chữ `number was something other than zero`.

#### Xử Lý Nhiều Điều Kiện với `else if`

Bạn có thể sử dụng nhiều điều kiện bằng cách kết hợp `if` và `else` trong một biểu thức `else if`. Ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Chương trình này có bốn hướng đi có thể thực hiện. Sau khi chạy nó, bạn sẽ thấy kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Khi chương trình này thực thi, nó kiểm tra lần lượt từng biểu thức `if` và thực thi thân của nhánh đầu tiên có điều kiện đánh giá thành `true`. Lưu ý rằng mặc dù 6 chia hết cho 2, chúng ta không thấy kết quả `number is divisible by 2`, cũng như không thấy văn bản `number is not divisible by 4, 3, or 2` từ khối `else`. Đó là vì Rust chỉ thực thi khối cho điều kiện `true` đầu tiên tìm thấy, và một khi đã tìm thấy, nó thậm chí không kiểm tra các điều kiện còn lại.

Sử dụng quá nhiều biểu thức `else if` có thể làm lộn xộn mã nguồn của bạn, vì vậy nếu bạn có nhiều điều kiện, bạn có thể muốn tái cấu trúc mã. Chương 6 mô tả một cấu trúc rẽ nhánh mạnh mẽ của Rust có tên là `match` dành cho các trường hợp này.

#### Sử Dụng `if` Trong Câu Lệnh `let`

Vì `if` là một biểu thức, chúng ta có thể sử dụng nó ở vế phải của một câu lệnh `let` để gán kết quả cho một biến, như trong Danh sách 3-2.

<Listing number="3-2" file-name="src/main.rs" caption="Gán kết quả của một biểu thức `if` cho một biến">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

Biến `number` sẽ được liên kết với một giá trị dựa trên kết quả của biểu thức `if`. Chạy đoạn mã này để xem điều gì xảy ra:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Hãy nhớ rằng các khối mã đánh giá thành biểu thức cuối cùng trong chúng, và bản thân các con số cũng là các biểu thức. Trong trường hợp này, giá trị của toàn bộ biểu thức `if` phụ thuộc vào khối mã nào được thực thi. Điều này có nghĩa là các giá trị có tiềm năng là kết quả từ mỗi nhánh của `if` **bắt buộc phải có cùng một kiểu dữ liệu**; trong Danh sách 3-2, kết quả của cả nhánh `if` và nhánh `else` đều là các số nguyên `i32`. Nếu các kiểu không khớp nhau, như trong ví dụ sau, chúng ta sẽ gặp lỗi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Khi chúng ta cố gắng biên dịch đoạn mã này, chúng ta sẽ gặp lỗi. Các nhánh `if` và `else` có các kiểu giá trị không tương thích:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

Biểu thức trong khối `if` đánh giá thành một số nguyên, và biểu thức trong khối `else` đánh giá thành một chuỗi. Điều này không hoạt động vì các biến bắt buộc phải có một kiểu duy nhất, và Rust cần biết chắc chắn tại thời điểm biên dịch biến `number` có kiểu gì.

{{#quiz ../quizzes/ch03-05-control-flow-sec1-if.toml}}

### Lặp Lại với Vòng Lặp (Loops)

Việc thực thi một khối mã nhiều lần là rất hữu ích. Đối với tác vụ này, Rust cung cấp một số loại _vòng lặp_ (loops), sẽ chạy qua đoạn mã bên trong thân vòng lặp cho đến cuối và sau đó ngay lập tức bắt đầu lại từ đầu. Để thử nghiệm với các vòng lặp, hãy tạo một dự án mới có tên là _loops_.

Rust có ba loại vòng lặp: `loop`, `while`, và `for`. Hãy cùng thử từng loại.

#### Lặp Lại Mã với `loop`

Từ khóa `loop` yêu cầu Rust thực thi một khối mã lặp đi lặp lại mãi mãi hoặc cho đến khi bạn yêu cầu nó dừng lại một cách tường minh.

Ví dụ: hãy thay đổi tệp _src/main.rs_ trong thư mục _loops_ của bạn thành như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Khi chúng ta chạy chương trình này, chúng ta sẽ thấy dòng chữ `again!` được in liên tục cho đến khi chúng ta dừng chương trình thủ công bằng tổ hợp phím <kbd>Ctrl</kbd>-<kbd>C</kbd>.

Rust cung cấp cách thoát khỏi một vòng lặp bằng mã nguồn: bạn có thể đặt từ khóa `break` bên trong vòng lặp để báo cho chương trình biết khi nào nên dừng thực thi vòng lặp. Bạn cũng có thể dùng từ khóa `continue` để bỏ qua các đoạn mã còn lại trong lần lặp hiện tại và chuyển ngay sang lần lặp tiếp theo.

#### Trả Về Giá Trị từ Vòng Lặp

Một trong những ứng dụng của `loop` là thử lại một thao tác mà bạn biết có thể thất bại, chẳng hạn như kiểm tra xem một luồng (thread) đã hoàn thành công việc của nó hay chưa. Bạn cũng có thể cần chuyển kết quả của thao tác đó ra khỏi vòng lặp đến phần còn lại của mã. Để làm điều này, bạn có thể thêm giá trị bạn muốn trả về sau biểu thức `break`; giá trị đó sẽ được trả về ra khỏi vòng lặp để bạn có thể sử dụng:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Trước vòng lặp, chúng ta khai báo một biến có tên là `counter` và khởi tạo nó bằng `0`. Sau đó, chúng ta khai báo một biến có tên là `result` để giữ giá trị trả về từ vòng lặp. Trong mỗi lần lặp của vòng lặp, chúng ta cộng `1` vào biến `counter`, và sau đó kiểm tra xem `counter` có bằng `10` hay không. Khi điều kiện thỏa mãn, chúng ta sử dụng từ khóa `break` với giá trị `counter * 2`. Sau vòng lặp, giá trị `20` được gán cho `result`.

Bạn cũng có thể sử dụng `return` từ bên trong một vòng lặp. Trong khi `break` chỉ thoát khỏi vòng lặp hiện tại, `return` luôn luôn thoát khỏi toàn bộ hàm hiện tại.

#### Nhãn Vòng Lặp để Phân Biệt Giữa Nhiều Vòng Lặp (Loop Labels)

Nếu bạn có các vòng lặp lồng nhau bên trong các vòng lặp khác, `break` và `continue` theo mặc định sẽ áp dụng cho vòng lặp trong cùng tại thời điểm đó. Bạn có thể tùy chọn chỉ định một _nhãn vòng lặp_ (loop label) trên một vòng lặp, sau đó sử dụng nhãn đó với `break` hoặc `continue` để chỉ định rằng các từ khóa đó áp dụng cho vòng lặp có nhãn thay vì vòng lặp trong cùng. Nhãn vòng lặp bắt buộc phải bắt đầu bằng một dấu nháy đơn. Dưới đây là một ví dụ với hai vòng lặp lồng nhau:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Vòng lặp bên ngoài có nhãn `'counting_up`, và nó sẽ đếm từ 0 đến 2. Vòng lặp bên trong không có nhãn đếm ngược từ 10 xuống 9. Lệnh `break` đầu tiên không chỉ định nhãn sẽ chỉ thoát khỏi vòng lặp bên trong. Câu lệnh `break 'counting_up;` sẽ thoát khỏi vòng lặp bên ngoài có nhãn. Đoạn mã này in ra kết quả:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

#### Vòng Lặp Có Điều Kiện với `while`

Một chương trình thường cần đánh giá một điều kiện bên trong một vòng lặp. Trong khi điều kiện là `true`, vòng lặp sẽ chạy. Khi điều kiện không còn là `true` nữa, chương trình sẽ dừng vòng lặp. Mẫu hình này phổ biến đến mức Rust có một cấu trúc ngôn ngữ tích hợp sẵn cho nó, được gọi là vòng lặp `while`. Trong Danh sách 3-3, chúng ta sử dụng `while` để lặp chương trình ba lần, đếm ngược mỗi lần, và sau đó in thông báo và thoát.

<Listing number="3-3" file-name="src/main.rs" caption="Sử dụng vòng lặp `while` để chạy mã trong khi một điều kiện đánh giá thành `true`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Trong khi điều kiện đánh giá thành `true`, đoạn mã sẽ chạy; nếu không, nó sẽ thoát khỏi vòng lặp.

#### Lặp Qua Một Tập Hợp với `for`

Bạn có thể sử dụng cấu trúc `while` để lặp qua các phần tử của một tập hợp dữ liệu, chẳng hạn như một mảng. Ví dụ: vòng lặp trong Danh sách 3-4 in từng phần tử trong mảng `a`.

<Listing number="3-4" file-name="src/main.rs" caption="Lặp qua từng phần tử của một tập hợp bằng vòng lặp `while`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Cách tiếp cận này rất dễ xảy ra lỗi: chúng ta có thể khiến chương trình panic nếu giá trị chỉ mục hoặc điều kiện kiểm tra không chính xác (ví dụ nếu đổi độ dài mảng mà quên đổi điều kiện). Nó cũng chậm hơn vì trình biên dịch phải chèn mã kiểm tra giới hạn chỉ mục tại mỗi lần lặp.

Để thay thế ngắn gọn và an toàn hơn, bạn có thể sử dụng vòng lặp `for` để thực thi mã cho từng mục trong một tập hợp dữ liệu. Vòng lặp `for` trông như trong Danh sách 3-5:

<Listing number="3-5" file-name="src/main.rs" caption="Lặp qua từng phần tử của một tập hợp bằng vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Khi chạy đoạn mã này, chúng ta sẽ thấy kết quả tương tự như trong Danh sách 3-4. Quan trọng hơn, chúng ta đã tăng cường tính an toàn của mã và loại bỏ hoàn toàn khả năng xảy ra lỗi do truy cập vượt quá giới hạn mảng.

Tính an toàn và súc tích của vòng lặp `for` làm cho chúng trở thành cấu trúc vòng lặp được sử dụng phổ biến nhất trong Rust. Ngay cả trong các tình huống mà bạn muốn chạy một đoạn mã một số lần nhất định, chẳng hạn như ví dụ đếm ngược sử dụng vòng lặp `while` trong Danh sách 3-3, hầu hết các Rustacean sẽ sử dụng vòng lặp `for` kết hợp với `Range` do thư viện chuẩn cung cấp cùng phương thức `rev` để đảo ngược dãy:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

{{#quiz ../quizzes/ch03-05-control-flow-sec2-loops.toml}}

## Tóm Tắt

Bạn đã hoàn thành một chương rất quan trọng! Trong chương này, bạn đã tìm hiểu về biến, các kiểu dữ liệu vô hướng và kết hợp, hàm, chú thích, biểu thức `if` và các vòng lặp! Để thực hành các khái niệm đã thảo luận trong chương này, hãy thử xây dựng các chương trình để:

- Chuyển đổi nhiệt độ giữa độ F và độ C.
- Tạo số Fibonacci thứ *n*.
- In lời bài hát mừng Giáng sinh “The Twelve Days of Christmas”, tận dụng tính chất lặp lại trong bài hát.

Khi bạn đã sẵn sàng tiếp tục, chúng ta sẽ tìm hiểu về một khái niệm trong Rust _không_ phổ biến trong các ngôn ngữ lập trình khác: **Quyền sở hữu (Ownership)**.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
