## Cú pháp Phương thức (Method Syntax)

_Phương thức_ (Methods) tương tự như hàm: chúng ta khai báo chúng bằng từ khóa `fn` và một
cái tên, chúng có thể có tham số và một giá trị trả về, và chúng chứa một đoạn mã
được chạy khi phương thức được gọi từ một nơi khác. Không giống như hàm,
các phương thức được định nghĩa trong ngữ cảnh của một struct (hoặc một enum hoặc một đối tượng trait,
mà chúng ta đề cập lần lượt trong [Chương 6][enums]<!-- ignore --> và [Chương
18][trait-objects]<!-- ignore -->), và tham số đầu tiên của chúng
luôn là `self`, đại diện cho chính thể hiện của struct mà phương thức đang được gọi trên đó.

### Định nghĩa các Phương thức

Hãy thay đổi hàm `area` vốn có một thể hiện `Rectangle` làm tham số
và thay vào đó hãy tạo một phương thức `area` được định nghĩa trên struct `Rectangle`, như được hiển thị
trong Listing 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="Định nghĩa một phương thức `area` trên struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Để định nghĩa hàm trong ngữ cảnh của `Rectangle`, chúng ta bắt đầu một khối `impl`
(implementation - triển khai) cho `Rectangle`. Mọi thứ trong khối `impl` này
sẽ được liên kết với kiểu `Rectangle`. Sau đó, chúng ta di chuyển hàm `area`
vào bên trong các dấu ngoặc nhọn của `impl` và thay đổi tham số đầu tiên (và trong trường hợp này là duy nhất)
thành `self` trong chữ ký hàm và mọi nơi bên trong thân hàm. Trong hàm
`main`, nơi chúng ta đã gọi hàm `area` và truyền `rect1` như một đối số,
thay vào đó chúng ta có thể sử dụng _cú pháp phương thức_ (method syntax) để gọi phương thức `area` trên thể hiện `Rectangle`
của chúng ta. Cú pháp phương thức được đặt sau một thể hiện: chúng ta thêm một dấu chấm theo sau là
tên phương thức, dấu ngoặc đơn và bất kỳ đối số nào.

Trong chữ ký của `area`, chúng ta sử dụng `&self` thay vì `rectangle: &Rectangle`.
`&self` thực chất là cách viết tắt của `self: &Self`. Bên trong một khối `impl`,
kiểu `Self` là một bí danh cho kiểu mà khối `impl` đó đang triển khai. Các phương thức phải
có một tham số tên là `self` kiểu `Self` làm tham số đầu tiên của chúng, vì vậy Rust
cho phép bạn viết tắt điều này chỉ với tên `self` ở vị trí tham số đầu tiên.
Lưu ý rằng chúng ta vẫn cần sử dụng dấu `&` phía trước cách viết tắt `self` để
chỉ ra rằng phương thức này vay mượn thể hiện `Self`, giống như chúng ta đã làm trong
`rectangle: &Rectangle`. Các phương thức có thể nắm quyền sở hữu `self`, vay mượn `self`
bất biến như chúng ta đã làm ở đây, hoặc vay mượn `self` có thể thay đổi, giống như chúng có thể với bất kỳ
tham số nào khác.

Chúng ta đã chọn `&self` ở đây vì cùng lý do chúng ta đã sử dụng `&Rectangle` trong phiên bản hàm:
chúng ta không muốn nắm quyền sở hữu, và chúng ta chỉ muốn đọc dữ liệu trong
struct chứ không ghi vào nó. Nếu chúng ta muốn thay đổi thể hiện mà chúng ta đã
gọi phương thức trên đó như một phần của những gì phương thức thực hiện, chúng ta sẽ sử dụng `&mut self` làm
tham số đầu tiên. Việc có một phương thức nắm quyền sở hữu thể hiện bằng cách chỉ
sử dụng `self` làm tham số đầu tiên là rất hiếm; kỹ thuật này thường được sử dụng
khi phương thức chuyển đổi `self` thành một thứ khác và bạn muốn ngăn
người gọi sử dụng thể hiện ban đầu sau khi chuyển đổi.

Lý do chính của việc sử dụng các phương thức thay vì hàm, ngoài việc
cung cấp cú pháp phương thức và không phải lặp lại kiểu của `self` trong mọi
chữ ký phương thức, là để tổ chức. Chúng ta đã đặt tất cả những gì chúng ta có thể làm
với một thể hiện của một kiểu vào trong một khối `impl` thay vì bắt những người dùng tương lai
trong mã của chúng ta phải tìm kiếm các khả năng của `Rectangle` ở nhiều nơi khác nhau trong
thư viện mà chúng ta cung cấp.

Lưu ý rằng chúng ta có thể chọn đặt cho phương thức cùng tên với một trong các
trường của struct. Ví dụ, chúng ta có thể định nghĩa một phương thức trên `Rectangle` cũng có tên là
`width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta chọn làm cho phương thức `width` trả về `true` nếu giá trị trong
trường `width` của thể hiện lớn hơn `0` và `false` nếu giá trị là
`0`: chúng ta có thể sử dụng một trường bên trong một phương thức cùng tên cho bất kỳ mục đích nào. Trong
hàm `main`, khi chúng ta viết `rect1.width` kèm theo dấu ngoặc đơn, Rust biết chúng ta muốn ám chỉ
phương thức `width`. Khi chúng ta không sử dụng dấu ngoặc đơn, Rust biết chúng ta muốn ám chỉ trường
`width`.

Thường thì, nhưng không phải luôn luôn, khi chúng ta đặt cho phương thức cùng tên với một trường, chúng ta muốn
nó chỉ trả về giá trị trong trường đó và không làm gì khác. Các phương thức như thế này
được gọi là _getters_, và Rust không tự động triển khai chúng cho các trường struct
như một số ngôn ngữ khác thực hiện. Getters hữu ích vì bạn có thể để
trường ở chế độ riêng tư (private) nhưng phương thức ở chế độ công khai (public), và do đó cho phép quyền truy cập chỉ đọc vào
trường đó như một phần của API công khai của kiểu dữ liệu. Chúng ta sẽ thảo luận về public và private là gì
và cách chỉ định một trường hoặc phương thức là public hay private trong [Chương
7][public]<!-- ignore -->.

### Các phương thức với nhiều tham số hơn

Hãy thực hành sử dụng phương thức bằng cách triển khai một phương thức thứ hai trên struct `Rectangle`.
Lần này chúng ta muốn một thể hiện của `Rectangle` nhận một thể hiện khác của `Rectangle`
và trả về `true` nếu hình chữ nhật thứ hai có thể nằm hoàn toàn bên trong
`self` (hình chữ nhật đầu tiên); nếu không, nó sẽ trả về `false`.
Tức là, một khi chúng ta đã định nghĩa phương thức `can_hold`, chúng ta muốn có thể viết
chương trình như trong Listing 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="Sử dụng phương thức `can_hold` chưa được viết">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

Kết quả đầu ra mong đợi sẽ trông như sau vì cả hai kích thước của
`rect2` đều nhỏ hơn kích thước của `rect1`, nhưng `rect3` lại rộng hơn
`rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Chúng ta biết mình muốn định nghĩa một phương thức, vì vậy nó sẽ nằm trong khối `impl Rectangle`.
Tên phương thức sẽ là `can_hold`, và nó sẽ nhận một phép vay mượn bất biến
của một `Rectangle` khác làm tham số. Chúng ta có thể biết kiểu của tham số
sẽ là gì bằng cách nhìn vào đoạn mã gọi phương thức:
`rect1.can_hold(&rect2)` truyền vào `&rect2`, vốn là một phép vay mượn bất biến của
`rect2`, một thể hiện của `Rectangle`. Điều này hợp lý vì chúng ta chỉ cần
đọc `rect2` (thay vì ghi, điều này có nghĩa là chúng ta cần một phép vay mượn có thể thay đổi),
và chúng ta muốn `main` vẫn giữ quyền sở hữu của `rect2` để chúng ta có thể sử dụng lại nó sau khi
gọi phương thức `can_hold`. Giá trị trả về của `can_hold` sẽ là một
kiểu Boolean, và việc triển khai sẽ kiểm tra xem chiều rộng và chiều cao của
`self` có lần lượt lớn hơn chiều rộng và chiều cao của hình chữ nhật kia (`other`) hay không.
Hãy thêm phương thức `can_hold` mới vào khối `impl` từ Listing 5-13, được hiển thị trong Listing 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="Triển khai phương thức `can_hold` trên `Rectangle` nhận một thể hiện `Rectangle` khác làm tham số">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Khi chúng ta chạy mã này với hàm `main` ở Listing 5-14, chúng ta sẽ nhận được
kết quả mong muốn. Các phương thức có thể nhận nhiều tham số mà chúng ta thêm vào
chữ ký sau tham số `self`, và những tham số đó hoạt động giống như
các tham số trong hàm.

### Các hàm liên kết

Tất cả các hàm được định nghĩa bên trong một khối `impl` được gọi là _các hàm liên kết_ (associated functions)
bởi vì chúng được liên kết với kiểu được đặt tên sau từ khóa `impl`. Chúng ta có thể định nghĩa
các hàm liên kết như là các hàm không có `self` làm tham số đầu tiên (và do đó
không phải là các phương thức) vì chúng không cần một thể hiện của kiểu để làm việc cùng.
Chúng ta đã từng sử dụng một hàm như thế này rồi: hàm `String::from`
được định nghĩa trên kiểu `String`.

Các hàm liên kết không phải là phương thức thường được sử dụng cho các hàm khởi tạo (constructors)
sẽ trả về một thể hiện mới của struct. Những hàm này thường được gọi là `new`, nhưng
`new` không phải là một cái tên đặc biệt và không được xây dựng sẵn trong ngôn ngữ. Ví dụ, chúng ta
có thể chọn cung cấp một hàm liên kết tên là `square` nhận vào một tham số kích thước
và sử dụng nó làm cả chiều rộng và chiều cao, do đó giúp việc tạo một `Rectangle` hình vuông
trở nên dễ dàng hơn thay vì phải chỉ định cùng một giá trị hai lần:

<span class="filename">Tên file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Các từ khóa `Self` trong kiểu trả về và trong thân hàm là
bí danh cho kiểu xuất hiện sau từ khóa `impl`, trong trường hợp này
là `Rectangle`.

Để gọi hàm liên kết này, chúng ta sử dụng cú pháp `::` với tên struct;
`let sq = Rectangle::square(3);` là một ví dụ. Hàm này được phân không gian tên (namespaced) bởi
struct: cú pháp `::` được sử dụng cho cả các hàm liên kết và các không gian tên
được tạo ra bởi các module. Chúng ta sẽ thảo luận về module trong [Chương
7][modules]<!-- ignore -->.

### Nhiều khối `impl`

Mỗi struct được phép có nhiều khối `impl`. Ví dụ, Listing
5-15 tương đương với đoạn mã được hiển thị trong Listing 5-16, trong đó mỗi phương thức nằm trong
khối `impl` của riêng nó.

<Listing number="5-16" caption="Viết lại Listing 5-15 bằng cách sử dụng nhiều khối `impl`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Không có lý do gì để tách các phương thức này thành nhiều khối `impl` ở đây,
nhưng đây là một cú pháp hợp lệ. Chúng ta sẽ thấy trường hợp mà việc sử dụng nhiều khối `impl` là
hữu ích trong Chương 10, nơi chúng ta thảo luận về các kiểu generic và các trait.

### Lời gọi Phương thức là Cú pháp Hình thức cho Lời gọi Hàm

Sử dụng các khái niệm mà chúng ta đã thảo luận cho đến nay, giờ đây chúng ta có thể thấy cách các lời gọi phương thức là cú pháp hình thức (syntactic sugar) cho các lời gọi hàm. Ví dụ, giả sử chúng ta có một struct rectangle với một phương thức `area` và một phương thức `set_width`:

```rust,ignore
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn set_width(&mut self, width: u32) {
        self.width = width;
    }
}
```

Và giả sử chúng ta có một hình chữ nhật `r`. Khi đó, các lời gọi phương thức `r.area()` và `r.set_width(2)` tương đương với điều này:

```rust
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
# impl Rectangle {
#     fn area(&self) -> u32 {
#        self.width * self.height
#      }
#
#     fn set_width(&mut self, width: u32) {
#         self.width = width;
#     }
# }
#
# fn main() {
    let mut r = Rectangle {
        width: 1,
        height: 2
    };
    let area1 = r.area();
    let area2 = Rectangle::area(&r);
    assert_eq!(area1, area2);

    r.set_width(2);
    Rectangle::set_width(&mut r, 2);
# }
```

Lời gọi phương thức `r.area()` trở thành `Rectangle::area(&r)`. Tên hàm là hàm liên kết `Rectangle::area`. Đối số của hàm là tham số `&self`. Rust tự động chèn toán tử vay mượn `&`.

> _Lưu ý:_ nếu bạn đã quen với C hoặc C++, bạn đã quen với hai cú pháp khác nhau cho các lời gọi phương thức: `r.area()` và `r->area()`. Rust không có toán tử tương đương với toán tử mũi tên `->`. Rust sẽ tự động tham chiếu (reference) và giải tham chiếu (dereference) đối tượng nhận phương thức khi bạn sử dụng toán tử dấu chấm.

Lời gọi phương thức `r.set_width(2)` tương tự trở thành `Rectangle::set_width(&mut r, 2)`. Phương thức này yêu cầu `&mut self`, vì vậy đối số đầu tiên là một phép vay mượn có thể thay đổi `&mut r`. Đối số thứ hai hoàn toàn giống nhau, là con số 2.

Như chúng ta đã mô tả trong Chương 4.2 [“Giải tham chiếu một Con trỏ để Truy cập Dữ liệu của nó”](ch04-02-references-and-borrowing.html#dereferencing-a-pointer-accesses-its-data), Rust sẽ chèn bao nhiêu phép tham chiếu và giải tham chiếu cần thiết để làm cho các kiểu khớp với nhau cho tham số `self`. Ví dụ, đây là hai lời gọi tương đương đến `area` cho một tham chiếu có thể thay đổi đến một boxed rectangle:

```rust
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
# impl Rectangle {
#     fn area(&self) -> u32 {
#        self.width * self.height
#      }
#
#     fn set_width(&mut self, width: u32) {
#         self.width = width;
#     }
# }
# fn main() {
    let r = &mut Box::new(Rectangle {
        width: 1,
        height: 2
    });
    let area1 = r.area();
    let area2 = Rectangle::area(&**r);
    assert_eq!(area1, area2);
# }
```

Rust sẽ thêm hai phép giải tham chiếu (một lần cho tham chiếu có thể thay đổi, một lần cho box) và sau đó là một phép vay mượn bất biến vì `area` yêu cầu `&Rectangle`. Lưu ý rằng đây cũng là một tình huống mà một tham chiếu có thể thay đổi bị "hạ cấp" thành một tham chiếu dùng chung, như chúng ta đã thảo luận trong [Chương 4.2](ch04-02-references-and-borrowing.html#mutable-references-provide-unique-and-non-owning-access-to-data). Ngược lại, bạn sẽ không được phép gọi `set_width` trên một giá trị có kiểu `&Rectangle` hoặc `&Box<Rectangle>`.

{{#quiz ../quizzes/ch05-03-method-syntax-sec1.toml}}

### Phương thức và Quyền sở hữu

Như chúng ta đã thảo luận trong Chương 4.2 [“Tham chiếu và Vay mượn”](ch04-02-references-and-borrowing.html), các phương thức phải được gọi trên các struct có các quyền hạn cần thiết. Như một ví dụ xuyên suốt, chúng ta sẽ sử dụng ba phương thức lần lượt nhận vào `&self`, `&mut self`, và `self`.

```rust,ignore
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn set_width(&mut self, width: u32) {
        self.width = width;
    }

    fn max(self, other: Rectangle) -> Rectangle {
        Rectangle {
            width: self.width.max(other.width),
            height: self.height.max(other.height),
        }
    }
}
```

#### Đọc và Ghi với `&self` và `&mut self`

Nếu chúng ta tạo một hình chữ nhật được sở hữu với `let rect = Rectangle { ... }`, thì `rect` có các quyền hạn @Perm{read} và @Perm{own}. Với những quyền hạn đó, việc gọi các phương thức `area` và `max` là được phép:

```aquascope,permissions,boundaries,stepper
#struct Rectangle {
#    width: u32,
#    height: u32,
#}
#impl Rectangle {
#  fn area(&self) -> u32 {
#    self.width * self.height
#  }
#
#  fn set_width(&mut self, width: u32) {
#    self.width = width;
#  }
#
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h
#    }
#  }
#}
#fn main() {
let rect = Rectangle {
    width: 0,
    height: 0
};`(focus,rxpaths:^rect$)`
println!("{}", rect.area());`{}`

let other_rect = Rectangle { width: 1, height: 1 };
let max_rect = rect.max(other_rect);`{}`
#}
```

Tuy nhiên, nếu chúng ta cố gắng gọi `set_width`, chúng ta bị thiếu quyền hạn @Perm{write}:

```aquascope,permissions,boundaries,shouldFail
#struct Rectangle {
#    width: u32,
#    height: u32,
#}
#impl Rectangle {
#  fn area(&self) -> u32 {
#    self.width * self.height
#  }
#
#  fn set_width(&mut self, width: u32) {
#    self.width = width;
#  }
#
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h
#    }
#  }
#}
#fn main() {
let rect = Rectangle {
    width: 0,
    height: 0
};
rect.set_width(0);`{}`
#}
```

Rust sẽ từ chối chương trình này với lỗi tương ứng:

```text
error[E0596]: cannot borrow `rect` as mutable, as it is not declared as mutable
  --> test.rs:28:1
   |
24 | let rect = Rectangle {
   |     ---- help: consider changing this to be mutable: `mut rect`
...
28 | rect.set_width(0);
   | ^^^^^^^^^^^^^^^^^ cannot borrow as mutable
```

Chúng ta sẽ nhận được một lỗi tương tự nếu chúng ta cố gắng gọi `set_width` trên một tham chiếu bất biến đến một `Rectangle`, ngay cả khi hình chữ nhật bên dưới là có thể thay đổi:

```aquascope,permissions,boundaries,stepper,shouldFail
#struct Rectangle {
#    width: u32,
#    height: u32,
#}
#impl Rectangle {
#  fn area(&self) -> u32 {
#    self.width * self.height
#  }
#
#  fn set_width(&mut self, width: u32) {
#    self.width = width;
#  }
#
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h
#    }
#  }
#}
#fn main() {
// Đã thêm từ khóa mut vào let-binding
let mut rect = Rectangle {
    width: 0,
    height: 0
};`(focus,rxpaths:^rect$)`
rect.set_width(1);`{}`     // điều này bây giờ ổn

let rect_ref = &rect;`(focus,rxpaths:^\*rect_ref$)`
rect_ref.set_width(2);`{}` // nhưng điều này vẫn không ổn
#}
```

#### Di chuyển với `self`

Gọi một phương thức yêu cầu `self` sẽ di chuyển (move) struct đầu vào (trừ khi struct đó triển khai `Copy`). Ví dụ, chúng ta không thể sử dụng một `Rectangle` sau khi truyền nó cho `max`:

```aquascope,permissions,boundaries,stepper,shouldFail
#struct Rectangle {
#    width: u32,
#    height: u32,
#}
#impl Rectangle {
#  fn area(&self) -> u32 {
#    self.width * self.height
#  }
#
#  fn set_width(&mut self, width: u32) {
#    self.width = width;
#  }
#
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h
#    }
#  }
#}
#fn main() {
let rect = Rectangle {
    width: 0,
    height: 0
};`(focus,rxpaths:^rect$)`
let other_rect = Rectangle {
    width: 1,
    height: 1
};
let max_rect = rect.max(other_rect);`(focus,rxpaths:^rect$)`
println!("{}", rect.area());`{}`
#}
```

Một khi chúng ta gọi `rect.max(..)`, chúng ta di chuyển `rect` và do đó mất tất cả các quyền hạn trên nó. Cố gắng biên dịch chương trình này sẽ cho chúng ta lỗi sau:

```text
error[E0382]: borrow of moved value: `rect`
  --> test.rs:33:16
   |
24 | let rect = Rectangle {
   |     ---- move occurs because `rect` has type `Rectangle`, which does not implement the `Copy` trait
...
32 | let max_rect = rect.max(other_rect);
   |                     --------------- `rect` moved due to this method call
33 | println!("{}", rect.area());
   |                ^^^^^^^^^^^ value borrowed here after move
```

Một tình huống tương tự nảy sinh nếu chúng ta cố gắng gọi một phương thức `self` trên một tham chiếu. Chẳng hạn, giả sử chúng ta cố gắng tạo một phương thức `set_to_max` gán `self` cho kết quả đầu ra của `self.max(..)`:

```aquascope,permissions,boundaries,stepper,shouldFail
#struct Rectangle {
#    width: u32,
#    height: u32,
#}
impl Rectangle {
#  fn area(&self) -> u32 {
#    self.width * self.height
#  }
#
#  fn set_width(&mut self, width: u32) {
#    self.width = width;
#  }
#
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h
#    }
#  }
    fn set_to_max(&mut self, other: Rectangle) {`(focus,rxpaths:^\*self$)`
        *self = self.max(other);`{}`
    }
}
```

Khi đó chúng ta có thể thấy rằng `self` thiếu quyền hạn @Perm{own} trong phép toán `self.max(..)`. Do đó, Rust từ chối chương trình này với lỗi sau:

```text
error[E0507]: cannot move out of `*self` which is behind a mutable reference
  --> test.rs:23:17
   |
23 |         *self = self.max(other);
   |                 ^^^^^----------
   |                 |    |
   |                 |    `*self` moved due to this method call
   |                 move occurs because `*self` has type `Rectangle`, which does not implement the `Copy` trait
   |
```

Đây là cùng một loại lỗi mà chúng ta đã thảo luận trong Chương 4.3 [“Sao chép và Di chuyển ra khỏi một Bộ sưu tập”](ch04-03-fixing-ownership-errors.html#fixing-an-unsafe-program-copying-vs-moving-out-of-a-collection).

#### Các bước Di chuyển Tốt và Các bước Di chuyển Xấu

Bạn có thể tự hỏi: tại sao việc chúng ta di chuyển ra khỏi `*self` lại quan trọng? Trên thực tế, đối với trường hợp của `Rectangle`, việc di chuyển ra khỏi `*self` thực sự là an toàn, mặc dù Rust không cho phép bạn làm điều đó. Ví dụ, nếu chúng ta mô phỏng một chương trình gọi phương thức `set_to_max` bị từ chối, bạn có thể thấy không có điều gì không an toàn xảy ra:

```aquascope,interpreter,shouldFail,horizontal
#struct Rectangle {
#    width: u32,
#    height: u32,
#}
impl Rectangle {
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h
#    }
#  }
    fn set_to_max(&mut self, other: Rectangle) {
        let max = self.max(other);`[]`
        *self = max;
    }
}

fn main() {
    let mut rect = Rectangle { width: 0, height: 1 };
    let other_rect = Rectangle { width: 1, height: 0 };`[]`
    rect.set_to_max(other_rect);`[]`
}
```

Lý do di chuyển ra khỏi `*self` an toàn là vì `Rectangle` không sở hữu bất kỳ dữ liệu heap nào.
Trên thực tế, chúng ta thực sự có thể làm cho Rust biên dịch `set_to_max` bằng cách đơn giản thêm `#[derive(Copy, Clone)]` vào định nghĩa của `Rectangle`:

```aquascope,permissions,boundaries,stepper
\#[derive(Copy, Clone)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h
#    }
#  }
    fn set_to_max(&mut self, other: Rectangle) {`(focus,rxpaths:^\*self$)`
        *self = self.max(other);`{}`
    }
}
```

Chú ý rằng không giống như trước đây, `self.max(other)` không còn yêu cầu quyền hạn @Perm{own} trên `*self` hoặc `other`. Hãy nhớ rằng `self.max(other)` được giải mã thành `Rectangle::max(*self, other)`. Giải tham chiếu `*self` không yêu cầu quyền sở hữu trên `*self` nếu `Rectangle` có thể copy.

Bạn có thể thắc mắc: tại sao Rust không tự động dẫn xuất (derive) `Copy` cho `Rectangle`? Rust không tự động dẫn xuất `Copy` để đảm bảo tính ổn định qua các thay đổi API. Hãy tưởng tượng rằng tác giả của kiểu `Rectangle` quyết định thêm một trường `name: String`. Khi đó tất cả mã máy khách dựa vào việc `Rectangle` là `Copy` sẽ đột ngột bị trình biên dịch từ chối. Để tránh vấn đề đó, các tác giả API phải thêm `#[derive(Copy)]` một cách rõ ràng để chỉ ra rằng họ mong đợi struct của mình luôn là `Copy`.

Để hiểu rõ hơn vấn đề, hãy chạy một mô phỏng. Giả sử chúng ta đã thêm `name: String` vào `Rectangle`. Điều gì sẽ xảy ra nếu Rust cho phép `set_to_max` biên dịch?

```aquascope,interpreter,shouldFail,horizontal
struct Rectangle {
    width: u32,
    height: u32,
    name: String,
}

impl Rectangle {
#  fn max(self, other: Self) -> Self {
#    let w = self.width.max(other.width);
#    let h = self.height.max(other.height);
#    Rectangle {
#      width: w,
#      height: h,
#      name: String::from("max")
#    }
#  }
    fn set_to_max(&mut self, other: Rectangle) {
        `[]`let max = self.max(other);`[]`
        drop(*self);`[]` // Điều này thường là ngầm định,
                         // nhưng được thêm vào đây để rõ ràng.
        *self = max;
    }
}

fn main() {
    let mut r1 = Rectangle {
        width: 9,
        height: 9,
        name: String::from("r1")
    };
    let r2 = Rectangle {
        width: 16,
        height: 16,
        name: String::from("r2")
    };
    r1.set_to_max(r2);
}
```

Trong chương trình này, chúng ta gọi `set_to_max` với hai hình chữ nhật `r1` và `r2`. `self` là một tham chiếu có thể thay đổi đến `r1` và `other` là một bước di chuyển của `r2`. Sau khi gọi `self.max(other)`, phương thức `max` tiêu thụ quyền sở hữu của cả hai hình chữ nhật. Khi `max` trả về, Rust giải phóng cả hai chuỗi "r1" và "r2" trong heap. Lưu ý vấn đề: tại vị trí L2, `*self` được cho là có thể đọc và viết được. Tuy nhiên, `(*self).name` (thực tế là `r1.name`) đã bị giải phóng.

Do đó, khi chúng ta thực hiện `*self = max`, chúng ta gặp phải hành vi không xác định (undefined behavior). Khi chúng ta ghi đè lên `*self`, Rust sẽ ngầm định drop dữ liệu trước đó trong `*self`. Để làm cho hành vi đó rõ ràng, chúng ta đã thêm `drop(*self)`. Sau khi gọi `drop(*self)`, Rust cố gắng giải phóng `(*self).name` lần thứ hai. Hành động đó là một lỗi double-free, vốn là hành vi không xác định.

Vì vậy, hãy nhớ: khi bạn thấy một lỗi như "cannot move out of `*self`", đó thường là vì bạn đang cố gọi một phương thức `self` trên một tham chiếu như `&self` hoặc `&mut self`. Rust đang bảo vệ bạn khỏi lỗi double-free.

## Tóm tắt

Struct cho phép bạn tạo ra các kiểu dữ liệu tùy chỉnh có ý nghĩa cho miền nghiệp vụ của mình. Bằng cách
sử dụng struct, bạn có thể giữ các phần dữ liệu liên quan kết nối với nhau
và đặt tên cho từng phần để làm cho mã của bạn trở nên rõ ràng. Trong các khối `impl`, bạn có thể định nghĩa
các hàm liên kết với kiểu của mình, và các phương thức là một loại hàm
liên kết cho phép bạn chỉ định hành vi mà các thể hiện của
struct của bạn có.

Nhưng struct không phải là cách duy nhất bạn có thể tạo các kiểu dữ liệu tùy chỉnh: hãy chuyển sang
tính năng enum của Rust để thêm một công cụ khác vào hộp dụng cụ của bạn.

{{#quiz ../quizzes/ch05-03-method-syntax-sec2.toml}}

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
