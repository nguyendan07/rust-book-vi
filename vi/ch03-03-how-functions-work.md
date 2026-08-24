## Hàm (Functions)

Hàm xuất hiện rất phổ biến trong mã nguồn Rust. Bạn đã thấy một trong những hàm quan trọng nhất của ngôn ngữ: hàm `main`, là điểm khởi đầu (entry point) của nhiều chương trình. Bạn cũng đã thấy từ khóa `fn`, cho phép bạn khai báo các hàm mới.

Mã nguồn Rust sử dụng phong cách _snake case_ làm chuẩn quy ước cho tên hàm và tên biến, trong đó tất cả các chữ cái đều là chữ thường và các từ được phân tách bằng dấu gạch dưới. Dưới đây là một chương trình chứa định nghĩa hàm ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Chúng ta định nghĩa một hàm trong Rust bằng cách nhập `fn` theo sau là tên hàm và một cặp dấu ngoặc đơn. Cặp dấu ngoặc nhọn cho trình biên dịch biết nơi thân hàm bắt đầu và kết thúc.

Chúng ta có thể gọi bất kỳ hàm nào đã định nghĩa bằng cách nhập tên của nó theo sau là một cặp dấu ngoặc đơn. Vì `another_function` được định nghĩa trong chương trình, nó có thể được gọi từ bên trong hàm `main`. Lưu ý rằng chúng ta đã định nghĩa `another_function` _sau_ hàm `main` trong mã nguồn; chúng ta cũng hoàn toàn có thể định nghĩa nó trước hàm `main`. Rust không quan tâm bạn định nghĩa các hàm của mình ở đâu trong tệp, miễn là chúng được định nghĩa ở một nơi nào đó trong phạm vi mà bên gọi có thể nhìn thấy.

Hãy bắt đầu một dự án mới có tên là _functions_ để khám phá sâu hơn về hàm. Đặt ví dụ `another_function` vào _src/main.rs_ và chạy nó. Bạn sẽ thấy kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Các dòng mã được thực thi theo thứ tự xuất hiện trong hàm `main`. Đầu tiên thông báo “Hello, world!” được in ra, sau đó `another_function` được gọi và thông báo của nó được in ra.

### Tham Số (Parameters)

Chúng ta có thể định nghĩa các hàm có _tham số_ (parameters), là các biến đặc biệt nằm trong chữ ký của một hàm. Khi một hàm có tham số, bạn có thể cung cấp cho nó các giá trị cụ thể cho các tham số đó. Về mặt kỹ thuật, các giá trị cụ thể được truyền vào được gọi là _đối số_ (arguments), nhưng trong giao tiếp thông thường, mọi người có xu hướng sử dụng hai từ _tham số_ và _đối số_ thay thế cho nhau cho cả các biến trong định nghĩa hàm hoặc các giá trị cụ thể được truyền vào khi bạn gọi một hàm.

Trong phiên bản `another_function` này, chúng ta thêm một tham số:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Chạy chương trình này; bạn sẽ nhận được kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

Khai báo của `another_function` có một tham số tên là `x`. Kiểu của `x` được chỉ định là `i32`. Khi chúng ta truyền `5` vào `another_function`, macro `println!` sẽ đặt `5` vào vị trí của cặp dấu ngoặc nhọn chứa `x` trong chuỗi định dạng.

Trong chữ ký hàm, bạn **bắt buộc phải khai báo kiểu của từng tham số**. Đây là một quyết định có chủ đích trong thiết kế của Rust: việc yêu cầu chú thích kiểu trong các định nghĩa hàm giúp trình biên dịch hầu như không bao giờ cần bạn phải chú thích kiểu ở những nơi khác trong mã để tìm ra kiểu dữ liệu bạn muốn. Trình biên dịch cũng có thể đưa ra các thông báo lỗi hữu ích hơn nhiều nếu nó biết các kiểu dữ liệu mà hàm mong đợi.

Khi định nghĩa nhiều tham số, hãy phân tách các khai báo tham số bằng dấu phẩy, như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Ví dụ này tạo một hàm có tên là `print_labeled_measurement` với hai tham số. Tham số đầu tiên có tên là `value` và thuộc kiểu `i32`. Tham số thứ hai có tên là `unit_label` và thuộc kiểu `char`. Sau đó, hàm in văn bản chứa cả `value` và `unit_label`.

Hãy chạy thử đoạn mã này. Thay thế chương trình hiện tại trong tệp _src/main.rs_ của dự án _functions_ bằng ví dụ trên và chạy nó bằng `cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Vì chúng ta đã gọi hàm với `5` là giá trị cho `value` và `'h'` là giá trị cho `unit_label`, kết quả đầu ra của chương trình chứa các giá trị đó.

{{#quiz ../quizzes/ch03-03-functions-sec1-parameters.toml}}

### Câu Lệnh (Statements) và Biểu Thức (Expressions)

Thân hàm được tạo thành từ một loạt các câu lệnh (statements) và có thể tùy chọn kết thúc bằng một biểu thức (expression). Cho đến nay, các hàm chúng ta đã tìm hiểu chưa bao gồm biểu thức kết thúc, nhưng bạn đã thấy một biểu thức như một phần của câu lệnh. Vì Rust là một ngôn ngữ dựa trên biểu thức (expression-based language), đây là một sự phân biệt rất quan trọng cần hiểu rõ. Các ngôn ngữ khác không có sự phân biệt tương tự, vì vậy hãy xem xét câu lệnh và biểu thức là gì và sự khác biệt của chúng ảnh hưởng như thế nào đến thân hàm.

- **Câu lệnh (Statements)** là các chỉ thị thực hiện một hành động nào đó và **không trả về giá trị**.
- **Biểu thức (Expressions)** đánh giá và **cho ra một giá trị kết quả**.

Hãy cùng xem xét một số ví dụ.

Thực ra chúng ta đã sử dụng cả câu lệnh và biểu thức. Tạo một biến và gán một giá trị cho nó bằng từ khóa `let` là một câu lệnh. Trong Danh sách 3-1, `let y = 6;` là một câu lệnh.

<Listing number="3-1" file-name="src/main.rs" caption="Khai báo hàm `main` chứa một câu lệnh">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Các định nghĩa hàm cũng là câu lệnh; toàn bộ ví dụ trên tự thân nó là một câu lệnh. (Tuy nhiên, như chúng ta sẽ thấy bên dưới, _lời gọi_ hàm không phải là một câu lệnh mà là một biểu thức).

Các câu lệnh không trả về giá trị. Do đó, bạn không thể gán một câu lệnh `let` cho một biến khác, như đoạn mã sau đang cố gắng thực hiện; bạn sẽ gặp lỗi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Khi bạn chạy chương trình này, lỗi bạn nhận được sẽ trông như thế này:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

Câu lệnh `let y = 6` không trả về giá trị, vì vậy không có giá trị nào để `x` liên kết tới. Điều này khác với những gì xảy ra trong các ngôn ngữ khác như C và Ruby, nơi phép gán trả về chính giá trị của phép gán đó. Trong các ngôn ngữ đó, bạn có thể viết `x = y = 6` và cả `x` lẫn `y` đều có giá trị `6`; điều đó không được phép trong Rust.

Biểu thức đánh giá thành một giá trị và chiếm phần lớn phần còn lại của mã mà bạn sẽ viết trong Rust. Hãy xem xét một phép toán toán học, chẳng hạn như `5 + 6`, là một biểu thức đánh giá thành giá trị `11`. Biểu thức có thể là một phần của câu lệnh: trong Danh sách 3-1, số `6` trong câu lệnh `let y = 6;` là một biểu thức đánh giá thành giá trị `6`. Gọi một hàm là một biểu thức. Gọi một macro là một biểu thức. Một khối phạm vi mới được tạo bằng dấu ngoặc nhọn cũng là một biểu thức, ví dụ:

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

là một khối mã mà trong trường hợp này, đánh giá thành giá trị `4`. Giá trị đó được liên kết với `y` như một phần của câu lệnh `let`. Lưu ý rằng dòng `x + 1` **không có dấu chấm phẩy ở cuối**, khác với hầu hết các dòng bạn đã thấy cho đến nay. **Biểu thức không bao gồm dấu chấm phẩy ở cuối**. Nếu bạn thêm dấu chấm phẩy vào cuối một biểu thức, bạn sẽ biến nó thành một câu lệnh, và khi đó nó sẽ không trả về giá trị. Hãy ghi nhớ điều này khi chúng ta khám phá các hàm có giá trị trả về tiếp theo.

### Hàm Có Giá Trị Trả Về

Các hàm có thể trả về giá trị cho đoạn mã gọi chúng. Chúng ta không đặt tên cho các giá trị trả về, nhưng chúng ta phải khai báo kiểu của chúng sau một dấu mũi tên (`->`). Trong Rust, giá trị trả về của hàm đồng nghĩa với giá trị của biểu thức cuối cùng trong khối thân hàm. Bạn có thể trả về sớm từ một hàm bằng cách sử dụng từ khóa `return` và chỉ định một giá trị, nhưng hầu hết các hàm đều trả về biểu thức cuối cùng một cách ngầm định. Dưới đây là một ví dụ về một hàm trả về một giá trị:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Không có lời gọi hàm, macro, hay thậm chí câu lệnh `let` nào trong hàm `five` — chỉ có duy nhất số `5`. Đó là một hàm hoàn toàn hợp lệ trong Rust. Lưu ý rằng kiểu trả về của hàm cũng được chỉ định là `-> i32`. Chạy đoạn mã này; kết quả đầu ra sẽ như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

Số `5` trong `five` là giá trị trả về của hàm, đó là lý do tại sao kiểu trả về là `i32`. Thân hàm là một con số `5` đơn độc không có dấu chấm phẩy vì nó là một biểu thức có giá trị mà chúng ta muốn trả về.

Hãy cùng xem một ví dụ khác:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Chạy mã này sẽ in ra `The value of x is: 6`. Nhưng nếu chúng ta đặt một dấu chấm phẩy ở cuối dòng chứa `x + 1`, biến nó từ một biểu thức thành một câu lệnh, chúng ta sẽ gặp lỗi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Biên dịch đoạn mã này sẽ sinh ra lỗi như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Thông báo lỗi chính, `mismatched types`, tiết lộ vấn đề cốt lõi với đoạn mã này. Định nghĩa của hàm `plus_one` tuyên bố rằng nó sẽ trả về một `i32`, nhưng các câu lệnh không đánh giá thành một giá trị nào, điều này được biểu thị bằng `()`, kiểu unit. Do đó, không có giá trị nào được trả về, mâu thuẫn với định nghĩa hàm và dẫn đến lỗi. Trong kết quả này, Rust cung cấp một gợi ý sửa lỗi hữu ích: đề xuất xóa dấu chấm phẩy để sửa lỗi.

{{#quiz ../quizzes/ch03-03-functions-sec2-expressions.toml}}
