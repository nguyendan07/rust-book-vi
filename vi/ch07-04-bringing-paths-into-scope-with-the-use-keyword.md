## Đưa các Đường dẫn vào Phạm vi với Từ khóa `use` (Bringing Paths into Scope with the `use` Keyword)

Việc phải viết ra toàn bộ các đường dẫn để gọi hàm có thể gây cảm giác bất tiện và
lặp đi lặp lại. Trong Listing 7-7, cho dù chúng ta chọn đường dẫn tuyệt đối hay tương đối đến
hàm `add_to_waitlist`, mỗi khi chúng ta muốn gọi `add_to_waitlist`,
chúng ta cũng phải chỉ định cả `front_of_house` và `hosting`. May mắn thay, có một
cách để đơn giản hóa quá trình này: chúng ta có thể tạo một lối tắt (shortcut) đến một đường dẫn với từ khóa `use`
một lần, và sau đó sử dụng cái tên ngắn hơn đó ở mọi nơi khác trong phạm vi (scope).

Trong Listing 7-11, chúng ta đưa module `crate::front_of_house::hosting` vào
phạm vi của hàm `eat_at_restaurant` để chúng ta chỉ cần chỉ định
`hosting::add_to_waitlist` để gọi hàm `add_to_waitlist` trong
`eat_at_restaurant`.

<Listing number="7-11" file-name="src/lib.rs" caption="Đưa một module vào phạm vi với `use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Thêm `use` và một đường dẫn trong một phạm vi tương tự như việc tạo một liên kết biểu tượng (symbolic link) trong
hệ thống tệp. Bằng cách thêm `use crate::front_of_house::hosting` vào crate
root, `hosting` giờ đây là một cái tên hợp lệ trong phạm vi đó, giống như thể module `hosting`
đã được định nghĩa trong crate root. Các đường dẫn được đưa vào phạm vi bằng `use`
cũng được kiểm tra tính riêng tư, giống như bất kỳ đường dẫn nào khác.

Lưu ý rằng `use` chỉ tạo lối tắt cho phạm vi cụ thể mà
`use` xuất hiện. Listing 7-12 di chuyển hàm `eat_at_restaurant` vào một
module con mới tên là `customer`, module này là một phạm vi khác với câu lệnh
`use`, vì vậy thân hàm sẽ không biên dịch được.

<Listing number="7-12" file-name="src/lib.rs" caption="Một câu lệnh `use` chỉ áp dụng trong phạm vi mà nó hiện diện.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

Lỗi trình biên dịch cho thấy lối tắt không còn áp dụng bên trong
module `customer` nữa:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Lưu ý rằng cũng có một cảnh báo rằng `use` không còn được sử dụng trong phạm vi của nó nữa! Để
khắc phục vấn đề này, hãy di chuyển `use` vào bên trong module `customer` luôn, hoặc tham chiếu
đến lối tắt trong module cha bằng `super::hosting` bên trong module con
`customer`.

### Tạo các Đường dẫn `use` theo Phong cách Đặc thù (Idiomatic)

Trong Listing 7-11, bạn có thể thắc mắc tại sao chúng ta chỉ định `use
crate::front_of_house::hosting` và sau đó gọi `hosting::add_to_waitlist` trong
`eat_at_restaurant`, thay vì chỉ định đường dẫn `use` đến tận
hàm `add_to_waitlist` để đạt được cùng một kết quả, như trong Listing 7-13.

<Listing number="7-13" file-name="src/lib.rs" caption="Đưa hàm `add_to_waitlist` vào phạm vi bằng `use`, điều này không đúng phong cách đặc thù">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Mặc dù cả Listing 7-11 và Listing 7-13 đều hoàn thành cùng một nhiệm vụ, Listing
7-11 là cách đặc thù (idiomatic) để đưa một hàm vào phạm vi với `use`. Việc đưa
module cha của hàm vào phạm vi với `use` có nghĩa là chúng ta phải chỉ định
module cha khi gọi hàm. Việc chỉ định module cha khi
gọi hàm giúp làm rõ rằng hàm đó không được định nghĩa tại địa phương
trong khi vẫn giảm thiểu việc lặp lại đường dẫn đầy đủ. Mã nguồn trong Listing 7-13 không
rõ ràng về nơi `add_to_waitlist` được định nghĩa.

Mặt khác, khi đưa vào các struct, enum và các mục khác bằng `use`,
phong cách đặc thù là chỉ định đường dẫn đầy đủ. Listing 7-14 cho thấy cách đặc thù
để đưa struct `HashMap` của thư viện tiêu chuẩn vào phạm vi của một binary
crate.

<Listing number="7-14" file-name="src/main.rs" caption="Đưa `HashMap` vào phạm vi theo cách đặc thù">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Không có lý do mạnh mẽ nào đằng sau phong cách đặc thù này: nó chỉ là quy ước đã
hình thành, và mọi người đã quen với việc đọc và viết mã Rust theo cách này.

Ngoại lệ cho phong cách đặc thù này là nếu chúng ta đưa hai mục có cùng tên
vào phạm vi bằng các câu lệnh `use`, vì Rust không cho phép điều đó. Listing 7-15
cho thấy cách đưa hai kiểu `Result` vào phạm vi có cùng tên nhưng
khác module cha, và cách tham chiếu đến chúng.

<Listing number="7-15" file-name="src/lib.rs" caption="Việc đưa hai kiểu có cùng tên vào cùng một phạm vi yêu cầu sử dụng các module cha của chúng.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Như bạn có thể thấy, việc sử dụng các module cha giúp phân biệt hai kiểu `Result`.
Nếu thay vào đó chúng ta chỉ định `use std::fmt::Result` và `use std::io::Result`, chúng ta sẽ
có hai kiểu `Result` trong cùng một phạm vi, và Rust sẽ không biết chúng ta
ám chỉ cái nào khi chúng ta sử dụng `Result`.

### Cung cấp Tên mới với Từ khóa `as`

Có một giải pháp khác cho vấn đề đưa hai kiểu cùng tên
vào cùng một phạm vi với `use`: sau đường dẫn, chúng ta có thể chỉ định `as` và một
tên cục bộ mới, hoặc _bí danh_ (alias), cho kiểu đó. Listing 7-16 cho thấy một cách khác để viết
mã nguồn trong Listing 7-15 bằng cách đổi tên một trong hai kiểu `Result` bằng `as`.

<Listing number="7-16" file-name="src/lib.rs" caption="Đổi tên một kiểu khi nó được đưa vào phạm vi bằng từ khóa `as`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

Trong câu lệnh `use` thứ hai, chúng ta đã chọn tên mới `IoResult` cho
kiểu `std::io::Result`, tên này sẽ không xung đột với `Result` từ `std::fmt`
mà chúng ta cũng đã đưa vào phạm vi. Cả Listing 7-15 và Listing 7-16 đều được
coi là đặc thù, vì vậy sự lựa chọn là tùy thuộc vào bạn!

### Tái xuất các Tên với `pub use`

Khi chúng ta đưa một cái tên vào phạm vi với từ khóa `use`, cái tên đó là riêng tư đối với
phạm vi mà chúng ta đã nhập nó vào. Để cho phép mã nguồn bên ngoài phạm vi đó tham chiếu
đến cái tên đó như thể nó đã được định nghĩa trong phạm vi đó, chúng ta có thể kết hợp `pub` và
`use`. Kỹ thuật này được gọi là _tái xuất_ (re-exporting) vì chúng ta đang đưa một mục vào
phạm vi nhưng cũng làm cho mục đó có sẵn để những người khác đưa vào phạm vi của họ.

Listing 7-17 cho thấy mã nguồn trong Listing 7-11 với `use` trong module gốc
được đổi thành `pub use`.

<Listing number="7-17" file-name="src/lib.rs" caption="Làm cho một cái tên có sẵn cho bất kỳ mã nguồn nào sử dụng từ một phạm vi mới với `pub use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Trước thay đổi này, mã nguồn bên ngoài sẽ phải gọi hàm `add_to_waitlist`
bằng cách sử dụng đường dẫn
`restaurant::front_of_house::hosting::add_to_waitlist()`, điều này cũng sẽ
yêu cầu module `front_of_house` phải được đánh dấu là `pub`. Bây giờ khi `pub
use` này đã tái xuất module `hosting` từ module gốc, mã nguồn bên ngoài
có thể sử dụng đường dẫn `restaurant::hosting::add_to_waitlist()` thay thế.

Tái xuất rất hữu ích khi cấu trúc nội bộ của mã nguồn của bạn khác
với cách các lập trình viên gọi mã của bạn sẽ nghĩ về lĩnh vực đó. Ví
dụ, trong phép ẩn dụ về nhà hàng này, những người điều hành nhà hàng nghĩ
về “phía trước” (front of house) và “phía sau” (back of house). Nhưng những khách hàng đến thăm một nhà hàng
có lẽ sẽ không nghĩ về các bộ phận của nhà hàng theo những thuật ngữ đó. Với `pub
use`, chúng ta có thể viết mã của mình với một cấu trúc nhưng để lộ một cấu trúc khác.
Làm như vậy giúp thư viện của chúng ta được tổ chức tốt cho các lập trình viên làm việc trên thư viện
và các lập trình viên gọi thư viện. Chúng ta sẽ xem xét một ví dụ khác về `pub use`
và cách nó ảnh hưởng đến tài liệu crate của bạn trong [“Xuất một API Công khai Tiện lợi
với `pub use`”][ch14-pub-use]<!-- ignore --> ở Chương 14.

### Sử dụng các Package bên ngoài

Trong Chương 2, chúng ta đã lập trình một dự án trò chơi đoán số có sử dụng một
package bên ngoài gọi là `rand` để lấy các số ngẫu nhiên. Để sử dụng `rand` trong dự án của chúng ta, chúng ta
đã thêm dòng này vào _Cargo.toml_:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

Việc thêm `rand` làm phụ thuộc trong _Cargo.toml_ bảo Cargo tải xuống
package `rand` và bất kỳ phụ thuộc nào từ [crates.io](https://crates.io/) và
làm cho `rand` có sẵn cho dự án của chúng ta.

Sau đó, để đưa các định nghĩa của `rand` vào phạm vi package của chúng ta, chúng ta đã thêm một
dòng `use` bắt đầu bằng tên của crate, `rand`, và liệt kê các mục chúng ta
muốn đưa vào phạm vi. Hãy nhớ lại rằng trong [“Tạo một Số ngẫu
nhiên”][rand]<!-- ignore --> ở Chương 2, chúng ta đã đưa trait `Rng` vào
phạm vi và gọi hàm `rand::thread_rng`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Các thành viên của cộng đồng Rust đã tạo ra nhiều package có sẵn tại
[crates.io](https://crates.io/), và việc kéo bất kỳ package nào trong số chúng vào package của bạn
đều bao gồm các bước tương tự: liệt kê chúng trong file _Cargo.toml_ của package của bạn và
sử dụng `use` để đưa các mục từ crate của chúng vào phạm vi.

Lưu ý rằng thư viện tiêu chuẩn `std` cũng là một crate nằm bên ngoài
package của chúng ta. Bởi vì thư viện tiêu chuẩn được đi kèm với ngôn ngữ Rust, chúng ta
không cần thay đổi _Cargo.toml_ để bao gồm `std`. Nhưng chúng ta cần tham chiếu đến
nó bằng `use` để đưa các mục từ đó vào phạm vi package của chúng ta. Ví dụ,
với `HashMap` chúng ta sẽ sử dụng dòng này:

```rust
use std::collections::HashMap;
```

Đây là một đường dẫn tuyệt đối bắt đầu bằng `std`, tên của crate thư viện
tiêu chuẩn.

### Sử dụng các Đường dẫn Lồng nhau để Làm gọn các Danh sách `use` Lớn

Nếu chúng ta đang sử dụng nhiều mục được định nghĩa trong cùng một crate hoặc cùng một module, việc liệt kê
từng mục trên dòng riêng của nó có thể chiếm nhiều không gian theo chiều dọc trong các file của chúng ta. Ví
dụ, hai câu lệnh `use` này chúng ta đã có trong trò chơi đoán số trong Listing 2-4
đưa các mục từ `std` vào phạm vi:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Thay vào đó, chúng ta có thể sử dụng các đường dẫn lồng nhau để đưa các mục tương tự vào phạm vi trong một
dòng. Chúng ta làm điều này bằng cách chỉ định phần chung của đường dẫn, theo sau là hai
dấu hai chấm, và sau đó là các dấu ngoặc nhọn bao quanh danh sách các phần của đường dẫn mà
khác nhau, như được hiển thị trong Listing 7-18.

<Listing number="7-18" file-name="src/main.rs" caption="Chỉ định một đường dẫn lồng nhau để đưa nhiều mục có cùng tiền tố vào phạm vi">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Trong các chương trình lớn hơn, việc đưa nhiều mục vào phạm vi từ cùng một crate hoặc
module bằng cách sử dụng các đường dẫn lồng nhau có thể làm giảm đáng kể số lượng các câu lệnh `use` riêng biệt
cần thiết!

Chúng ta có thể sử dụng một đường dẫn lồng nhau ở bất kỳ cấp độ nào trong một đường dẫn, điều này hữu ích khi kết hợp
hai câu lệnh `use` có chung một đường dẫn con. Ví dụ, Listing 7-19 cho thấy hai
câu lệnh `use`: một câu lệnh đưa `std::io` vào phạm vi và một câu lệnh đưa
`std::io::Write` vào phạm vi.

<Listing number="7-19" file-name="src/lib.rs" caption="Hai câu lệnh `use` trong đó một câu lệnh là đường dẫn con của câu lệnh kia">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

Phần chung của hai đường dẫn này là `std::io`, và đó là đường dẫn đầu tiên
đầy đủ. Để hợp nhất hai đường dẫn này vào một câu lệnh `use`, chúng ta có thể sử dụng `self` trong
đường dẫn lồng nhau, như được hiển thị trong Listing 7-20.

<Listing number="7-20" file-name="src/lib.rs" caption="Kết hợp các đường dẫn trong Listing 7-19 vào một câu lệnh `use`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Dòng này đưa cả `std::io` và `std::io::Write` vào phạm vi.

### Toán tử Glob

Nếu chúng ta muốn đưa _tất cả_ các mục công khai được định nghĩa trong một đường dẫn vào phạm vi, chúng ta có thể
chỉ định đường dẫn đó theo sau là toán tử glob `*`:

```rust
use std::collections::*;
```

Câu lệnh `use` này đưa tất cả các mục công khai được định nghĩa trong `std::collections` vào
phạm vi hiện tại. Hãy cẩn thận khi sử dụng toán tử glob! Glob có thể làm cho nó
khó khăn hơn để biết tên nào đang nằm trong phạm vi và nơi một cái tên được sử dụng trong chương trình của bạn
được định nghĩa. Ngoài ra, nếu phụ thuộc thay đổi các định nghĩa của nó, những gì
bạn đã nhập cũng thay đổi theo, điều này có thể dẫn đến lỗi trình biên dịch khi bạn
nâng cấp phụ thuộc nếu phụ thuộc đó thêm một định nghĩa có cùng tên
với một định nghĩa của bạn trong cùng một phạm vi, chẳng hạn.

Toán tử glob thường được sử dụng khi kiểm thử (testing) để đưa mọi thứ đang được kiểm thử vào
module `tests`; chúng ta sẽ nói về điều đó trong [“Cách Viết
Kiểm thử”][writing-tests]<!-- ignore --> ở Chương 11. Toán tử glob đôi khi cũng
được sử dụng như một phần của mô hình prelude: xem [tài liệu thư viện
tiêu chuẩn](../std/prelude/index.html#other-preludes)<!-- ignore --> để biết thêm
thông tin về mô hình đó.

{{#quiz ../quizzes/ch07-04-use.toml}}

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number

## [writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
