## Unsafe Rust

Tất cả mã nguồn chúng ta đã thảo luận cho đến nay đều được trình biên dịch Rust
thực thi các đảm bảo an toàn bộ nhớ (memory safety) tại thời điểm biên dịch (compile time).
Tuy nhiên, Rust có một "ngôn ngữ thứ hai" ẩn bên trong nó mà không áp đặt các đảm bảo
an toàn bộ nhớ nghiêm ngặt này: nó được gọi là _unsafe Rust_. Unsafe Rust hoạt động
giống như Rust thông thường nhưng cung cấp thêm cho chúng ta những "siêu năng lực" đặc biệt.

Unsafe Rust tồn tại vì, về bản chất, phân tích tĩnh (static analysis) của trình biên
dịch luôn có tính bảo thủ (conservative). Khi trình biên dịch cố gắng xác định xem
mã nguồn có đảm bảo an toàn hay không, nó thà từ chối một số chương trình hợp lệ còn
hơn là chấp nhận một chương trình không an toàn. Mặc dù mã nguồn _có thể_ hoàn toàn
đúng đắn, nhưng nếu trình biên dịch Rust không có đủ thông tin để khẳng định chắc chắn,
nó sẽ từ chối biên dịch. Trong những trường hợp này, bạn có thể sử dụng mã `unsafe`
để nói với trình biên dịch rằng: “Hãy tin tôi, tôi biết mình đang làm gì.” Tuy nhiên,
hãy lưu ý rằng bạn tự chịu rủi ro khi sử dụng unsafe Rust: nếu sử dụng không đúng cách,
các sự cố nghiêm trọng về bộ nhớ có thể xảy ra, chẳng hạn như giải tham chiếu con trỏ null
(null pointer dereferencing — khác với Python sẽ ném ra ngoại lệ `AttributeError` khi
gặp `None`, trong Rust và C việc giải tham chiếu con trỏ null sẽ làm crash chương trình
ngay lập tức với lỗi phân đoạn hoặc gây lỗ hổng bảo mật).

Một lý do khác khiến Rust có một "mặt tối" unsafe là vì phần cứng máy tính bên dưới
vốn dĩ không an toàn. Nếu Rust không cho phép bạn thực hiện các thao tác cấp thấp, bạn
sẽ không thể thực hiện một số tác vụ nhất định. Rust cần cho phép bạn lập trình hệ thống
cấp thấp, chẳng hạn như tương tác trực tiếp với hệ điều hành hoặc thậm chí tự viết một
hệ điều hành. Lập trình hệ thống cấp thấp là một trong những mục tiêu cốt lõi của Rust.
Hãy cùng khám phá những gì chúng ta có thể làm với unsafe Rust và cách thực hiện nó.

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### Thực Hiện Các Siêu Năng Lực Của Unsafe

Để chuyển sang unsafe Rust, hãy sử dụng từ khóa `unsafe` và bắt đầu một khối lệnh mới
chứa mã unsafe. Bạn có thể thực hiện năm hành động trong unsafe Rust mà không thể làm
trong Rust an toàn (safe Rust), chúng ta gọi đó là _các siêu năng lực của unsafe_ (unsafe superpowers).
Những siêu năng lực đó bao gồm:

1. Giải tham chiếu một con trỏ thô (dereference a raw pointer).
2. Gọi một hàm hoặc phương thức unsafe (call an unsafe function or method).
3. Truy cập hoặc sửa đổi một biến tĩnh có thể thay đổi (access or modify a mutable static variable).
4. Triển khai một trait unsafe (implement an unsafe trait).
5. Truy cập các trường của một `union`.

Điều quan trọng cần hiểu là `unsafe` không tắt bộ kiểm tra mượn (borrow checker) hoặc
vô hiệu hóa bất kỳ kiểm tra an toàn nào khác của Rust: nếu bạn sử dụng một tham chiếu
trong mã unsafe, nó vẫn sẽ được kiểm tra quyền mượn bình thường. Từ khóa `unsafe` chỉ
cung cấp cho bạn quyền truy cập vào năm tính năng trên mà trình biên dịch không thể tự
động kiểm tra an toàn bộ nhớ. Bạn vẫn được thừa hưởng mức độ an toàn vốn có của Rust
bên trong một khối `unsafe`.

Ngoài ra, `unsafe` không có nghĩa là mã nguồn bên trong khối nhất thiết phải nguy hiểm
hoặc chắc chắn sẽ gặp lỗi: mục đích là với tư cách lập trình viên, bạn cam kết rằng mã
nguồn bên trong khối `unsafe` sẽ truy cập bộ nhớ một cách hợp lệ.

Con người luôn có thể mắc sai lầm, nhưng bằng cách yêu cầu năm thao tác unsafe này
phải nằm trong các khối được đánh dấu bằng `unsafe`, bạn sẽ dễ dàng khoanh vùng rằng
mọi lỗi liên quan đến an toàn bộ nhớ chắc chắn phải bắt nguồn từ bên trong một khối
`unsafe`. Hãy giữ các khối `unsafe` càng nhỏ càng tốt; bạn sẽ thấy điều này vô cùng
hữu ích khi cần truy vết và sửa lỗi bộ nhớ sau này.

Để cô lập mã unsafe tốt nhất có thể, bạn nên bao bọc (encapsulate) mã đó bên trong một
trừu tượng an toàn (safe abstraction) và cung cấp một API an toàn ra bên ngoài. Các phần
trong thư viện chuẩn của Rust cũng thường được triển khai dưới dạng các trừu tượng an toàn
bọc quanh mã unsafe đã được kiểm định kỹ lưỡng. Việc này giúp ngăn chặn `unsafe` rò rỉ
ra khắp nơi trong mã của bạn hoặc người dùng, bởi vì việc gọi một hàm an toàn thì luôn
luôn an toàn.

Hãy lần lượt xem xét từng siêu năng lực trong số năm siêu năng lực của unsafe.

### Giải Tham Chiếu Một Con Trỏ Thô

Trong phần [“Borrow checker phát hiện các vi phạm quyền”][permission-violations]<!-- ignore --> ở
Chương 4, chúng ta đã biết cách trình biên dịch đảm bảo các tham chiếu luôn hợp lệ. Unsafe
Rust cung cấp hai kiểu dữ liệu mới gọi là _con trỏ thô_ (raw pointers), tương tự như các
con trỏ trong ngôn ngữ C. Giống như tham chiếu, con trỏ thô có thể là bất biến (immutable)
hoặc khả biến (mutable), được viết lần lượt là `*const T` và `*mut T`. Dấu hoa thị `*` ở đây
không phải là toán tử giải tham chiếu mà là một phần trong tên kiểu dữ liệu. Trong ngữ cảnh
của con trỏ thô, _bất biến_ có nghĩa là con trỏ không thể được gán giá trị mới trực tiếp
sau khi đã giải tham chiếu.

Khác với tham chiếu (`&T`, `&mut T`) và con trỏ thông minh (smart pointers như `Box`, `Rc`), con trỏ thô:

- Được phép bỏ qua các quy tắc mượn (borrowing rules): có thể có đồng thời cả con trỏ bất
  biến và con trỏ khả biến, hoặc nhiều con trỏ khả biến cùng trỏ tới một vị trí bộ nhớ.
- Không được đảm bảo là luôn trỏ tới vùng bộ nhớ hợp lệ.
- Được phép mang giá trị `null` (khác với tham chiếu trong Rust không bao giờ có thể là null).
- Không tự động dọn dẹp bộ nhớ khi đi ra ngoài phạm vi (không tự động gọi `Drop`).

Bằng cách chọn không để Rust tự động thực thi các đảm bảo này, bạn chấp nhận đánh đổi sự
an toàn tuyệt đối để lấy hiệu năng cao hơn hoặc khả năng giao tiếp (interop) với phần cứng
hoặc các ngôn ngữ khác (như C/C++) nơi mà các quy tắc của Rust không áp dụng.

Listing 20-1 cho thấy cách tạo một con trỏ thô bất biến và một con trỏ thô khả biến.

<Listing number="20-1" caption="Tạo các con trỏ thô bằng các toán tử mượn thô (raw borrow operators)">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta không cần dùng từ khóa `unsafe` trong đoạn mã này. Chúng ta có thể tạo
con trỏ thô hoàn toàn trong mã an toàn (safe code); chúng ta chỉ không thể giải tham chiếu
(dereference) chúng bên ngoài khối `unsafe`, như bạn sẽ thấy ngay sau đây.

Chúng ta đã tạo các con trỏ thô bằng các toán tử mượn thô (raw borrow operators): `&raw const num`
tạo ra một con trỏ thô bất biến `*const i32`, và `&raw mut num` tạo ra một con trỏ thô khả biến
`*mut i32`. Vì chúng ta tạo trực tiếp từ một biến cục bộ hợp lệ, chúng ta biết chắc các con
trỏ này là hợp lệ, nhưng trình biên dịch sẽ không mặc định giả định điều đó cho mọi con trỏ thô.

Để chứng minh điều này, tiếp theo chúng ta sẽ tạo một con trỏ thô trỏ đến một địa chỉ bộ nhớ
bất kỳ bằng cách ép kiểu qua từ khóa `as` thay vì dùng toán tử mượn thô. Listing 20-2 cho thấy
cách tạo con trỏ thô đến một địa chỉ tùy ý trong bộ nhớ.

<Listing number="20-2" caption="Tạo một con trỏ thô đến một địa chỉ bộ nhớ tùy ý">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Cố gắng truy cập một vùng nhớ tùy ý là hành vi không xác định (undefined behavior - UB): có thể
có dữ liệu tại địa chỉ đó hoặc không, trình biên dịch có thể tối ưu hóa bỏ qua thao tác truy cập,
hoặc chương trình sẽ bị crash với lỗi phân đoạn (segmentation fault). Thông thường không có lý
do gì để viết mã như thế này, nhưng về mặt kỹ thuật thì Rust cho phép tạo ra nó.

Hãy nhớ rằng việc tạo con trỏ thô trong safe code là hợp lệ, nhưng chúng ta không thể _giải
tham chiếu_ (`*ptr`) để đọc/ghi dữ liệu mà không có khối `unsafe`. Trong Listing 20-3, chúng ta
sử dụng toán tử giải tham chiếu `*` trên con trỏ thô bên trong một khối `unsafe`.

<Listing number="20-3" caption="Giải tham chiếu các con trỏ thô bên trong một khối `unsafe`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Bản thân việc tạo một con trỏ không gây hại gì; chỉ khi chúng ta giải tham chiếu để truy cập
dữ liệu mà nó trỏ tới, chúng ta mới có nguy cơ gặp phải vùng nhớ không hợp lệ.

Lưu ý rằng trong Listing 20-1 và 20-3, chúng ta đã tạo cả `*const i32` và `*mut i32` cùng trỏ tới
một vị trí bộ nhớ của `num`. Nếu chúng ta cố gắng tạo một tham chiếu bất biến `&num` và một tham
chiếu khả biến `&mut num` cùng lúc trong safe Rust, mã sẽ bị từ chối biên dịch ngay lập tức vì
vi phạm quy tắc ownership và borrowing của Rust. Với con trỏ thô, bạn có thể bỏ qua quy tắc này,
nhưng nếu nhiều con trỏ cùng ghi dữ liệu đồng thời, bạn có thể tạo ra tình trạng tranh chấp dữ
liệu (data race). Hãy hết sức cẩn thận!

Với tất cả những nguy cơ này, tại sao lại cần dùng con trỏ thô? Trường hợp phổ biến nhất là khi
giao tiếp với mã C (FFI), như bạn sẽ thấy ở phần tiếp theo. Một trường hợp khác là khi xây dựng
các cấu trúc dữ liệu hoặc trừu tượng an toàn mà bộ kiểm tra mượn (borrow checker) không thể tự
chứng minh được tính an toàn.

### Gọi Một Hàm Hoặc Phương Thức Unsafe

Thao tác thứ hai bạn có thể thực hiện trong một khối `unsafe` là gọi các hàm unsafe. Hàm và
phương thức unsafe trông giống hệt hàm thông thường, nhưng có thêm từ khóa `unsafe` đặt trước
từ khóa `fn`. Từ khóa `unsafe` ở đây chỉ ra rằng hàm này có các điều kiện tiên quyết (preconditions)
mà người gọi phải tự đảm bảo, vì trình biên dịch Rust không thể tự kiểm tra chúng.

Dưới đây là một hàm unsafe có tên là `dangerous` không thực hiện thao tác gì trong thân hàm:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Chúng ta bắt buộc phải gọi hàm `dangerous` bên trong một khối `unsafe`. Nếu gọi trực tiếp, trình
biên dịch sẽ báo lỗi:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Bằng việc đặt lệnh gọi trong khối `unsafe`, bạn khẳng định với Rust rằng bạn đã đọc tài liệu của
hàm, hiểu cách sử dụng và cam kết đáp ứng đúng hợp đồng an toàn của hàm đó.

Để thực hiện các thao tác unsafe bên trong thân của một hàm unsafe, bạn vẫn cần sử dụng một
khối `unsafe` (trừ khi được cấu hình khác ở các phiên bản cũ). Điều này giúp giữ cho phạm vi
`unsafe` nhỏ nhất có thể, tránh việc cả thân hàm dài đều bị coi là không kiểm soát.

#### Tạo Một Trừu Tượng An Toàn Bao Bọc Mã Unsafe

Chỉ vì một hàm có chứa mã unsafe không có nghĩa là chúng ta phải đánh dấu toàn bộ hàm đó là
`unsafe`. Trên thực tế, bọc mã unsafe trong một hàm an toàn là mô hình thiết kế rất phổ biến trong
Rust. Ví dụ, hãy xem xét hàm `split_at_mut` từ thư viện chuẩn. Hàm an toàn này nhận vào một slice
khả biến và chia nó thành hai slice không chồng lấn tại vị trí chỉ số `mid`. Listing 20-4 minh họa
cách sử dụng `split_at_mut`.

<Listing number="20-4" caption="Sử dụng hàm an toàn `split_at_mut`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Chúng ta không thể triển khai hàm này nếu chỉ dùng safe Rust. Hãy xem một nỗ lực triển khai
trong Listing 20-5 (đoạn mã này sẽ không thể biên dịch).

<Listing number="20-5" caption="Thử nghiệm triển khai `split_at_mut` chỉ bằng safe Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Khi biên dịch Listing 20-5, chúng ta sẽ nhận được lỗi từ borrow checker:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Borrow checker của Rust không thể hiểu rằng chúng ta đang mượn hai phần tách biệt không chồng lấn
của cùng một slice; nó chỉ thấy rằng chúng ta đang cố mượn khả biến (`&mut`) từ cùng một slice hai
lần. Vì chúng ta là lập trình viên và biết chắc chắn hai nửa slice này hoàn toàn độc lập, đây là
lúc thích hợp để sử dụng mã `unsafe`.

Listing 20-6 cho thấy cách sử dụng khối `unsafe`, con trỏ thô và các hàm unsafe để làm cho
`split_at_mut` hoạt động.

<Listing number="20-6" caption="Sử dụng mã unsafe trong việc triển khai hàm `split_at_mut`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Hãy nhớ lại từ [“Kiểu Slice”][the-slice-type]<!-- ignore --> ở Chương 4 rằng một slice bản chất là
một con trỏ trỏ tới dữ liệu và độ dài của nó. Phương thức `as_mut_ptr` trả về một con trỏ thô
`*mut i32`. Sau đó hàm `slice::from_raw_parts_mut` nhận con trỏ thô cùng độ dài để tái tạo thành
một slice. Phương thức `add` trên con trỏ thô dùng để dịch chuyển địa chỉ con trỏ tới vị trí `mid`.

Cả `slice::from_raw_parts_mut` và `add` đều là hàm `unsafe` vì chúng tin tưởng tuyệt đối rằng
con trỏ truyền vào là hợp lệ. Nhờ có câu lệnh `assert!(mid <= len)`, chúng ta đảm bảo rằng các con
trỏ luôn nằm trong phạm vi vùng nhớ của slice. Do đó, hàm `split_at_mut` hoàn toàn an toàn để gọi
từ bên ngoài mà không cần phải gắn nhãn `unsafe`.

Ngược lại, việc sử dụng `slice::from_raw_parts_mut` với một địa chỉ tùy ý như trong Listing 20-7
sẽ rất dễ gây crash chương trình khi slice được truy cập.

<Listing number="20-7" caption="Tạo một slice từ một vị trí bộ nhớ tùy ý">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Chúng ta không sở hữu vùng nhớ tại địa chỉ tùy ý này, và không có gì đảm bảo 10.000 phần tử đó là
các số `i32` hợp lệ. Thao tác này dẫn thẳng tới hành vi không xác định (undefined behavior).

#### Sử Dụng Các Hàm `extern` Để Gọi Mã Ngoại Lai (FFI)

Đôi khi, mã Rust cần tương tác với mã được viết bằng ngôn ngữ khác (chẳng hạn như C). Để làm điều
này, Rust cung cấp từ khóa `extern` để thiết lập _Foreign Function Interface (FFI)_ (Giao diện hàm
ngoại lai — tương tự như module `ctypes` hoặc `cffi` trong Python).

Listing 20-8 minh họa cách gọi hàm `abs` từ thư viện chuẩn C.

<Listing number="20-8" file-name="src/main.rs" caption="Khai báo và gọi một hàm `extern` được định nghĩa trong một ngôn ngữ khác">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

Các hàm khai báo trong khối `extern` theo mặc định là không an toàn khi gọi từ Rust, vì vậy khối
`extern` phải được đánh dấu là `unsafe extern`. Phần `"C"` chỉ định _Application Binary Interface (ABI)_
của C — định nghĩa cách truyền tham số và gọi hàm ở cấp độ assembly.

Tuy nhiên, một số hàm FFI như `abs` hoàn toàn không có nguy cơ mất an toàn bộ nhớ. Trong trường
hợp này, chúng ta có thể sử dụng từ khóa `safe` bên trong khối `unsafe extern` để đánh dấu rằng
hàm này an toàn khi gọi, giúp chúng ta gọi hàm mà không cần bọc trong khối `unsafe`, như hiển thị
trong Listing 20-9.

<Listing number="20-9" file-name="src/main.rs" caption="Đánh dấu rõ ràng một hàm là `safe` bên trong một khối `unsafe extern` và gọi nó một cách an toàn">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Lưu ý rằng việc đánh dấu `safe` là một lời cam kết của bạn với Rust. Bạn phải chịu trách nhiệm
đảm bảo hàm C đó thực sự an toàn.

#### Gọi Các Hàm Rust Từ Các Ngôn Ngữ Khác

Chúng ta cũng có thể dùng `extern` để cho phép các ngôn ngữ khác gọi hàm viết bằng Rust. Chúng ta
thêm từ khóa `extern "C"` trước `fn` và thêm thuộc tính `#[unsafe(no_mangle)]` để trình biên dịch
không làm biến đổi (mangle) tên hàm:

```rust
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

_Name mangling_ là quá trình trình biên dịch thay đổi tên hàm thành một chuỗi ký tự chứa thêm
thông tin nội bộ. Việc tắt mangling bằng `#[unsafe(no_mangle)]` là bắt buộc để ngôn ngữ khác (như C/Python)
có thể tìm thấy đúng tên hàm trong thư viện liên kết động (`.so` / `.dll`).

### Truy Cập Hoặc Sửa Đổi Một Biến Tĩnh Khả Biến (Mutable Static Variable)

Trong Rust, các biến toàn cục được gọi là các _biến tĩnh_ (static variables). Trong safe Rust,
việc có biến toàn cục khả biến là điều tối kỵ vì nếu nhiều luồng cùng truy cập và sửa đổi một biến
toàn cục, nó sẽ gây ra tình trạng tranh chấp dữ liệu (data race).

Listing 20-10 cho thấy cách khai báo một biến tĩnh bất biến:

<Listing number="20-10" file-name="src/main.rs" caption="Định nghĩa và sử dụng một biến tĩnh bất biến">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Tên biến tĩnh theo quy ước luôn viết hoa theo kiểu `SCREAMING_SNAKE_CASE` và bắt buộc phải có chú
thích kiểu. Biến tĩnh chỉ có thể chứa các tham chiếu có lifetime `'static`. Khác với hằng số (`const`)
có thể bị nhân bản dữ liệu ở mỗi nơi sử dụng, biến tĩnh có một địa chỉ bộ nhớ cố định duy nhất.

Khi một biến tĩnh được khai báo là khả biến (`static mut`), việc đọc hoặc ghi vào nó là **unsafe**.
Listing 20-11 minh họa cách khai báo và sửa đổi `COUNTER`.

<Listing number="20-11" file-name="src/main.rs" caption="Đọc từ hoặc ghi vào một biến tĩnh có thể thay đổi là unsafe">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Mọi thao tác đọc/ghi `COUNTER` đều phải đặt trong khối `unsafe` hoặc hàm `unsafe`. Ngoài ra, trình
biên dịch Rust mặc định sẽ từ chối việc tạo tham chiếu trực tiếp đến `static mut` thông qua lint
`static_mut_refs`. Bạn phải truy cập nó thông qua con trỏ thô được tạo bằng toán tử mượn thô
(`&raw mut` / `&raw const`) như trong Listing 20-11, hoặc tường minh tắt cảnh báo bằng
`#[allow(static_mut_refs)]`.

Theo quy ước chuẩn mực trong Rust (idiomatic Rust), bất cứ khi nào bạn viết một hàm hoặc thao tác
unsafe, hãy thêm một chú thích bắt đầu bằng `// SAFETY:` để giải thích rõ lý do vì sao thao tác này
được đảm bảo an toàn.

### Triển Khai Một Trait Unsafe

Một trait được coi là unsafe khi có ít nhất một phương thức hoặc quy ước bên trong nó chứa các điều
kiện bất biến (invariants) mà trình biên dịch không thể tự xác minh. Chúng ta khai báo trait là
`unsafe trait` và triển khai nó bằng cú pháp `unsafe impl`, như trong Listing 20-12.

<Listing number="20-12" caption="Định nghĩa và triển khai một trait unsafe">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

Ví dụ điển hình là hai marker trait `Send` và `Sync` trong lập trình đa luồng (Chương 16): trình
biên dịch tự động triển khai `Send` và `Sync` cho các kiểu dữ liệu nếu tất cả các trường bên trong nó
đều là `Send` và `Sync`. Nhưng nếu kiểu của bạn chứa con trỏ thô `*const T` (vốn không tự động là `Send`/`Sync`),
và bạn muốn khẳng định rằng kiểu của mình vẫn an toàn khi gửi qua các luồng, bạn bắt buộc phải dùng
`unsafe impl Send for MyType {}` và tự chịu trách nhiệm kiểm tra an toàn luồng.

### Truy Cập Các Trường Của Một `union`

Thao tác unsafe cuối cùng là truy cập các trường của một `union`. Một `union` tương tự như một `struct`,
nhưng tất cả các trường cùng chia sẻ chung một vùng nhớ duy nhất, và tại một thời điểm chỉ có một
trường được sử dụng. `union` chủ yếu được dùng khi tương tác với các cấu trúc union trong mã nguồn C.
Vì Rust không thể biết trường nào hiện đang thực sự chứa dữ liệu hợp lệ, việc đọc trường của một
`union` bắt buộc phải nằm trong khối `unsafe`.

### Sử Dụng Miri Để Kiểm Tra Mã Unsafe

Khi viết mã unsafe, làm thế nào để biết mã của bạn có thực sự an toàn và không gây ra lỗi ẩn (undefined
behavior)? Công cụ chính thức tốt nhất của Rust cho việc này là **Miri**.

Trong khi borrow checker là công cụ phân tích _tĩnh_ (static analysis) chạy lúc biên dịch, Miri là
công cụ phân tích _động_ (dynamic analysis) chạy lúc runtime. Nó chạy chương trình hoặc bộ kiểm thử
của bạn trong một môi trường giả lập để phát hiện các hành vi vi phạm quy tắc bộ nhớ (như dangling
pointer, out-of-bounds, data race).

Miri yêu cầu phiên bản Rust Nightly. Bạn có thể cài đặt bằng lệnh:

```console
$ rustup +nightly component add miri
```

Sau đó chạy kiểm tra dự án bằng:

```console
$ cargo +nightly miri run
$ cargo +nightly miri test
```

Ví dụ, khi chạy Miri trên đoạn mã trong Listing 20-7 (tạo slice từ địa chỉ con trỏ tùy ý):

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri sẽ lập tức cảnh báo việc ép kiểu số nguyên thành con trỏ và báo lỗi hành vi không xác định do
con trỏ lơ lửng (dangling pointer). Nhờ Miri, bạn có thể dễ dàng phát hiện các lỗi nguy hiểm mà
trình biên dịch thông thường không bắt được.

Tuy nhiên, lưu ý rằng Miri là công cụ động: nó chỉ kiểm tra những dòng mã thực sự được thực thi trong
quá trình chạy test. Do đó, bạn vẫn cần viết các bài test bao quát kết hợp với Miri để tăng độ tin cậy.

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### Sử Dụng Mã Unsafe Đúng Cách

Việc sử dụng `unsafe` không phải là điều cấm kỵ hay bị phản đối trong Rust; trên thực tế, nó là nền
tảng để xây dựng nên toàn bộ thư viện chuẩn và hệ sinh thái hiệu năng cao của Rust. Điểm mấu chốt là
hãy luôn bọc mã unsafe trong các trừu tượng an toàn (safe abstractions), giới hạn phạm vi khối `unsafe`
càng nhỏ càng tốt, luôn viết chú thích `// SAFETY:` giải thích rõ ràng, và sử dụng công cụ Miri để
kiểm tra.

Để tìm hiểu sâu hơn nữa về cách viết unsafe Rust an toàn và chuẩn xác, bạn có thể tham khảo cuốn sách
chính thức: [The Rustonomicon][nomicon].

{{#quiz ../quizzes/ch19-01-unsafe-rust.toml}}

[permission-violations]: ch04-02-references-and-borrowing.html#the-borrow-checker-finds-permission-violations
[ABI]: https://doc.rust-lang.org/reference/items/external-blocks.html#abi
[the-slice-type]: ch04-04-slices.html#the-slice-type
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[unions]: https://doc.rust-lang.org/reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
