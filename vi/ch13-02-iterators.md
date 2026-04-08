## Xử lý một chuỗi các mục với Iterator

Mẫu iterator (iterator pattern) cho phép bạn thực hiện một số tác vụ trên một chuỗi các mục
lần lượt. Một iterator chịu trách nhiệm về logic của việc lặp qua từng mục và
xác định khi nào chuỗi kết thúc. Khi bạn sử dụng iterator, bạn không
phải tự mình triển khai lại logic đó.

Trong Rust, các iterator là _lười biếng_ (lazy), nghĩa là chúng không có tác dụng gì cho đến khi bạn gọi
các phương thức tiêu thụ iterator để sử dụng hết nó. Ví dụ, đoạn mã trong
Danh sách 13-10 tạo ra một iterator trên các mục trong vector `v1` bằng cách gọi
phương thức `iter` được định nghĩa trên `Vec<T>`. Bản thân đoạn mã này không làm bất cứ điều gì
hữu ích.

<Listing number="13-10" file-name="src/main.rs" caption="Tạo một iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

Iterator được lưu trữ trong biến `v1_iter`. Một khi chúng ta đã tạo ra một
iterator, chúng ta có thể sử dụng nó theo nhiều cách khác nhau. Trong Danh sách 3-5 ở Chương 3, chúng ta
đã lặp qua một mảng bằng vòng lặp `for` để thực thi một số mã trên mỗi
mục của nó. Bên dưới lớp vỏ, điều này đã ngầm tạo ra và sau đó tiêu thụ một iterator,
nhưng cho đến tận bây giờ chúng ta mới xem xét kỹ cách thức hoạt động chính xác của nó.

Trong ví dụ ở Danh sách 13-11, chúng ta tách biệt việc tạo iterator khỏi
việc sử dụng iterator trong vòng lặp `for`. Khi vòng lặp `for` được gọi bằng cách sử dụng
iterator trong `v1_iter`, mỗi phần tử trong iterator được sử dụng trong một
lần lặp của vòng lặp, việc này in ra từng giá trị.

<Listing number="13-11" file-name="src/main.rs" caption="Sử dụng một iterator trong một vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Trong các ngôn ngữ không có iterator được cung cấp bởi thư viện chuẩn của chúng,
bạn có thể sẽ viết chức năng tương tự này bằng cách bắt đầu một biến tại chỉ số
0, sử dụng biến đó để truy cập vào vector nhằm lấy một giá trị, và
tăng giá trị biến trong một vòng lặp cho đến khi nó đạt đến tổng số
mục trong vector.

Iterator xử lý tất cả logic đó cho bạn, cắt giảm mã lặp lại mà bạn
có khả năng làm hỏng. Iterator mang lại cho bạn sự linh hoạt hơn để sử dụng cùng một
logic với nhiều loại chuỗi khác nhau, không chỉ các cấu trúc dữ liệu bạn có thể
truy cập bằng chỉ số, như vector. Hãy cùng xem xét cách các iterator thực hiện điều đó.

### Trait `Iterator` và phương thức `next`

Tất cả các iterator đều triển khai một trait tên là `Iterator` được định nghĩa trong
thư viện chuẩn. Định nghĩa của trait trông như thế này:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // các phương thức với các triển khai mặc định đã được lược bỏ
}
```

Lưu ý rằng định nghĩa này sử dụng một số cú pháp mới: `type Item` và `Self::Item`,
đang định nghĩa một _kiểu liên kết_ (associated type) với trait này. Chúng ta sẽ nói về
các kiểu liên kết một cách chuyên sâu trong Chương 20. Hiện tại, tất cả những gì bạn cần biết là
đoạn mã này nói rằng việc triển khai trait `Iterator` yêu cầu bạn cũng phải định nghĩa
một kiểu `Item`, và kiểu `Item` này được sử dụng trong kiểu trả về của phương thức `next`.
Nói cách khác, kiểu `Item` sẽ là kiểu được trả về từ iterator.

Trait `Iterator` chỉ yêu cầu những người triển khai định nghĩa một phương thức: phương thức
`next`, trả về từng mục của iterator tại một thời điểm, được bọc trong
`Some` và khi quá trình lặp kết thúc, trả về `None`.

Chúng ta có thể gọi trực tiếp phương thức `next` trên các iterator; Danh sách 13-12 minh họa
những giá trị nào được trả về từ các lần gọi lặp lại tới `next` trên iterator được tạo
từ vector.

<Listing number="13-12" file-name="src/lib.rs" caption="Gọi phương thức `next` trên một iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cần làm cho `v1_iter` có khả năng thay đổi (mutable): việc gọi phương thức `next` trên một
iterator làm thay đổi trạng thái nội bộ mà iterator sử dụng để theo dõi vị trí của nó
trong chuỗi. Nói cách khác, đoạn mã này _tiêu thụ_ (consume), hoặc sử dụng hết,
iterator. Mỗi lần gọi tới `next` sẽ "ăn" mất một mục từ iterator. Chúng ta không cần
phải làm cho `v1_iter` có khả năng thay đổi khi chúng ta sử dụng vòng lặp `for` vì vòng lặp đã lấy
quyền sở hữu của `v1_iter` và làm cho nó có khả năng thay đổi ở phía sau hậu trường.

Cũng lưu ý rằng các giá trị chúng ta nhận được từ các lần gọi tới `next` là các tham chiếu bất biến
đến các giá trị trong vector. Phương thức `iter` tạo ra một iterator trên
các tham chiếu bất biến. Nếu chúng ta muốn tạo một iterator lấy quyền sở hữu của `v1`
và trả về các giá trị được sở hữu, chúng ta có thể gọi `into_iter` thay vì
`iter`. Tương tự, nếu chúng ta muốn lặp trên các tham chiếu khả biến, chúng ta có thể gọi
`iter_mut` thay vì `iter`.

### Các phương thức tiêu thụ Iterator

Trait `Iterator` có một số phương thức khác nhau với các triển khai mặc định
được cung cấp bởi thư viện chuẩn; bạn có thể tìm hiểu về các phương thức này
bằng cách xem trong tài liệu API thư viện chuẩn cho trait `Iterator`.
Một số phương thức này gọi phương thức `next` trong định nghĩa của chúng, đó là
lý do tại sao bạn được yêu cầu triển khai phương thức `next` khi triển khai
trait `Iterator`.

Các phương thức gọi `next` được gọi là các _adapter tiêu thụ_ (consuming adapters) bởi vì việc gọi chúng
sẽ sử dụng hết iterator. Một ví dụ là phương thức `sum`, phương thức này lấy quyền sở hữu của
iterator và lặp qua các mục bằng cách gọi lặp lại `next`, do đó
tiêu thụ iterator. Khi nó lặp qua, nó cộng từng mục vào một tổng đang chạy
và trả về tổng đó khi quá trình lặp hoàn tất. Danh sách 13-13 có một
bản kiểm tra minh họa việc sử dụng phương thức `sum`.

<Listing number="13-13" file-name="src/lib.rs" caption="Gọi phương thức `sum` để lấy tổng của tất cả các mục trong iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

Chúng ta không được phép sử dụng `v1_iter` sau khi gọi `sum` vì `sum` lấy
quyền sở hữu của iterator mà chúng ta gọi nó.

### Các phương thức tạo ra các Iterator khác

_Adapter iterator_ (iterator adapters) là các phương thức được định nghĩa trên trait `Iterator` mà không
tiêu thụ iterator. Thay vào đó, chúng tạo ra các iterator khác nhau bằng cách thay đổi
một số khía cạnh của iterator ban đầu.

Danh sách 13-14 cho thấy một ví dụ về việc gọi phương thức adapter iterator `map`,
phương thức này nhận một closure để gọi trên mỗi mục khi các mục được lặp qua.
Phương thức `map` trả về một iterator mới tạo ra các mục đã được sửa đổi.
Closure ở đây tạo ra một iterator mới trong đó mỗi mục từ vector sẽ được
tăng thêm 1:

<Listing number="13-14" file-name="src/main.rs" caption="Gọi adapter iterator `map` để tạo một iterator mới">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

Tuy nhiên, đoạn mã này tạo ra một cảnh báo:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Đoạn mã trong Danh sách 13-14 không làm gì cả; closure chúng ta đã chỉ định
không bao giờ được gọi. Cảnh báo nhắc nhở chúng ta tại sao: các adapter iterator là lười biếng, và
chúng ta cần tiêu thụ iterator ở đây.

Để khắc phục cảnh báo này và tiêu thụ iterator, chúng ta sẽ sử dụng phương thức `collect`,
phương thức mà chúng ta đã sử dụng trong Chương 12 với `env::args` trong Danh sách 12-1. Phương thức này
tiêu thụ iterator và thu thập các giá trị kết quả vào một kiểu dữ liệu bộ sưu tập (collection).

Trong Danh sách 13-15, chúng ta thu thập các kết quả của việc lặp qua iterator được
trả về từ lời gọi tới `map` vào một vector. Vector này cuối cùng sẽ
chứa từng mục từ vector ban đầu, được tăng thêm 1.

<Listing number="13-15" file-name="src/main.rs" caption="Gọi phương thức `map` để tạo một iterator mới và sau đó gọi phương thức `collect` để tiêu thụ iterator mới và tạo một vector">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Bởi vì `map` nhận một closure, chúng ta có thể chỉ định bất kỳ thao tác nào chúng ta muốn thực hiện
trên mỗi mục. Đây là một ví dụ tuyệt vời về cách các closure cho phép bạn tùy chỉnh một số
hành vi trong khi tái sử dụng hành vi lặp mà trait `Iterator` cung cấp.

Bạn có thể chuỗi nhiều lời gọi tới các adapter iterator để thực hiện các hành động phức tạp theo
một cách dễ đọc. Nhưng vì tất cả các iterator đều lười biếng, bạn phải gọi một trong các
phương thức adapter tiêu thụ để nhận kết quả từ các lần gọi tới adapter iterator.

### Sử dụng các Closure có ghi lại môi trường của chúng

Nhiều adapter iterator nhận các closure làm đối số, và thông thường các closure
chúng ta sẽ chỉ định làm đối số cho các adapter iterator sẽ là các closure ghi lại
môi trường của chúng.

Đối với ví dụ này, chúng ta sẽ sử dụng phương thức `filter` nhận một closure.
Closure nhận một mục từ iterator và trả về một `bool`. Nếu closure
trả về `true`, giá trị sẽ được đưa vào quá trình lặp được tạo ra bởi
`filter`. Nếu closure trả về `false`, giá trị sẽ không được bao gồm.

Trong Danh sách 13-16, chúng ta sử dụng `filter` với một closure ghi lại biến `shoe_size`
từ môi trường của nó để lặp qua một bộ sưu tập các instance struct `Shoe`.
Nó sẽ chỉ trả về những đôi giày có kích cỡ được chỉ định.

<Listing number="13-16" file-name="src/lib.rs" caption="Sử dụng phương thức `filter` với một closure ghi lại `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

Hàm `shoes_in_size` lấy quyền sở hữu của một vector các đôi giày và một kích cỡ giày
làm các tham số. Nó trả về một vector chỉ chứa những đôi giày có kích cỡ
được chỉ định.

Trong phần thân của `shoes_in_size`, chúng ta gọi `into_iter` để tạo một iterator
lấy quyền sở hữu của vector. Sau đó chúng ta gọi `filter` để điều chỉnh iterator đó
thành một iterator mới chỉ chứa các phần tử mà closure trả về `true`.

Closure ghi lại tham số `shoe_size` từ môi trường và
so sánh giá trị đó với kích cỡ của mỗi đôi giày, chỉ giữ lại những đôi giày có kích cỡ
được chỉ định. Cuối cùng, việc gọi `collect` sẽ tập hợp các giá trị được trả về bởi
iterator đã được điều chỉnh vào một vector được trả về bởi hàm.

Bản kiểm tra cho thấy khi chúng ta gọi `shoes_in_size`, chúng ta chỉ nhận lại những đôi giày
có cùng kích cỡ với giá trị chúng ta đã chỉ định.

{{#quiz ../quizzes/ch13-02-iterators.toml}}
