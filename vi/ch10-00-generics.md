# Kiểu Generic, Trait, và Lifetime

Mọi ngôn ngữ lập trình đều có các công cụ để xử lý hiệu quả sự trùng lặp
các khái niệm. Trong Rust, một công cụ như vậy là _generics_: các vật thay thế trừu tượng cho
các kiểu cụ thể hoặc các thuộc tính khác. Chúng ta có thể diễn đạt hành vi của generics hoặc
cách chúng liên quan đến các generics khác mà không cần biết cái gì sẽ ở vị trí của chúng
khi biên dịch và chạy mã nguồn.

Các hàm có thể nhận các tham số của một kiểu generic nào đó, thay vì một kiểu cụ thể
như `i32` hoặc `String`, giống như cách chúng nhận các tham số với những giá trị
chưa biết để chạy cùng một mã nguồn trên nhiều giá trị cụ thể. Thực tế, chúng ta đã
sử dụng generics trong Chương 6 với `Option<T>`, trong Chương 8 với `Vec<T>` và
`HashMap<K, V>`, và trong Chương 9 với `Result<T, E>`. Trong chương này, bạn sẽ
khám phá cách định nghĩa các kiểu, hàm và phương thức của riêng bạn với generics!

Đầu tiên, chúng ta sẽ xem xét cách trích xuất một hàm để giảm bớt sự trùng lặp mã nguồn. Sau đó,
chúng ta sẽ sử dụng cùng kỹ thuật đó để tạo ra một hàm generic từ hai hàm chỉ
khác nhau về kiểu dữ liệu của các tham số. Chúng ta cũng sẽ giải thích cách sử dụng
các kiểu generic trong định nghĩa struct và enum.

Tiếp theo, bạn sẽ học cách sử dụng _traits_ (đặc tính) để định nghĩa hành vi theo cách generic.
Bạn có thể kết hợp traits với các kiểu generic để ràng buộc một kiểu generic chỉ chấp nhận
những kiểu có một hành vi cụ thể, thay vì bất kỳ kiểu nào.

Cuối cùng, chúng ta sẽ thảo luận về _lifetimes_ (vòng đời): một loại generics cung cấp cho
trình biên dịch thông tin về cách các tham chiếu liên quan đến nhau. Lifetimes cho
phép chúng ta cung cấp cho trình biên dịch đủ thông tin về các giá trị được mượn để nó có thể
đảm bảo các tham chiếu sẽ hợp lệ trong nhiều tình huống hơn là khi không có sự trợ giúp của chúng ta.

## Loại bỏ sự trùng lặp bằng cách trích xuất một hàm

Generics cho phép chúng ta thay thế các kiểu cụ thể bằng một vật thay thế đại diện cho
nhiều kiểu để loại bỏ sự trùng lặp mã nguồn. Trước khi đi sâu vào cú pháp generics,
hãy cùng xem cách loại bỏ sự trùng lặp theo cách không liên quan đến các kiểu generic bằng cách
trích xuất một hàm thay thế các giá trị cụ thể bằng một vật thay thế đại diện cho
nhiều giá trị. Sau đó, chúng ta sẽ áp dụng cùng kỹ thuật đó để trích xuất một hàm generic!
Bằng cách xem xét cách nhận biết mã nguồn bị trùng lặp mà bạn có thể trích xuất vào một hàm,
bạn sẽ bắt đầu nhận ra mã nguồn trùng lặp có thể sử dụng generics.

Chúng ta sẽ bắt đầu với chương trình ngắn trong Liệt kê 10-1 tìm số
lớn nhất trong một danh sách.

<Listing number="10-1" file-name="src/main.rs" caption="Tìm số lớn nhất trong một danh sách các số">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Chúng ta lưu trữ một danh sách các số nguyên trong biến `number_list` và đặt một tham chiếu
đến số đầu tiên trong danh sách vào một biến tên là `largest`. Sau đó, chúng ta lặp
qua tất cả các số trong danh sách, và nếu số hiện tại lớn hơn số được lưu trữ
trong `largest`, chúng ta thay thế tham chiếu trong biến đó.
Tuy nhiên, nếu số hiện tại nhỏ hơn hoặc bằng số lớn nhất đã thấy cho
đến nay, biến đó không thay đổi, và mã nguồn chuyển sang số tiếp theo
trong danh sách. Sau khi xem xét tất cả các số trong danh sách, `largest` sẽ
tham chiếu đến số lớn nhất, trong trường hợp này là 100.

Bây giờ chúng ta được giao nhiệm vụ tìm số lớn nhất trong hai danh sách số
khác nhau. Để làm như vậy, chúng ta có thể chọn sao chép mã nguồn trong Liệt kê 10-1 và sử dụng
cùng một logic tại hai nơi khác nhau trong chương trình, như được hiển thị trong Liệt kê 10-2.

<Listing number="10-2" file-name="src/main.rs" caption="Mã nguồn để tìm số lớn nhất trong *hai* danh sách các số">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Mặc dù mã nguồn này hoạt động, nhưng việc sao chép mã nguồn rất tẻ nhạt và dễ mắc lỗi. Chúng ta cũng
phải nhớ cập nhật mã nguồn ở nhiều nơi khi muốn thay đổi
nó.

Để loại bỏ sự trùng lặp này, chúng ta sẽ tạo ra một sự trừu tượng hóa bằng cách định nghĩa một
hàm hoạt động trên bất kỳ danh sách số nguyên nào được truyền vào như một tham số.
Giải pháp này giúp mã nguồn của chúng ta rõ ràng hơn và cho phép chúng ta diễn đạt khái niệm
tìm số lớn nhất trong một danh sách một cách trừu tượng.

Trong Liệt kê 10-3, chúng ta trích xuất mã nguồn tìm số lớn nhất vào một
hàm tên là `largest`. Sau đó, chúng ta gọi hàm để tìm số lớn nhất
trong hai danh sách từ Liệt kê 10-2. Chúng ta cũng có thể sử dụng hàm này cho bất kỳ
danh sách các giá trị `i32` nào khác mà chúng ta có thể có trong tương lai.

<Listing number="10-3" file-name="src/main.rs" caption="Mã nguồn đã được trừu tượng hóa để tìm số lớn nhất trong hai danh sách">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

Hàm `largest` có một tham số gọi là `list`, đại diện cho bất kỳ
slice cụ thể nào của các giá trị `i32` mà chúng ta có thể truyền vào hàm. Kết quả là,
khi chúng ta gọi hàm, mã nguồn sẽ chạy trên các giá trị cụ thể mà chúng ta truyền
vào.

Tóm lại, đây là các bước chúng ta đã thực hiện để thay đổi mã nguồn từ Liệt kê 10-2 sang
Liệt kê 10-3:

1. Xác định mã nguồn trùng lặp.
1. Trích xuất mã nguồn trùng lặp vào thân hàm, và chỉ định các
   đầu vào và giá trị trả về của mã nguồn đó trong chữ ký hàm.
1. Cập nhật hai phiên bản mã nguồn bị trùng lặp để gọi hàm thay thế.

Tiếp theo, chúng ta sẽ sử dụng chính các bước này với generics để giảm bớt sự trùng lặp mã nguồn. Theo
cùng một cách mà thân hàm có thể hoạt động trên một `list` trừu tượng thay vì
các giá trị cụ thể, generics cho phép mã nguồn hoạt động trên các kiểu trừu tượng.

Ví dụ, giả sử chúng ta có hai hàm: một hàm tìm mục lớn nhất trong
một slice các giá trị `i32` và một hàm tìm mục lớn nhất trong một slice các giá trị
`char`. Làm thế nào để chúng ta loại bỏ sự trùng lặp đó? Hãy cùng tìm hiểu!
