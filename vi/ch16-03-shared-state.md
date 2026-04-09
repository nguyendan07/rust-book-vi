## Concurrency với trạng thái được chia sẻ (Shared-State Concurrency)

Truyền thông điệp là một cách tốt để xử lý concurrency, nhưng nó không phải là cách
duy nhất. Một phương pháp khác là cho nhiều luồng truy cập vào cùng một dữ liệu
được chia sẻ. Hãy xem xét lại phần này của câu khẩu hiệu từ tài liệu ngôn ngữ Go
một lần nữa: “Đừng giao tiếp bằng cách chia sẻ bộ nhớ.”

Việc giao tiếp bằng cách chia sẻ bộ nhớ sẽ trông như thế nào? Ngoài ra, tại sao
những người ủng hộ truyền thông điệp lại cảnh báo không nên sử dụng chia sẻ bộ nhớ?

Theo một cách nào đó, các kênh trong bất kỳ ngôn ngữ lập trình nào cũng tương tự như quyền sở hữu đơn lẻ,
bởi vì một khi bạn chuyển một giá trị xuống một kênh, bạn không nên sử dụng giá trị
đó nữa. Concurrency bộ nhớ chia sẻ giống như đa sở hữu (multiple ownership): nhiều luồng
có thể truy cập cùng một vị trí bộ nhớ tại cùng một thời điểm. Như bạn đã thấy trong Chương 15,
nơi các con trỏ thông minh (smart pointers) làm cho đa sở hữu trở nên khả thi, đa sở hữu có
thể làm tăng độ phức tạp vì những chủ sở hữu khác nhau này cần được quản lý. Hệ thống kiểu
và các quy tắc sở hữu của Rust hỗ trợ rất nhiều trong việc thực hiện quản lý này một cách chính xác. Để lấy
một ví dụ, hãy xem xét các mutex, một trong những nguyên ngữ concurrency phổ biến hơn
cho bộ nhớ chia sẻ.

### Sử dụng Mutex để cho phép truy cập dữ liệu từ một luồng tại một thời điểm

_Mutex_ là từ viết tắt của _mutual exclusion_ (loại trừ lẫn nhau), giống như một mutex chỉ cho phép
một luồng truy cập vào một số dữ liệu tại bất kỳ thời điểm nào. Để truy cập dữ liệu trong một
mutex, một luồng trước tiên phải báo hiệu rằng nó muốn truy cập bằng cách yêu cầu lấy được
_khóa_ (lock) của mutex. Khóa là một cấu trúc dữ liệu thuộc về mutex để
theo dõi xem ai hiện đang có quyền truy cập độc quyền vào dữ liệu. Do đó,
mutex được mô tả là _bảo vệ_ (guarding) dữ liệu mà nó nắm giữ thông qua hệ thống khóa.

Mutex nổi tiếng là khó sử dụng vì bạn phải
nhớ hai quy tắc:

1. Bạn phải cố gắng lấy khóa trước khi sử dụng dữ liệu.
2. Khi bạn hoàn thành với dữ liệu mà mutex bảo vệ, bạn phải mở khóa
   dữ liệu để các luồng khác có thể lấy khóa.

Để có một phép ẩn dụ thực tế cho một mutex, hãy tưởng tượng một cuộc thảo luận nhóm tại một
hội nghị chỉ có một micrô. Trước khi một tham luận viên có thể nói, họ phải
yêu cầu hoặc báo hiệu rằng họ muốn sử dụng micrô. Khi họ nhận được
micrô, họ có thể nói bao lâu tùy thích và sau đó đưa
micrô cho tham luận viên tiếp theo yêu cầu được nói. Nếu một tham luận viên quên
trao micrô khi họ nói xong, không ai khác có thể
nói được. Nếu việc quản lý micrô dùng chung gặp trục trặc, cuộc thảo luận sẽ không hoạt động
như kế hoạch!

Việc quản lý các mutex có thể cực kỳ khó khăn để thực hiện đúng, đó là lý do tại sao
rất nhiều người nhiệt tình với các kênh. Tuy nhiên, nhờ vào hệ thống kiểu
và các quy tắc sở hữu của Rust, bạn không thể thực hiện sai việc khóa và mở khóa.

#### API của `Mutex<T>`

Để làm ví dụ về cách sử dụng mutex, hãy bắt đầu bằng cách sử dụng một mutex trong một
ngữ cảnh đơn luồng, như được hiển thị trong Listing 16-12.

<Listing number="16-12" file-name="src/main.rs" caption="Khám phá API của `Mutex<T>` trong ngữ cảnh đơn luồng cho đơn giản">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Cũng giống như nhiều kiểu dữ liệu khác, chúng ta tạo một `Mutex<T>` bằng cách sử dụng hàm liên kết `new`.
Để truy cập dữ liệu bên trong mutex, chúng ta sử dụng phương thức `lock` để lấy
khóa. Lời gọi này sẽ chặn luồng hiện tại để nó không thể làm bất kỳ công việc nào cho đến khi
đến lượt chúng ta có khóa.

Lời gọi `lock` sẽ thất bại nếu một luồng khác đang giữ khóa bị panic. Trong
trường hợp đó, không ai có thể lấy được khóa nữa, vì vậy chúng ta đã chọn
`unwrap` và để luồng này panic nếu chúng ta rơi vào tình huống đó.

Sau khi chúng ta đã lấy được khóa, chúng ta có thể coi giá trị trả về, được đặt tên là `num` trong
trường hợp này, như một tham chiếu có thể thay đổi (mutable reference) đến dữ liệu bên trong. Hệ thống kiểu đảm bảo
rằng chúng ta lấy được khóa trước khi sử dụng giá trị trong `m`. Kiểu của `m` là
`Mutex<i32>`, không phải `i32`, vì vậy chúng ta _phải_ gọi `lock` để có thể sử dụng giá trị
`i32`. Chúng ta không thể quên; hệ thống kiểu sẽ không cho phép chúng ta truy cập `i32`
bên trong nếu không làm vậy.

Như bạn có thể nghi ngờ, `Mutex<T>` là một con trỏ thông minh. Chính xác hơn, lời gọi
đến `lock` _trả về_ một con trỏ thông minh được gọi là `MutexGuard`, được bọc trong một
`LockResult` mà chúng ta đã xử lý bằng lời gọi `unwrap`. Con trỏ thông minh `MutexGuard`
triển khai `Deref` để trỏ vào dữ liệu bên trong của chúng ta; con trỏ thông minh này cũng
có một triển khai `Drop` giúp tự động giải phóng khóa khi một
`MutexGuard` ra khỏi phạm vi, điều này xảy ra ở cuối phạm vi bên trong. Kết
quả là, chúng ta không có rủi ro quên giải phóng khóa và ngăn chặn mutex
khỏi việc được sử dụng bởi các luồng khác, bởi vì việc giải phóng khóa diễn ra
tự động.

Sau khi giải phóng khóa, chúng ta có thể in giá trị mutex và thấy rằng chúng ta đã có thể
thay đổi `i32` bên trong thành 6.

#### Chia sẻ một `Mutex<T>` giữa nhiều luồng

Bây giờ hãy thử chia sẻ một giá trị giữa nhiều luồng bằng cách sử dụng `Mutex<T>`. Chúng ta sẽ
tạo ra 10 luồng và cho mỗi luồng tăng một giá trị bộ đếm thêm 1, để
bộ đếm đi từ 0 đến 10. Ví dụ trong Listing 16-13 sẽ có một lỗi
biên dịch, và chúng ta sẽ sử dụng lỗi đó để tìm hiểu thêm về cách sử dụng `Mutex<T>` và cách
Rust giúp chúng ta sử dụng nó một cách chính xác.

<Listing number="16-13" file-name="src/main.rs" caption="Mười luồng, mỗi luồng tăng một bộ đếm được bảo vệ bởi một `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

Chúng ta tạo một biến `counter` để giữ một `i32` bên trong một `Mutex<T>`, như chúng ta đã làm
trong Listing 16-12. Tiếp theo, chúng ta tạo 10 luồng bằng cách lặp qua một phạm vi các
con số. Chúng ta sử dụng `thread::spawn` và đưa cho tất cả các luồng cùng một closure: một
closure di chuyển bộ đếm vào luồng, lấy một khóa trên `Mutex<T>` bằng cách
gọi phương thức `lock`, và sau đó cộng 1 vào giá trị trong mutex. Khi một
luồng chạy xong closure của nó, `num` sẽ ra khỏi phạm vi và giải phóng
khóa để một luồng khác có thể lấy nó.

Trong luồng chính, chúng ta thu thập tất cả các join handles. Sau đó, như chúng ta đã làm trong Listing
16-2, chúng ta gọi `join` trên mỗi handle để đảm bảo tất cả các luồng đều kết thúc. Tại
thời điểm đó, luồng chính sẽ lấy khóa và in kết quả của
chương trình này.

Chúng ta đã gợi ý rằng ví dụ này sẽ không biên dịch được. Bây giờ hãy tìm hiểu tại sao!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Thông báo lỗi cho biết giá trị `counter` đã bị di chuyển trong lần lặp
trước đó của vòng lặp. Rust đang bảo chúng ta rằng chúng ta không thể di chuyển quyền sở hữu
của khóa `counter` vào nhiều luồng. Hãy sửa lỗi biên dịch bằng
phương pháp đa sở hữu mà chúng ta đã thảo luận trong Chương 15.

#### Đa sở hữu với nhiều luồng

Trong Chương 15, chúng ta đã trao một giá trị cho nhiều chủ sở hữu bằng cách sử dụng con trỏ thông minh
`Rc<T>` để tạo một giá trị được đếm tham chiếu. Hãy làm tương tự ở đây và xem
điều gì xảy ra. Chúng ta sẽ bọc `Mutex<T>` trong `Rc<T>` trong Listing 16-14 và nhân bản
`Rc<T>` trước khi di chuyển quyền sở hữu sang luồng.

<Listing number="16-14" file-name="src/main.rs" caption="Cố gắng sử dụng `Rc<T>` để cho phép nhiều luồng sở hữu `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Một lần nữa, chúng ta biên dịch và nhận được… các lỗi khác nhau! Trình biên dịch đang dạy chúng ta
rất nhiều điều.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Wow, thông báo lỗi đó rất dài dòng! Đây là phần quan trọng cần tập trung vào:
`` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. Trình biên dịch cũng
đang cho chúng ta biết lý do tại sao: `` the trait `Send` is not implemented for
`Rc<Mutex<i32>>` ``. Chúng ta sẽ nói về `Send` trong phần tiếp theo: nó là một trong
những trait đảm bảo các kiểu dữ liệu chúng ta sử dụng với các luồng được thiết kế để sử dụng trong
các tình huống đồng thời.

Thật không may, `Rc<T>` không an toàn để chia sẻ giữa các luồng. Khi `Rc<T>`
quản lý số lượng tham chiếu, nó cộng thêm vào số lượng cho mỗi lần gọi `clone` và
trừ đi từ số lượng khi mỗi bản sao bị hủy. Nhưng nó không sử dụng bất kỳ
nguyên ngữ concurrency nào để đảm bảo rằng các thay đổi đối với số lượng không thể bị
ngắt quãng bởi một luồng khác. Điều này có thể dẫn đến số lượng sai—những lỗi tinh vi
mà đến lượt nó có thể dẫn đến rò rỉ bộ nhớ hoặc một giá trị bị hủy trước khi chúng ta dùng
xong nó. Những gì chúng ta cần là một kiểu dữ liệu giống hệt `Rc<T>` nhưng là một kiểu thực hiện
các thay đổi đối với số lượng tham chiếu theo cách an toàn cho luồng (thread-safe).

#### Đếm tham chiếu nguyên tử với `Arc<T>`

May mắn thay, `Arc<T>` _là_ một kiểu dữ liệu giống như `Rc<T>` và an toàn để sử dụng trong
các tình huống đồng thời. Chữ _a_ là viết tắt của _atomic_ (nguyên tử), nghĩa là nó là một kiểu
_đếm tham chiếu nguyên tử_ (atomically reference-counted). Atomics là một loại nguyên ngữ
concurrency bổ sung mà chúng ta sẽ không đề cập chi tiết ở đây: hãy xem tài liệu
thư viện tiêu chuẩn cho [`std::sync::atomic`][atomic]<!-- ignore --> để biết thêm
chi tiết. Tại thời điểm này, bạn chỉ cần biết rằng atomics hoạt động giống như các kiểu nguyên thủy
nhưng an toàn để chia sẻ qua các luồng.

Bạn có thể thắc mắc tại sao tất cả các kiểu nguyên thủy không phải là nguyên tử và tại sao các kiểu
thư viện tiêu chuẩn không được triển khai để sử dụng `Arc<T>` theo mặc định. Lý do là
an toàn luồng đi kèm với một hình phạt về hiệu suất mà bạn chỉ muốn trả khi
bạn thực sự cần. Nếu bạn chỉ thực hiện các thao tác trên các giá trị trong một
luồng duy nhất, mã của bạn có thể chạy nhanh hơn nếu nó không phải thực thi các
đảm bảo mà atomics cung cấp.

Hãy quay lại ví dụ của chúng ta: `Arc<T>` và `Rc<T>` có cùng một API, vì vậy chúng ta sửa
chương trình của mình bằng cách thay đổi dòng `use`, lời gọi `new`, và lời gọi
`clone`. Mã trong Listing 16-15 cuối cùng sẽ biên dịch và chạy.

<Listing number="16-15" file-name="src/main.rs" caption="Sử dụng một `Arc<T>` để bọc `Mutex<T>` để có thể chia sẻ quyền sở hữu qua nhiều luồng">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Mã này sẽ in ra kết quả sau:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Chúng ta đã làm được! Chúng ta đã đếm từ 0 đến 10, điều này có vẻ không mấy ấn tượng, nhưng nó
đã dạy chúng ta rất nhiều về `Mutex<T>` và an toàn luồng. Bạn cũng có thể sử dụng
cấu trúc của chương trình này để thực hiện các thao tác phức tạp hơn là chỉ tăng một
bộ đếm. Sử dụng chiến lược này, bạn có thể chia một phép tính thành các phần độc lập,
chia các phần đó qua các luồng, và sau đó sử dụng một `Mutex<T>` để cho mỗi
luồng cập nhật kết quả cuối cùng với phần của nó.

Lưu ý rằng nếu bạn đang thực hiện các thao tác số học đơn giản, có những kiểu đơn giản
hơn kiểu `Mutex<T>` được cung cấp bởi [module `std::sync::atomic` của
thư viện tiêu chuẩn][atomic]<!-- ignore -->. Các kiểu này cung cấp quyền truy cập nguyên tử,
đồng thời, an toàn vào các kiểu nguyên thủy. Chúng ta đã chọn sử dụng `Mutex<T>` với một kiểu
nguyên thủy cho ví dụ này để chúng ta có thể tập trung vào cách `Mutex<T>` hoạt động.

### Sự tương đồng giữa `RefCell<T>`/`Rc<T>` và `Mutex<T>`/`Arc<T>`

Bạn có thể đã nhận thấy rằng `counter` là bất biến nhưng chúng ta có thể lấy một tham chiếu
có thể thay đổi đến giá trị bên trong nó; điều này có nghĩa là `Mutex<T>` cung cấp tính đột biến
nội thất (interior mutability), giống như gia đình `Cell` vẫn làm. Theo cùng một cách chúng ta đã sử dụng `RefCell<T>` trong
Chương 15 để cho phép chúng ta thay đổi nội dung bên trong một `Rc<T>`, chúng ta sử dụng `Mutex<T>`
để thay đổi nội dung bên trong một `Arc<T>`.

Một chi tiết khác cần lưu ý là Rust không thể bảo vệ bạn khỏi tất cả các loại lỗi
logic khi bạn sử dụng `Mutex<T>`. Hãy nhớ lại từ Chương 15 rằng việc sử dụng `Rc<T>` đi kèm
với rủi ro tạo ra các chu kỳ tham chiếu, nơi hai giá trị `Rc<T>` tham chiếu đến
nhau, gây ra rò rỉ bộ nhớ. Tương tự, `Mutex<T>` đi kèm với rủi ro tạo ra
_bế tắc_ (deadlocks). Những điều này xảy ra khi một thao tác cần khóa hai tài nguyên
và hai luồng mỗi luồng đã lấy được một trong các khóa, khiến chúng chờ
đợi lẫn nhau mãi mãi. Nếu bạn quan tâm đến bế tắc, hãy thử tạo một chương trình Rust
có bế tắc; sau đó nghiên cứu các chiến lược giảm thiểu bế tắc cho
mutex trong bất kỳ ngôn ngữ nào và thử triển khai chúng trong Rust. Tài liệu
API thư viện tiêu chuẩn cho `Mutex<T>` và `MutexGuard` cung cấp thông tin hữu ích.

Chúng ta sẽ kết thúc chương này bằng cách nói về các trait `Send` và `Sync` và
cách chúng ta có thể sử dụng chúng với các kiểu dữ liệu tùy chỉnh.

{{#quiz ../quizzes/ch16-03-shared-state.toml}}

[atomic]: https://doc.rust-lang.org/std/sync/atomic/index.html
