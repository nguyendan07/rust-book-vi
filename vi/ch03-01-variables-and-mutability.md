## Biến và Tính Khả Biến (Variables and Mutability)

Như đã đề cập trong phần [“Lưu trữ Giá trị bằng Biến”][storing-values-with-variables]<!-- ignore -->, theo mặc định, các biến trong Rust là bất biến (immutable). Đây là một trong nhiều định hướng mà Rust mang lại để bạn viết mã tận dụng được tính an toàn và khả năng xử lý đồng thời (concurrency) dễ dàng mà Rust cung cấp. Tuy nhiên, bạn vẫn có tùy chọn làm cho các biến của mình trở nên khả biến (mutable). Hãy cùng tìm hiểu cách thức và lý do tại sao Rust khuyến khích bạn ưu tiên tính bất biến, và tại sao đôi khi bạn lại muốn chọn tính khả biến.

Khi một biến là bất biến, một khi giá trị đã được liên kết với một tên biến, bạn không thể thay đổi giá trị đó. Để minh họa điều này, hãy tạo một dự án mới có tên là _variables_ trong thư mục _projects_ của bạn bằng lệnh `cargo new variables`.

Sau đó, trong thư mục _variables_ mới, hãy mở _src/main.rs_ và thay thế mã của nó bằng đoạn mã sau, đoạn mã này hiện tại sẽ chưa biên dịch được:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Lưu và chạy chương trình bằng lệnh `cargo run`. Bạn sẽ nhận được một thông báo lỗi liên quan đến tính bất biến, như được hiển thị trong kết quả này:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Ví dụ này cho thấy cách trình biên dịch giúp bạn phát hiện lỗi trong chương trình của mình. Lỗi biên dịch có thể gây khó chịu lúc đầu, nhưng thực tế chúng chỉ có nghĩa là chương trình của bạn chưa thực hiện một cách an toàn những gì bạn muốn; chúng _không_ có nghĩa là bạn không phải là một lập trình viên giỏi! Ngay cả những Rustacean giàu kinh nghiệm vẫn thường xuyên gặp lỗi biên dịch.

Bạn nhận được thông báo lỗi `` cannot assign twice to immutable variable `x` `` (không thể gán giá trị lần thứ hai cho biến bất biến `x`) vì bạn đã cố gắng gán giá trị thứ hai cho biến bất biến `x`.

Việc chúng ta nhận được lỗi tại thời gian biên dịch khi cố gắng thay đổi một giá trị được chỉ định là bất biến là rất quan trọng vì chính tình huống này có thể dẫn đến các lỗi logic (bug). Nếu một phần trong mã của chúng ta hoạt động dựa trên giả định rằng một giá trị sẽ không bao giờ thay đổi và một phần khác lại thay đổi giá trị đó, thì rất có thể phần mã thứ nhất sẽ không hoạt động đúng như thiết kế. Nguyên nhân của loại lỗi này thường rất khó truy vết sau này, đặc biệt là khi phần mã thứ hai chỉ thỉnh thoảng mới thay đổi giá trị. Trình biên dịch Rust đảm bảo rằng khi bạn tuyên bố một giá trị sẽ không thay đổi, nó thực sự sẽ không bao giờ thay đổi, do đó bạn không cần phải tự mình theo dõi nó. Mã nguồn của bạn vì thế sẽ dễ suy luận hơn rất nhiều.

Nhưng tính khả biến (mutability) cũng có thể rất hữu ích và có thể làm cho việc viết mã trở nên thuận tiện hơn. Mặc dù các biến là bất biến theo mặc định, bạn có thể làm cho chúng khả biến bằng cách thêm từ khóa `mut` vào trước tên biến như bạn đã làm trong [Chương 2][storing-values-with-variables]<!-- ignore -->. Việc thêm `mut` cũng truyền đạt ý định rõ ràng cho những người đọc mã trong tương lai rằng các phần khác của mã sẽ thay đổi giá trị của biến này.

Ví dụ, hãy thay đổi _src/main.rs_ thành như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Khi chạy chương trình bây giờ, chúng ta nhận được kết quả:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Chúng ta được phép thay đổi giá trị liên kết với `x` từ `5` thành `6` khi `mut` được sử dụng. Cuối cùng, việc quyết định có sử dụng tính khả biến hay không là tùy thuộc vào bạn và phụ thuộc vào những gì bạn cho là rõ ràng nhất trong tình huống cụ thể đó.

{{#quiz ../quizzes/ch03-01-variables-and-mutability-sec1-variables.toml}}

### Hằng Số (Constants)

Giống như các biến bất biến, _hằng số_ (constants) là các giá trị được liên kết với một cái tên và không được phép thay đổi, nhưng có một vài điểm khác biệt quan trọng giữa hằng số và biến.

Đầu tiên, bạn không được phép sử dụng `mut` với hằng số. Hằng số không chỉ bất biến theo mặc định — chúng luôn luôn bất biến. Bạn khai báo hằng số bằng từ khóa `const` thay vì từ khóa `let`, và kiểu của giá trị _bắt buộc_ phải được chú thích kiểu tường minh. Chúng ta sẽ tìm hiểu về các kiểu dữ liệu và chú thích kiểu trong phần tiếp theo, [“Kiểu Dữ Liệu”][data-types]<!-- ignore -->, vì vậy đừng lo lắng về chi tiết ngay bây giờ. Chỉ cần nhớ rằng bạn luôn phải chú thích kiểu cho hằng số.

Hằng số có thể được khai báo trong bất kỳ phạm vi nào, bao gồm cả phạm vi toàn cục (global scope), điều này làm cho chúng hữu ích cho các giá trị mà nhiều phần trong mã cần biết đến.

Điểm khác biệt cuối cùng là hằng số chỉ có thể được gán cho một biểu thức hằng (constant expression), chứ không phải kết quả của một giá trị chỉ có thể được tính toán tại thời gian chạy (runtime).

Dưới đây là một ví dụ về khai báo hằng số:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Tên của hằng số là `THREE_HOURS_IN_SECONDS` và giá trị của nó được đặt bằng kết quả của phép nhân 60 (số giây trong một phút) với 60 (số phút trong một giờ) với 3 (số giờ chúng ta muốn tính trong chương trình này). Quy ước đặt tên của Rust cho các hằng số là sử dụng tất cả các chữ in hoa với dấu gạch dưới giữa các từ. Trình biên dịch có khả năng tính toán một tập hợp các phép toán giới hạn ngay tại thời điểm biên dịch, cho phép chúng ta viết ra giá trị này theo cách dễ hiểu và dễ xác minh hơn, thay vì đặt hằng số này thành giá trị 10,800. Xem [phần đánh giá hằng số trong Tài liệu Tham khảo Rust][const-eval] để biết thêm thông tin về các phép toán có thể được sử dụng khi khai báo hằng số.

Hằng số có hiệu lực trong toàn bộ thời gian chương trình chạy, bên trong phạm vi mà chúng được khai báo. Thuộc tính này làm cho hằng số trở nên hữu ích cho các giá trị trong miền ứng dụng của bạn mà nhiều phần của chương trình có thể cần biết, chẳng hạn như số điểm tối đa mà bất kỳ người chơi nào được phép đạt được trong một trò chơi, hoặc tốc độ ánh sáng.

Đặt tên cho các giá trị được mã hóa cứng (hardcoded) được sử dụng xuyên suốt chương trình của bạn dưới dạng hằng số rất hữu ích trong việc truyền đạt ý nghĩa của giá trị đó cho những người bảo trì mã trong tương lai. Nó cũng giúp bạn chỉ cần sửa đổi tại một vị trí duy nhất nếu giá trị đó cần được cập nhật sau này.

{{#quiz ../quizzes/ch03-01-variables-and-mutability-sec2-constants.toml}}

### Che Khuất Biến (Shadowing)

Như bạn đã thấy trong hướng dẫn trò chơi đoán số ở [Chương 2][comparing-the-guess-to-the-secret-number]<!-- ignore -->, bạn có thể khai báo một biến mới có cùng tên với một biến trước đó. Các Rustacean nói rằng biến đầu tiên đã bị _che khuất_ (shadowed) bởi biến thứ hai, điều đó có nghĩa là biến thứ hai là những gì trình biên dịch sẽ nhìn thấy khi bạn sử dụng tên biến đó. Trên thực tế, biến thứ hai che khuất biến thứ nhất, chuyển hướng mọi thao tác sử dụng tên biến về chính nó cho đến khi bản thân nó bị che khuất hoặc phạm vi kết thúc. Chúng ta có thể che khuất một biến bằng cách sử dụng cùng tên biến và lặp lại việc sử dụng từ khóa `let` như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Chương trình này đầu tiên liên kết `x` với giá trị `5`. Sau đó nó tạo một biến `x` mới bằng cách lặp lại `let x =`, lấy giá trị ban đầu và cộng thêm `1` để giá trị của `x` trở thành `6`. Sau đó, bên trong một phạm vi con được tạo bởi cặp dấu ngoặc nhọn, câu lệnh `let` thứ ba cũng che khuất `x` và tạo ra một biến mới, nhân giá trị trước đó với `2` để `x` có giá trị là `12`. Khi phạm vi con đó kết thúc, sự che khuất bên trong kết thúc và `x` quay trở lại là `6`. Khi chúng ta chạy chương trình này, nó sẽ xuất ra kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Shadowing khác với việc đánh dấu một biến là `mut` vì chúng ta sẽ gặp lỗi biên dịch nếu vô tình gán lại giá trị cho biến này mà không sử dụng từ khóa `let`. Bằng cách sử dụng `let`, chúng ta có thể thực hiện một vài phép biến đổi trên một giá trị nhưng biến vẫn là bất biến sau khi các phép biến đổi đó hoàn tất.

Điểm khác biệt khác giữa `mut` và shadowing là vì chúng ta thực sự đang tạo ra một biến hoàn toàn mới khi sử dụng lại từ khóa `let`, chúng ta có thể **thay đổi kiểu của giá trị** nhưng vẫn tái sử dụng cùng một tên biến. Ví dụ: giả sử chương trình của chúng ta yêu cầu người dùng nhập số lượng khoảng trắng họ muốn giữa các đoạn văn bản bằng cách nhập các ký tự khoảng trắng, và sau đó chúng ta muốn lưu trữ đầu vào đó dưới dạng một số:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Biến `spaces` đầu tiên có kiểu chuỗi và biến `spaces` thứ hai có kiểu số. Shadowing giúp chúng ta không phải nghĩ ra các tên biến khác nhau như `spaces_str` và `spaces_num`; thay vào đó, chúng ta có thể tái sử dụng tên `spaces` ngắn gọn hơn. Tuy nhiên, nếu chúng ta cố gắng sử dụng `mut` cho trường hợp này, chúng ta sẽ gặp lỗi biên dịch:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Lỗi cho biết chúng ta không được phép thay đổi kiểu dữ liệu của một biến khả biến:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Bây giờ chúng ta đã tìm hiểu cách các biến hoạt động, hãy cùng khám phá thêm các kiểu dữ liệu mà chúng có thể nhận.

{{#quiz ../quizzes/ch03-01-variables-and-mutability-sec3-shadowing.toml}}

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: https://doc.rust-lang.org/reference/const_eval.html
