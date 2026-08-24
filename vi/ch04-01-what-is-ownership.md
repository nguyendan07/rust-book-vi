## Quyền Sở Hữu Là Gì?

Quyền sở hữu (Ownership) là một kỷ luật nhằm đảm bảo **sự an toàn bộ nhớ** của các chương trình Rust. Để hiểu về quyền sở hữu, trước tiên chúng ta cần hiểu điều gì làm cho một chương trình Rust an toàn (hoặc không an toàn).

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

> _Lưu ý_: Trong chương này, chúng tôi sẽ sử dụng nhiều ví dụ mã không biên dịch được. Hãy nhớ tìm kiếm biểu tượng chú cua Ferris mang dấu hỏi nếu bạn không chắc chắn liệu một chương trình có biên dịch thành công hay không.

Chương trình thứ hai này không an toàn vì `read(x)` mong đợi `x` có một giá trị thuộc kiểu `bool`, nhưng `x` lại chưa có giá trị nào.

Khi một chương trình như thế này được thực thi bởi một trình thông dịch, thì việc đọc `x` trước khi nó được định nghĩa sẽ gây ra một ngoại lệ (exception) như [`NameError`] của Python hay [`ReferenceError`] của Javascript. Nhưng các ngoại lệ đi kèm với một cái giá về hiệu năng: mỗi khi một chương trình thông dịch đọc một biến, trình thông dịch bắt buộc phải kiểm tra xem biến đó đã được định nghĩa hay chưa.

Mục tiêu của Rust là biên dịch các chương trình thành các tệp nhị phân có hiệu năng cao nhất, yêu cầu càng ít bước kiểm tra tại thời gian chạy (runtime) càng tốt. Do đó, Rust không kiểm tra tại _thời gian chạy_ xem một biến đã được định nghĩa hay chưa trước khi sử dụng. Thay vào đó, Rust kiểm tra toàn diện ngay tại _thời gian biên dịch_ (compile-time). Nếu bạn cố gắng biên dịch chương trình không an toàn, bạn sẽ nhận được lỗi này:

```text
error[E0425]: cannot find value `x` in this scope
 --> src/main.rs:8:10
  |
8 |     read(x); // oh no! x isn't defined!
  |          ^ not found in this scope
```

Bạn có thể có trực giác rằng việc Rust đảm bảo các biến được định nghĩa trước khi chúng được sử dụng là điều hiển nhiên. Nhưng tại sao? Để hiểu rõ quy tắc này, chúng ta phải đặt câu hỏi: **điều gì sẽ xảy ra nếu Rust cho phép biên dịch một chương trình bị lỗi như vậy?**

Trước tiên hãy xem xét cách chương trình an toàn biên dịch và thực thi. Trên một máy tính có bộ vi xử lý sử dụng kiến trúc [x86](https://en.wikipedia.org/wiki/X86), Rust tạo ra mã hợp ngữ (assembly) sau cho hàm `main` trong chương trình an toàn ([xem mã hợp ngữ đầy đủ tại đây](https://rust.godbolt.org/z/xnT1fzsqv)):

```x86asm
main:
    ; ...
    mov     edi, 1
    call    read
    ; ...
```

> _Lưu ý_: Nếu bạn chưa quen thuộc với mã hợp ngữ (assembly), đừng lo lắng! Phần này chứa một vài ví dụ về hợp ngữ chỉ để cho bạn thấy cách Rust thực sự vận hành ở cấp độ phần cứng bên dưới (under the hood). Bạn hoàn toàn không cần biết hợp ngữ để học và làm chủ Rust.

Mã hợp ngữ này sẽ:

-   Di chuyển số 1, đại diện cho `true`, vào một "thanh ghi" (register - một loại ô nhớ tốc độ cao của CPU) được gọi là `edi`.
-   Gọi hàm `read`, hàm này mong đợi đối số đầu tiên `y` của nó nằm trong thanh ghi `edi`.

Nếu hàm không an toàn được phép biên dịch, mã hợp ngữ của nó có thể trông như thế này:

```x86asm
main:
    ; ...
    call    read
    mov     edi, 1    ; mov nằm sau call
    ; ...
```

Chương trình này không an toàn vì `read` sẽ mong đợi `edi` là một boolean, tức là số `0` hoặc `1`. Nhưng `edi` lúc này có thể chứa bất kỳ giá trị rác nào trong bộ nhớ: `2`, `100`, hay `0x1337BEEF`. Khi `read` muốn sử dụng đối số `y` của nó cho bất kỳ mục đích nào, nó sẽ ngay lập tức gây ra _**HÀNH VI CHƯA XÁC ĐỊNH (UNDEFINED BEHAVIOR - UB)!**_

Rust không quy định điều gì xảy ra nếu bạn cố gắng chạy `if y { .. }` khi `y` không phải là `true` hay `false`. _Hành vi_ đó, hay những gì xảy ra sau khi thực thi lệnh, là _chưa xác định_. Một điều gì đó sẽ xảy ra lúc chạy, ví dụ:

-   Mã thực thi mà không gặp sự cố, và không ai nhận thấy vấn đề gì lúc đó.
-   Mã ngay lập tức bị sập (crash) do [lỗi phân đoạn](https://en.wikipedia.org/wiki/Segmentation_fault) (segmentation fault) hoặc một loại lỗi hệ điều hành khác.
-   Mã thực thi bình thường trong một thời gian, cho đến khi một kẻ tấn công tạo ra dữ liệu đầu vào đặc biệt để làm sập hệ thống, ghi đè dữ liệu hoặc chiếm đoạt các tài nguyên nhạy cảm.

**Một mục tiêu nền tảng của Rust là đảm bảo rằng các chương trình của bạn không bao giờ xảy ra hành vi chưa xác định (Undefined Behavior).** Đó chính là ý nghĩa của "sự an toàn bộ nhớ". Hành vi chưa xác định đặc biệt nguy hiểm đối với các chương trình cấp hệ thống có quyền truy cập trực tiếp vào bộ nhớ. Khoảng [70% các lỗ hổng bảo mật nghiêm trọng](https://msrc.microsoft.com/blog/2019/07/a-proactive-approach-to-more-secure-code/) trong các hệ thống phần mềm lớn bằng C/C++ là do lỗi hỏng bộ nhớ (memory corruption) — một dạng điển hình của hành vi chưa xác định.

Mục tiêu thứ hai của Rust là ngăn chặn hành vi chưa xác định ngay tại _thời gian biên dịch_ thay vì để đến _thời gian chạy_. Mục tiêu này mang lại hai lợi ích lớn:

1. Bắt lỗi tại thời gian biên dịch giúp loại bỏ hoàn toàn các lỗi bộ nhớ trong môi trường thực tế (production), nâng cao độ tin cậy của phần mềm.
2. Bắt lỗi tại thời gian biên dịch giúp giảm bớt các bước kiểm tra kiểm tra an toàn lúc runtime, tối đa hóa hiệu suất của phần mềm.

Rust không thể ngăn chặn mọi lỗi logic nghiệp vụ. Nếu một ứng dụng để lộ công khai một endpoint `/delete-production-database` không cần xác thực, thì một kẻ xấu không cần lỗi bộ nhớ nào cũng có thể xóa cơ sở dữ liệu. Nhưng các cơ chế bảo vệ của Rust chắc chắn giúp các chương trình an toàn hơn rất nhiều so với việc sử dụng các ngôn ngữ không có kiểm tra bộ nhớ tĩnh, điều đã được chứng minh qua các báo cáo thực tế từ [đội ngũ Android của Google](https://security.googleblog.com/2022/12/memory-safe-languages-in-android-13.html).

### Quyền Sở Hữu Như Là Một Kỷ Luật Cho An Toàn Bộ Nhớ

Vì an toàn là sự vắng mặt của hành vi chưa xác định, và vì quyền sở hữu đảm bảo sự an toàn, nên chúng ta cần hiểu quyền sở hữu dưới góc độ của các hành vi chưa xác định mà nó ngăn chặn. Tài liệu tham khảo Rust (Rust Reference) duy trì một danh mục lớn các ["Hành vi được coi là chưa xác định"](https://doc.rust-lang.org/reference/behavior-considered-undefined.html). Hiện tại, chúng ta sẽ tập trung vào một danh mục quan trọng nhất: các thao tác trên bộ nhớ.

Bộ nhớ là không gian nơi dữ liệu được lưu trữ trong quá trình thực thi của một chương trình. Có nhiều cách để tư duy về bộ nhớ:

-   Nếu bạn chưa quen với lập trình hệ thống, bạn có thể nghĩ về bộ nhớ ở mức trừu tượng như "bộ nhớ là RAM trong máy tính" hoặc "bộ nhớ là thứ sẽ bị đầy nếu tải quá nhiều dữ liệu".
-   Nếu bạn đã quen thuộc với C/C++, bạn có thể nghĩ về bộ nhớ ở mức rất thấp như "bộ nhớ là một mảng các byte" hoặc "bộ nhớ là các con trỏ nhận về từ hàm `malloc`".

Cả hai cách hiểu này đều _đúng_, nhưng chúng chưa phải là cách _tối ưu_ để hiểu cách Rust hoạt động. Cách hiểu trừu tượng thì thiếu cụ thể để giải thích cơ chế của con trỏ. Cách hiểu mức quá thấp lại quá chi tiết vì Rust không cho phép bạn thao tác tự do trên mảng byte thô mà không có kiểm soát.

Rust cung cấp một mô hình tư duy rõ ràng và an toàn về bộ nhớ. Quyền sở hữu (Ownership) là một kỷ luật để quản lý bộ nhớ an toàn theo mô hình đó.

### Biến Được Lưu Trữ Trong Stack

Dưới đây là một chương trình minh họa việc định nghĩa một số `n` và gọi hàm `plus_one` trên `n`. Bên dưới chương trình là sơ đồ trực quan hóa nội dung của bộ nhớ tại ba thời điểm thực thi.

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

Các biến tồn tại trong các **khung ngăn xếp** (stack frames). Một khung là một bảng ánh xạ từ các tên biến tới các giá trị trong một phạm vi (scope) duy nhất, chẳng hạn như một hàm. Ví dụ:

-   Khung của hàm `main` tại vị trí L1 chứa `n = 5`.
-   Khung của hàm `plus_one` tại L2 chứa `x = 5`.
-   Khung của hàm `main` tại vị trí L3 chứa `n = 5; y = 6`.

Các khung được tổ chức theo cấu trúc **ngăn xếp** (stack) của các hàm hiện đang được gọi. Ví dụ, tại L2, khung của `main` nằm dưới khung của hàm đang được gọi `plus_one`. Sau khi một hàm thực thi xong và trả về, Rust sẽ tự động thu hồi (deallocate) khung của hàm đó. (Việc thu hồi này còn được gọi là **giải phóng** - freeing hoặc **hủy bỏ** - dropping). Cấu trúc này gọi là ngăn xếp (stack) vì khung được thêm vào sau cùng luôn là khung đầu tiên được giải phóng (nguyên lý LIFO - Last In, First Out).

Khi một biểu thức đọc một biến có kiểu dữ liệu nguyên thủy (như số nguyên, boolean), giá trị của biến đó được sao chép (copy) từ vị trí của nó trong khung stack. Ví dụ:

```aquascope,interpreter,horizontal
#fn main() {
let a = 5;`[]`
let mut b = a;`[]`
b += 1;`[]`
#}
```

Giá trị của `a` được sao chép vào `b`, và `a` vẫn hoàn toàn độc lập, không bị thay đổi ngay cả khi `b` tăng lên 1.

### Box Được Lưu Trữ Trong Heap

Tuy nhiên, việc sao chép toàn bộ dữ liệu có thể chiếm rất nhiều bộ nhớ và làm giảm hiệu năng nếu dữ liệu có kích thước lớn. Ví dụ, chương trình sau sao chép một mảng có 1 triệu phần tử:

```aquascope,interpreter
#fn main() {
let a = [0; 1_000_000];`[]`
let b = a;`[]`
#}
```

Quan sát thấy rằng việc sao chép `a` sang `b` khiến khung stack của hàm `main` phải chứa tới 2 triệu phần tử (tốn gấp đôi bộ nhớ stack).

Để chuyển quyền truy cập dữ liệu lớn mà không cần sao chép tốn kém, Rust sử dụng các **con trỏ** (pointers). Một con trỏ là một giá trị lưu địa chỉ của một vị trí trong bộ nhớ. Giá trị mà con trỏ trỏ tới được gọi là **vùng nhớ đích** (pointee) của nó.

Một cách phổ biến để tạo con trỏ là cấp phát bộ nhớ trong vùng **Heap**. Heap là một vùng bộ nhớ rộng lớn nơi dữ liệu có thể tồn tại linh hoạt, không bị gắn chặt vào thời gian sống của một khung stack cụ thể nào. Rust cung cấp một kiểu dữ liệu thông minh gọi là [`Box`](https://doc.rust-lang.org/std/boxed/index.html) để lưu trữ dữ liệu trên heap. Ví dụ, chúng ta có thể đặt mảng một triệu phần tử vào bên trong `Box::new`:

```aquascope,interpreter
#fn main() {
let a = Box::new([0; 1_000_000]);`[]`
let b = a;`[]`
#}
```

Lúc này, trong bộ nhớ Heap chỉ tồn tại duy nhất một mảng dữ liệu. Tại L1, giá trị của `a` là một con trỏ nhỏ (trên stack) trỏ tới mảng dữ liệu nằm trên heap. Câu lệnh `let b = a` chỉ sao chép địa chỉ con trỏ từ `a` sang `b`, dữ liệu 1 triệu phần tử trên heap không bị sao chép lại. Đáng chú ý, biến `a` sau đó bị vô hiệu hóa vì quyền sở hữu đã bị **chuyển giao** (_moved_) sang `b`.

{{#quiz ../quizzes/ch04-01-ownership-sec1-stackheap.toml}}

### Rust Không Cho Phép Quản Lý Bộ Nhớ Thủ Công

Quản lý bộ nhớ là quá trình cấp phát và thu hồi bộ nhớ khi không còn sử dụng. Khung stack được Rust tự động quản lý hoàn toàn theo vòng đời của hàm.

Với vùng nhớ heap, dữ liệu được cấp phát khi gọi `Box::new(..)`. Nhưng khi nào dữ liệu heap được thu hồi? Nếu Rust cho phép lập trình viên tự gọi hàm `free()` bất kỳ lúc nào (như trong C/C++), điều này rất dễ dẫn đến lỗi nghiêm trọng: lập trình viên có thể vô tình đọc lại con trỏ trỏ tới vùng nhớ đã bị giải phóng (*Dangling Pointer / Use-after-free*):

```aquascope,interpreter,shouldFail
#fn free<T>(_t: T) {}
#fn main() {
let b = Box::new([0; 100]);`[]`
free(b);`[]`
assert!(b[0] == 0);`[]`
#}
```

Khi gọi `free(b)`, vùng nhớ heap của `b` bị thu hồi. Sau đó, nếu cố gắng đọc `b[0]`, chương trình sẽ truy cập vào vùng nhớ không còn hợp lệ, dẫn đến hành vi chưa xác định (crash chương trình hoặc đọc phải dữ liệu rác).

Rust ngăn chặn hoàn toàn nguy cơ này bằng cách **không cho phép lập trình viên thu hồi bộ nhớ thủ công**.

### Chủ Sở Hữu Của Box Quản Lý Việc Thu Hồi

Thay vào đó, Rust tự động giải phóng bộ nhớ heap của box dựa trên quyền sở hữu:

> **Nguyên tắc thu hồi Box:** Nếu một biến sở hữu một box, khi khung stack chứa biến đó bị giải phóng (khi biến ra khỏi scope), Rust sẽ tự động thu hồi vùng nhớ heap mà box đó đang sở hữu.

Hãy xem điều gì xảy ra khi chúng ta gán hai biến cho cùng một box:

```rust,ignore
# fn main() {
let a = Box::new([0; 1_000_000]);
let b = a;
# }
```

Nếu cả `a` và `b` cùng sở hữu mảng trên heap, khi hàm kết thúc, Rust sẽ cố gắng giải phóng vùng nhớ đó **hai lần** (*Double Free*). Đây là một lỗi nghiêm trọng dẫn đến hỏng bộ nhớ!

Để loại trừ triệt để lỗi này, Rust đặt ra quy tắc: khi `a` được gán giá trị `Box::new(...)`, `a` là **chủ sở hữu duy nhất** của box đó. Câu lệnh `let b = a` đã **chuyển giao (move)** quyền sở hữu từ `a` sang `b`. Biến `a` bị vô hiệu hóa ngay lập tức. Khi kết thúc scope, Rust chỉ giải phóng vùng nhớ heap một lần duy nhất cho `b`.

### Các Cấu Trúc Dữ Liệu Tập Hợp Sử Dụng Heap

Các cấu trúc dữ liệu quen thuộc trong Rust[^boxed-data-structures] như [`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html), [`String`](https://doc.rust-lang.org/std/string/struct.String.html), và [`HashMap`](https://doc.rust-lang.org/std/collections/struct.HashMap.html) đều quản lý dữ liệu động trên heap. Ví dụ, đây là chương trình tạo, chuyển quyền sở hữu và thay đổi một chuỗi (`String`):

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

Các bước diễn ra như sau:

1. Tại L1, chuỗi `"Ferris"` được cấp phát trên heap, được sở hữu bởi `first`.
2. Tại L2, khi gọi hàm `add_suffix(first)`, quyền sở hữu chuỗi được **move** từ `first` sang tham số `name`. Con trỏ được sao chép nhưng dữ liệu trên heap không bị duplicate.
3. Tại L3, `name.push_str(" Jr.")` cấp phát vùng nhớ mới lớn hơn trên heap để chứa `"Ferris Jr."` và giải phóng vùng nhớ cũ.
4. Tại L4, khi `add_suffix` kết thúc, hàm trả về `name`, chuyển tiếp quyền sở hữu cho biến `full` trong `main`.

### Các Biến Không Thể Được Sử Dụng Sau Khi Bị Di Chuyển (Moved)

Nếu bạn cố gắng sử dụng lại biến `first` trong `main` sau khi đã chuyển quyền sở hữu cho hàm `add_suffix`:

```aquascope,interpreter,shouldFail
fn main() {
    let first = String::from("Ferris");
    let full = add_suffix(first);
    println!("{full}, originally {first}");`[]` // lỗi: first đã bị move!
}

fn add_suffix(mut name: String) -> String {
    name.push_str(" Jr.");
    name
}
```

Trình biên dịch Rust sẽ từ chối biên dịch ngay lập tức và báo lỗi rõ ràng:

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

> **Nguyên tắc dữ liệu heap bị di chuyển:** Nếu một biến `x` đã chuyển giao quyền sở hữu dữ liệu heap cho một biến `y` khác, thì `x` không còn hợp lệ và không thể được sử dụng lại sau đó.

> [!NOTE]
> **So sánh với Python:**
> Trong Python, khi bạn viết `b = a` (với `a` là một list hoặc object), cả `a` và `b` cùng trỏ đến chung một đối tượng trên heap và bạn vẫn có thể dùng `a` bình thường.
> Trong Rust, với các kiểu dữ liệu cấp phát trên heap như `String` hay `Vec`, phép gán `b = a` là một hành động **Move (chuyển giao quyền sở hữu)**. Biến `a` bị vô hiệu hóa ngay lập tức. Nếu bạn muốn `a` vẫn dùng được, bạn phải chủ động sao chép toàn bộ dữ liệu bằng `.clone()`.

### Sao Chép Dữ Liệu Bằng `.clone()` Để Tránh Move

Nếu bạn muốn tạo một bản sao độc lập của dữ liệu heap để biến ban đầu vẫn có thể sử dụng được, bạn có thể gọi phương thức `.clone()` (sao chép sâu - deep copy):

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

Tại L1, `first.clone()` sao chép toàn bộ nội dung của `"Ferris"` sang một vùng nhớ heap mới cho `first_clone`. Nhờ đó, khi `first_clone` bị di chuyển vào `add_suffix`, biến `first` ban đầu vẫn giữ nguyên vẹn và an toàn để sử dụng ở dòng `println!`.

{{#quiz ../quizzes/ch04-01-ownership-sec2-moves.toml}}

### Tóm Tắt

Quyền sở hữu (Ownership) là kỷ luật cốt lõi để quản lý bộ nhớ Heap của Rust:

-   Mỗi vùng dữ liệu trên heap luôn có **chính xác một** biến làm chủ sở hữu tại một thời điểm.
-   Khi chủ sở hữu ra khỏi phạm vi (scope), dữ liệu trên heap sẽ được **tự động giải phóng**.
-   Quyền sở hữu được **chuyển giao (move)** khi gán biến hoặc truyền/trả về từ hàm.
-   Dữ liệu chỉ có thể được truy cập thông qua **chủ sở hữu hiện tại**, không thể truy cập qua chủ sở hữu cũ sau khi đã move.

[^boxed-data-structures]: Các cấu trúc dữ liệu này không nhất thiết dùng trực tiếp kiểu `Box` mà dùng các cấu trúc quản lý con trỏ thô bên dưới như `RawVec`, nhưng chúng đều có cùng cơ chế sở hữu bộ nhớ heap.
[^pointer-management]: Quyền sở hữu cũng là nền tảng cho việc quản lý con trỏ an toàn, điều mà chúng ta sẽ tìm hiểu ở phần tiếp theo về Tham chiếu (References).

[`NameError`]: https://docs.python.org/3/library/exceptions.html#NameError
[`ReferenceError`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError
