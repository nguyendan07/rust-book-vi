## Mở rộng Cargo với các Lệnh Tùy chỉnh

Cargo được thiết kế để bạn có thể mở rộng nó với các lệnh con mới mà không cần phải
sửa đổi nó. Nếu một tệp nhị phân trong `$PATH` của bạn được đặt tên là `cargo-something`, bạn có thể chạy
nó như thể nó là một lệnh con của Cargo bằng cách chạy `cargo something`. Các lệnh
tùy chỉnh như thế này cũng được liệt kê khi bạn chạy `cargo --list`. Việc có thể
sử dụng `cargo install` để cài đặt các phần mở rộng và sau đó chạy chúng giống như các
công cụ Cargo tích hợp sẵn là một lợi ích cực kỳ thuận tiện trong thiết kế của Cargo!

## Tổng kết

Chia sẻ mã nguồn với Cargo và [crates.io](https://crates.io/)<!-- ignore --> là
một phần của những gì làm cho hệ sinh thái Rust trở nên hữu ích cho nhiều tác vụ khác nhau. Thư viện
chuẩn của Rust nhỏ và ổn định, nhưng các crate thì dễ dàng để chia sẻ, sử dụng và
cải thiện theo một lộ trình khác với lộ trình của ngôn ngữ. Đừng ngần ngại về việc
chia sẻ mã nguồn hữu ích cho bạn trên [crates.io](https://crates.io/)<!-- ignore
-->; rất có khả năng nó cũng sẽ hữu ích cho một ai đó khác!
