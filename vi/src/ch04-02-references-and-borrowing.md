## Tham chiếu và Mượn (References and Borrowing)

Vấn đề với đoạn mã tuple trong Listing 4-5 là chúng ta phải trả lại `String` cho hàm gọi để vẫn có thể sử dụng `String` sau khi gọi `calculate_length`, bởi vì `String` đã bị chuyển quyền sở hữu vào `calculate_length`. Thay vào đó, chúng ta có thể cung cấp một tham chiếu tới giá trị `String`. Một _tham chiếu_ giống như một con trỏ ở chỗ nó là một địa chỉ mà chúng ta có thể theo để truy cập dữ liệu được lưu tại địa chỉ đó; dữ liệu đó được sở hữu bởi một biến khác. Khác với con trỏ, một tham chiếu được đảm bảo sẽ trỏ tới một giá trị hợp lệ của một kiểu cụ thể trong suốt vòng đời của tham chiếu đó.

Đây là cách bạn định nghĩa và sử dụng hàm `calculate_length` nhận tham chiếu tới một đối tượng làm tham số thay vì lấy quyền sở hữu giá trị:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

Đầu tiên, hãy chú ý rằng toàn bộ mã tuple trong khai báo biến và giá trị trả về của hàm đã biến mất. Thứ hai, lưu ý rằng chúng ta truyền `&s1` vào `calculate_length` và trong định nghĩa của nó, chúng ta nhận `&String` thay vì `String`. Những dấu `&` này đại diện cho _tham chiếu_, và chúng cho phép bạn tham chiếu tới một giá trị mà không lấy quyền sở hữu nó. Hình 4-6 minh họa khái niệm này.

<img alt="Ba bảng: bảng cho s chỉ chứa một con trỏ tới bảng cho s1. Bảng cho s1 chứa dữ liệu stack cho s1 và trỏ tới dữ liệu chuỗi trên heap." src="img/trpl04-06.svg" class="center" />

<span class="caption">Hình 4-6: Sơ đồ `&String s` trỏ tới `String s1`</span>

> Lưu ý: Đối lập với việc tham chiếu bằng cách sử dụng `&` là _giải tham chiếu_, được thực hiện với toán tử giải tham chiếu, `*`. Chúng ta sẽ thấy một số ví dụ về toán tử này ở Chương 8 và bàn về chi tiết giải tham chiếu ở Chương 15.

Hãy xem kỹ hơn về lời gọi hàm ở đây:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

Cú pháp `&s1` cho phép chúng ta tạo một tham chiếu _tham chiếu_ tới giá trị của `s1` nhưng không sở hữu nó. Vì tham chiếu không sở hữu giá trị, nên giá trị mà nó trỏ tới sẽ không bị hủy khi tham chiếu ngừng được sử dụng.

Tương tự, chữ ký của hàm sử dụng `&` để chỉ ra rằng kiểu của tham số `s` là một tham chiếu. Hãy thêm một số chú thích giải thích:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

Phạm vi mà biến `s` hợp lệ giống như phạm vi của bất kỳ tham số hàm nào, nhưng giá trị được tham chiếu bởi tham chiếu sẽ không bị hủy khi `s` ngừng được sử dụng, bởi vì `s` không có quyền sở hữu. Khi các hàm nhận tham chiếu làm tham số thay vì giá trị thực, chúng ta sẽ không cần trả lại giá trị để chuyển lại quyền sở hữu, bởi vì chúng ta chưa từng sở hữu nó.

Hành động tạo một tham chiếu được gọi là _mượn_. Giống như trong đời thực, nếu một người sở hữu thứ gì đó, bạn có thể mượn nó từ họ. Khi bạn xong việc, bạn phải trả lại. Bạn không sở hữu nó.

Vậy điều gì xảy ra nếu chúng ta cố gắng sửa đổi thứ mà chúng ta đang mượn? Hãy thử đoạn mã trong Listing 4-6. Bật mí: nó không hoạt động!

<Listing number="4-6" file-name="src/main.rs" caption="Cố gắng sửa đổi một giá trị được mượn">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Cũng giống như biến mặc định là bất biến, tham chiếu cũng vậy. Chúng ta không được phép sửa đổi thứ mà chúng ta chỉ có tham chiếu tới.

### Tham chiếu có thể thay đổi

Chúng ta có thể sửa đoạn mã từ Listing 4-6 để cho phép sửa đổi giá trị được mượn chỉ với một vài thay đổi nhỏ bằng cách sử dụng _tham chiếu có thể thay đổi_:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

Đầu tiên chúng ta đổi `s` thành `mut`. Sau đó, chúng ta tạo một tham chiếu có thể thay đổi với `&mut s` khi gọi hàm `change`, và cập nhật chữ ký hàm để nhận tham chiếu có thể thay đổi với `some_string: &mut String`. Điều này làm rõ rằng hàm `change` sẽ thay đổi giá trị mà nó mượn.

Tham chiếu có thể thay đổi có một hạn chế lớn: nếu bạn có một tham chiếu có thể thay đổi tới một giá trị, bạn không thể có bất kỳ tham chiếu nào khác tới giá trị đó. Đoạn mã này cố gắng tạo hai tham chiếu có thể thay đổi tới `s` sẽ bị lỗi:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Lỗi này nói rằng đoạn mã này không hợp lệ vì chúng ta không thể mượn `s` dưới dạng có thể thay đổi nhiều hơn một lần cùng lúc. Lần mượn có thể thay đổi đầu tiên là ở `r1` và phải kéo dài cho tới khi nó được sử dụng trong `println!`, nhưng giữa lúc tạo tham chiếu có thể thay đổi đó và lúc sử dụng nó, chúng ta đã cố gắng tạo một tham chiếu có thể thay đổi khác ở `r2` mượn cùng dữ liệu với `r1`.

Hạn chế ngăn chặn nhiều tham chiếu có thể thay đổi tới cùng dữ liệu cùng lúc cho phép sửa đổi nhưng theo cách rất kiểm soát. Đây là điều mà người mới học Rust thường gặp khó khăn vì hầu hết các ngôn ngữ cho phép bạn sửa đổi bất cứ khi nào bạn muốn. Lợi ích của hạn chế này là Rust có thể ngăn chặn các lỗi tranh chấp dữ liệu ngay tại thời điểm biên dịch. Một _tranh chấp dữ liệu_ giống như một điều kiện tranh chấp và xảy ra khi ba hành vi sau xuất hiện:

- Hai hoặc nhiều con trỏ truy cập cùng dữ liệu cùng lúc.
- Ít nhất một con trỏ được sử dụng để ghi dữ liệu.
- Không có cơ chế nào được sử dụng để đồng bộ hóa việc truy cập dữ liệu.

Tranh chấp dữ liệu gây ra hành vi không xác định và có thể rất khó phát hiện và sửa khi bạn cố gắng tìm lỗi lúc chạy; Rust ngăn chặn vấn đề này bằng cách từ chối biên dịch mã có tranh chấp dữ liệu!

Như thường lệ, chúng ta có thể sử dụng dấu ngoặc nhọn để tạo một phạm vi mới, cho phép nhiều tham chiếu có thể thay đổi, chỉ là không _đồng thời_:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust cũng áp dụng quy tắc tương tự khi kết hợp tham chiếu có thể thay đổi và tham chiếu bất biến. Đoạn mã này sẽ gây lỗi:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Chà! Chúng ta _cũng_ không thể có một tham chiếu có thể thay đổi trong khi đang có một tham chiếu bất biến tới cùng giá trị.

Người dùng của tham chiếu bất biến không mong đợi giá trị bị thay đổi bất ngờ! Tuy nhiên, nhiều tham chiếu bất biến được phép vì không ai chỉ đọc dữ liệu có thể ảnh hưởng tới việc đọc của người khác.

Lưu ý rằng phạm vi của một tham chiếu bắt đầu từ nơi nó được giới thiệu và kéo dài tới lần cuối cùng tham chiếu đó được sử dụng. Ví dụ, đoạn mã này sẽ biên dịch vì lần sử dụng cuối cùng của các tham chiếu bất biến là trong `println!`, trước khi tham chiếu có thể thay đổi được tạo ra:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Phạm vi của các tham chiếu bất biến `r1` và `r2` kết thúc sau `println!` nơi chúng được sử dụng lần cuối, trước khi tham chiếu có thể thay đổi `r3` được tạo ra. Các phạm vi này không giao nhau, nên đoạn mã này được phép: trình biên dịch có thể xác định rằng tham chiếu không còn được sử dụng tại một điểm trước khi kết thúc phạm vi.

Dù đôi khi lỗi mượn có thể gây khó chịu, hãy nhớ rằng đó là trình biên dịch Rust chỉ ra một lỗi tiềm ẩn sớm (tại thời điểm biên dịch thay vì lúc chạy) và chỉ rõ chính xác vị trí vấn đề. Nhờ vậy bạn không phải mất công tìm hiểu tại sao dữ liệu của mình không như mong đợi.

### Tham chiếu treo

Trong các ngôn ngữ có con trỏ, rất dễ vô tình tạo ra một _con trỏ treo_—một con trỏ tham chiếu tới một vị trí bộ nhớ có thể đã được cấp phát cho ai đó khác—bằng cách giải phóng bộ nhớ trong khi vẫn giữ một con trỏ tới bộ nhớ đó. Ngược lại, trong Rust, trình biên dịch đảm bảo rằng tham chiếu sẽ không bao giờ là tham chiếu treo: nếu bạn có một tham chiếu tới dữ liệu nào đó, trình biên dịch sẽ đảm bảo rằng dữ liệu đó sẽ không bị ra khỏi phạm vi trước khi tham chiếu tới dữ liệu đó kết thúc.

Hãy thử tạo một tham chiếu treo để xem Rust ngăn chặn chúng bằng lỗi biên dịch như thế nào:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Thông báo lỗi này đề cập tới một tính năng mà chúng ta chưa học: vòng đời (lifetimes). Chúng ta sẽ bàn về vòng đời chi tiết ở Chương 10. Nhưng nếu bạn bỏ qua phần về vòng đời, thông báo vẫn chứa ý chính về lý do tại sao đoạn mã này có vấn đề:

```text
kiểu trả về của hàm này chứa một giá trị được mượn, nhưng không có giá trị nào để mượn từ đó
```

Hãy xem kỹ hơn từng bước của đoạn mã `dangle`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

Vì `s` được tạo bên trong `dangle`, khi mã của `dangle` kết thúc, `s` sẽ bị giải phóng. Nhưng chúng ta lại cố trả về một tham chiếu tới nó. Điều đó nghĩa là tham chiếu này sẽ trỏ tới một `String` không hợp lệ. Điều này không ổn! Rust sẽ không cho phép chúng ta làm vậy.

Giải pháp ở đây là trả về trực tiếp `String`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Cách này hoạt động mà không gặp vấn đề gì. Quyền sở hữu được chuyển ra ngoài, và không có gì bị giải phóng.

### Các quy tắc của tham chiếu

Hãy tổng kết lại những gì chúng ta đã học về tham chiếu:

- Tại một thời điểm, bạn chỉ có _hoặc_ một tham chiếu có thể thay đổi _hoặc_ bất kỳ số lượng tham chiếu bất biến nào.
- Tham chiếu luôn phải hợp lệ.

Tiếp theo, chúng ta sẽ tìm hiểu về một loại tham chiếu khác: slice.

