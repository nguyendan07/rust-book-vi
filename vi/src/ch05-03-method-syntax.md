## Cú pháp Method

_Method_ (phương thức) tương tự như _function_ (hàm): chúng ta khai báo chúng bằng từ khóa `fn` và một cái tên, chúng có thể có tham số và giá trị trả về, và chúng chứa một số mã lệnh được chạy khi method được gọi từ một nơi khác. Không giống như function, method được định nghĩa trong ngữ cảnh của một struct (hoặc một enum hay một trait object, chúng ta sẽ đề cập trong [Chương 6][enums]<!-- ignore --> và [Chương 18][trait-objects]<!-- ignore -->), và tham số đầu tiên của chúng luôn là `self`, đại diện cho thực thể của struct mà method đang được gọi trên đó.

### Định nghĩa Method

Hãy thay đổi hàm `area` có một thực thể `Rectangle` làm tham số và thay vào đó tạo một method `area` được định nghĩa trên struct `Rectangle`, như trong Listing 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="Định nghĩa một method `area` trên struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Để định nghĩa hàm trong ngữ cảnh của `Rectangle`, chúng ta bắt đầu một khối `impl` (viết tắt của implementation - triển khai) cho `Rectangle`. Mọi thứ bên trong khối `impl` này sẽ được liên kết với kiểu `Rectangle`. Sau đó, chúng ta di chuyển hàm `area` vào bên trong cặp ngoặc nhọn của `impl` và thay đổi tham số đầu tiên (và trong trường hợp này là duy nhất) thành `self` trong chữ ký và ở mọi nơi trong thân hàm. Trong `main`, nơi chúng ta đã gọi hàm `area` và truyền `rect1` làm đối số, thay vào đó chúng ta có thể sử dụng _cú pháp method_ để gọi method `area` trên thực thể `Rectangle` của mình. Cú pháp method đi sau một thực thể: chúng ta thêm một dấu chấm theo sau là tên method, cặp ngoặc đơn và bất kỳ đối số nào.

Trong chữ ký của `area`, chúng ta sử dụng `&self` thay vì `rectangle: &Rectangle`. `&self` thực ra là viết tắt của `self: &Self`. Bên trong một khối `impl`, kiểu `Self` là một bí danh cho kiểu mà khối `impl` đó dành cho. Các method phải có một tham số tên là `self` của kiểu `Self` làm tham số đầu tiên, vì vậy Rust cho phép bạn viết tắt điều này chỉ bằng tên `self` ở vị trí tham số đầu tiên. Lưu ý rằng chúng ta vẫn cần sử dụng `&` trước `self` viết tắt để chỉ ra rằng method này mượn thực thể `Self`, giống như chúng ta đã làm trong `rectangle: &Rectangle`. Các method có thể lấy quyền sở hữu của `self`, mượn `self` một cách bất biến (immutable) như chúng ta đã làm ở đây, hoặc mượn `self` một cách khả biến (mutable), giống như bất kỳ tham số nào khác.

Chúng ta đã chọn `&self` ở đây vì cùng lý do chúng ta đã sử dụng `&Rectangle` trong phiên bản hàm: chúng ta không muốn lấy quyền sở hữu, và chúng ta chỉ muốn đọc dữ liệu trong struct, không phải ghi vào nó. Nếu chúng ta muốn thay đổi thực thể mà chúng ta đã gọi method trên đó như một phần của những gì method làm, chúng ta sẽ sử dụng `&mut self` làm tham số đầu tiên. Việc có một method lấy quyền sở hữu của thực thể bằng cách chỉ sử dụng `self` làm tham số đầu tiên là rất hiếm; kỹ thuật này thường được sử dụng khi method biến đổi `self` thành một thứ khác và bạn muốn ngăn người gọi sử dụng thực thể ban đầu sau khi biến đổi.

Lý do chính để sử dụng method thay vì function, ngoài việc cung cấp cú pháp method và không phải lặp lại kiểu của `self` trong chữ ký của mỗi method, là để tổ chức. Chúng ta đã đặt tất cả những thứ chúng ta có thể làm với một thực thể của một kiểu vào một khối `impl` thay vì khiến những người dùng tương lai của mã nguồn của chúng ta phải tìm kiếm các khả năng của `Rectangle` ở nhiều nơi khác nhau trong thư viện mà chúng ta cung cấp.

Lưu ý rằng chúng ta có thể chọn đặt tên cho một method trùng với tên một trong các trường của struct. Ví dụ, chúng ta có thể định nghĩa một method trên `Rectangle` cũng có tên là `width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta chọn làm cho method `width` trả về `true` nếu giá trị trong trường `width` của thực thể lớn hơn `0` và `false` nếu giá trị là `0`: chúng ta có thể sử dụng một trường bên trong một method cùng tên cho bất kỳ mục đích nào. Trong `main`, khi chúng ta theo sau `rect1.width` bằng cặp ngoặc đơn, Rust biết chúng ta muốn nói đến method `width`. Khi chúng ta không sử dụng cặp ngoặc đơn, Rust biết chúng ta muốn nói đến trường `width`.

Thường thì, nhưng không phải lúc nào cũng vậy, khi chúng ta đặt tên cho một method trùng với tên một trường, chúng ta muốn nó chỉ trả về giá trị trong trường đó và không làm gì khác. Các method như thế này được gọi là _getter_, và Rust không tự động triển khai chúng cho các trường của struct như một số ngôn ngữ khác. Getter hữu ích vì bạn có thể đặt trường ở chế độ riêng tư (private) nhưng method ở chế độ công khai (public), và do đó cho phép truy cập chỉ đọc vào trường đó như một phần của API công khai của kiểu. Chúng ta sẽ thảo luận về công khai và riêng tư là gì và cách chỉ định một trường hoặc method là công khai hay riêng tư trong [Chương 7][public]<!-- ignore -->.

> ### Toán tử `->` ở đâu?
>
> Trong C và C++, hai toán tử khác nhau được sử dụng để gọi method: bạn sử dụng `.` nếu bạn đang gọi một method trên đối tượng trực tiếp và `->` nếu bạn đang gọi method trên một con trỏ tới đối tượng và cần giải tham chiếu con trỏ trước. Nói cách khác, nếu `object` là một con trỏ, `object->something()` tương tự như `(*object).something()`.
>
> Rust không có toán tử tương đương với `->`; thay vào đó, Rust có một tính năng gọi là _tham chiếu và giải tham chiếu tự động_ (automatic referencing and dereferencing). Gọi method là một trong số ít nơi trong Rust có hành vi này.
>
> Đây là cách nó hoạt động: khi bạn gọi một method với `object.something()`, Rust tự động thêm vào `&`, `&mut`, hoặc `*` để `object` khớp với chữ ký của method. Nói cách khác, những điều sau đây là như nhau:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Cách đầu tiên trông gọn gàng hơn nhiều. Hành vi tham chiếu tự động này hoạt động vì các method có một "người nhận" (receiver) rõ ràng—kiểu của `self`. Dựa vào người nhận và tên của một method, Rust có thể xác định chắc chắn liệu method đó đang đọc (`&self`), thay đổi (`&mut self`), hay tiêu thụ (`self`). Việc Rust làm cho việc mượn trở nên ngầm định đối với người nhận method là một phần quan trọng giúp cho việc quản lý quyền sở hữu trở nên thuận tiện trong thực tế.

### Method với nhiều tham số hơn

Hãy thực hành sử dụng method bằng cách triển khai một method thứ hai trên struct `Rectangle`. Lần này chúng ta muốn một thực thể của `Rectangle` nhận một thực thể khác của `Rectangle` và trả về `true` nếu `Rectangle` thứ hai có thể nằm hoàn toàn bên trong `self` (`Rectangle` đầu tiên); nếu không, nó sẽ trả về `false`. Tức là, một khi chúng ta đã định nghĩa method `can_hold`, chúng ta muốn có thể viết chương trình như trong Listing 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="Sử dụng method `can_hold` chưa được viết">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

Kết quả mong đợi sẽ như sau vì cả hai chiều của `rect2` đều nhỏ hơn các chiều của `rect1`, nhưng `rect3` lại rộng hơn `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Chúng ta biết rằng chúng ta muốn định nghĩa một method, vì vậy nó sẽ nằm trong khối `impl Rectangle`. Tên method sẽ là `can_hold`, và nó sẽ nhận một tham chiếu mượn bất biến của một `Rectangle` khác làm tham số. Chúng ta có thể biết kiểu của tham số bằng cách nhìn vào mã lệnh gọi method: `rect1.can_hold(&rect2)` truyền vào `&rect2`, là một tham chiếu mượn bất biến đến `rect2`, một thực thể của `Rectangle`. Điều này hợp lý vì chúng ta chỉ cần đọc `rect2` (thay vì ghi, điều đó có nghĩa là chúng ta cần một tham chiếu mượn khả biến), và chúng ta muốn `main` giữ lại quyền sở hữu của `rect2` để chúng ta có thể sử dụng lại nó sau khi gọi method `can_hold`. Giá trị trả về của `can_hold` sẽ là một Boolean, và phần triển khai sẽ kiểm tra xem chiều rộng và chiều cao của `self` có lớn hơn chiều rộng và chiều cao của `Rectangle` kia hay không. Hãy thêm method `can_hold` mới vào khối `impl` từ Listing 5-13, được hiển thị trong Listing 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="Triển khai method `can_hold` trên `Rectangle` nhận một thực thể `Rectangle` khác làm tham số">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Khi chúng ta chạy mã này với hàm `main` trong Listing 5-14, chúng ta sẽ nhận được kết quả mong muốn. Các method có thể nhận nhiều tham số mà chúng ta thêm vào chữ ký sau tham số `self`, và các tham số đó hoạt động giống như các tham số trong function.

### Associated Functions (Hàm liên kết)

Tất cả các hàm được định nghĩa trong một khối `impl` được gọi là _associated functions_ (hàm liên kết) vì chúng được liên kết với kiểu được đặt tên sau `impl`. Chúng ta có thể định nghĩa các hàm liên kết không có `self` làm tham số đầu tiên (và do đó không phải là method) vì chúng không cần một thực thể của kiểu để làm việc. Chúng ta đã sử dụng một hàm như thế này: hàm `String::from` được định nghĩa trên kiểu `String`.

Các hàm liên kết không phải là method thường được sử dụng cho các hàm khởi tạo (constructor) sẽ trả về một thực thể mới của struct. Chúng thường được gọi là `new`, nhưng `new` không phải là một tên đặc biệt và không được tích hợp sẵn trong ngôn ngữ. Ví dụ, chúng ta có thể chọn cung cấp một hàm liên kết tên là `square` sẽ có một tham số kích thước và sử dụng nó cho cả chiều rộng và chiều cao, do đó giúp việc tạo một `Rectangle` hình vuông dễ dàng hơn thay vì phải chỉ định cùng một giá trị hai lần:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Từ khóa `Self` trong kiểu trả về và trong thân hàm là bí danh cho kiểu xuất hiện sau từ khóa `impl`, trong trường hợp này là `Rectangle`.

Để gọi hàm liên kết này, chúng ta sử dụng cú pháp `::` với tên struct; `let sq = Rectangle::square(3);` là một ví dụ. Hàm này được định không gian tên bởi struct: cú pháp `::` được sử dụng cho cả các hàm liên kết và các không gian tên được tạo bởi các module. Chúng ta sẽ thảo luận về module trong [Chương 7][modules]<!-- ignore -->.

### Nhiều khối `impl`

Mỗi struct được phép có nhiều khối `impl`. Ví dụ, Listing 5-15 tương đương với mã được hiển thị trong Listing 5-16, trong đó mỗi method nằm trong khối `impl` riêng của nó.

<Listing number="5-16" caption="Viết lại Listing 5-15 bằng cách sử dụng nhiều khối `impl`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Không có lý do gì để tách các method này thành nhiều khối `impl` ở đây, nhưng đây là cú pháp hợp lệ. Chúng ta sẽ thấy một trường hợp mà nhiều khối `impl` hữu ích trong Chương 10, khi chúng ta thảo luận về các kiểu generic và trait.

## Tóm tắt

Struct cho phép bạn tạo các kiểu tùy chỉnh có ý nghĩa cho lĩnh vực của bạn. Bằng cách sử dụng struct, bạn có thể giữ các mẩu dữ liệu liên quan kết nối với nhau và đặt tên cho từng mẩu để làm cho mã của bạn rõ ràng. Trong các khối `impl`, bạn có thể định nghĩa các hàm được liên kết với kiểu của bạn, và method là một loại hàm liên kết cho phép bạn chỉ định hành vi mà các thực thể của struct của bạn có.

Nhưng struct không phải là cách duy nhất bạn có thể tạo các kiểu tùy chỉnh: hãy chuyển sang tính năng enum của Rust để thêm một công cụ khác vào bộ công cụ của bạn.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
