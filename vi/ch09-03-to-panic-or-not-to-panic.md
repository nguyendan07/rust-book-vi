## Nên `panic!` hay Không nên `panic!`

Vậy làm thế nào để bạn quyết định khi nào nên gọi `panic!` và khi nào nên trả về
`Result`? Khi mã bị panic, không có cách nào để phục hồi. Bạn có thể gọi `panic!`
cho bất kỳ tình huống lỗi nào, cho dù có cách khả thi để phục hồi hay không, nhưng
khi đó bạn đang thay mặt mã gọi (calling code) đưa ra quyết định rằng một tình huống là không thể phục hồi. Khi bạn chọn trả về một giá trị `Result`, bạn cung cấp các tùy chọn cho mã gọi. Mã gọi có thể chọn thử phục hồi theo
cách phù hợp với tình huống của nó, hoặc nó có thể quyết định rằng một giá trị `Err`
trong trường hợp này là không thể phục hồi, vì vậy nó có thể gọi `panic!` và biến
lỗi có thể phục hồi của bạn thành một lỗi không thể phục hồi. Do đó, trả về `Result` là một
lựa chọn mặc định tốt khi bạn định nghĩa một hàm có thể thất bại.

Trong các tình huống như ví dụ, mã nguyên mẫu (prototype code) và kiểm thử (tests), việc
viết mã gây panic thay vì trả về một `Result` sẽ phù hợp hơn. Hãy cùng
tìm hiểu lý do, sau đó thảo luận về các tình huống mà trình biên dịch không thể biết được rằng
thất bại là không thể xảy ra, nhưng bạn với tư cách là con người thì có thể. Chương này sẽ kết thúc với
một số hướng dẫn chung về cách quyết định xem có nên gây panic trong mã thư viện hay không.

### Ví dụ, Mã nguyên mẫu và Kiểm thử

Khi bạn đang viết một ví dụ để minh họa một khái niệm nào đó, việc bao gồm cả
mã xử lý lỗi mạnh mẽ có thể làm cho ví dụ trở nên kém rõ ràng hơn. Trong các ví dụ, mọi người
ngầm hiểu rằng một lời gọi đến một phương thức như `unwrap` vốn có thể gây panic chỉ là một
trình giữ chỗ (placeholder) cho cách mà bạn muốn ứng dụng của mình xử lý lỗi, điều này có thể
khác nhau tùy thuộc vào những gì phần còn lại của mã đang thực hiện.

Tương tự, các phương thức `unwrap` và `expect` rất tiện lợi khi viết mã nguyên mẫu,
trước khi bạn sẵn sàng quyết định cách xử lý lỗi. Chúng để lại những dấu hiệu rõ ràng trong
mã của bạn cho đến khi bạn sẵn sàng làm cho chương trình của mình mạnh mẽ hơn.

Nếu một lời gọi phương thức thất bại trong một bài kiểm thử, bạn sẽ muốn toàn bộ bài kiểm thử đó thất bại, ngay cả khi
phương thức đó không phải là chức năng đang được kiểm thử. Vì `panic!` là cách một bài kiểm thử
được đánh dấu là thất bại, nên việc gọi `unwrap` hoặc `expect` chính xác là những gì nên
xảy ra.

### Các trường hợp mà bạn có nhiều thông tin hơn trình biên dịch

Việc gọi `expect` cũng sẽ phù hợp khi bạn có một số logic khác
đảm bảo rằng `Result` sẽ có giá trị `Ok`, nhưng logic đó không phải là thứ mà
trình biên dịch hiểu được. Bạn vẫn sẽ có một giá trị `Result` mà bạn
cần xử lý: bất kỳ thao tác nào bạn đang gọi vẫn có khả năng
thất bại nói chung, mặc dù nó là không thể về mặt logic trong tình huống cụ thể của bạn.
Nếu bạn có thể đảm bảo bằng cách kiểm tra thủ công mã nguồn rằng bạn sẽ không bao giờ
có biến thể `Err`, thì việc gọi `expect` là hoàn toàn có thể chấp nhận được và hãy ghi lại
lý do bạn nghĩ rằng bạn sẽ không bao giờ có biến thể `Err` trong văn bản đối số.
Dưới đây là một ví dụ:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Chúng ta đang tạo một thực thể `IpAddr` bằng cách phân tích cú pháp một chuỗi được viết cứng (hardcoded). Chúng ta có thể thấy
rằng `127.0.0.1` là một địa chỉ IP hợp lệ, vì vậy việc sử dụng `expect` ở đây là chấp nhận được.
Tuy nhiên, việc có một chuỗi hợp lệ được viết cứng không làm thay đổi kiểu trả về
của phương thức `parse`: chúng ta vẫn nhận được một giá trị `Result`, và trình biên dịch vẫn
bắt chúng ta xử lý `Result` như thể biến thể `Err` là một khả năng có thể xảy ra
vì trình biên dịch không đủ thông minh để thấy rằng chuỗi này luôn là một địa chỉ IP hợp lệ.
Nếu chuỗi địa chỉ IP đến từ người dùng thay vì được viết cứng vào chương trình và do đó
_thực sự_ có khả năng thất bại, chúng ta chắc chắn sẽ muốn xử lý `Result` theo
một cách mạnh mẽ hơn. Việc đề cập đến giả định rằng địa chỉ IP này được viết cứng sẽ nhắc nhở chúng ta
thay đổi `expect` thành mã xử lý lỗi tốt hơn nếu trong tương lai, chúng ta cần lấy
địa chỉ IP từ một nguồn khác thay thế.

### Hướng dẫn xử lý lỗi

Bạn nên để mã của mình panic khi có khả năng mã của bạn có thể
rơi vào một trạng thái xấu (bad state). Trong ngữ cảnh này, một _trạng thái xấu_ là khi một giả định,
đảm bảo, hợp đồng (contract) hoặc bất biến (invariant) nào đó bị phá vỡ, chẳng hạn như khi các giá trị không hợp lệ,
giá trị mâu thuẫn hoặc giá trị bị thiếu được truyền vào mã của bạn—cộng với một hoặc
nhiều điều sau đây:

- Trạng thái xấu là thứ không được mong đợi, trái ngược với thứ
  có khả năng thỉnh thoảng xảy ra, như việc người dùng nhập dữ liệu sai
  định dạng.
- Mã của bạn sau thời điểm này cần dựa trên việc không nằm trong trạng thái xấu này,
  thay vì kiểm tra vấn đề ở mọi bước.
- Không có cách nào tốt để mã hóa thông tin này vào các kiểu dữ liệu bạn sử dụng. Chúng ta sẽ
  xem qua một ví dụ về ý của chúng tôi trong [“Mã hóa Trạng thái và Hành vi dưới dạng
  Kiểu dữ liệu”][encoding]<!-- ignore --> ở Chương 18.

Nếu ai đó gọi mã của bạn và truyền vào các giá trị không hợp lý, tốt nhất
là trả về một lỗi nếu bạn có thể để người dùng thư viện có thể quyết định họ muốn
làm gì trong trường hợp đó. Tuy nhiên, trong những trường hợp mà việc tiếp tục có thể
không an toàn hoặc có hại, lựa chọn tốt nhất có thể là gọi `panic!` và cảnh báo cho
người đang sử dụng thư viện của bạn về bug trong mã của họ để họ có thể sửa nó trong quá trình
phát triển. Tương tự, `panic!` thường phù hợp nếu bạn đang gọi
mã bên ngoài nằm ngoài tầm kiểm soát của bạn và nó trả về một trạng thái không hợp lệ mà
bạn không có cách nào sửa chữa.

Tuy nhiên, khi thất bại là điều được dự đoán trước, việc trả về một `Result` sẽ phù hợp hơn
là thực hiện một lời gọi `panic!`. Các ví dụ bao gồm một trình phân tích cú pháp (parser) được cung cấp dữ liệu sai định dạng
hoặc một yêu cầu HTTP trả về một trạng thái cho biết bạn đã chạm giới hạn tốc độ (rate limit).
Trong những trường hợp này, việc trả về một `Result` cho thấy rằng thất bại là một khả năng được dự đoán trước mà
mã gọi phải quyết định cách xử lý.

Khi mã của bạn thực hiện một thao tác có thể khiến người dùng gặp rủi ro nếu nó
được gọi bằng các giá trị không hợp lệ, mã của bạn nên xác minh các giá trị đó là hợp lệ trước
và panic nếu các giá trị đó không hợp lệ. Điều này chủ yếu vì lý do an toàn:
việc cố gắng thao tác trên dữ liệu không hợp lệ có thể khiến mã của bạn gặp phải các lỗ hổng bảo mật.
Đây là lý do chính mà thư viện tiêu chuẩn sẽ gọi `panic!` nếu bạn cố gắng
truy cập bộ nhớ ngoài giới hạn (out-of-bounds): cố gắng truy cập bộ nhớ không thuộc về
cấu trúc dữ liệu hiện tại là một vấn đề bảo mật phổ biến. Các hàm thường có các
_hợp đồng_ (contracts): hành vi của chúng chỉ được đảm bảo nếu các đầu vào đáp ứng các yêu cầu
cụ thể. Việc gây panic khi hợp đồng bị vi phạm là hợp lý vì vi phạm hợp đồng
luôn cho thấy một bug ở phía người gọi, và đó không phải là loại lỗi mà bạn muốn mã gọi phải xử lý một cách rõ ràng. Thực tế, không có cách nào hợp lý để mã gọi có thể phục hồi; những _lập trình viên_ gọi hàm cần
phải sửa lại mã. Các hợp đồng cho một hàm, đặc biệt là khi vi phạm sẽ gây ra
panic, nên được giải thích trong tài liệu API cho hàm đó.

Tuy nhiên, việc có rất nhiều bước kiểm tra lỗi trong tất cả các hàm của bạn sẽ rất dài dòng
và phiền phức. May mắn thay, bạn có thể sử dụng hệ thống kiểu của Rust (và do đó là việc kiểm tra kiểu
được thực hiện bởi trình biên dịch) để thực hiện nhiều bước kiểm tra cho bạn. Nếu hàm của bạn
có một kiểu dữ liệu cụ thể làm tham số, bạn có thể tiếp tục với logic của mã
khi biết rằng trình biên dịch đã đảm bảo bạn có một giá trị hợp lệ. Ví dụ,
nếu bạn có một kiểu dữ liệu thay vì một `Option`, chương trình của bạn mong đợi có _thứ gì đó_ thay vì _không có gì_. Khi đó mã của bạn không phải xử lý
hai trường hợp cho các biến thể `Some` và `None`: nó sẽ chỉ có một trường hợp cho việc
chắc chắn có một giá trị. Mã cố gắng truyền "không có gì" vào hàm của bạn thậm chí sẽ
không biên dịch được, vì vậy hàm của bạn không phải kiểm tra trường hợp đó khi chạy (runtime).
Một ví dụ khác là sử dụng kiểu số nguyên không dấu như `u32`, điều này đảm bảo
tham số không bao giờ là số âm.

### Tạo các kiểu tùy chỉnh để xác thực

Hãy đưa ý tưởng sử dụng hệ thống kiểu của Rust để đảm bảo chúng ta có một giá trị hợp lệ
tiến thêm một bước nữa và xem xét việc tạo một kiểu tùy chỉnh để xác thực. Hãy nhớ lại
trò chơi đoán số trong Chương 2, trong đó mã của chúng ta yêu cầu người dùng đoán một số
từ 1 đến 100. Chúng ta chưa bao giờ xác thực rằng dự đoán của người dùng nằm trong khoảng các số
đó trước khi kiểm tra nó với số bí mật của chúng ta; chúng ta chỉ xác thực rằng
dự đoán là số dương. Trong trường hợp này, hậu quả không quá thảm khốc: kết quả
đầu ra "Quá cao" hoặc "Quá thấp" của chúng ta vẫn chính xác. Nhưng sẽ là một sự cải tiến hữu ích
để hướng dẫn người dùng thực hiện các dự đoán hợp lệ và có hành vi khác nhau khi người dùng
đoán một số nằm ngoài phạm vi so với khi người dùng nhập chữ cái chẳng hạn.

Một cách để làm điều này là phân tích dự đoán thành một `i32` thay vì chỉ là một
`u32` để cho phép các số có khả năng là số âm, sau đó thêm một bước kiểm tra xem
số đó có nằm trong phạm vi hay không, như thế này:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

Biểu thức `if` kiểm tra xem giá trị của chúng ta có nằm ngoài phạm vi hay không, thông báo cho người dùng
về vấn đề, và gọi `continue` để bắt đầu lần lặp tiếp theo của vòng lặp
và yêu cầu một dự đoán khác. Sau biểu thức `if`, chúng ta có thể tiếp tục với các
so sánh giữa `guess` và số bí mật khi biết rằng `guess` nằm giữa 1 và 100.

Tuy nhiên, đây không phải là một giải pháp lý tưởng: nếu việc chương trình chỉ thao tác trên các giá trị từ 1 đến 100 là cực kỳ quan trọng, và nó có nhiều hàm
với yêu cầu này, việc có một bước kiểm tra như thế này trong mọi hàm sẽ rất
tẻ nhạt (và có thể ảnh hưởng đến hiệu suất).

Thay vào đó, chúng ta có thể tạo một kiểu mới trong một module chuyên dụng và đặt các bước xác thực vào
một hàm để tạo một thực thể của kiểu đó thay vì lặp lại các bước xác thực ở khắp mọi nơi. Bằng cách đó, các hàm có thể sử dụng kiểu mới một cách an toàn trong
chữ ký của chúng và tự tin sử dụng các giá trị mà chúng nhận được. Listing 9-13 cho thấy
một cách để định nghĩa một kiểu `Guess` mà sẽ chỉ tạo một thực thể của `Guess` nếu
hàm `new` nhận được một giá trị từ 1 đến 100.

<Listing number="9-13" caption="Một kiểu `Guess` sẽ chỉ tiếp tục với các giá trị từ 1 đến 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Lưu ý rằng mã này trong _src/guessing_game.rs_ phụ thuộc vào việc thêm một khai báo
module `mod guessing_game;` trong _src/lib.rs_ mà chúng tôi chưa hiển thị ở đây.
Bên trong file của module mới này, chúng ta định nghĩa một struct trong module đó tên là `Guess`
có một trường tên là `value` chứa một `i32`. Đây là nơi con số
sẽ được lưu trữ.

Sau đó, chúng ta triển khai một hàm liên kết (associated function) tên là `new` trên `Guess` để tạo ra
các thực thể của giá trị `Guess`. Hàm `new` được định nghĩa là có một
tham số tên là `value` kiểu `i32` và trả về một `Guess`. Mã trong
thân của hàm `new` kiểm tra `value` để đảm bảo nó nằm trong khoảng từ 1 đến 100.
Nếu `value` không vượt qua bài kiểm tra này, chúng ta thực hiện một lời gọi `panic!`, điều này sẽ cảnh báo cho
lập trình viên đang viết mã gọi rằng họ có một bug cần phải sửa, bởi vì việc tạo một `Guess` với một `value` nằm ngoài phạm vi này sẽ vi phạm
hợp đồng mà `Guess::new` đang dựa vào. Các điều kiện mà `Guess::new` có thể gây panic nên được thảo luận trong tài liệu API công khai của nó; chúng ta sẽ đề cập đến các quy ước về tài liệu chỉ ra khả năng xảy ra của một
`panic!` trong tài liệu API mà bạn tạo ở Chương 14. Nếu
`value` vượt qua bài kiểm tra, chúng ta tạo một `Guess` mới với trường `value` của nó được đặt
thành tham số `value` và trả về `Guess`.

Tiếp theo, chúng ta triển khai một phương thức tên là `value` mượn `self`, không có bất kỳ
tham số nào khác, và trả về một `i32`. Loại phương thức này đôi khi được gọi
là một _getter_ vì mục đích của nó là lấy một số dữ liệu từ các trường của nó và trả
nó về. Phương thức công khai này là cần thiết vì trường `value` của struct `Guess` là riêng tư. Việc trường `value` là riêng tư là rất quan trọng để mã
sử dụng struct `Guess` không được phép đặt `value` trực tiếp: mã bên ngoài
module `guessing_game` _phải_ sử dụng hàm `Guess::new` để tạo một
thực thể của `Guess`, từ đó đảm bảo không có cách nào để một `Guess` có một
`value` chưa được kiểm tra bởi các điều kiện trong hàm `Guess::new`.

Một hàm có tham số hoặc chỉ trả về các số từ 1 đến 100 sau đó có thể
khai báo trong chữ ký của nó rằng nó nhận hoặc trả về một `Guess` thay vì một
`i32` và sẽ không cần thực hiện bất kỳ bước kiểm tra bổ sung nào trong thân hàm của nó.

{{#quiz ../quizzes/ch09-03-panic-or-not.toml}}

## Tóm tắt

Các tính năng xử lý lỗi của Rust được thiết kế để giúp bạn viết mã mạnh mẽ hơn.
Macro `panic!` báo hiệu rằng chương trình của bạn đang ở trạng thái mà nó không thể xử lý và
cho phép bạn yêu cầu tiến trình dừng lại thay vì cố gắng tiếp tục với các giá trị không hợp lệ hoặc
không chính xác. Enum `Result` sử dụng hệ thống kiểu của Rust để chỉ ra rằng các
thao tác có thể thất bại theo cách mà mã của bạn có thể phục hồi. Bạn có thể sử dụng
`Result` để nói với mã gọi mã của bạn rằng nó cũng cần xử lý sự thành công hoặc thất bại tiềm ẩn. Việc sử dụng `panic!` và `Result` trong các tình huống thích hợp
sẽ làm cho mã của bạn đáng tin cậy hơn khi đối mặt với những vấn đề không thể tránh khỏi.

Bây giờ bạn đã thấy những cách hữu ích mà thư viện tiêu chuẩn sử dụng generic với
các enum `Option` và `Result`, chúng ta sẽ nói về cách generic hoạt động và cách bạn
có thể sử dụng chúng trong mã của mình.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
