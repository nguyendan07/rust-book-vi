## Chú thích

Mọi lập trình viên đều cố gắng làm cho mã của họ dễ hiểu, nhưng đôi khi cần thêm giải thích. Trong những trường hợp đó, lập trình viên để lại _chú thích_ trong mã nguồn mà trình biên dịch sẽ bỏ qua nhưng người đọc mã có thể thấy hữu ích.

Đây là một chú thích đơn giản:

```rust
// hello, world
```

Trong Rust, kiểu chú thích theo phong cách chuẩn bắt đầu một chú thích bằng hai dấu gạch chéo, và chú thích sẽ kéo dài đến cuối dòng. Với chú thích vượt quá một dòng, bạn sẽ cần thêm `//` ở mỗi dòng, như sau:

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

Chú thích cũng có thể đặt ở cuối các dòng chứa mã:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Nhưng bạn sẽ thường thấy chúng được dùng theo dạng này hơn, với chú thích nằm trên một dòng riêng phía trên đoạn mã mà nó ghi chú:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust còn có một loại chú thích khác, chú thích tài liệu (documentation comments), mà chúng ta sẽ thảo luận trong phần [“Xuất bản một Crate lên Crates.io”][publishing]<!-- ignore --> của Chương 14.

[publishing]: ch14-02-publishing-to-crates-io.html
