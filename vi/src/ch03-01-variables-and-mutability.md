## Biến và Tính Thay Đổi

Như đã đề cập trong phần [“Lưu trữ giá trị với biến”][storing-values-with-variables]<!-- ignore -->, theo mặc định, các biến là bất biến. Đây là một trong nhiều lời khuyên Rust đưa ra để bạn viết mã theo cách tận dụng tính an toàn và khả năng đồng thời dễ kiểm soát mà Rust cung cấp. Tuy nhiên, bạn vẫn có thể làm cho biến của mình có thể thay đổi được. Hãy cùng khám phá cách và lý do Rust khuyến khích bạn ưu tiên bất biến và tại sao đôi khi bạn có thể muốn bỏ qua điều đó.

Khi một biến là bất biến, một khi một giá trị được gán cho một tên, bạn không thể thay đổi giá trị đó. Để minh họa điều này, hãy tạo một dự án mới có tên là _variables_ trong thư mục _projects_ bằng lệnh `cargo new variables`.

Sau đó, trong thư mục _variables_ mới, mở _src/main.rs_ và thay thế mã bằng đoạn mã sau, chương trình sẽ chưa biên dịch được ngay:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Lưu và chạy chương trình bằng `cargo run`. Bạn sẽ nhận được thông báo lỗi liên quan đến tính bất biến, như trong đầu ra sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Ví dụ này cho thấy cách trình biên dịch giúp bạn tìm lỗi trong chương trình. Lỗi biên dịch có thể khiến bạn khó chịu, nhưng thật ra chúng chỉ có nghĩa là chương trình của bạn chưa thực hiện an toàn những gì bạn muốn; chúng _không_ có nghĩa là bạn không phải là một lập trình viên giỏi! Ngay cả những Rustacean có kinh nghiệm vẫn gặp lỗi biên dịch.

Bạn nhận được thông báo lỗi `` cannot assign twice to immutable variable `x` `` vì bạn đã cố gắng gán giá trị lần thứ hai cho biến `x` vốn là bất biến.

Việc nhận lỗi tại thời điểm biên dịch khi cố gắng thay đổi một giá trị được khai báo bất biến là rất quan trọng vì chính tình huống này có thể dẫn đến lỗi. Nếu một phần mã của chúng ta hoạt động với giả định rằng một giá trị sẽ không thay đổi và một phần khác thay đổi giá trị đó, có thể phần mã đầu tiên sẽ không hoạt động như được thiết kế. Nguyên nhân của loại lỗi này có thể khó phát hiện sau khi xảy ra, đặc biệt khi phần mã thứ hai chỉ đôi khi thay đổi giá trị. Trình biên dịch Rust đảm bảo rằng khi bạn nói một giá trị sẽ không thay đổi, nó thực sự sẽ không thay đổi, vì vậy bạn không phải tự theo dõi điều đó. Do đó mã của bạn dễ suy luận hơn.

Nhưng tính thay đổi (mutability) có thể rất hữu ích và giúp mã dễ viết hơn. Mặc dù biến theo mặc định là bất biến, bạn có thể làm cho chúng có thể thay đổi bằng cách thêm `mut` trước tên biến như bạn đã làm trong [Chương 2][storing-values-with-variables]<!-- ignore -->. Việc thêm `mut` cũng truyền đạt ý định cho người đọc mã sau này bằng cách chỉ ra rằng các phần khác của mã sẽ thay đổi giá trị của biến này.

Ví dụ, hãy thay đổi _src/main.rs_ thành như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Khi chạy chương trình bây giờ, chúng ta nhận được:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Chúng ta được phép thay đổi giá trị được gán cho `x` từ `5` thành `6` khi sử dụng `mut`. Cuối cùng, việc quyết định có dùng mutability hay không tùy thuộc vào bạn và phụ thuộc vào điều bạn cho là rõ ràng nhất trong tình huống cụ thể đó.

### Hằng số

Giống như biến bất biến, _hằng số_ là các giá trị được gán cho một tên và không được thay đổi, nhưng có một vài khác biệt giữa hằng số và biến.

Thứ nhất, bạn không được phép sử dụng `mut` với hằng số. Hằng số không chỉ bất biến theo mặc định — chúng luôn luôn bất biến. Bạn khai báo hằng số bằng từ khóa `const` thay vì `let`, và kiểu của giá trị _phải_ được chú thích. Chúng ta sẽ đề cập đến các kiểu và chú thích kiểu trong phần tiếp theo, [“Các kiểu dữ liệu”][data-types]<!-- ignore -->, nên đừng lo chi tiết ngay bây giờ. Chỉ cần biết rằng bạn luôn phải chú thích kiểu.

Hằng số có thể được khai báo trong bất kỳ phạm vi nào, bao gồm cả phạm vi toàn cục, điều này làm cho chúng hữu ích cho các giá trị mà nhiều phần của mã cần biết tới.

Sự khác biệt cuối cùng là hằng số chỉ có thể được gán cho một biểu thức hằng, không phải kết quả của một giá trị chỉ có thể tính toán trong thời gian chạy.

Đây là ví dụ khai báo hằng số:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Tên của hằng số là `THREE_HOURS_IN_SECONDS` và giá trị của nó được đặt thành kết quả của phép nhân 60 (số giây trong một phút) với 60 (số phút trong một giờ) với 3 (số giờ mà chúng ta muốn đếm trong chương trình này). Quy ước đặt tên của Rust cho hằng số là dùng chữ in hoa toàn bộ với dấu gạch dưới giữa các từ. Trình biên dịch có thể đánh giá một tập hợp giới hạn các phép toán tại thời điểm biên dịch, điều này cho phép chúng ta viết giá trị này theo cách dễ hiểu và xác minh hơn, thay vì đặt hằng số này thành 10.800. Xem phần về đánh giá hằng trong Tài liệu tham khảo Rust ([const-eval]) để biết thêm thông tin về các phép toán nào có thể sử dụng khi khai báo hằng số.

Hằng số có hiệu lực trong toàn bộ thời gian chương trình chạy, trong phạm vi mà chúng được khai báo. Thuộc tính này làm cho hằng số hữu ích cho các giá trị trong miền ứng dụng của bạn mà nhiều phần của chương trình có thể cần biết, chẳng hạn như số điểm tối đa mà bất kỳ người chơi nào trong một trò chơi được phép đạt, hoặc tốc độ ánh sáng.

Đặt tên các giá trị cố định (hardcoded) xuất hiện trong chương trình của bạn thành hằng số có ích trong việc truyền đạt ý nghĩa của giá trị đó cho những người bảo trì mã sau này. Nó cũng giúp chỉ có một nơi trong mã mà bạn cần thay đổi nếu giá trị cố định đó cần được cập nhật trong tương lai.

### Che phủ (Shadowing)

Như bạn đã thấy trong bài hướng dẫn trò chơi đoán ở [Chương 2][comparing-the-guess-to-the-secret-number]<!-- ignore -->, bạn có thể khai báo một biến mới cùng tên với một biến trước đó. Các Rustacean nói rằng biến đầu tiên bị _che phủ_ (shadowed) bởi biến thứ hai, nghĩa là biến thứ hai là biến mà trình biên dịch sẽ thấy khi bạn dùng tên biến đó. Thực tế, biến thứ hai lấn át biến thứ nhất, chiếm mọi lần sử dụng tên biến đó cho đến khi chính nó bị che phủ hoặc phạm vi kết thúc. Chúng ta có thể che phủ một biến bằng cách dùng cùng tên biến và lặp lại từ khóa `let` như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Chương trình này trước tiên gán `x` với giá trị `5`. Sau đó nó tạo một biến mới `x` bằng cách lặp lại `let x =`, lấy giá trị ban đầu và cộng thêm `1` nên giá trị của `x` trở thành `6`. Rồi, trong một phạm vi con được tạo bởi dấu ngoặc nhọn, câu lệnh `let` thứ ba cũng che phủ `x` và tạo một biến mới, nhân giá trị trước đó với `2` để cho `x` có giá trị `12`. Khi phạm vi đó kết thúc, việc che phủ bên trong cũng kết thúc và `x` trở lại giá trị `6`. Khi chạy chương trình này, nó sẽ in ra như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Shadowing khác với việc đánh dấu một biến là `mut` vì chúng ta sẽ nhận lỗi biên dịch nếu vô tình cố gắng gán lại cho biến đó mà không dùng từ khóa `let`. Bằng cách dùng `let`, chúng ta có thể thực hiện vài biến đổi trên một giá trị nhưng giữ cho biến bất biến sau khi những biến đổi đó hoàn tất.

Sự khác biệt khác giữa `mut` và shadowing là vì khi chúng ta thực sự tạo một biến mới khi dùng lại từ khóa `let`, chúng ta có thể thay đổi kiểu của giá trị nhưng tái sử dụng cùng tên. Ví dụ, giả sử chương trình của chúng ta yêu cầu người dùng cho biết bao nhiêu ký tự khoảng trắng họ muốn giữa một số văn bản bằng cách nhập các ký tự khoảng trắng, và sau đó chúng ta muốn lưu đầu vào đó dưới dạng một số:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Biến `spaces` đầu tiên là kiểu chuỗi và biến `spaces` thứ hai là kiểu số. Shadowing do đó giúp chúng ta khỏi phải nghĩ ra những tên khác nhau, như `spaces_str` và `spaces_num`; thay vào đó, ta có thể tái sử dụng tên `spaces` đơn giản hơn. Tuy nhiên, nếu chúng ta cố dùng `mut` cho điều này, như được thể hiện ở đây, chúng ta sẽ nhận lỗi biên dịch:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Lỗi nói rằng chúng ta không được phép thay đổi kiểu của một biến khi dùng mutate:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Bây giờ chúng ta đã khám phá cách các biến hoạt động, hãy xem thêm các kiểu dữ liệu mà chúng có thể có.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
