## Xác thực các Tham chiếu với Lifetimes

Lifetimes (vòng đời) là một loại generic khác mà chúng ta đã và đang sử dụng. Thay
vì đảm bảo rằng một kiểu có hành vi mà chúng ta muốn, lifetimes đảm bảo rằng
các tham chiếu sẽ hợp lệ trong khoảng thời gian mà chúng ta cần chúng.

Một chi tiết mà chúng ta đã không thảo luận trong phần [“Tham chiếu và
Mượn”][references-and-borrowing]<!-- ignore --> ở Chương 4 là
mọi tham chiếu trong Rust đều có một _lifetime_, đó là phạm vi (scope) mà
tham chiếu đó hợp lệ. Hầu hết thời gian, lifetimes là ngầm định và được suy luận,
giống như hầu hết thời gian, các kiểu dữ liệu được suy luận. Chúng ta chỉ bắt buộc phải
chú thích kiểu khi có thể có nhiều kiểu dữ liệu khác nhau. Theo cách tương tự, chúng ta phải
chú thích lifetimes khi các lifetimes của các tham chiếu có thể liên quan đến nhau theo một vài
cách khác nhau. Rust yêu cầu chúng ta chú thích các mối quan hệ này bằng cách sử dụng các tham số
vòng đời generic (generic lifetime parameters) để đảm bảo các tham chiếu thực tế được sử dụng khi chạy chương trình chắc chắn
sẽ hợp lệ.

Chú thích lifetimes không phải là một khái niệm mà hầu hết các ngôn ngữ lập trình khác có, vì vậy
điều này sẽ mang lại cảm giác lạ lẫm. Mặc dù chúng ta sẽ không trình bày toàn bộ về lifetimes
trong chương này, chúng ta sẽ thảo luận về các cách phổ biến mà bạn có thể gặp
cú pháp lifetime để bạn có thể làm quen với khái niệm này.

### Ngăn chặn các Tham chiếu lơ lửng với Lifetimes

Mục tiêu chính của lifetimes là ngăn chặn _tham chiếu lơ lửng_ (dangling references), thứ khiến một
chương trình tham chiếu đến dữ liệu không phải là dữ liệu mà nó định tham chiếu.
Hãy xem xét chương trình không an toàn trong Liệt kê 10-16, chương trình này có một phạm vi bên ngoài và một
phạm vi bên trong.

<!-- TODO(aquascope): support for nested scopes -->
<Listing number="10-16" caption="Một nỗ lực sử dụng một tham chiếu mà giá trị của nó đã nằm ngoài phạm vi">

```rust,ignore,does_not_compile
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

</Listing>

> Lưu ý: Các ví dụ trong Liệt kê 10-16, 10-17, và 10-23 khai báo các biến
> mà không cấp cho chúng một giá trị ban đầu, vì vậy tên biến tồn tại trong phạm vi
> bên ngoài. Thoạt nhìn, điều này có vẻ mâu thuẫn với việc Rust không có
> giá trị null. Tuy nhiên, nếu chúng ta cố gắng sử dụng một biến trước khi cấp cho nó một giá trị,
> chúng ta sẽ nhận được lỗi lúc biên dịch, điều này cho thấy Rust thực sự không cho phép
> các giá trị null.

Phạm vi bên ngoài khai báo một biến tên là `r` không có giá trị ban đầu, và
phạm vi bên trong khai báo một biến tên là `x` với giá trị ban đầu là `5`. Bên trong
phạm vi bên trong, chúng ta cố gắng đặt giá trị của `r` làm tham chiếu đến `x`. Sau đó
phạm vi bên trong kết thúc, và chúng ta cố gắng in giá trị trong `r`. Mã này sẽ không
biên dịch được vì giá trị mà `r` đang tham chiếu đến đã nằm ngoài phạm vi trước khi
chúng ta cố gắng sử dụng nó. Đây là thông báo lỗi:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Thông báo lỗi nói rằng biến `x` “không sống đủ lâu” (does not live long enough).
Lý do là `x` sẽ nằm ngoài phạm vi khi phạm vi bên trong kết thúc ở dòng 7.
Nhưng `r` vẫn hợp lệ đối với phạm vi bên ngoài; vì phạm vi của nó lớn hơn, chúng ta nói
rằng nó “sống lâu hơn”. Nếu Rust cho phép mã này hoạt động, `r` sẽ
tham chiếu đến vùng nhớ đã được giải phóng khi `x` nằm ngoài phạm vi, và
bất cứ điều gì chúng ta cố gắng làm với `r` sẽ không hoạt động chính xác. Vậy làm thế nào để Rust
xác định rằng mã này không hợp lệ? Nó sử dụng một bộ kiểm tra mượn (borrow checker).

### Bộ kiểm tra mượn đảm bảo dữ liệu sống lâu hơn các tham chiếu của nó

Trình biên dịch Rust có một _bộ kiểm tra mượn_ (borrow checker) để so sánh các phạm vi nhằm xác định
liệu tất cả các lần mượn có hợp lệ hay không. Liệt kê 10-17 hiển thị cùng mã nguồn như Liệt kê
10-16 nhưng có các chú thích hiển thị lifetimes của các biến.

<Listing number="10-17" caption="Các chú thích về lifetimes của r và x, lần lượt được đặt tên là 'a và 'b">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã chú thích lifetime của `r` là `'a` và lifetime của `x`
là `'b`. Như bạn có thể thấy, khối `'b` bên trong nhỏ hơn nhiều so với khối
lifetime `'a` bên ngoài. Tại thời điểm biên dịch, Rust so sánh kích thước của hai
lifetimes và thấy rằng `r` có lifetime là `'a` nhưng nó lại tham chiếu đến vùng nhớ
có lifetime là `'b`. Chương trình bị từ chối vì `'b` ngắn hơn
`'a`: đối tượng của tham chiếu không sống lâu bằng tham chiếu đó.

Liệt kê 10-18 sửa lại mã nguồn để nó không có tham chiếu lơ lửng và nó
biên dịch mà không có bất kỳ lỗi nào.

<Listing number="10-18" caption="Một tham chiếu hợp lệ vì dữ liệu có lifetime dài hơn tham chiếu">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Ở đây, `x` có lifetime `'b`, trong trường hợp này lớn hơn `'a`. Điều này
nghĩa là `r` có thể tham chiếu đến `x` vì Rust biết rằng tham chiếu trong `r` sẽ
luôn hợp lệ trong khi `x` còn hợp lệ.

Bây giờ bạn đã biết lifetimes của các tham chiếu là gì và cách Rust phân tích
lifetimes để đảm bảo các tham chiếu sẽ luôn hợp lệ, hãy cùng khám phá
generic lifetimes của các tham số và giá trị trả về trong ngữ cảnh của các hàm.

### Generic Lifetimes trong các Hàm

Chúng ta sẽ viết một hàm trả về chuỗi dài hơn trong hai string slices. Hàm
này sẽ nhận vào hai string slices và trả về một string slice duy nhất. Sau khi
chúng ta triển khai hàm `longest`, mã nguồn trong Liệt kê 10-19 sẽ
in ra `The longest string is abcd`.

<Listing number="10-19" file-name="src/main.rs" caption="Một hàm main gọi hàm longest để tìm chuỗi dài hơn trong hai string slices">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

Lưu ý rằng chúng ta muốn hàm nhận vào các string slices, vốn là các tham chiếu,
thay vì các strings, bởi vì chúng ta không muốn hàm `longest` chiếm
quyền sở hữu (ownership) các tham số của nó. Tham khảo phần [“String Slices làm
Tham số”][string-slices-as-parameters]<!-- ignore --> trong Chương 4 để thảo luận
thêm về lý do tại sao các tham số chúng ta sử dụng trong Liệt kê 10-19 là những thứ chúng ta
muốn.

Nếu chúng ta cố gắng triển khai hàm `longest` như hiển thị trong Liệt kê 10-20, nó
sẽ không biên dịch được.

<Listing number="10-20" file-name="src/main.rs" caption="Một bản triển khai của hàm longest trả về chuỗi dài hơn trong hai string slices nhưng vẫn chưa thể biên dịch">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

Thay vào đó, chúng ta nhận được lỗi sau đây nói về lifetimes:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Văn bản trợ giúp tiết lộ rằng kiểu trả về cần một tham số lifetime generic
trên đó vì Rust không thể biết tham chiếu được trả về đang tham chiếu đến
`x` hay `y`. Thực tế, chính chúng ta cũng không biết, vì khối `if` trong thân
hàm này trả về một tham chiếu đến `x` và khối `else` trả về một
tham chiếu đến `y`!

Khi chúng ta định nghĩa hàm này, chúng ta không biết các giá trị cụ thể sẽ
được truyền vào hàm này, vì vậy chúng ta không biết trường hợp `if` hay trường hợp
`else` sẽ thực thi. Chúng ta cũng không biết lifetimes cụ thể của các
tham chiếu sẽ được truyền vào, vì vậy chúng ta không thể xem xét các phạm vi như chúng ta đã làm trong
Liệt kê 10-17 và 10-18 để xác định liệu tham chiếu chúng ta trả về sẽ
luôn hợp lệ hay không. Bộ kiểm tra mượn cũng không thể xác định điều này, vì nó
không biết lifetimes của `x` và `y` liên quan như thế nào đến lifetime của
giá trị trả về. Để khắc phục lỗi này, chúng ta sẽ thêm các tham số lifetime generic
định nghĩa mối quan hệ giữa các tham chiếu để bộ kiểm tra mượn có thể
thực hiện phân tích của nó.

### Cú pháp Chú thích Lifetime

Các chú thích lifetime không làm thay đổi thời gian sống của bất kỳ tham chiếu nào. Thay vào đó,
chúng mô tả các mối quan hệ về lifetimes của nhiều tham chiếu với nhau
mà không ảnh hưởng đến các lifetimes đó. Giống như các hàm có thể chấp nhận bất kỳ kiểu nào
khi chữ ký chỉ định một tham số kiểu generic, các hàm có thể chấp nhận các
tham chiếu với bất kỳ lifetime nào bằng cách chỉ định một tham số lifetime generic.

Các chú thích lifetime có một cú pháp hơi lạ: tên của các tham số lifetime
phải bắt đầu bằng một dấu nháy đơn (`'`) và thường là tất cả các chữ cái thường
và rất ngắn, giống như các kiểu generic. Hầu hết mọi người sử dụng tên `'a` cho chú thích
lifetime đầu tiên. Chúng ta đặt các chú thích tham số lifetime sau ký tự `&` của một
tham chiếu, sử dụng một khoảng trắng để phân tách chú thích với kiểu của tham chiếu.

Dưới đây là một số ví dụ: một tham chiếu đến một `i32` không có tham số lifetime, một
tham chiếu đến một `i32` có tham số lifetime tên là `'a`, và một
tham chiếu có thể thay đổi (mutable reference) đến một `i32` cũng có lifetime là `'a`.

```rust,ignore
&i32        // một tham chiếu
&'a i32     // một tham chiếu với một lifetime rõ ràng
&'a mut i32 // một tham chiếu có thể thay đổi với một lifetime rõ ràng
```

Một chú thích lifetime đứng một mình thì không có nhiều ý nghĩa vì các
chú thích này nhằm mục đích cho Rust biết cách các tham số lifetime generic của nhiều
tham chiếu liên quan đến nhau. Hãy cùng kiểm tra cách các chú thích lifetime
liên quan đến nhau trong ngữ cảnh của hàm `longest`.

### Chú thích Lifetime trong Chữ ký Hàm

Để sử dụng các chú thích lifetime trong các chữ ký hàm, chúng ta cần khai báo các tham số
_lifetime_ generic bên trong dấu ngoặc nhọn giữa tên hàm
và danh sách tham số, giống như cách chúng ta đã làm với các tham số _kiểu_ generic.

Chúng ta muốn chữ ký thể hiện ràng buộc sau: tham chiếu được trả về
sẽ hợp lệ chừng nào cả hai tham số đều hợp lệ. Đây là mối
quan hệ giữa lifetimes của các tham số và giá trị trả về. Chúng ta sẽ
đặt tên cho lifetime là `'a` và sau đó thêm nó vào mỗi tham chiếu, như được hiển thị trong Liệt kê
10-21.

<Listing number="10-21" file-name="src/main.rs" caption="Định nghĩa hàm longest chỉ định rằng tất cả các tham chiếu trong chữ ký phải có cùng lifetime 'a">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

Mã nguồn này sẽ biên dịch được và tạo ra kết quả chúng ta muốn khi chúng ta sử dụng nó với
hàm `main` trong Liệt kê 10-19.

Chữ ký hàm giờ đây cho Rust biết rằng đối với một lifetime `'a` nào đó, hàm
nhận vào hai tham số, cả hai đều là string slices sống ít nhất là
bằng lifetime `'a`. Chữ ký hàm cũng cho Rust biết rằng string slice
được trả về từ hàm sẽ sống ít nhất là bằng lifetime `'a`.
Trong thực tế, nó có nghĩa là lifetime của tham chiếu được trả về bởi hàm
`longest` giống với lifetime nhỏ hơn trong số các lifetimes của các giá trị
được tham chiếu bởi các đối số của hàm. Những mối quan hệ này là những gì chúng ta muốn
Rust sử dụng khi phân tích mã nguồn này.

Hãy nhớ rằng, khi chúng ta chỉ định các tham số lifetime trong chữ ký hàm này,
chúng ta không làm thay đổi lifetimes của bất kỳ giá trị nào được truyền vào hoặc trả về. Thay vào đó,
chúng ta đang chỉ định rằng bộ kiểm tra mượn nên từ chối bất kỳ giá trị nào không
tuân thủ các ràng buộc này. Lưu ý rằng hàm `longest` không cần phải
biết chính xác `x` và `y` sẽ sống trong bao lâu, chỉ cần biết rằng một phạm vi nào đó có thể được
thay thế cho `'a` mà sẽ thỏa mãn chữ ký này.

Khi chú thích lifetimes trong các hàm, các chú thích sẽ nằm trong chữ ký hàm,
chứ không phải trong thân hàm. Các chú thích lifetime trở thành một phần của
hợp đồng của hàm, giống như các kiểu dữ liệu trong chữ ký. Việc để
các chữ ký hàm chứa hợp đồng lifetime có nghĩa là quá trình phân tích mà trình biên dịch Rust thực hiện
có thể đơn giản hơn. Nếu có vấn đề với cách một hàm được chú thích
hoặc cách nó được gọi, các lỗi trình biên dịch có thể chỉ ra phần mã nguồn
của chúng ta và các ràng buộc một cách chính xác hơn. Nếu, thay vào đó, trình biên dịch Rust
thực hiện nhiều suy luận hơn về những gì chúng ta dự định về mối quan hệ của các lifetimes,
trình biên dịch có thể chỉ có thể chỉ ra một lần sử dụng mã nguồn của chúng ta cách xa nhiều bước
so với nguyên nhân của vấn đề.

Khi chúng ta truyền các tham chiếu cụ thể cho `longest`, lifetime cụ thể được
thay thế cho `'a` là phần phạm vi của `x` trùng lặp với
phạm vi của `y`. Nói cách khác, lifetime generic `'a` sẽ nhận được lifetime cụ thể
bằng với lifetime nhỏ hơn trong số các lifetimes của `x` và `y`. Bởi vì
chúng ta đã chú thích tham chiếu được trả về với cùng một tham số lifetime `'a`,
tham chiếu được trả về cũng sẽ hợp lệ trong khoảng thời gian bằng với lifetime nhỏ hơn trong số
các lifetimes của `x` và `y`.

Hãy xem cách các chú thích lifetime hạn chế hàm `longest` bằng cách
truyền vào các tham chiếu có lifetimes cụ thể khác nhau. Liệt kê 10-22 là
một ví dụ đơn giản.

<Listing number="10-22" file-name="src/main.rs" caption="Sử dụng hàm longest với các tham chiếu đến các giá trị String có lifetimes cụ thể khác nhau">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

Trong ví dụ này, `string1` hợp lệ cho đến khi kết thúc phạm vi bên ngoài, `string2`
hợp lệ cho đến khi kết thúc phạm vi bên trong, và `result` tham chiếu đến một thứ
hợp lệ cho đến khi kết thúc phạm vi bên trong. Chạy mã này và bạn sẽ thấy
rằng bộ kiểm tra mượn chấp thuận; nó sẽ biên dịch và in ra `The longest string
is long string is long`.

Tiếp theo, hãy thử một ví dụ cho thấy lifetime của tham chiếu trong
`result` phải là lifetime nhỏ hơn trong hai đối số. Chúng ta sẽ di chuyển
khai báo của biến `result` ra bên ngoài phạm vi bên trong nhưng để lại việc
gán giá trị cho biến `result` bên trong phạm vi cùng với
`string2`. Sau đó chúng ta sẽ di chuyển `println!` sử dụng `result` ra bên ngoài
phạm vi bên trong, sau khi phạm vi bên trong đã kết thúc. Mã nguồn trong Liệt kê 10-23 sẽ
không biên dịch được.

<Listing number="10-23" file-name="src/main.rs" caption="Cố gắng sử dụng result sau khi string2 đã nằm ngoài phạm vi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

Khi chúng ta cố gắng biên dịch mã nguồn này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Lỗi cho thấy để `result` hợp lệ cho câu lệnh `println!`,
`string2` sẽ cần phải hợp lệ cho đến khi kết thúc phạm vi bên ngoài. Rust biết
điều này bởi vì chúng ta đã chú thích lifetimes của các tham số hàm và các giá trị
trả về bằng cách sử dụng cùng một tham số lifetime `'a`.

Với tư cách là con người, chúng ta có thể nhìn vào mã nguồn này và thấy rằng `string1` dài hơn
`string2`, và do đó, `result` sẽ chứa một tham chiếu đến `string1`.
Bởi vì `string1` chưa nằm ngoài phạm vi, một tham chiếu đến `string1` sẽ
vẫn hợp lệ cho câu lệnh `println!`. Tuy nhiên, trình biên dịch không thể thấy
rằng tham chiếu đó là hợp lệ trong trường hợp này. Chúng ta đã nói với Rust rằng lifetime của
tham chiếu được trả về bởi hàm `longest` giống với lifetime nhỏ hơn trong số
các lifetimes của các tham chiếu được truyền vào. Do đó, bộ kiểm tra mượn
không cho phép mã nguồn trong Liệt kê 10-23 vì có khả năng có một tham chiếu không hợp lệ.

Hãy thử thiết kế thêm các thử nghiệm thay đổi các giá trị và lifetimes của các
tham chiếu được truyền vào hàm `longest` và cách tham chiếu được trả về
được sử dụng. Đưa ra các giả thuyết về việc liệu các thử nghiệm của bạn có vượt qua được
bộ kiểm tra mượn hay không trước khi bạn biên dịch; sau đó kiểm tra xem bạn có đúng không!

{{#quiz ../quizzes/ch10-03-lifetimes-sec1.toml}}

### Suy nghĩ dưới góc độ Lifetimes

Cách mà bạn cần chỉ định các tham số lifetime phụ thuộc vào những gì
hàm của bạn đang làm. Ví dụ, nếu chúng ta thay đổi bản triển khai của hàm
`longest` để luôn trả về tham số đầu tiên thay vì string slice dài nhất,
chúng ta sẽ không cần chỉ định một lifetime cho tham số `y`.
Mã nguồn sau đây sẽ biên dịch được:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

Chúng ta đã chỉ định một tham số lifetime `'a` cho tham số `x` và kiểu
trả về, nhưng không chỉ định cho tham số `y`, vì lifetime của `y` không có
bất kỳ mối quan hệ nào với lifetime của `x` hoặc giá trị trả về.

Khi trả về một tham chiếu từ một hàm, tham số lifetime cho
kiểu trả về cần phải khớp với tham số lifetime của một trong các tham số. Nếu
tham chiếu được trả về _không_ tham chiếu đến một trong các tham số, nó phải tham chiếu
đến một giá trị được tạo ra bên trong hàm này. Tuy nhiên, đây sẽ là một tham chiếu
lơ lửng vì giá trị đó sẽ nằm ngoài phạm vi khi kết thúc hàm.
Hãy xem xét nỗ lực triển khai hàm `longest` này, nó sẽ không
biên dịch được:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

Ở đây, mặc dù chúng ta đã chỉ định một tham số lifetime `'a` cho kiểu
trả về, bản triển khai này sẽ không biên dịch được vì lifetime của giá trị trả về
không liên quan chút nào đến lifetime của các tham số. Đây là thông báo
lỗi chúng ta nhận được:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Vấn đề là `result` nằm ngoài phạm vi và bị dọn dẹp khi kết thúc hàm
`longest`. Chúng ta cũng đang cố gắng trả về một tham chiếu đến `result`
từ hàm. Không có cách nào chúng ta có thể chỉ định các tham số lifetime để có thể
thay đổi được tham chiếu lơ lửng đó, và Rust sẽ không để chúng ta tạo ra một tham chiếu lơ lửng. Trong trường hợp này, cách khắc phục tốt nhất là trả về một kiểu dữ liệu sở hữu (owned data type)
thay vì một tham chiếu để hàm gọi sau đó chịu trách nhiệm
dọn dẹp giá trị đó.

Cuối cùng, cú pháp lifetime là về việc kết nối lifetimes của các
tham số khác nhau và các giá trị trả về của hàm. Khi chúng đã được kết nối, Rust có
đủ thông tin để cho phép các thao tác an toàn về bộ nhớ và không cho phép các thao tác
có thể tạo ra các con trỏ lơ lửng hoặc vi phạm an toàn bộ nhớ theo cách khác.

### Chú thích Lifetime trong các Định nghĩa Struct

Cho đến nay, các struct chúng ta đã định nghĩa đều giữ các kiểu sở hữu (owned types). Chúng ta có thể định nghĩa các struct
để giữ các tham chiếu, nhưng trong trường hợp đó chúng ta sẽ cần thêm một chú thích lifetime
trên mỗi tham chiếu trong định nghĩa của struct. Liệt kê 10-24 có một struct tên là
`ImportantExcerpt` giữ một string slice.

<Listing number="10-24" file-name="src/main.rs" caption="Một struct giữ một tham chiếu, yêu cầu một chú thích lifetime">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

Struct này có trường duy nhất `part` giữ một string slice, vốn là một
tham chiếu. Giống như với các kiểu dữ liệu generic, chúng ta khai báo tên của tham số
lifetime generic bên trong dấu ngoặc nhọn sau tên của struct để chúng ta có thể
sử dụng tham số lifetime trong thân của định nghĩa struct. Chú thích
này có nghĩa là một instance của `ImportantExcerpt` không thể sống lâu hơn tham chiếu
mà nó nắm giữ trong trường `part` của nó.

Hàm `main` ở đây tạo ra một instance của struct `ImportantExcerpt`
giữ một tham chiếu đến câu đầu tiên của `String` được sở hữu bởi
biến `novel`. Dữ liệu trong `novel` tồn tại trước khi instance
`ImportantExcerpt` được tạo ra. Ngoài ra, `novel` không nằm ngoài phạm vi cho đến sau khi
`ImportantExcerpt` nằm ngoài phạm vi, vì vậy tham chiếu trong
instance `ImportantExcerpt` là hợp lệ.

### Lifetime Elision (Lược bỏ Lifetime)

Bạn đã học được rằng mọi tham chiếu đều có một lifetime và bạn cần chỉ định
các tham số lifetime cho các hàm hoặc struct sử dụng các tham chiếu. Tuy nhiên, chúng ta
đã có một hàm trong Liệt kê 4-9, được hiển thị lại trong Liệt kê 10-25, đã biên dịch được
mà không cần chú thích lifetime.

<Listing number="10-25" file-name="src/lib.rs" caption="Một hàm chúng ta đã định nghĩa trong Liệt kê 4-9 đã biên dịch được mà không cần các chú thích lifetime, mặc dù tham số và kiểu trả về là các tham chiếu">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

Lý do hàm này biên dịch được mà không cần các chú thích lifetime là do lịch sử:
trong các phiên bản đầu tiên (trước 1.0) của Rust, mã nguồn này sẽ không biên dịch được vì
mọi tham chiếu đều cần một lifetime rõ ràng. Vào thời điểm đó, chữ ký
hàm sẽ được viết như thế này:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Sau khi viết rất nhiều mã nguồn Rust, đội ngũ phát triển Rust nhận thấy rằng các lập trình viên Rust
đã nhập đi nhập lại cùng một loại chú thích lifetime trong các tình huống
cụ thể. Những tình huống này có thể dự đoán được và tuân theo một vài
mẫu xác định. Các nhà phát triển đã lập trình các mẫu này vào mã nguồn của trình biên dịch để
bộ kiểm tra mượn có thể suy luận lifetimes trong những tình huống này và sẽ
không cần các chú thích rõ ràng.

Phần lịch sử này của Rust có liên quan vì có khả năng nhiều
mẫu xác định hơn sẽ xuất hiện và được thêm vào trình biên dịch. Trong tương lai,
có thể thậm chí còn ít chú thích lifetime được yêu cầu hơn.

Các mẫu được lập trình vào quá trình phân tích các tham chiếu của Rust được gọi là các
_quy tắc lược bỏ lifetime_ (lifetime elision rules). Đây không phải là các quy tắc để các lập trình viên tuân theo; chúng
là một tập hợp các trường hợp cụ thể mà trình biên dịch sẽ xem xét, và nếu mã nguồn của bạn
khớp với những trường hợp này, bạn không cần phải viết các lifetimes một cách rõ ràng.

Các quy tắc lược bỏ không cung cấp khả năng suy luận đầy đủ. Nếu vẫn còn sự mơ hồ
về việc các tham chiếu có lifetimes nào sau khi Rust áp dụng các quy tắc,
trình biên dịch sẽ không đoán xem lifetime của các tham chiếu còn lại nên là gì.
Thay vì đoán, trình biên dịch sẽ gửi cho bạn một lỗi mà bạn có thể giải quyết bằng cách
thêm các chú thích lifetime.

Lifetimes trên các tham số của hàm hoặc phương thức được gọi là _input lifetimes_ (vòng đời đầu vào), và
lifetimes trên các giá trị trả về được gọi là _output lifetimes_ (vòng đời đầu ra).

Trình biên dịch sử dụng ba quy tắc để tìm ra lifetimes của các tham chiếu
khi không có các chú thích rõ ràng. Quy tắc đầu tiên áp dụng cho input
lifetimes, và quy tắc thứ hai và thứ ba áp dụng cho output lifetimes. Nếu
trình biên dịch đi đến cuối ba quy tắc mà vẫn còn các tham chiếu mà
nó không thể tìm ra lifetimes, trình biên dịch sẽ dừng lại với một lỗi.
Các quy tắc này áp dụng cho các định nghĩa `fn` cũng như các khối `impl`.

<!-- BEGIN INTERVENTION: d03748df-8dcf-4ec8-bd30-341927544665 -->

Quy tắc đầu tiên là trình biên dịch gán một tham số vòng đời khác nhau cho mỗi vòng đời trong mỗi kiểu đầu vào. Các tham chiếu như `&'_ i32` cần một tham số vòng đời, và các cấu trúc như `ImportantExcerpt<'_>` cũng cần một tham số vòng đời. Ví dụ:

- Hàm `fn foo(x: &i32)` sẽ nhận được một tham số vòng đời và trở thành `fn foo<'a>(x: &'a i32)`.
- Hàm `fn foo(x: &i32, y: &i32)` sẽ nhận được hai tham số vòng đời và trở thành `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`.
- Hàm `fn foo(x: &ImportantExcerpt)` sẽ nhận được hai tham số vòng đời và trở thành `fn foo<'a, 'b>(x: &'a ImportantExcerpt<'b>)`.
<!-- END INTERVENTION -->

Quy tắc thứ hai là, nếu có chính xác một tham số input lifetime,
lifetime đó được gán cho tất cả các tham số output lifetime: `fn foo<'a>(x: &'a i32)
-> &'a i32`.

Quy tắc thứ ba là, nếu có nhiều tham số input lifetime, nhưng
một trong số đó là `&self` hoặc `&mut self` vì đây là một phương thức, lifetime của
`self` được gán cho tất cả các tham số output lifetime. Quy tắc thứ ba này giúp
các phương thức trở nên dễ đọc và dễ viết hơn nhiều vì cần ít biểu tượng hơn.

Hãy giả vờ chúng ta là trình biên dịch. Chúng ta sẽ áp dụng các quy tắc này để tìm ra
lifetimes của các tham chiếu trong chữ ký của hàm `first_word` trong
Liệt kê 10-25. Chữ ký bắt đầu mà không có bất kỳ lifetimes nào liên kết với
các tham chiếu:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Sau đó trình biên dịch áp dụng quy tắc đầu tiên, quy định rằng mỗi tham số
nhận được lifetime của riêng nó. Chúng ta sẽ gọi nó là `'a` như thường lệ, vì vậy bây giờ chữ ký là
thế này:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Quy tắc thứ hai được áp dụng vì có chính xác một input lifetime. Quy tắc
thứ hai quy định rằng lifetime của một tham số đầu vào đó được gán cho
output lifetime, vì vậy chữ ký bây giờ là thế này:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Bây giờ tất cả các tham chiếu trong chữ ký hàm này đều có lifetimes, và
trình biên dịch có thể tiếp tục phân tích mà không cần lập trình viên chú thích
lifetimes trong chữ ký hàm này.

Hãy xem một ví dụ khác, lần này sử dụng hàm `longest` mà không có
tham số lifetime nào khi chúng ta bắt đầu làm việc với nó trong Liệt kê 10-20:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Hãy áp dụng quy tắc đầu tiên: mỗi tham số nhận được lifetime của riêng nó. Lần này chúng ta
có hai tham số thay vì một, vì vậy chúng ta có hai lifetimes:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Bạn có thể thấy rằng quy tắc thứ hai không được áp dụng vì có nhiều hơn một
input lifetime. Quy tắc thứ ba cũng không áp dụng, vì `longest` là một
hàm chứ không phải là một phương thức, vì vậy không có tham số nào là `self`. Sau khi
làm việc qua cả ba quy tắc, chúng ta vẫn chưa tìm ra được lifetime của kiểu trả về
là gì. Đây là lý do tại sao chúng ta gặp lỗi khi cố gắng biên dịch mã nguồn trong
Liệt kê 10-20: trình biên dịch đã làm việc qua các quy tắc lược bỏ lifetime nhưng vẫn
không thể tìm ra tất cả các lifetimes của các tham chiếu trong chữ ký.

Bởi vì quy tắc thứ ba thực sự chỉ áp dụng trong các chữ ký phương thức, chúng ta sẽ xem xét
lifetimes trong ngữ cảnh đó tiếp theo để thấy tại sao quy tắc thứ ba có nghĩa là chúng ta không phải
chú thích lifetimes trong các chữ ký phương thức rất thường xuyên.

### Chú thích Lifetime trong các Định nghĩa Phương thức

Khi chúng ta triển khai các phương thức trên một struct có lifetimes, chúng ta sử dụng cùng một cú pháp như
của các tham số kiểu generic, như được hiển thị trong Liệt kê 10-11. Nơi chúng ta khai báo và
sử dụng các tham số lifetime phụ thuộc vào việc liệu chúng có liên quan đến các trường của struct
hay các tham số của phương thức và các giá trị trả về.

Tên lifetime cho các trường của struct luôn cần được khai báo sau từ khóa
`impl` và sau đó được sử dụng sau tên của struct vì những lifetimes đó là một phần
của kiểu của struct.

Trong các chữ ký phương thức bên trong khối `impl`, các tham chiếu có thể được gắn liền với
lifetime của các tham chiếu trong các trường của struct, hoặc chúng có thể độc lập. Thêm
vào đó, các quy tắc lược bỏ lifetime thường làm cho các chú thích lifetime
không cần thiết trong các chữ ký phương thức. Hãy xem một số ví dụ sử dụng
struct tên là `ImportantExcerpt` mà chúng ta đã định nghĩa trong Liệt kê 10-24.

Đầu tiên chúng ta sẽ sử dụng một phương thức tên là `level` có tham số duy nhất là một tham chiếu đến
`self` và có giá trị trả về là một `i32`, vốn không phải là một tham chiếu đến bất cứ thứ gì:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

Việc khai báo tham số lifetime sau `impl` và việc sử dụng nó sau tên kiểu
là bắt buộc, nhưng chúng ta không bắt buộc phải chú thích lifetime của tham chiếu
đến `self` vì quy tắc lược bỏ đầu tiên.

Dưới đây là một ví dụ mà quy tắc lược bỏ lifetime thứ ba được áp dụng:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Có hai input lifetimes, vì vậy Rust áp dụng quy tắc lược bỏ lifetime đầu tiên
và cấp cho cả `&self` và `announcement` lifetimes riêng của chúng. Sau đó, bởi vì
một trong các tham số là `&self`, kiểu trả về nhận được lifetime của `&self`,
và tất cả các lifetimes đã được tính toán xong.

### Vòng đời Tĩnh (The Static Lifetime)

Một lifetime đặc biệt mà chúng ta cần thảo luận là `'static`, biểu thị rằng
tham chiếu bị ảnh hưởng _có thể_ sống trong toàn bộ thời gian của chương trình. Tất cả
các string literals đều có lifetime là `'static`, chúng ta có thể chú thích như sau:

```rust
let s: &'static str = "I have a static lifetime.";
```

Văn bản của chuỗi này được lưu trữ trực tiếp trong mã nhị phân của chương trình, vốn
luôn có sẵn. Do đó, lifetime của tất cả các string literals là `'static`.

Bạn có thể thấy các đề xuất trong thông báo lỗi về việc sử dụng lifetime `'static`. Nhưng
trước khi chỉ định `'static` làm lifetime cho một tham chiếu, hãy nghĩ về việc
liệu tham chiếu bạn có thực sự sống trong toàn bộ lifetime của chương trình của bạn
hay không, và liệu bạn có muốn nó như vậy không. Hầu hết thời gian, một thông báo lỗi
đề xuất lifetime `'static` là kết quả của việc cố gắng tạo ra một tham chiếu lơ lửng
hoặc sự không khớp giữa các lifetimes có sẵn. Trong những trường hợp như vậy, giải pháp
là khắc phục những vấn đề đó, chứ không phải là chỉ định lifetime `'static`.

### Tham số Kiểu Generic, Trait Bounds, và Lifetimes Cùng Nhau

Hãy cùng xem nhanh cú pháp chỉ định các tham số kiểu generic, trait
bounds, và lifetimes tất cả trong một hàm!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Đây là hàm `longest` từ Liệt kê 10-21 trả về chuỗi dài hơn trong
hai string slices. Nhưng bây giờ nó có thêm một tham số tên là `ann` thuộc kiểu
generic `T`, kiểu này có thể được lấp đầy bởi bất kỳ kiểu nào có triển khai trait `Display`
như được chỉ định bởi câu lệnh `where`. Tham số bổ sung này sẽ được in ra
bằng cách sử dụng `{}`, đó là lý do tại sao trait bound `Display` là cần thiết. Bởi vì
lifetimes là một loại generic, các khai báo của tham số lifetime
`'a` và tham số kiểu generic `T` nằm trong cùng một danh sách bên trong dấu ngoặc nhọn
sau tên hàm.

{{#quiz ../quizzes/ch10-03-lifetimes-sec2.toml}}

### Tóm tắt

Chúng ta đã trình bày rất nhiều điều trong chương này! Bây giờ bạn đã biết về các tham số kiểu
generic, traits và trait bounds, và các tham số lifetime generic, bạn đã
sẵn sàng để viết mã nguồn không lặp lại hoạt động trong nhiều tình huống khác nhau.
Các tham số kiểu generic cho phép bạn áp dụng mã nguồn cho các kiểu khác nhau. Traits và
trait bounds đảm bảo rằng mặc dù các kiểu là generic, chúng sẽ có các
hành vi mà mã nguồn cần. Bạn đã học cách sử dụng các chú thích lifetime để đảm bảo
rằng mã nguồn linh hoạt này sẽ không có bất kỳ tham chiếu lơ lửng nào. Và tất cả những
phân tích này diễn ra tại thời điểm biên dịch, điều này không ảnh hưởng đến hiệu suất lúc chạy!

Tin hay không tùy bạn, còn rất nhiều điều để học về các chủ đề chúng ta đã thảo luận trong
chương này: Chương 18 thảo luận về trait objects, vốn là một cách khác để sử dụng
traits. Cũng có những tình huống phức tạp hơn liên quan đến các chú thích lifetime
mà bạn sẽ chỉ cần trong những tình huống rất nâng cao; đối với những tình huống đó, bạn nên đọc
[Tài liệu tham khảo Rust (Rust Reference)][reference]. Nhưng tiếp theo, bạn sẽ học cách viết các bài kiểm tra (tests) trong
Rust để bạn có thể đảm bảo mã nguồn của mình đang hoạt động theo cách nó nên hoạt động.

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-04-slices.html#string-slices-as-parameters
[reference]: ../reference/index.html
[lifetime-permissions]: ch04-02-references-and-borrowing.html#permissions-are-returned-at-the-end-of-a-references-lifetime
