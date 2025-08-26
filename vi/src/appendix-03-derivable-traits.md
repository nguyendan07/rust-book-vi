## Appendix C: Derivable Traits

Nhiều chỗ trong cuốn sách, chúng ta đã thảo luận về thuộc tính `derive`, mà bạn
có thể áp dụng cho định nghĩa struct hoặc enum. Thuộc tính `derive` sinh
mã sẽ triển khai một trait với triển khai mặc định cho kiểu mà bạn đã chú thích
bằng cú pháp `derive`.

Trong phụ lục này, chúng tôi cung cấp một tham chiếu của tất cả các trait trong
thư viện chuẩn mà bạn có thể sử dụng với `derive`. Mỗi phần bao gồm:

- Những toán tử và phương thức mà việc derive trait này sẽ kích hoạt
- Việc triển khai trait do `derive` cung cấp làm gì
- Việc triển khai trait biểu thị điều gì về kiểu dữ liệu
- Các điều kiện cho phép hoặc không cho phép bạn triển khai trait
- Ví dụ về các thao tác yêu cầu trait

Nếu bạn muốn hành vi khác với hành vi do thuộc tính `derive` cung cấp,
hãy tham khảo [standard library documentation](../std/index.html)<!-- ignore -->
cho mỗi trait để biết chi tiết về cách tự triển khai chúng.

Các trait được liệt kê ở đây là những trait duy nhất do thư viện chuẩn định nghĩa
mà bạn có thể triển khai trên các kiểu của mình bằng `derive`. Các trait khác
được định nghĩa trong thư viện chuẩn không có hành vi mặc định hợp lý, vì vậy
bạn phải tự triển khai chúng theo cách phù hợp với mục đích bạn muốn đạt được.

Một ví dụ về trait không thể derive là `Display`, trait này xử lý
định dạng cho người dùng cuối. Bạn luôn nên cân nhắc cách thích hợp để
hiển thị một kiểu dữ liệu cho người dùng cuối. Những phần nào của kiểu dữ liệu
một người dùng nên được phép thấy? Những phần nào là có liên quan với họ?
Định dạng dữ liệu nào sẽ phù hợp nhất? Trình biên dịch Rust không có hiểu biết này, vì vậy
nó không thể cung cấp hành vi mặc định phù hợp cho bạn.

Danh sách các trait có thể derive được cung cấp trong phụ lục này không đầy đủ:
thư viện có thể hiện thực `derive` cho các trait của riêng họ, làm cho danh sách
các trait bạn có thể dùng `derive` trở nên mở rộng. Việc hiện thực `derive`
liên quan đến việc sử dụng procedural macro, điều này được đề cập trong phần
[“Macros”][macros]<!-- ignore --> của Chương 20.

### `Debug` for Programmer Output

Trait `Debug` cho phép định dạng debug trong các chuỗi định dạng, mà bạn
chỉ định bằng cách thêm `:?` bên trong các placeholder `{}`.

Trait `Debug` cho phép bạn in các thể hiện của một kiểu để phục vụ việc gỡ lỗi,
để bạn và các lập trình viên khác sử dụng kiểu của bạn có thể kiểm tra một thể hiện
tại một điểm nhất định trong quá trình thực thi chương trình.

Trait `Debug` được yêu cầu, ví dụ, khi dùng macro `assert_eq!`. Macro này in giá trị
của các thể hiện được truyền làm đối số nếu khẳng định về sự bằng nhau thất bại để
lập trình viên có thể thấy lý do vì sao hai thể hiện không bằng nhau.

### `PartialEq` and `Eq` for Equality Comparisons

Trait `PartialEq` cho phép bạn so sánh các thể hiện của một kiểu để kiểm tra
sự bằng nhau và kích hoạt việc sử dụng các toán tử `==` và `!=`.

Khi derive `PartialEq` sẽ triển khai phương thức `eq`. Khi `PartialEq` được derive trên
các struct, hai thể hiện bằng nhau chỉ khi tất cả các trường đều bằng nhau, và hai
thể hiện sẽ không bằng nhau nếu bất kỳ trường nào không bằng nhau. Khi derive trên enum,
mỗi biến thể (variant) chỉ bằng chính nó và không bằng các biến thể khác.

Trait `PartialEq` được yêu cầu, ví dụ, khi sử dụng macro `assert_eq!`, macro này cần
có khả năng so sánh hai thể hiện của một kiểu để kiểm tra sự bằng nhau.

Trait `Eq` không có phương thức nào. Mục đích của nó là báo hiệu rằng với mọi giá trị
của kiểu được chú thích, giá trị đó bằng chính nó. Trait `Eq` chỉ có thể áp dụng cho
các kiểu cũng triển khai `PartialEq`, mặc dù không phải tất cả các kiểu triển khai
`PartialEq` đều có thể triển khai `Eq`. Một ví dụ là các kiểu số dấu phẩy động: triển
khai cho số dấu phẩy quy định rằng hai thể hiện của giá trị not-a-number (`NaN`) không
bằng nhau với nhau.

Một ví dụ khi `Eq` được yêu cầu là cho khóa trong `HashMap<K, V>` để `HashMap<K, V>`
có thể xác định hai khóa có giống nhau hay không.

### `PartialOrd` and `Ord` for Ordering Comparisons

Trait `PartialOrd` cho phép bạn so sánh các thể hiện của một kiểu để phục vụ việc sắp xếp.
Một kiểu triển khai `PartialOrd` có thể được dùng với các toán tử `<`, `>`, `<=`, và `>=`.
Bạn chỉ có thể áp dụng trait `PartialOrd` cho các kiểu cũng triển khai `PartialEq`.

Khi derive `PartialOrd` sẽ triển khai phương thức `partial_cmp`, trả về một
`Option<Ordering>` sẽ là `None` khi hai giá trị đưa vào không tạo ra thứ tự. Một ví dụ
về giá trị không tạo ra thứ tự, mặc dù hầu hết các giá trị của kiểu đó có thể được so sánh,
là giá trị not-a-number (`NaN`) của số dấu phẩy. Gọi `partial_cmp` với bất kỳ số dấu phẩy nào
và giá trị `NaN` sẽ trả về `None`.

Khi derive trên struct, `PartialOrd` so sánh hai thể hiện bằng cách so sánh giá trị trong
mỗi trường theo thứ tự trường xuất hiện trong định nghĩa struct. Khi derive trên enum,
các biến thể enum khai báo trước được coi là nhỏ hơn các biến thể khai báo sau.

Trait `PartialOrd` được yêu cầu, ví dụ, cho phương thức `gen_range` từ crate `rand`
dùng để sinh một giá trị ngẫu nhiên trong khoảng được chỉ định bởi một biểu thức range.

Trait `Ord` cho phép bạn biết rằng đối với bất kỳ hai giá trị nào của kiểu được chú thích,
luôn tồn tại một thứ tự hợp lệ. Trait `Ord` triển khai phương thức `cmp`, trả về một
`Ordering` thay vì `Option<Ordering>` bởi vì một thứ tự hợp lệ luôn có thể xác định được.
Bạn chỉ có thể áp dụng trait `Ord` cho các kiểu cũng triển khai `PartialOrd` và `Eq`
(và `Eq` lại yêu cầu `PartialEq`). Khi derive trên struct và enum, `cmp` hoạt động
theo cùng cách như triển khai derive cho `partial_cmp` với `PartialOrd`.

Một ví dụ khi `Ord` được yêu cầu là khi lưu trữ giá trị trong `BTreeSet<T>`,
một cấu trúc dữ liệu lưu trữ dữ liệu dựa trên thứ tự sắp xếp của các giá trị.

### `Clone` and `Copy` for Duplicating Values

Trait `Clone` cho phép bạn tạo một bản sao sâu (deep copy) một cách rõ ràng của một giá trị, và
quá trình nhân bản có thể liên quan đến việc chạy mã tùy ý và sao chép dữ liệu trên heap.
Xem [Variables and Data Interacting with
Clone”][variables-and-data-interacting-with-clone]<!-- ignore --> trong Chương 4
để biết thêm thông tin về `Clone`.

Khi derive `Clone` sẽ triển khai phương thức `clone`, khi được triển khai cho toàn bộ kiểu,
sẽ gọi `clone` trên từng phần của kiểu. Điều này có nghĩa là tất cả các trường hoặc giá trị
trong kiểu cũng phải triển khai `Clone` để có thể derive `Clone`.

Một ví dụ khi `Clone` được yêu cầu là khi gọi phương thức `to_vec` trên một
slice. Slice không sở hữu các thể hiện mà nó chứa, nhưng vector trả về từ `to_vec`
sẽ cần sở hữu các thể hiện đó, vì vậy `to_vec` gọi `clone` trên mỗi phần tử. Do đó,
kiểu được lưu trong slice phải triển khai `Clone`.

Trait `Copy` cho phép bạn nhân bản một giá trị chỉ bằng cách sao chép các bit lưu trên
stack; không có mã tùy ý nào cần thực thi. Xem [“Stack-Only Data:
Copy”][stack-only-data-copy]<!-- ignore --> trong Chương 4 để biết thêm thông tin về
`Copy`.

Trait `Copy` không định nghĩa bất kỳ phương thức nào để ngăn lập trình viên
ghi đè những phương thức đó và làm vi phạm giả định rằng không có mã tùy ý nào đang
chạy. Nhờ vậy, tất cả lập trình viên có thể giả định rằng việc sao chép một giá trị
sẽ rất nhanh.

Bạn có thể derive `Copy` trên bất kỳ kiểu nào mà tất cả các phần của nó đều triển khai `Copy`.
Một kiểu triển khai `Copy` cũng phải triển khai `Clone`, bởi vì kiểu triển khai `Copy`
có một triển khai `Clone` đơn giản thực hiện cùng công việc như `Copy`.

Trait `Copy` hiếm khi là bắt buộc; các kiểu triển khai `Copy` có những tối ưu
khả dụng, nghĩa là bạn không phải gọi `clone`, làm cho mã ngắn gọn hơn.

Mọi điều có thể làm với `Copy` bạn cũng có thể đạt được bằng `Clone`, nhưng mã
có thể chậm hơn hoặc phải dùng `clone` ở một số chỗ.

### `Hash` for Mapping a Value to a Value of Fixed Size

Trait `Hash` cho phép bạn lấy một thể hiện của một kiểu có kích thước tuỳ ý và
ánh xạ thể hiện đó thành một giá trị có kích thước cố định bằng một hàm băm. Khi derive
`Hash` sẽ triển khai phương thức `hash`. Việc triển khai `hash` do derive tạo kết hợp
kết quả của việc gọi `hash` trên từng phần của kiểu, nghĩa là tất cả các trường hoặc
giá trị cũng phải triển khai `Hash` để có thể derive `Hash`.

Một ví dụ khi `Hash` được yêu cầu là khi lưu khóa trong `HashMap<K, V>`
để lưu trữ dữ liệu một cách hiệu quả.

### `Default` for Default Values

Trait `Default` cho phép bạn tạo một giá trị mặc định cho một kiểu. Khi derive
`Default` sẽ triển khai hàm `default`. Việc triển khai `default` do derive tạo
sẽ gọi `default` trên từng phần của kiểu, nghĩa là tất cả các trường hoặc giá trị
trong kiểu cũng phải triển khai `Default` để có thể derive `Default`.

Hàm `Default::default` thường được sử dụng kết hợp với cú pháp cập nhật struct được
thảo luận trong [“Creating Instances from Other Instances with Struct
Update
Syntax”][creating-instances-from-other-instances-with-struct-update-syntax]<!--
ignore --> trong Chương 5. Bạn có thể tùy chỉnh một vài trường của struct và sau đó
đặt và sử dụng giá trị mặc định cho phần còn lại của các trường bằng cách dùng
`..Default::default()`.

Trait `Default` được yêu cầu khi bạn dùng phương thức `unwrap_or_default` trên
các thể hiện `Option<T>`, ví dụ. Nếu `Option<T>` là `None`, phương thức
`unwrap_or_default` sẽ trả về kết quả của `Default::default` cho kiểu
`T` được lưu trong `Option<T>`.

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[macros]: ch20-05-macros.html#macros
