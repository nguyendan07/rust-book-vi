## Hàm

Các hàm xuất hiện rất nhiều trong mã Rust. Bạn đã thấy một trong những hàm quan trọng nhất của ngôn ngữ: hàm `main`, là điểm bắt đầu của nhiều chương trình. Bạn cũng đã thấy từ khóa `fn`, cho phép bạn khai báo các hàm mới.

Mã Rust sử dụng kiểu đặt tên _snake case_ như một phong cách quy ước cho tên hàm và biến, trong đó tất cả các chữ cái đều là chữ thường và dấu gạch dưới phân tách các từ. Dưới đây là một chương trình chứa một ví dụ định nghĩa hàm:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Chúng ta định nghĩa một hàm trong Rust bằng cách viết `fn` theo sau là tên hàm và một cặp dấu ngoặc tròn. Các dấu ngoặc nhọn cho trình biên dịch biết nơi phần thân hàm bắt đầu và kết thúc.

Chúng ta có thể gọi bất kỳ hàm nào đã định nghĩa bằng cách viết tên của nó theo sau là một cặp ngoặc tròn. Vì `another_function` được định nghĩa trong chương trình, nó có thể được gọi từ bên trong hàm `main`. Lưu ý rằng chúng ta định nghĩa `another_function` _sau_ hàm `main` trong mã nguồn; ta cũng có thể định nghĩa nó trước. Rust không quan tâm bạn định nghĩa hàm ở đâu, chỉ cần chúng được định nghĩa ở đâu đó trong một phạm vi mà người gọi có thể nhìn thấy.

Hãy bắt đầu một dự án nhị phân mới tên là _functions_ để khám phá thêm về hàm. Đặt ví dụ `another_function` vào _src/main.rs_ và chạy nó. Bạn sẽ thấy kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Các dòng được thực thi theo thứ tự mà chúng xuất hiện trong hàm `main`. Trước tiên, thông điệp “Hello, world!” được in ra, rồi `another_function` được gọi và thông điệp của nó được in ra.

### Tham số

Chúng ta có thể định nghĩa hàm có _tham số_, là các biến đặc biệt thuộc về chữ ký của hàm. Khi một hàm có tham số, bạn có thể cung cấp cho nó các giá trị cụ thể cho những tham số đó. Về mặt kỹ thuật, các giá trị cụ thể được gọi là _đối số_ (arguments), nhưng trong giao tiếp hàng ngày, mọi người thường dùng lẫn lộn các từ _parameter_ (tham số) và _argument_ (đối số) cho cả các biến trong định nghĩa hàm lẫn các giá trị cụ thể được truyền vào khi bạn gọi hàm.

Trong phiên bản `another_function` này, chúng ta thêm một tham số:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Hãy chạy chương trình này; bạn sẽ nhận được kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

Khai báo `another_function` có một tham số tên là `x`. Kiểu của `x` được chỉ định là `i32`. Khi chúng ta truyền `5` vào `another_function`, macro `println!` sẽ đặt `5` vào vị trí cặp ngoặc nhọn chứa `x` trong chuỗi định dạng.

Trong chữ ký hàm, bạn _phải_ khai báo kiểu của từng tham số. Đây là một quyết định có chủ đích trong thiết kế của Rust: yêu cầu chú thích kiểu ở định nghĩa hàm có nghĩa là trình biên dịch hầu như không bao giờ cần bạn sử dụng chúng ở nơi khác trong mã để suy ra kiểu bạn muốn. Trình biên dịch cũng có thể đưa ra các thông báo lỗi hữu ích hơn nếu nó biết hàm mong đợi những kiểu nào.

Khi định nghĩa nhiều tham số, hãy phân tách các khai báo tham số bằng dấu phẩy, như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Ví dụ này tạo một hàm tên `print_labeled_measurement` với hai tham số. Tham số đầu tiên tên là `value` và có kiểu `i32`. Tham số thứ hai tên là `unit_label` và có kiểu `char`. Hàm sau đó in ra văn bản chứa cả `value` và `unit_label`.

Hãy thử chạy đoạn mã này. Thay thế chương trình hiện có trong tệp _src/main.rs_ của dự án _functions_ bằng ví dụ ở trên và chạy nó bằng `cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Bởi vì chúng ta gọi hàm với `5` làm giá trị cho `value` và `'h'` làm giá trị cho `unit_label`, đầu ra của chương trình sẽ chứa các giá trị đó.

### Câu lệnh và Biểu thức

Phần thân hàm được tạo thành từ một chuỗi các câu lệnh và có thể kết thúc bằng một biểu thức. Cho đến giờ, các hàm mà chúng ta đã đề cập chưa bao gồm một biểu thức kết thúc, nhưng bạn đã thấy một biểu thức như một phần của câu lệnh. Vì Rust là một ngôn ngữ dựa trên biểu thức, đây là một khác biệt quan trọng cần hiểu. Các ngôn ngữ khác không có sự phân biệt tương tự, nên hãy xem câu lệnh và biểu thức là gì và sự khác nhau của chúng ảnh hưởng đến phần thân hàm như thế nào.

- Câu lệnh là các chỉ dẫn thực hiện một hành động và không trả về giá trị.
- Biểu thức được đánh giá thành một giá trị kết quả.

Hãy xem một vài ví dụ.

Thực ra chúng ta đã sử dụng câu lệnh và biểu thức rồi. Tạo một biến và gán giá trị cho nó bằng từ khóa `let` là một câu lệnh. Trong Liệt kê 3-1, `let y = 6;` là một câu lệnh.

<Listing number="3-1" file-name="src/main.rs" caption="Một khai báo hàm `main` chứa một câu lệnh">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Định nghĩa hàm cũng là các câu lệnh; toàn bộ ví dụ trước đó tự nó là một câu lệnh. (Như bạn sẽ thấy ở bên dưới, _gọi_ một hàm không phải là một câu lệnh.)

Các câu lệnh không trả về giá trị. Do đó, bạn không thể gán một câu lệnh `let` cho một biến khác, như đoạn mã sau cố gắng làm; bạn sẽ nhận lỗi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Khi bạn chạy chương trình này, lỗi bạn nhận được sẽ trông như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

Câu lệnh `let y = 6` không trả về giá trị, vì vậy không có gì để `x` có thể ràng buộc. Điều này khác với những gì xảy ra ở các ngôn ngữ khác, như C và Ruby, nơi phép gán trả về chính giá trị được gán. Trong những ngôn ngữ đó, bạn có thể viết `x = y = 6` và cả `x` lẫn `y` đều có giá trị `6`; điều đó không đúng trong Rust.

Biểu thức được đánh giá thành một giá trị và chiếm phần lớn phần còn lại của mã mà bạn sẽ viết trong Rust. Hãy xem xét một phép toán, chẳng hạn `5 + 6`, đó là một biểu thức đánh giá ra giá trị `11`. Biểu thức có thể là một phần của câu lệnh: trong Liệt kê 3-1, `6` trong câu lệnh `let y = 6;` là một biểu thức đánh giá ra giá trị `6`. Gọi một hàm là một biểu thức. Gọi một macro là một biểu thức. Một khối phạm vi mới được tạo bằng ngoặc nhọn là một biểu thức, ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Biểu thức này:

```rust,ignore
{
  let x = 3;
  x + 1
}
```

là một khối, trong trường hợp này, được đánh giá thành `4`. Giá trị đó được ràng buộc vào `y` như một phần của câu lệnh `let`. Lưu ý rằng dòng `x + 1` không có dấu chấm phẩy ở cuối, điều này khác với hầu hết các dòng bạn đã thấy từ trước đến nay. Biểu thức không bao gồm dấu chấm phẩy ở cuối. Nếu bạn thêm dấu chấm phẩy vào cuối một biểu thức, bạn biến nó thành một câu lệnh, và khi đó nó sẽ không trả về giá trị. Hãy ghi nhớ điều này khi bạn khám phá giá trị trả về của hàm và các biểu thức ở phần tiếp theo.

### Hàm với giá trị trả về

Hàm có thể trả về giá trị cho đoạn mã gọi chúng. Chúng ta không đặt tên cho giá trị trả về, nhưng phải khai báo kiểu của chúng sau một mũi tên (`->`). Trong Rust, giá trị trả về của hàm tương đương với giá trị của biểu thức cuối cùng trong khối phần thân của hàm. Bạn có thể trả về sớm từ một hàm bằng cách dùng từ khóa `return` và chỉ định một giá trị, nhưng hầu hết các hàm trả về biểu thức cuối cùng một cách ngầm định. Dưới đây là một ví dụ về một hàm trả về giá trị:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Không có lời gọi hàm, macro, hay thậm chí câu lệnh `let` nào trong hàm `five`—chỉ có số `5` đứng một mình. Đó là một hàm hoàn toàn hợp lệ trong Rust. Lưu ý rằng kiểu trả về của hàm cũng được chỉ định, là `-> i32`. Hãy thử chạy đoạn mã này; đầu ra sẽ như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

Số `5` trong `five` là giá trị trả về của hàm, đó là lý do tại sao kiểu trả về là `i32`. Hãy xem xét kỹ hơn. Có hai điểm quan trọng: đầu tiên, dòng `let x = five();` cho thấy chúng ta đang sử dụng giá trị trả về của một hàm để khởi tạo một biến. Bởi vì hàm `five` trả về `5`, dòng đó tương đương với:

```rust
let x = 5;
```

Thứ hai, hàm `five` không có tham số và định nghĩa kiểu của giá trị trả về, nhưng phần thân của hàm chỉ là một `5` đơn lẻ không có dấu chấm phẩy vì đó là một biểu thức có giá trị mà chúng ta muốn trả về.

Hãy xem một ví dụ khác:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Chạy đoạn mã này sẽ in `The value of x is: 6`. Nhưng nếu chúng ta đặt một dấu chấm phẩy ở cuối dòng chứa `x + 1`, biến nó từ một biểu thức thành một câu lệnh, chúng ta sẽ gặp lỗi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Biên dịch đoạn mã này sẽ tạo ra một lỗi như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Thông báo lỗi chính, `mismatched types`, cho thấy vấn đề cốt lõi của đoạn mã này. Định nghĩa của hàm `plus_one` nói rằng nó sẽ trả về một `i32`, nhưng các câu lệnh không được đánh giá thành một giá trị, điều này được biểu diễn bằng `()`, kiểu đơn vị. Do đó, không có gì được trả về, trái với định nghĩa của hàm và dẫn đến lỗi. Trong đầu ra này, Rust cung cấp một gợi ý để có thể khắc phục vấn đề: nó đề nghị xóa dấu chấm phẩy, điều này sẽ sửa lỗi.
