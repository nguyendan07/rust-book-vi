## Các đặc tính của các ngôn ngữ hướng đối tượng

Không có sự đồng thuận trong cộng đồng lập trình về những tính năng mà một
ngôn ngữ phải có để được coi là hướng đối tượng. Rust bị ảnh hưởng bởi nhiều
mô hình lập trình, bao gồm cả OOP; ví dụ, chúng ta đã khám phá các tính năng
đến từ lập trình hàm trong Chương 13. Có thể cho rằng, các ngôn ngữ OOP
chia sẻ một số đặc tính chung nhất định, cụ thể là đối tượng (objects), tính đóng gói (encapsulation), và tính kế thừa (inheritance). Hãy xem mỗi
đặc tính đó có nghĩa là gì và liệu Rust có hỗ trợ nó hay không.

### Đối tượng chứa dữ liệu và hành vi

Cuốn sách _Design Patterns: Elements of Reusable Object-Oriented Software_ của
Erich Gamma, Richard Helm, Ralph Johnson, và John Vlissides (Addison-Wesley,
1994), thường được gọi là cuốn sách _The Gang of Four_ (Bộ tứ), là một danh mục các
mẫu thiết kế hướng đối tượng. Nó định nghĩa OOP theo cách này:

> Các chương trình hướng đối tượng được tạo thành từ các đối tượng. Một **đối tượng** (object) đóng gói cả
> dữ liệu và các thủ tục hoạt động trên dữ liệu đó. Các thủ tục thường
> được gọi là **phương thức** (methods) hoặc **thao tác** (operations).

Sử dụng định nghĩa này, Rust là hướng đối tượng: các `struct` và `enum` có dữ liệu,
và các khối `impl` cung cấp các phương thức trên các `struct` và `enum` đó. Mặc dù các `struct` và
`enum` với các phương thức không được _gọi_ là đối tượng, nhưng chúng cung cấp các
chức năng tương tự, theo định nghĩa về đối tượng của Bộ tứ.

### Tính đóng gói che giấu chi tiết triển khai

Một khía cạnh khác thường được liên kết với OOP là ý tưởng về _tính đóng gói_ (encapsulation),
nghĩa là các chi tiết triển khai của một đối tượng không thể truy cập được bởi mã nguồn
sử dụng đối tượng đó. Do đó, cách duy nhất để tương tác với một đối tượng là
thông qua API công khai của nó; mã sử dụng đối tượng không nên có khả năng đi sâu vào
bên trong đối tượng và thay đổi dữ liệu hoặc hành vi trực tiếp. Điều này cho phép
lập trình viên thay đổi và cấu trúc lại các phần bên trong của một đối tượng mà không cần
thay đổi mã nguồn sử dụng đối tượng đó.

Chúng ta đã thảo luận về cách kiểm soát tính đóng gói trong Chương 7: chúng ta có thể sử dụng từ khóa `pub`
để quyết định mô-đun, kiểu dữ liệu, hàm và phương thức nào trong mã của chúng ta
nên là công khai (public), và theo mặc định mọi thứ khác là riêng tư (private). Ví dụ, chúng ta
có thể định nghĩa một struct `AveragedCollection` có một trường chứa một vector
gồm các giá trị `i32`. Struct này cũng có thể có một trường chứa giá trị trung bình của
các giá trị trong vector, nghĩa là giá trị trung bình không cần phải được tính toán
theo yêu cầu bất cứ khi nào ai đó cần nó. Nói cách khác, `AveragedCollection` sẽ
lưu bộ nhớ đệm (cache) giá trị trung bình đã tính toán cho chúng ta. Liệt kê 18-1 có định nghĩa của
struct `AveragedCollection`:

<Listing number="18-1" file-name="src/lib.rs" caption="Một struct `AveragedCollection` duy trì một danh sách các số nguyên và giá trị trung bình của các mục trong bộ sưu tập">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

Struct này được đánh dấu `pub` để mã khác có thể sử dụng nó, nhưng các trường bên trong
struct vẫn ở chế độ riêng tư. Điều này quan trọng trong trường hợp này vì chúng ta muốn
đảm bảo rằng bất cứ khi nào một giá trị được thêm vào hoặc xóa khỏi danh sách, giá trị trung bình
cũng được cập nhật. Chúng ta thực hiện việc này bằng cách triển khai các phương thức `add`, `remove`, và `average`
trên struct, như được hiển thị trong Liệt kê 18-2:

<Listing number="18-2" file-name="src/lib.rs" caption="Triển khai các phương thức công khai `add`, `remove`, và `average` trên `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Các phương thức công khai `add`, `remove`, và `average` là những cách duy nhất để truy cập
hoặc sửa đổi dữ liệu trong một thực thể (instance) của `AveragedCollection`. Khi một mục được thêm
vào `list` bằng phương thức `add` hoặc bị xóa bằng phương thức `remove`, các
triển khai của mỗi phương thức sẽ gọi phương thức riêng tư `update_average` để xử lý
việc cập nhật trường `average`.

Chúng ta để các trường `list` và `average` ở chế độ riêng tư để mã bên ngoài không có cách nào
thêm hoặc xóa các mục vào hoặc từ trường `list` một cách trực tiếp;
nếu không, trường `average` có thể bị mất đồng bộ khi `list`
thay đổi. Phương thức `average` trả về giá trị trong trường `average`,
cho phép mã bên ngoài đọc `average` nhưng không được sửa đổi nó.

Bởi vì chúng ta đã đóng gói các chi tiết triển khai của struct
`AveragedCollection`, chúng ta có thể dễ dàng thay đổi các khía cạnh, chẳng hạn như cấu trúc dữ liệu,
trong tương lai. Ví dụ, chúng ta có thể sử dụng một `HashSet<i32>` thay vì một
`Vec<i32>` cho trường `list`. Miễn là chữ ký của các phương thức công khai `add`,
`remove`, và `average` được giữ nguyên, mã sử dụng
`AveragedCollection` sẽ không cần phải thay đổi. Nếu chúng ta để `list` ở chế độ công khai,
điều này chưa chắc đã đúng: `HashSet<i32>` và `Vec<i32>` có các
phương thức khác nhau để thêm và xóa các mục, vì vậy mã bên ngoài có thể sẽ phải thay đổi
nếu nó đang sửa đổi trực tiếp `list`.

Nếu tính đóng gói là một khía cạnh bắt buộc để một ngôn ngữ được coi là
hướng đối tượng, thì Rust đáp ứng yêu cầu đó. Tùy chọn sử dụng `pub` hoặc
không cho các phần khác nhau của mã cho phép đóng gói các chi tiết triển khai.

### Kế thừa như một Hệ thống kiểu và như việc Chia sẻ mã nguồn

_Tính kế thừa_ (Inheritance) là một cơ chế mà theo đó một đối tượng có thể kế thừa các thành phần từ
định nghĩa của một đối tượng khác, từ đó có được dữ liệu và hành vi của đối tượng cha
mà bạn không cần phải định nghĩa lại chúng.

Nếu một ngôn ngữ phải có tính kế thừa để được coi là hướng đối tượng, thì Rust không phải là một
ngôn ngữ như vậy. Không có cách nào để định nghĩa một struct kế thừa các trường và
triển khai phương thức của struct cha mà không sử dụng macro.

Tuy nhiên, nếu bạn đã quen với việc có tính kế thừa trong hộp công cụ lập trình của mình, bạn
có thể sử dụng các giải pháp khác trong Rust, tùy thuộc vào lý do bạn tìm đến
kế thừa ngay từ đầu.

Bạn sẽ chọn kế thừa vì hai lý do chính. Một là để tái sử dụng mã:
bạn có thể triển khai hành vi cụ thể cho một kiểu, và tính kế thừa cho phép bạn
tái sử dụng triển khai đó cho một kiểu khác. Bạn có thể làm điều này theo một cách hạn chế
trong mã Rust bằng cách sử dụng các triển khai phương thức trait mặc định, mà bạn đã thấy trong
Liệt kê 10-14 khi chúng ta thêm một triển khai mặc định của phương thức `summarize`
trên trait `Summary`. Bất kỳ kiểu nào triển khai trait `Summary` đều sẽ có
phương thức `summarize` khả dụng mà không cần thêm bất kỳ mã nào. Điều này
tương tự như một lớp cha có một triển khai của một phương thức và một
lớp con kế thừa cũng có triển khai của phương thức đó. Chúng ta cũng có thể
ghi đè (override) triển khai mặc định của phương thức `summarize` khi chúng ta
triển khai trait `Summary`, điều này tương tự như một lớp con ghi đè
triển khai của một phương thức được kế thừa từ một lớp cha.

Lý do khác để sử dụng tính kế thừa liên quan đến hệ thống kiểu: để cho phép một
kiểu con có thể được sử dụng ở cùng những nơi như kiểu cha. Điều này còn được
gọi là _tính đa hình_ (polymorphism), có nghĩa là bạn có thể thay thế nhiều đối tượng cho
nhau tại thời điểm chạy (runtime) nếu chúng chia sẻ một số đặc tính nhất định.

> ### Tính đa hình
>
> Đối với nhiều người, tính đa hình đồng nghĩa với tính kế thừa. Nhưng nó
> thực sự là một khái niệm tổng quát hơn, đề cập đến mã có thể hoạt động với dữ liệu
> thuộc nhiều kiểu khác nhau. Đối với tính kế thừa, những kiểu đó thường là các lớp con.
>
> Thay vào đó, Rust sử dụng generics để trừu tượng hóa các kiểu có thể có khác nhau và
> trait bounds để áp đặt các ràng buộc về những gì các kiểu đó phải cung cấp. Điều này
> đôi khi được gọi là _đa hình tham số có giới hạn_ (bounded parametric polymorphism).

Tính kế thừa gần đây đã không còn được ưa chuộng như một giải pháp thiết kế lập trình trong
nhiều ngôn ngữ lập trình vì nó thường có nguy cơ chia sẻ nhiều mã hơn
mức cần thiết. Các lớp con không phải lúc nào cũng nên chia sẻ tất cả các đặc tính của lớp cha
nhưng sẽ làm như vậy với tính kế thừa. Điều này có thể làm cho thiết kế của chương trình
kém linh hoạt hơn. Nó cũng dẫn đến khả năng gọi các phương thức trên các lớp con
mà không có ý nghĩa hoặc gây ra lỗi vì các phương thức đó không áp dụng cho
lớp con. Ngoài ra, một số ngôn ngữ sẽ chỉ cho phép đơn kế thừa (nghĩa là một
lớp con chỉ có thể kế thừa từ một lớp), làm hạn chế thêm tính linh hoạt của
thiết kế chương trình.

Vì những lý do này, Rust thực hiện cách tiếp cận khác là sử dụng trait object
thay vì tính kế thừa. Hãy xem cách trait object cho phép tính đa hình trong
Rust.

{{#quiz ../quizzes/ch17-01-what-is-oo.toml}}
