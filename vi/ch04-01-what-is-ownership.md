## Quyền Sở Hữu Là Gì?

Quyền sở hữu (Ownership) là một kỷ luật để đảm bảo **sự an toàn** của các chương trình Rust. Để hiểu về quyền sở hữu, trước tiên chúng ta cần hiểu điều gì làm cho một chương trình Rust an toàn (hoặc không an toàn).

### An Toàn Là Sự Vắng Mặt Của Hành Vi Chưa Xác Định

Hãy bắt đầu với một ví dụ. Chương trình này an toàn để thực thi:

```rust
fn read(y: bool) {
    if y {
        println!("y is true!");
    }
}

fn main() {
    let x = true;
    read(x);
}
```

Chúng ta có thể làm cho chương trình này không an toàn để thực thi bằng cách di chuyển lời gọi hàm `read` lên trước phần định nghĩa của `x`:

```rust,ignore,does_not_compile
fn read(y: bool) {
    if y {
        println!("y is true!");
    }
}

fn main() {
    read(x); // ôi không! x chưa được định nghĩa!
    let x = true;
}
```

> _Lưu ý_: trong chương này, chúng tôi sẽ sử dụng nhiều ví dụ mã không biên dịch được. Hãy nhớ tìm kiếm chú cua dấu hỏi nếu bạn không chắc chắn liệu một chương trình có nên biên dịch hay không.

Chương trình thứ hai này không an toàn vì `read(x)` mong đợi `x` có một giá trị thuộc kiểu `bool`, nhưng `x` lại chưa có giá trị nào.

Khi một chương trình như thế này được thực thi bởi một trình thông dịch, thì việc đọc `x` trước khi nó được định nghĩa sẽ gây ra một ngoại lệ (exception) như [`NameError`] của Python hay [`ReferenceError`] của Javascript. Nhưng các ngoại lệ đi kèm với một cái giá. Mỗi khi một chương trình thông dịch đọc một biến, trình thông dịch phải kiểm tra xem biến đó đã được định nghĩa hay chưa.

Mục tiêu của Rust là biên dịch các chương trình thành các tệp nhị phân hiệu quả yêu cầu càng ít kiểm tra tại thời gian chạy (runtime) càng tốt. Do đó, Rust không kiểm tra tại _thời gian chạy_ xem một biến đã được định nghĩa hay chưa trước khi sử dụng. Thay vào đó, Rust kiểm tra tại _thời gian biên dịch_ (compile-time). Nếu bạn cố gắng biên dịch chương trình không an toàn, bạn sẽ nhận được lỗi này:

```text
error[E0425]: cannot find value `x` in this scope
 --> src/main.rs:8:10
  |
8 |     read(x); // oh no! x isn't defined!
  |          ^ not found in this scope
```

Bạn có thể có trực giác rằng việc Rust đảm bảo các biến được định nghĩa trước khi chúng được sử dụng là điều tốt. Nhưng tại sao? Để biện minh cho quy tắc này, chúng ta phải đặt câu hỏi: **điều gì sẽ xảy ra nếu Rust cho phép biên dịch một chương trình bị từ chối?**

Trước tiên hãy xem xét cách chương trình an toàn biên dịch và thực thi. Trên một máy tính có bộ vi xử lý sử dụng kiến trúc [x86](https://en.wikipedia.org/wiki/X86), Rust tạo ra mã hợp ngữ (assembly) sau cho hàm `main` trong chương trình an toàn ([xem mã hợp ngữ đầy đủ tại đây](https://rust.godbolt.org/z/xnT1fzsqv)):

```x86asm
main:
    ; ...
    mov     edi, 1
    call    read
    ; ...
```

> _Lưu ý_: nếu bạn không quen thuộc với mã hợp ngữ, không sao cả! Phần này chứa một vài ví dụ về hợp ngữ chỉ để cho bạn thấy cách Rust thực sự hoạt động "bên dưới mui xe". Bạn thường không cần biết về hợp ngữ để hiểu Rust.

Mã hợp ngữ này sẽ:

-   Di chuyển số 1, đại diện cho `true`, vào một "thanh ghi" (register - một loại biến của hợp ngữ) được gọi là `edi`.
-   Gọi hàm `read`, hàm này mong đợi đối số đầu tiên `y` của nó nằm trong thanh ghi `edi`.

Nếu hàm không an toàn được phép biên dịch, mã hợp ngữ của nó có thể trông như thế này:

```x86asm
main:
    ; ...
    call    read
    mov     edi, 1    ; mov nằm sau call
    ; ...
```

Chương trình này không an toàn vì `read` sẽ mong đợi `edi` là một boolean, tức là số `0` hoặc `1`. Nhưng `edi` có thể là bất cứ thứ gì: `2`, `100`, `0x1337BEEF`. Khi `read` muốn sử dụng đối số `y` của nó cho bất kỳ mục đích nào, nó sẽ ngay lập tức gây ra _**UNDEFINED BEHAVIOR (HÀNH VI CHƯA XÁC ĐỊNH)!**_

Rust không quy định điều gì xảy ra nếu bạn cố gắng chạy `if y { .. }` khi `y` không phải là `true` hay `false`. _Hành vi_ đó, hay những gì xảy ra sau khi thực thi lệnh, là _chưa xác định_. Một điều gì đó sẽ xảy ra, ví dụ:

-   Mã thực thi mà không gặp sự cố, và không ai nhận thấy vấn đề gì.
-   Mã ngay lập tức gặp sự cố do [lỗi phân đoạn](https://en.wikipedia.org/wiki/Segmentation_fault) (segmentation fault) hoặc một loại lỗi hệ điều hành khác.
-   Mã thực thi mà không gặp sự cố, cho đến khi một tác nhân độc hại tạo ra đầu vào thích hợp để xóa cơ sở dữ liệu production, ghi đè các bản sao lưu và đánh cắp tiền ăn trưa của bạn.

**Một mục tiêu nền tảng của Rust là đảm bảo rằng các chương trình của bạn không bao giờ có hành vi chưa xác định.** Đó là ý nghĩa của "sự an toàn". Hành vi chưa xác định đặc biệt nguy hiểm đối với các chương trình cấp thấp có quyền truy cập trực tiếp vào bộ nhớ. Khoảng [70% các lỗ hổng bảo mật được báo cáo](https://msrc.microsoft.com/blog/2019/07/a-proactive-approach-to-more-secure-code/) trong các hệ thống cấp thấp là do hỏng bộ nhớ (memory corruption), đây là một dạng của hành vi chưa xác định.

Một mục tiêu thứ hai của Rust là ngăn chặn hành vi chưa xác định tại _thời gian biên dịch_ thay vì _thời gian chạy_. Mục tiêu này có hai động lực:

1. Bắt lỗi tại thời gian biên dịch đồng nghĩa với việc tránh được những lỗi đó trong môi trường production, cải thiện độ tin cậy của phần mềm.
2. Bắt lỗi tại thời gian biên dịch đồng nghĩa với việc ít phải kiểm tra lỗi đó khi chạy hơn, cải thiện hiệu suất của phần mềm.

Rust không thể ngăn chặn tất cả các lỗi. Nếu một ứng dụng để lộ công khai một endpoint `/delete-production-database` không cần xác thực, thì một tác nhân độc hại không cần một câu lệnh if đáng ngờ để xóa cơ sở dữ liệu. Nhưng các biện pháp bảo vệ của Rust vẫn có khả năng làm cho các chương trình an toàn hơn so với việc sử dụng một ngôn ngữ có ít biện pháp bảo vệ hơn, ví dụ như được tìm thấy bởi [đội ngũ Android của Google](https://security.googleblog.com/2022/12/memory-safe-languages-in-android-13.html).

### Quyền Sở Hữu Như Là Một Kỷ Luật Cho An Toàn Bộ Nhớ

Vì an toàn là sự vắng mặt của hành vi chưa xác định, và vì quyền sở hữu nói về sự an toàn, nên chúng ta cần hiểu quyền sở hữu dưới góc độ của các hành vi chưa xác định mà nó ngăn chặn. Tài liệu tham khảo Rust (Rust Reference) duy trì một danh sách lớn các ["Hành vi được coi là chưa xác định"](https://doc.rust-lang.org/reference/behavior-considered-undefined.html). Hiện tại, chúng ta sẽ tập trung vào một danh mục: các thao tác trên bộ nhớ.

Bộ nhớ là không gian nơi dữ liệu được lưu trữ trong quá trình thực thi của một chương trình. Có nhiều cách để suy nghĩ về bộ nhớ:

-   Nếu bạn chưa quen với lập trình hệ thống, bạn có thể nghĩ về bộ nhớ ở mức cao như "bộ nhớ là RAM trong máy tính của tôi" hoặc "bộ nhớ là thứ sẽ hết nếu tôi tải quá nhiều dữ liệu".
-   Nếu bạn đã quen thuộc với lập trình hệ thống, bạn có thể nghĩ về bộ nhớ ở mức thấp như "bộ nhớ là một mảng các byte" hoặc "bộ nhớ là các con trỏ tôi nhận lại từ `malloc`".

Cả hai mô hình bộ nhớ này đều _hợp lệ_, nhưng chúng không phải là những cách _hữu ích_ để suy nghĩ về cách Rust hoạt động. Mô hình cấp cao quá trừu tượng để giải thích cách Rust hoạt động. Ví dụ, bạn sẽ cần hiểu khái niệm về con trỏ. Mô hình cấp thấp lại quá cụ thể để giải thích cách Rust hoạt động. Ví dụ, Rust không cho phép bạn diễn giải bộ nhớ như một mảng các byte.

Rust cung cấp một cách cụ thể để suy nghĩ về bộ nhớ. Quyền sở hữu là một kỷ luật để sử dụng bộ nhớ một cách an toàn theo lối suy nghĩ đó. Phần còn lại của chương này sẽ giải thích mô hình bộ nhớ của Rust.

### Variables Live in the Stack

Dưới đây là một chương trình giống như chương trình bạn đã thấy trong Mục 3.3, định nghĩa một số `n` và gọi hàm `plus_one` trên `n`. Bên dưới chương trình là một loại sơ đồ mới. Sơ đồ này trực quan hóa nội dung của bộ nhớ trong quá trình thực thi chương trình tại ba điểm được đánh dấu.

```aquascope,interpreter,horizontal
fn main() {
    let n = 5;`[]`
    let y = plus_one(n);`[]`
    println!("The value of y is: {y}");
}

fn plus_one(x: i32) -> i32 {
    `[]`x + 1
}
```

Các biến sống trong các **khung** (frame). Một khung là một ánh xạ từ các biến tới các giá trị trong một phạm vi (scope) duy nhất, chẳng hạn như một hàm. Ví dụ:

-   Khung cho `main` tại vị trí L1 chứa `n = 5`.
-   Khung cho `plus_one` tại L2 chứa `x = 5`.
-   Khung cho `main` tại vị trí L3 chứa `n = 5; y = 6`.

Các khung được tổ chức thành một **ngăn xếp** (stack) của các hàm hiện đang được gọi. Ví dụ, tại L2, khung cho `main` nằm trên khung cho hàm được gọi `plus_one`. Sau khi một hàm trả về, Rust sẽ thu hồi khung của hàm đó. (Việc thu hồi - Deallocation còn được gọi là **giải phóng** - freeing hoặc **thả** - dropping, và chúng tôi sử dụng các thuật ngữ đó thay thế cho nhau.) Chuỗi các khung này được gọi là một ngăn xếp (stack) vì khung được thêm vào gần đây nhất luôn là khung tiếp theo được giải phóng.

> _Lưu ý:_ mô hình bộ nhớ này không mô tả đầy đủ cách Rust thực sự hoạt động! Như chúng ta đã thấy trước đó với mã hợp ngữ, trình biên dịch Rust có thể đặt `n` hoặc `x` vào một thanh ghi thay vì một khung stack. Nhưng sự phân biệt đó là một chi tiết cài đặt. Nó không nên làm thay đổi hiểu biết của bạn về sự an toàn trong Rust, vì vậy chúng ta có thể tập trung vào trường hợp đơn giản hơn là các biến chỉ nằm trong khung.

Khi một biểu thức đọc một biến, giá trị của biến đó được sao chép từ khe của nó trong khung stack. Ví dụ, nếu chúng ta chạy chương trình này:

```aquascope,interpreter,horizontal
#fn main() {
let a = 5;`[]`
let mut b = a;`[]`
b += 1;`[]`
#}
```

Giá trị của `a` được sao chép vào `b`, và `a` vẫn không thay đổi, ngay cả sau khi thay đổi `b`.

### Boxes Live in the Heap

Tuy nhiên, việc sao chép dữ liệu có thể chiếm nhiều bộ nhớ. Ví dụ, đây là một chương trình hơi khác. Chương trình này sao chép một mảng có 1 triệu phần tử:

```aquascope,interpreter
#fn main() {
let a = [0; 1_000_000];`[]`
let b = a;`[]`
#}
```

Hãy quan sát rằng việc sao chép `a` vào `b` khiến khung `main` chứa 2 triệu phần tử.

Để chuyển quyền truy cập dữ liệu mà không cần sao chép nó, Rust sử dụng các **con trỏ** (pointer). Một con trỏ là một giá trị mô tả một vị trí trong bộ nhớ. Giá trị mà một con trỏ trỏ tới được gọi là **pointee** của nó. Một cách phổ biến để tạo con trỏ là cấp phát bộ nhớ trong **heap**. Heap là một vùng bộ nhớ riêng biệt nơi dữ liệu có thể sống vô thời hạn. Dữ liệu trên heap không gắn liền với một khung stack cụ thể nào. Rust cung cấp một cấu trúc gọi là [`Box`](https://doc.rust-lang.org/std/boxed/index.html) để đặt dữ liệu lên heap. Ví dụ, chúng ta có thể bọc mảng một triệu phần tử trong `Box::new` như thế này:

```aquascope,interpreter
#fn main() {
let a = Box::new([0; 1_000_000]);`[]`
let b = a;`[]`
#}
```

Hãy quan sát rằng bây giờ, chỉ có duy nhất một mảng tại một thời điểm. Tại L1, giá trị của `a` là một con trỏ (được biểu diễn bằng dấu chấm với mũi tên) tới mảng bên trong heap. Câu lệnh `let b = a` sao chép con trỏ từ `a` sang `b`, nhưng dữ liệu được trỏ tới không bị sao chép. Lưu ý rằng `a` bây giờ bị làm mờ vì nó đã bị _di chuyển_ (moved) &mdash; chúng ta sẽ xem điều đó có nghĩa là gì trong chốc lát.

{{#quiz ../quizzes/ch04-01-ownership-sec1-stackheap.toml}}

### Rust Không Cho Phép Quản Lý Bộ Nhớ Thủ Công

Quản lý bộ nhớ là quá trình cấp phát bộ nhớ và thu hồi bộ nhớ. Nói cách khác, đó là quá trình tìm bộ nhớ không sử dụng và sau đó trả lại bộ nhớ đó khi nó không còn được sử dụng nữa. Các khung stack được Rust tự động quản lý. Khi một hàm được gọi, Rust cấp phát một khung stack cho hàm được gọi. Khi lời gọi kết thúc, Rust thu hồi khung stack đó.

Như chúng ta đã thấy ở trên, dữ liệu heap được cấp phát khi gọi `Box::new(..)`. Nhưng khi nào dữ liệu heap được thu hồi? Hãy tưởng tượng rằng Rust có một hàm `free()` dùng để giải phóng một cấp phát heap. Hãy tưởng tượng rằng Rust cho phép lập trình viên gọi `free` bất cứ khi nào họ muốn. Kiểu quản lý bộ nhớ "thủ công" này dễ dẫn đến lỗi. Ví dụ, chúng ta có thể đọc một con trỏ tới bộ nhớ đã được giải phóng:

```aquascope,interpreter,shouldFail
#fn free<T>(_t: T) {}
#fn main() {
let b = Box::new([0; 100]);`[]`
free(b);`[]`
assert!(b[0] == 0);`[]`
#}
```

> _Lưu ý:_ bạn có thể thắc mắc làm thế nào chúng tôi đang thực thi chương trình Rust không biên dịch được này. Chúng tôi sử dụng [các công cụ đặc biệt](https://github.com/cognitive-engineering-lab/aquascope) để mô phỏng Rust như thể bộ kiểm tra mượn (borrow checker) đã bị tắt, cho mục đích giáo dục. Bằng cách đó, chúng ta có thể trả lời các câu hỏi giả định, như: điều gì sẽ xảy ra nếu Rust cho phép biên dịch chương trình không an toàn này?

Ở đây, chúng ta cấp phát một mảng trên heap. Sau đó chúng ta gọi `free(b)`, lệnh này thu hồi bộ nhớ heap của `b`. Do đó, giá trị của `b` là một con trỏ tới bộ nhớ không hợp lệ, thứ mà chúng ta biểu diễn bằng biểu tượng "⦻". Chưa có hành vi chưa xác định nào xảy ra! Chương trình vẫn an toàn tại L2. Việc có một con trỏ không hợp lệ không nhất thiết là một vấn đề.

Hành vi chưa xác định xảy ra khi chúng ta cố gắng _sử dụng_ con trỏ bằng cách đọc `b[0]`. Điều đó sẽ cố gắng truy cập bộ nhớ không hợp lệ, có thể khiến chương trình bị crash (sự cố). Hoặc tệ hơn, nó có thể không crash và trả về dữ liệu tùy ý. Do đó chương trình này là **không an toàn**.

Rust không cho phép các chương trình thu hồi bộ nhớ một cách thủ công. Chính sách đó tránh được các loại hành vi chưa xác định được hiển thị ở trên.

### Chủ Sở Hữu Của Box Quản Lý Việc Thu Hồi

Thay vào đó, Rust _tự động_ giải phóng bộ nhớ heap của box. Đây là mô tả _gần_ đúng về chính sách của Rust cho việc giải phóng box:

> **Nguyên tắc thu hồi Box (gần đúng):** Nếu một biến được liên kết với một box, khi Rust thu hồi khung của biến đó, thì Rust sẽ thu hồi bộ nhớ heap của box đó.

Ví dụ, hãy theo dõi một chương trình cấp phát và giải phóng một box:

```aquascope,interpreter,horizontal
fn main() {
    let a_num = 4;`[]`
    make_and_drop();`[]`
}

fn make_and_drop() {
    let a_box = Box::new(5);`[]`
}
```

Tại L1, trước khi gọi `make_and_drop`, trạng thái của bộ nhớ chỉ là khung stack cho `main`. Sau đó tại L2, trong khi gọi `make_and_drop`, `a_box` trỏ tới `5` trên heap. Khi `make_and_drop` kết thúc, Rust thu hồi khung stack của nó. `make_and_drop` chứa biến `a_box`, vì vậy Rust cũng thu hồi dữ liệu heap trong `a_box`. Do đó heap trống rỗng tại L3.

Bộ nhớ heap của box đã được quản lý thành công. Nhưng điều gì sẽ xảy ra nếu chúng ta lạm dụng hệ thống này? Quay trở lại ví dụ trước đó của chúng ta, điều gì xảy ra khi chúng ta liên kết hai biến với cùng một box?

```rust,ignore
# fn main() {
let a = Box::new([0; 1_000_000]);
let b = a;
# }
```

Mảng trong box bây giờ đã được liên kết với cả `a` và `b`. Theo nguyên tắc "gần đúng" của chúng ta, Rust sẽ cố gắng giải phóng bộ nhớ heap của box _hai lần_ thay mặt cho cả hai biến. Đó cũng là hành vi chưa xác định!

Để tránh tình huống này, cuối cùng chúng ta đi đến quyền sở hữu (ownership). Khi `a` được liên kết với `Box::new([0; 1_000_000])`, chúng ta nói rằng `a` **sở hữu** box đó. Câu lệnh `let b = a` **di chuyển** (move) quyền sở hữu box từ `a` sang `b`. Với các khái niệm này, chính sách của Rust cho việc giải phóng box được mô tả chính xác hơn là:

> **Nguyên tắc thu hồi Box (hoàn toàn chính xác):** Nếu một biến sở hữu một box, khi Rust thu hồi khung của biến đó, thì Rust sẽ thu hồi bộ nhớ heap của box đó.

Trong ví dụ trên, `b` sở hữu mảng trong box. Do đó khi phạm vi kết thúc, Rust chỉ thu hồi box một lần thay mặt cho `b`, không phải `a`.

### Các Bộ Sưu Tập Sử Dụng Box

Các Box được sử dụng bởi các cấu trúc dữ liệu của Rust[^boxed-data-structures] như [`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html), [`String`](https://doc.rust-lang.org/std/string/struct.String.html), và [`HashMap`](https://doc.rust-lang.org/std/collections/struct.HashMap.html) để chứa số lượng phần tử thay đổi. Ví dụ, đây là một chương trình tạo, di chuyển, và thay đổi một chuỗi (string):

```aquascope,interpreter,horizontal
fn main() {
    let first = String::from("Ferris");`[]`
    let full = add_suffix(first);`[]`
    println!("{full}");
}

fn add_suffix(mut name: String) -> String {
    `[]`name.push_str(" Jr.");`[]`
    name
}
```

Chương trình này phức tạp hơn, vì vậy hãy chắc chắn bạn theo dõi từng bước:

1. Tại L1, chuỗi "Ferris" đã được cấp phát trên heap. Nó được sở hữu bởi `first`.
2. Tại L2, hàm `add_suffix(first)` đã được gọi. Điều này di chuyển quyền sở hữu của chuỗi từ `first` sang `name`. Dữ liệu chuỗi không bị sao chép, nhưng con trỏ tới dữ liệu được sao chép.
3. Tại L3, hàm `name.push_str(" Jr.")` thay đổi kích thước cấp phát heap của chuỗi. Điều này làm ba việc. Đầu tiên, nó tạo ra một cấp phát mới lớn hơn. Thứ hai, nó ghi "Ferris Jr." vào cấp phát mới. Thứ ba, nó giải phóng bộ nhớ heap ban đầu. `first` bây giờ trỏ tới bộ nhớ đã được giải phóng.
4. Tại L4, khung cho `add_suffix` đã biến mất. Hàm này đã trả về `name`, chuyển quyền sở hữu chuỗi cho `full`.

### Các Biến Không Thể Được Sử Dụng Sau Khi Bị Di Chuyển

Chương trình chuỗi giúp minh họa một nguyên tắc an toàn chính cho quyền sở hữu. Hãy tưởng tượng rằng `first` được sử dụng trong `main` sau khi gọi `add_suffix`. Chúng ta có thể mô phỏng một chương trình như vậy và xem hành vi chưa xác định xảy ra:

```aquascope,interpreter,shouldFail
fn main() {
    let first = String::from("Ferris");
    let full = add_suffix(first);
    println!("{full}, originally {first}");`[]` // first bây giờ được sử dụng ở đây
}

fn add_suffix(mut name: String) -> String {
    name.push_str(" Jr.");
    name
}
```

`first` trỏ tới bộ nhớ đã được giải phóng sau khi gọi `add_suffix`. Việc đọc `first` trong `println!` do đó sẽ là một vi phạm an toàn bộ nhớ (hành vi chưa xác định). Hãy nhớ: việc `first` trỏ tới bộ nhớ đã được giải phóng không phải là vấn đề. Vấn đề là chúng ta đã cố gắng _sử dụng_ `first` sau khi nó trở nên không hợp lệ.

Rất may, Rust sẽ từ chối biên dịch chương trình này, đưa ra lỗi sau:

```text
error[E0382]: borrow of moved value: `first`
 --> test.rs:4:35
  |
2 |     let first = String::from("Ferris");
  |         ----- move occurs because `first` has type `String`, which does not implement the `Copy` trait
3 |     let full = add_suffix(first);
  |                           ----- value moved here
4 |     println!("{full}, originally {first}"); // first is now used here
  |                                   ^^^^^ value borrowed here after move
```

Hãy đi qua các bước của lỗi này. Rust nói rằng `first` bị di chuyển khi chúng ta gọi `add_suffix(first)` ở dòng 3. Lỗi làm rõ rằng `first` bị di chuyển vì nó có kiểu `String`, kiểu này không triển khai trait `Copy`. Chúng ta sẽ thảo luận về `Copy` sớm &mdash; tóm lại, bạn sẽ không gặp lỗi này nếu bạn sử dụng `i32` thay vì `String`. Cuối cùng, lỗi nói rằng chúng ta sử dụng `first` sau khi bị di chuyển (nó được "mượn" - borrowed, điều mà chúng ta sẽ thảo luận trong phần tiếp theo).

Vì vậy, nếu bạn di chuyển một biến, Rust sẽ ngăn bạn sử dụng biến đó sau này. Nói rộng hơn, trình biên dịch sẽ thực thi nguyên tắc này:

> **Nguyên tắc dữ liệu heap bị di chuyển:** nếu một biến `x` di chuyển quyền sở hữu dữ liệu heap cho một biến `y` khác, thì `x` không thể được sử dụng sau khi di chuyển.

Bây giờ bạn sẽ bắt đầu thấy mối quan hệ giữa quyền sở hữu, di chuyển, và sự an toàn. Việc di chuyển quyền sở hữu dữ liệu heap giúp tránh hành vi chưa xác định từ việc đọc bộ nhớ đã được giải phóng.

### Cloning Tránh Việc Di Chuyển

Một cách để tránh di chuyển dữ liệu là _clone_ (tạo bản sao) nó bằng cách sử dụng phương thức `.clone()`. Ví dụ, chúng ta có thể sửa vấn đề an toàn trong chương trình trước bằng một lần clone:

```aquascope,interpreter
fn main() {
    let first = String::from("Ferris");
    let first_clone = first.clone();`[]`
    let full = add_suffix(first_clone);`[]`
    println!("{full}, originally {first}");
}

fn add_suffix(mut name: String) -> String {
    name.push_str(" Jr.");
    name
}
```

Hãy quan sát rằng tại L1, `first_clone` đã không sao chép "nông" (shallow copy) con trỏ trong `first`, mà thay vào đó đã sao chép "sâu" (deep copy) dữ liệu chuỗi vào một cấp phát heap mới. Do đó tại L2, trong khi `first_clone` đã bị di chuyển và vô hiệu hóa bởi `add_suffix`, biến `first` ban đầu không thay đổi. Việc tiếp tục sử dụng `first` là an toàn.

{{#quiz ../quizzes/ch04-01-ownership-sec2-moves.toml}}

### Tóm Tắt

Quyền sở hữu chủ yếu là một kỷ luật về quản lý heap:[^pointer-management]

-   Tất cả dữ liệu heap phải được sở hữu bởi chính xác một biến.
-   Rust thu hồi dữ liệu heap khi chủ sở hữu của nó ra khỏi phạm vi.
-   Quyền sở hữu có thể được chuyển giao bằng các thao tác di chuyển (moves), xảy ra khi gán và gọi hàm.
-   Dữ liệu heap chỉ có thể được truy cập thông qua chủ sở hữu hiện tại của nó, không phải chủ sở hữu trước đó.

Chúng tôi đã nhấn mạnh không chỉ _cách_ các biện pháp bảo vệ của Rust hoạt động, mà còn _lý do_ chúng tránh được hành vi chưa xác định. Khi bạn nhận được thông báo lỗi từ trình biên dịch Rust, rất dễ cảm thấy nản lòng nếu bạn không hiểu tại sao Rust lại phàn nàn. Những nền tảng khái niệm này sẽ giúp bạn diễn giải các thông báo lỗi của Rust. Chúng cũng sẽ giúp bạn thiết kế các API mang phong cách Rust hơn.

[^boxed-data-structures]: Các cấu trúc dữ liệu này không sử dụng kiểu `Box` theo nghĩa đen. Ví dụ, `String` được triển khai với `Vec`, và `Vec` được triển khai với [`RawVec`](https://doc.rust-lang.org/nomicon/vec/vec-raw.html) thay vì `Box`. Nhưng các kiểu như `RawVec` vẫn giống như box (box-like): chúng sở hữu bộ nhớ trong heap.
[^pointer-management]: Theo một nghĩa khác, quyền sở hữu là một kỷ luật về quản lý _con trỏ_. Nhưng chúng tôi chưa mô tả cách tạo con trỏ tới bất kỳ đâu khác ngoài heap. Chúng ta sẽ đến đó trong phần tiếp theo.

[`NameError`]: https://docs.python.org/3/library/exceptions.html#NameError
[`ReferenceError`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError
