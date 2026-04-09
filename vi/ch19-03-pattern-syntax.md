## Cú pháp mẫu (Pattern Syntax)

Trong phần này, chúng ta tập hợp tất cả các cú pháp hợp lệ trong các mẫu và thảo luận
về lý do tại sao và khi nào bạn có thể muốn sử dụng từng loại.

### Khớp với các hằng (Literals)

Như bạn đã thấy trong Chương 6, bạn có thể khớp các mẫu trực tiếp với các hằng. Đoạn
mã sau đây đưa ra một số ví dụ:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Mã này in ra `one` bởi vì giá trị trong `x` là 1. Cú pháp này hữu ích
khi bạn muốn mã của mình thực hiện một hành động nếu nó nhận được một giá trị cụ thể
cụ thể.

### Khớp với các biến đã đặt tên

Các biến đã đặt tên là các mẫu irrefutable khớp với bất kỳ giá trị nào, và chúng ta đã sử dụng
chúng nhiều lần trong cuốn sách này. Tuy nhiên, có một sự phức tạp khi bạn sử dụng
các biến đã đặt tên trong các biểu thức `match`, `if let`, hoặc `while let`. Bởi vì mỗi
loại biểu thức này bắt đầu một phạm vi (scope) mới, các biến được khai báo như một phần của
mẫu bên trong biểu thức sẽ che bóng (shadow) những biến có cùng tên ở bên ngoài, như
là trường hợp của tất cả các biến. Trong Liệt kê 19-11, chúng ta khai báo một biến tên là
`x` với giá trị `Some(5)` và một biến `y` với giá trị `10`. Sau đó chúng ta
tạo một biểu thức `match` trên giá trị `x`. Hãy nhìn vào các mẫu trong các nhánh match
và `println!` ở cuối, và cố gắng tìm hiểu xem mã sẽ in ra gì
trước khi chạy mã này hoặc đọc tiếp.

<Listing number="19-11" file-name="src/main.rs" caption="Một biểu thức `match` với một nhánh giới thiệu một biến mới che bóng một biến `y` hiện có">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

Hãy cùng xem qua những gì xảy ra khi biểu thức `match` chạy. Mẫu
trong nhánh match đầu tiên không khớp với giá trị đã định nghĩa của `x`, vì vậy mã
tiếp tục.

Mẫu trong nhánh match thứ hai giới thiệu một biến mới tên là `y` mà
sẽ khớp với bất kỳ giá trị nào bên trong một giá trị `Some`. Bởi vì chúng ta đang ở trong một phạm vi mới bên trong
biểu thức `match`, đây là một biến `y` mới, không phải biến `y` chúng ta đã khai báo ở
đầu với giá trị `10`. Liên kết `y` mới này sẽ khớp với bất kỳ giá trị nào
bên trong một `Some`, đó là những gì chúng ta có trong `x`. Do đó, `y` mới này liên kết với
giá trị bên trong của `Some` trong `x`. Giá trị đó là `5`, vì vậy biểu thức cho
nhánh đó thực thi và in ra `Matched, y = 5`.

Nếu `x` là một giá trị `None` thay vì `Some(5)`, các mẫu trong hai
nhánh đầu tiên sẽ không khớp, vì vậy giá trị sẽ khớp với dấu
gạch dưới. Chúng ta đã không giới thiệu biến `x` trong mẫu của nhánh
gạch dưới, vì vậy `x` trong biểu thức vẫn là `x` bên ngoài chưa
bị che bóng. Trong trường hợp giả định này, `match` sẽ in ra `Default
case, x = None`.

Khi biểu thức `match` kết thúc, phạm vi của nó kết thúc, và phạm vi của
biến `y` bên trong cũng vậy. Dòng `println!` cuối cùng tạo ra `at the end: x = Some(5), y = 10`.

Để tạo một biểu thức `match` so sánh các giá trị của `x` và
`y` bên ngoài, thay vì giới thiệu một biến mới che bóng biến `y`
hiện có, chúng ta sẽ cần sử dụng một điều kiện match guard (bảo vệ khớp) thay thế. Chúng ta sẽ nói
về match guard sau trong phần [“Các điều kiện bổ sung với Match
Guards”](#extra-conditionals-with-match-guards)<!-- ignore -->.

### Nhiều mẫu

Bạn có thể khớp nhiều mẫu bằng cú pháp `|`, đây là toán tử _hoặc_ (or) của
mẫu. Ví dụ, trong đoạn mã sau, chúng ta khớp giá trị của `x` với
các nhánh match, nhánh đầu tiên có một tùy chọn _hoặc_, nghĩa là nếu giá trị của
`x` khớp với một trong hai giá trị trong nhánh đó, mã của nhánh đó sẽ chạy:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Mã này in ra `one or two`.

### Khớp các khoảng giá trị với `..=`

Cú pháp `..=` cho phép chúng ta khớp với một khoảng giá trị bao gồm cả hai đầu (inclusive range). Trong
đoạn mã sau, khi một mẫu khớp với bất kỳ giá trị nào trong khoảng
đã cho, nhánh đó sẽ thực thi:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Nếu `x` là `1`, `2`, `3`, `4`, hoặc `5`, nhánh đầu tiên sẽ khớp. Cú pháp này
tiện lợi hơn cho nhiều giá trị match so với việc sử dụng toán tử `|` để diễn đạt
cùng một ý tưởng; nếu chúng ta sử dụng `|` chúng ta sẽ phải chỉ định `1 | 2 | 3 | 4 |
5`. Chỉ định một khoảng ngắn hơn nhiều, đặc biệt nếu chúng ta muốn khớp, ví dụ, bất kỳ
số nào từ 1 đến 1.000!

Trình biên dịch kiểm tra xem khoảng đó có trống hay không tại thời điểm biên dịch, và bởi vì
các kiểu duy nhất mà Rust có thể biết liệu một khoảng có trống hay không là `char` và
các giá trị số, các khoảng chỉ được phép với các giá trị số hoặc `char`.

Dưới đây là một ví dụ sử dụng các khoảng giá trị `char`:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust có thể biết rằng `'c'` nằm trong khoảng của mẫu đầu tiên và in ra `early
ASCII letter`.

### Phá cấu trúc để chia nhỏ các giá trị

Chúng ta cũng có thể sử dụng các mẫu để phá cấu trúc (destructure) struct, enum, và tuple để sử dụng
các phần khác nhau của các giá trị này. Hãy cùng đi qua từng loại giá trị.

#### Phá cấu trúc Struct

Liệt kê 19-12 hiển thị một struct `Point` với hai trường, `x` và `y`, mà chúng ta có thể
chia nhỏ bằng cách sử dụng một mẫu với câu lệnh `let`.

<Listing number="19-12" file-name="src/main.rs" caption="Phá cấu trúc các trường của một struct thành các biến riêng biệt">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Mã này tạo ra các biến `a` và `b` khớp với các giá trị của các trường `x`
và `y` của struct `p`. Ví dụ này cho thấy tên của các
biến trong mẫu không nhất thiết phải khớp với tên trường của struct.
Tuy nhiên, việc khớp tên biến với tên trường là phổ biến để làm cho nó
dễ nhớ hơn biến nào đến từ trường nào. Vì cách sử dụng phổ biến này,
và vì việc viết `let Point { x: x, y: y } = p;` chứa rất nhiều sự lặp lại,
Rust có một cách viết tắt cho các mẫu khớp với các trường struct:
bạn chỉ cần liệt kê tên của trường struct, và các biến được tạo
từ mẫu sẽ có cùng tên. Liệt kê 19-13 hoạt động theo cùng cách
với mã trong Liệt kê 19-12, nhưng các biến được tạo trong mẫu `let`
là `x` và `y` thay vì `a` và `b`.

<Listing number="19-13" file-name="src/main.rs" caption="Phá cấu trúc các trường struct bằng cách sử dụng cách viết tắt trường struct">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Mã này tạo ra các biến `x` và `y` khớp với các trường `x` và `y`
của biến `p`. Kết quả là các biến `x` và `y` chứa các
giá trị từ struct `p`.

Chúng ta cũng có thể phá cấu trúc với các giá trị hằng như một phần của mẫu struct
thay vì tạo các biến cho tất cả các trường. Làm như vậy cho phép chúng ta kiểm tra
một số trường cho các giá trị cụ thể trong khi tạo các biến để
phá cấu trúc các trường khác.

Trong Liệt kê 19-14, chúng ta có một biểu thức `match` phân tách các giá trị `Point`
thành ba trường hợp: các điểm nằm trực tiếp trên trục `x` (đúng khi
`y = 0`), trên trục `y` (`x = 0`), hoặc không nằm trên trục nào.

<Listing number="19-14" file-name="src/main.rs" caption="Phá cấu trúc và khớp các giá trị hằng trong một mẫu">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

Nhánh đầu tiên sẽ khớp với bất kỳ điểm nào nằm trên trục `x` bằng cách chỉ định rằng
trường `y` khớp nếu giá trị của nó khớp với hằng `0`. Mẫu vẫn
tạo ra một biến `x` mà chúng ta có thể sử dụng trong mã cho nhánh này.

Tương tự, nhánh thứ hai khớp với bất kỳ điểm nào trên trục `y` bằng cách chỉ định rằng
trường `x` khớp nếu giá trị của nó là `0` và tạo một biến `y` cho
giá trị của trường `y`. Nhánh thứ ba không chỉ định bất kỳ hằng nào, vì vậy nó
khớp với bất kỳ `Point` nào khác và tạo các biến cho cả hai trường `x` và `y`.

Trong ví dụ này, giá trị `p` khớp với nhánh thứ hai nhờ vào việc `x`
chứa giá trị `0`, vì vậy mã này sẽ in ra `On the y axis at 7`.

Hãy nhớ rằng một biểu thức `match` sẽ ngừng kiểm tra các nhánh khi nó đã tìm thấy
mẫu khớp đầu tiên, vì vậy ngay cả khi `Point { x: 0, y: 0}` nằm trên trục `x`
và trục `y`, mã này sẽ chỉ in ra `On the x axis at 0`.

#### Phá cấu trúc Enum

Chúng ta đã phá cấu trúc enum trong cuốn sách này (ví dụ, Liệt kê 6-5), nhưng chúng ta chưa
thảo luận rõ ràng rằng mẫu để phá cấu trúc một enum tương ứng với
cách dữ liệu được lưu trữ bên trong enum được định nghĩa. Ví dụ, trong Liệt kê
19-15 chúng ta sử dụng enum `Message` từ Liệt kê 6-2 và viết một `match` với
các mẫu sẽ phá cấu trúc từng giá trị bên trong.

<Listing number="19-15" file-name="src/main.rs" caption="Phá cấu trúc các biến thể enum chứa các loại giá trị khác nhau">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Mã này sẽ in ra `Change color to red 0, green 160, and blue 255`. Hãy thử
thay đổi giá trị của `msg` để thấy mã từ các nhánh khác chạy.

Đối với các biến thể enum không có bất kỳ dữ liệu nào, như `Message::Quit`, chúng ta không thể phá cấu trúc
giá trị thêm nữa. Chúng ta chỉ có thể khớp trên giá trị hằng `Message::Quit`,
và không có biến nào trong mẫu đó.

Đối với các biến thể enum giống struct, chẳng hạn như `Message::Move`, chúng ta có thể sử dụng một mẫu
tương tự như mẫu chúng ta chỉ định để khớp struct. Sau tên biến thể, chúng ta
đặt các dấu ngoặc nhọn và sau đó liệt kê các trường với các biến để chúng ta chia nhỏ
các phần để sử dụng trong mã cho nhánh này. Ở đây chúng ta sử dụng dạng viết tắt
như chúng ta đã làm trong Liệt kê 19-13.

Đối với các biến thể enum giống tuple, như `Message::Write` giữ một tuple với một
phần tử và `Message::ChangeColor` giữ một tuple với ba phần tử,
mẫu tương tự như mẫu chúng ta chỉ định để khớp tuple. Số lượng
biến trong mẫu phải khớp với số lượng phần tử trong biến thể mà chúng ta
đang khớp.

#### Phá cấu trúc các Struct và Enum lồng nhau

Cho đến nay, các ví dụ của chúng ta đều là khớp struct hoặc enum ở mức sâu một cấp,
nhưng việc khớp có thể hoạt động trên cả các mục lồng nhau! Ví dụ, chúng ta có thể tái cấu trúc
mã trong Liệt kê 19-15 để hỗ trợ các màu RGB và HSV trong thông báo
`ChangeColor`, như được hiển thị trong Liệt kê 19-16.

<Listing number="19-16" caption="Khớp trên các enum lồng nhau">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

Mẫu của nhánh đầu tiên trong biểu thức `match` khớp với một biến thể enum
`Message::ChangeColor` chứa một biến thể `Color::Rgb`; sau đó
mẫu liên kết với ba giá trị `i32` bên trong. Mẫu của nhánh thứ hai
cũng khớp với một biến thể enum `Message::ChangeColor`, nhưng enum bên trong
khớp với `Color::Hsv` thay thế. Chúng ta có thể chỉ định các điều kiện phức tạp này trong một
biểu thức `match`, mặc dù có hai enum liên quan.

#### Phá cấu trúc Struct và Tuple

Chúng ta có thể trộn, khớp và lồng các mẫu phá cấu trúc theo những cách thậm chí còn phức tạp hơn.
Ví dụ sau đây cho thấy một sự phá cấu trúc phức tạp nơi chúng ta lồng các struct và
tuple bên trong một tuple và phá cấu trúc tất cả các giá trị nguyên thủy ra ngoài:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Mã này cho phép chúng ta chia các kiểu phức tạp thành các phần thành phần của chúng để chúng ta có thể sử dụng
các giá trị mà chúng ta quan tâm một cách riêng biệt.

Phá cấu trúc với các mẫu là một cách thuận tiện để sử dụng các phần của giá trị, chẳng
hạn như giá trị từ mỗi trường trong một struct, riêng biệt với nhau.

### Bỏ qua các giá trị trong một mẫu

Bạn đã thấy rằng đôi khi việc bỏ qua các giá trị trong một mẫu là hữu ích, chẳng hạn
như trong nhánh cuối cùng của một `match`, để có một mẫu bao quát tất cả (catchall) mà thực tế không
làm gì cả nhưng vẫn tính đến tất cả các giá trị khả thi còn lại. Có một vài
cách để bỏ qua toàn bộ giá trị hoặc các phần của giá trị trong một mẫu: sử dụng mẫu `_`
(mà bạn đã thấy), sử dụng mẫu `_` bên trong một mẫu khác, sử dụng một tên bắt đầu
với dấu gạch dưới, hoặc sử dụng `..` để bỏ qua các phần còn lại của một giá trị. Hãy cùng khám phá
cách thức và lý do tại sao nên sử dụng từng loại mẫu này.

<!-- Old link, do not remove -->

<a id="ignoring-an-entire-value-with-_"></a>

#### Toàn bộ một giá trị với `_`

Chúng ta đã sử dụng dấu gạch dưới như một mẫu ký tự đại diện (wildcard) sẽ khớp với bất kỳ giá trị nào nhưng
không liên kết với giá trị đó. Điều này đặc biệt hữu ích như là nhánh cuối cùng trong một biểu thức
`match`, nhưng chúng ta cũng có thể sử dụng nó trong bất kỳ mẫu nào, bao gồm cả các tham số
hàm, như được hiển thị trong Liệt kê 19-17.

<Listing number="19-17" file-name="src/main.rs" caption="Sử dụng `_` trong một chữ ký hàm">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Mã này sẽ hoàn toàn bỏ qua giá trị `3` được truyền vào như đối số đầu tiên,
và sẽ in ra `This code only uses the y parameter: 4`.

Trong hầu hết các trường hợp khi bạn không còn cần một tham số hàm cụ thể nào đó, bạn
sẽ thay đổi chữ ký để nó không bao gồm tham số không sử dụng đó. Bỏ qua
một tham số hàm có thể đặc biệt hữu ích trong các trường hợp, ví dụ,
bạn đang triển khai một trait nơi bạn cần một chữ ký kiểu nhất định nhưng
thân hàm trong triển khai của bạn không cần một trong các tham số. Sau đó bạn
tránh nhận được cảnh báo của trình biên dịch về các tham số hàm không được sử dụng, như bạn
sẽ gặp nếu bạn sử dụng một cái tên thay thế.

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### Các phần của một giá trị với một `_` lồng nhau

Chúng ta cũng có thể sử dụng `_` bên trong một mẫu khác để chỉ bỏ qua một phần của một giá trị, ví
dụ, khi chúng ta muốn kiểm tra chỉ một phần của một giá trị nhưng không có nhu cầu sử dụng
các phần khác trong mã tương ứng mà chúng ta muốn chạy. Liệt kê 19-18 cho thấy mã
chịu trách nhiệm quản lý giá trị của một cài đặt. Các yêu cầu nghiệp vụ là
người dùng không được phép ghi đè lên một tùy chỉnh hiện có của một cài đặt
nhưng có thể hủy cài đặt và cung cấp cho nó một giá trị nếu nó hiện đang chưa được thiết lập.

<Listing number="19-18" caption=" Sử dụng một dấu gạch dưới bên trong các mẫu khớp với các biến thể `Some` khi chúng ta không cần sử dụng giá trị bên trong `Some`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Mã này sẽ in ra `Can't overwrite an existing customized value` và sau đó
`setting is Some(5)`. Trong nhánh match đầu tiên, chúng ta không cần khớp hoặc sử dụng
các giá trị bên trong cả hai biến thể `Some`, nhưng chúng ta cần kiểm tra trường hợp
khi `setting_value` và `new_setting_value` là biến thể `Some`. Trong trường hợp
đó, chúng ta in ra lý do không thay đổi `setting_value`, và nó không bị
thay đổi.

Trong tất cả các trường hợp khác (nếu `setting_value` hoặc `new_setting_value` là `None`)
được biểu thị bằng mẫu `_` trong nhánh thứ hai, chúng ta muốn cho phép
`setting_value` được thiết lập thành `new_setting_value`.

Chúng ta cũng có thể sử dụng dấu gạch dưới ở nhiều nơi trong một mẫu để bỏ qua
các giá trị cụ thể. Liệt kê 19-19 cho thấy một ví dụ về việc bỏ qua giá trị thứ hai và
thứ tư trong một tuple có năm mục.

<Listing number="19-19" caption="Bỏ qua nhiều phần của một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

Mã này sẽ in ra `Some numbers: 2, 8, 32`, và các giá trị `4` và `16` sẽ
bị bỏ qua.

<!-- Old link, do not remove -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### Một biến không sử dụng bằng cách bắt đầu tên của nó với `_`

Nếu bạn tạo một biến nhưng không sử dụng nó ở bất cứ đâu, Rust thường sẽ đưa ra một
cảnh báo vì một biến không được sử dụng có thể là một lỗi. Tuy nhiên, đôi khi việc
tạo một biến mà bạn chưa sử dụng ngay là hữu ích, chẳng hạn như khi bạn đang
tạo nguyên mẫu hoặc mới bắt đầu một dự án. Trong tình huống này, bạn có thể bảo Rust
đừng cảnh báo bạn về biến không được sử dụng bằng cách bắt đầu tên của biến
với một dấu gạch dưới. Trong Liệt kê 19-20, chúng ta tạo hai biến không được sử dụng, nhưng khi
chúng ta biên dịch mã này, chúng ta chỉ nhận được cảnh báo về một trong số chúng.

<Listing number="19-20" file-name="src/main.rs" caption="Bắt đầu tên biến bằng dấu gạch dưới để tránh nhận cảnh báo biến không sử dụng">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Ở đây, chúng ta nhận được cảnh báo về việc không sử dụng biến `y`, nhưng chúng ta không nhận được
cảnh báo về việc không sử dụng `_x`.

Lưu ý rằng có một sự khác biệt nhỏ giữa việc chỉ sử dụng `_` và sử dụng một tên
bắt đầu bằng dấu gạch dưới. Cú pháp `_x` vẫn liên kết giá trị với
biến, trong khi `_` hoàn toàn không liên kết. Để cho thấy một trường hợp mà sự
khác biệt này quan trọng, Liệt kê 19-21 sẽ cung cấp cho chúng ta một lỗi.

<Listing number="19-21" caption="Một biến không được sử dụng bắt đầu bằng dấu gạch dưới vẫn liên kết giá trị, điều này có thể lấy quyền sở hữu của giá trị">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Chúng ta sẽ nhận được một lỗi vì giá trị `s` vẫn sẽ được chuyển (move) vào `_s`,
điều này ngăn cản chúng ta sử dụng lại `s`. Tuy nhiên, việc sử dụng dấu gạch dưới một mình
không bao giờ liên kết với giá trị. Liệt kê 19-22 sẽ biên dịch mà không có bất kỳ lỗi nào
vì `s` không bị chuyển vào `_`.

<Listing number="19-22" caption="Sử dụng dấu gạch dưới không liên kết giá trị">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

Mã này hoạt động tốt vì chúng ta không bao giờ liên kết `s` với bất cứ thứ gì; nó không bị chuyển đi.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### Các phần còn lại của một giá trị với `..`

Với các giá trị có nhiều phần, chúng ta có thể sử dụng cú pháp `..` để sử dụng các phần
cụ thể và bỏ qua phần còn lại, tránh việc phải liệt kê các dấu gạch dưới cho mỗi
giá trị bị bỏ qua. Mẫu `..` bỏ qua bất kỳ phần nào của một giá trị mà chúng ta chưa
khớp rõ ràng trong phần còn lại của mẫu. Trong Liệt kê 19-23, chúng ta có một
struct `Point` giữ một tọa độ trong không gian ba chiều. Trong biểu thức
`match`, chúng ta chỉ muốn thao tác trên tọa độ `x` và bỏ qua
các giá trị trong các trường `y` và `z`.

<Listing number="19-23" caption="Bỏ qua tất cả các trường của một `Point` ngoại trừ `x` bằng cách sử dụng `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

Chúng ta liệt kê giá trị `x` và sau đó chỉ bao gồm mẫu `..`. Điều này nhanh hơn
so với việc phải liệt kê `y: _` và `z: _`, đặc biệt là khi chúng ta đang làm việc với
các struct có nhiều trường trong các tình huống mà chỉ có một hoặc hai trường là
có liên quan.

Cú pháp `..` sẽ mở rộng thành bao nhiêu giá trị tùy thích. Liệt kê 19-24
cho thấy cách sử dụng `..` với một tuple.

<Listing number="19-24" file-name="src/main.rs" caption="Chỉ khớp các giá trị đầu tiên và cuối cùng trong một tuple và bỏ qua tất cả các giá trị khác">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

Trong mã này, giá trị đầu tiên và cuối cùng được khớp với `first` và `last`.
`..` sẽ khớp và bỏ qua mọi thứ ở giữa.

Tuy nhiên, việc sử dụng `..` phải không được mơ hồ. Nếu không rõ giá trị nào
được dự định để khớp và giá trị nào nên được bỏ qua, Rust sẽ báo lỗi cho chúng ta.
Liệt kê 19-25 cho thấy một ví dụ về việc sử dụng `..` một cách mơ hồ, vì vậy nó sẽ không
biên dịch.

<Listing number="19-25" file-name="src/main.rs" caption="Một nỗ lực sử dụng `..` theo cách mơ hồ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Khi chúng ta biên dịch ví dụ này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust không thể xác định được có bao nhiêu giá trị trong tuple cần bỏ qua
trước khi khớp một giá trị với `second` và sau đó có bao nhiêu giá trị tiếp theo cần bỏ qua
sau đó. Mã này có thể có nghĩa là chúng ta muốn bỏ qua `2`, liên kết
`second` với `4`, và sau đó bỏ qua `8`, `16`, và `32`; hoặc chúng ta muốn bỏ qua
`2` và `4`, liên kết `second` với `8`, và sau đó bỏ qua `16` và `32`; vân vân.
Tên biến `second` không có ý nghĩa gì đặc biệt đối với Rust, vì vậy chúng ta nhận được một
lỗi biên dịch vì việc sử dụng `..` ở hai nơi như thế này là mơ hồ.

### Các điều kiện bổ sung với Match Guard

Một _match guard_ là một điều kiện `if` bổ sung, được chỉ định sau mẫu trong
một nhánh `match`, điều kiện này cũng phải khớp để nhánh đó được chọn. Match guard
hữu ích để diễn đạt các ý tưởng phức tạp hơn so với chỉ dùng mẫu đơn thuần. Tuy nhiên,
lưu ý rằng chúng chỉ khả dụng trong các biểu thức `match`, không có trong các biểu thức `if let` hoặc
`while let`.

Điều kiện có thể sử dụng các biến được tạo trong mẫu. Liệt kê 19-26 cho thấy một
`match` nơi nhánh đầu tiên có mẫu `Some(x)` và cũng có một match guard
là `if x % 2 == 0` (sẽ là `true` nếu số đó là số chẵn).

<Listing number="19-26" caption="Thêm một match guard vào một mẫu">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Ví dụ này sẽ in ra `The number 4 is even`. Khi `num` được so sánh với
mẫu trong nhánh đầu tiên, nó khớp vì `Some(4)` khớp với `Some(x)`. Sau đó
match guard kiểm tra xem phần dư của phép chia `x` cho 2 có bằng
0 hay không, và vì nó bằng 0, nhánh đầu tiên được chọn.

Nếu `num` là `Some(5)` thay thế, match guard trong nhánh đầu tiên sẽ
là `false` vì phần dư của 5 chia cho 2 là 1, không
bằng 0. Rust sau đó sẽ chuyển sang nhánh thứ hai, nhánh này sẽ khớp vì
nhánh thứ hai không có match guard và do đó khớp với bất kỳ biến thể `Some` nào.

Không có cách nào để diễn đạt điều kiện `if x % 2 == 0` bên trong một mẫu, vì vậy
match guard cho chúng ta khả năng diễn đạt logic này. Nhược điểm của
khả năng diễn đạt bổ sung này là các nhánh có match guard không được "tính" vào
tính đầy đủ (exhaustiveness). Vì vậy, ngay cả khi chúng ta thêm `Some(x) if x % 2 == 1` như một nhánh bổ sung, chúng ta vẫn
cần nhánh `Some(x)` không có guard.

Trong Liệt kê 19-11, chúng ta đã đề cập rằng chúng ta có thể sử dụng match guard để giải quyết
vấn đề che bóng mẫu của mình. Hãy nhớ lại rằng chúng ta đã tạo một biến mới bên trong
mẫu trong biểu thức `match` thay vì sử dụng biến bên ngoài
`match`. Biến mới đó có nghĩa là chúng ta không thể kiểm tra giá trị của
biến bên ngoài. Liệt kê 19-27 cho thấy cách chúng ta có thể sử dụng một match guard để khắc phục
vấn đề này.

<Listing number="19-27" file-name="src/main.rs" caption="Sử dụng một match guard để kiểm tra sự bằng nhau với một biến bên ngoài">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Mã này bây giờ sẽ in ra `Default case, x = Some(5)`. Mẫu trong nhánh match
thứ hai không giới thiệu một biến `y` mới che bóng `y` bên ngoài,
nghĩa là chúng ta có thể sử dụng `y` bên ngoài trong match guard. Thay vì chỉ định
mẫu là `Some(y)`, mẫu này sẽ che bóng `y` bên ngoài, chúng ta chỉ định
`Some(n)`. Điều này tạo ra một biến `n` mới không che bóng bất cứ thứ gì vì
không có biến `n` nào bên ngoài `match`.

Match guard `if n == y` không phải là một mẫu và do đó không giới thiệu các
biến mới. `y` này _là_ `y` bên ngoài thay vì một `y` mới che bóng nó, và
chúng ta có thể tìm kiếm một giá trị có cùng giá trị với `y` bên ngoài bằng cách so sánh
`n` với `y`.

Bạn cũng có thể sử dụng toán tử _hoặc_ `|` trong một match guard để chỉ định nhiều
mẫu; điều kiện match guard sẽ áp dụng cho tất cả các mẫu. Liệt kê
19-28 cho thấy thứ tự ưu tiên (precedence) khi kết hợp một mẫu sử dụng `|` với một match
guard. Phần quan trọng của ví dụ này là match guard `if y`
áp dụng cho `4`, `5`, _và_ `6`, mặc dù nó có vẻ như `if y` chỉ
áp dụng cho `6`.

<Listing number="19-28" caption="Kết hợp nhiều mẫu với một match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

Điều kiện match quy định rằng nhánh chỉ khớp nếu giá trị của `x`
bằng `4`, `5`, hoặc `6` _và_ nếu `y` là `true`. Khi mã này chạy,
mẫu của nhánh đầu tiên khớp vì `x` là `4`, nhưng match guard `if y`
là `false`, vì vậy nhánh đầu tiên không được chọn. Mã chuyển sang nhánh thứ hai,
nhánh này khớp, và chương trình này in ra `no`. Lý do là điều kiện
`if` áp dụng cho toàn bộ mẫu `4 | 5 | 6`, chứ không chỉ cho giá trị cuối cùng
`6`. Nói cách khác, thứ tự ưu tiên của một match guard so với một mẫu
hoạt động như thế này:

```text
(4 | 5 | 6) if y => ...
```

thay vì thế này:

```text
4 | 5 | (6 if y) => ...
```

Sau khi chạy mã, hành vi ưu tiên là rõ ràng: nếu match guard
chỉ được áp dụng cho giá trị cuối cùng trong danh sách các giá trị được chỉ định bằng
toán tử `|`, nhánh đó sẽ khớp và chương trình sẽ in ra
`yes`.

### Liên kết `@` (@ Bindings)

Toán tử _at_ `@` cho phép chúng ta tạo một biến giữ một giá trị đồng thời với
việc chúng ta kiểm tra giá trị đó để khớp mẫu. Trong Liệt kê 19-29, chúng ta muốn
kiểm tra xem một trường `id` của `Message::Hello` có nằm trong khoảng `3..=7` hay không. Chúng ta cũng
muốn liên kết giá trị đó với biến `id_variable` để có thể sử dụng nó trong
mã liên kết với nhánh đó. Chúng ta có thể đặt tên biến này là `id`, giống như tên
trường, nhưng trong ví dụ này chúng ta sẽ sử dụng một cái tên khác.

<Listing number="19-29" caption="Sử dụng `@` để liên kết với một giá trị trong một mẫu đồng thời kiểm tra nó">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Ví dụ này sẽ in ra `Found an id in range: 5`. Bằng cách chỉ định `id_variable
@` trước khoảng `3..=7`, chúng ta đang nắm bắt bất kỳ giá trị nào khớp với khoảng
đồng thời kiểm tra xem giá trị đó có khớp với mẫu khoảng hay không.

Trong nhánh thứ hai, nơi chúng ta chỉ có một khoảng được chỉ định trong mẫu, mã
liên kết với nhánh đó không có một biến chứa giá trị thực tế của
trường `id`. Giá trị của trường `id` có thể là 10, 11, hoặc 12, nhưng
mã đi kèm với mẫu đó không biết nó là giá trị nào. Mã mẫu
không thể sử dụng giá trị từ trường `id`, vì chúng ta chưa lưu
giá trị `id` vào một biến.

Trong nhánh cuối cùng, nơi chúng ta đã chỉ định một biến mà không có khoảng, chúng ta có
giá trị sẵn sàng để sử dụng trong mã của nhánh trong một biến tên là `id`. Lý
do là chúng ta đã sử dụng cú pháp viết tắt trường struct. Nhưng chúng ta chưa
áp dụng bất kỳ kiểm tra nào cho giá trị trong trường `id` ở nhánh này, như chúng ta đã làm với
hai nhánh đầu tiên: bất kỳ giá trị nào cũng sẽ khớp với mẫu này.

Sử dụng `@` cho phép chúng ta kiểm tra một giá trị và lưu nó vào một biến trong cùng một mẫu.

{{#quiz ../quizzes/ch18-03-pattern-syntax.toml}}

## Tổng kết

Các mẫu của Rust rất hữu ích trong việc phân biệt giữa các loại dữ liệu khác
nhau. Khi được sử dụng trong các biểu thức `match`, Rust đảm bảo các mẫu của bạn bao quát mọi
giá trị có thể, nếu không chương trình của bạn sẽ không biên dịch. Các mẫu trong các câu lệnh `let` và
các tham số hàm làm cho các cấu trúc đó hữu ích hơn, cho phép
phá cấu trúc các giá trị thành các phần nhỏ hơn đồng thời gán các
phần đó cho các biến. Chúng ta có thể tạo các mẫu đơn giản hoặc phức tạp để phù hợp với nhu cầu của mình.

Tiếp theo, cho chương áp chót của cuốn sách, chúng ta sẽ xem xét một số khía cạnh nâng cao
của nhiều tính năng khác nhau của Rust.
