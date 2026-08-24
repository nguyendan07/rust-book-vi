## Chú Thích (Comments)

Mọi lập trình viên đều cố gắng làm cho mã nguồn của mình dễ hiểu, nhưng đôi khi cần phải có thêm lời giải thích. Trong những trường hợp này, các lập trình viên để lại _chú thích_ (comments) trong mã nguồn của họ mà trình biên dịch sẽ bỏ qua nhưng những người đọc mã nguồn có thể thấy hữu ích.

Dưới đây là một chú thích đơn giản:

```rust
// hello, world
```

Trong Rust, phong cách chú thích chuẩn mực bắt đầu một chú thích bằng hai dấu gạch chéo (`//`), và chú thích kéo dài cho đến hết dòng. Đối với các chú thích kéo dài qua nhiều dòng, bạn sẽ cần bao gồm `//` trên mỗi dòng, như sau:

```rust
// Ở đây chúng ta đang làm một điều gì đó phức tạp, đủ dài để chúng ta cần
// nhiều dòng chú thích để giải thích! Hy vọng chú thích này sẽ
// giải thích rõ ràng điều gì đang diễn ra.
```

Hoặc bạn có thể sử dụng cú pháp chú thích nhiều dòng với `/*` và `*/`:

```rust
/* Ở đây chúng ta đang làm một điều gì đó phức tạp, đủ dài để chúng ta cần
   nhiều dòng chú thích để giải thích! Hy vọng chú thích này sẽ
   giải thích rõ ràng điều gì đang diễn ra. */
```

Chú thích cũng có thể được đặt ở cuối các dòng chứa mã nguồn:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Nhưng bạn sẽ thường thấy chúng được sử dụng theo định dạng này hơn, với chú thích nằm trên một dòng riêng biệt phía trên đoạn mã mà nó giải thích:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust cũng có một loại chú thích khác là chú thích tài liệu (documentation comments), chúng ta sẽ thảo luận trong phần [“Phát Hành Crate Lên Crates.io”][publishing]<!-- ignore --> của Chương 14.

[publishing]: ch14-02-publishing-to-crates-io.html
