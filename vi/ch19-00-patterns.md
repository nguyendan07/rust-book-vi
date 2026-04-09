# Mẫu và Khớp mẫu (Patterns and Matching)

_Mẫu_ (Patterns) là một cú pháp đặc biệt trong Rust để khớp với cấu trúc của
các kiểu dữ liệu, cả phức tạp và đơn giản. Sử dụng mẫu kết hợp với
biểu thức `match` và các cấu trúc khác giúp bạn kiểm soát nhiều hơn đối với
luồng điều khiển (control flow) của chương trình. Một mẫu bao gồm một số sự kết hợp của các thành phần sau:

- Các hằng (Literals)
- Các mảng, enum, struct, hoặc tuple đã được phá cấu trúc (destructured)
- Biến
- Các ký tự đại diện (Wildcards)
- Các trình giữ chỗ (Placeholders)

Một số mẫu ví dụ bao gồm `x`, `(a, 3)`, và `Some(Color::Red)`. Trong
các ngữ cảnh mà mẫu hợp lệ, các thành phần này mô tả hình dạng của
dữ liệu. Chương trình của chúng ta sau đó sẽ khớp các giá trị với các mẫu để xác định xem
nó có đúng hình dạng dữ liệu để tiếp tục chạy một đoạn mã cụ thể hay không.

Để sử dụng một mẫu, chúng ta so sánh nó với một giá trị nào đó. Nếu mẫu khớp với
giá trị, chúng ta sử dụng các phần của giá trị đó trong mã của mình. Hãy nhớ lại các biểu thức `match` trong
Chương 6 đã sử dụng các mẫu, chẳng hạn như ví dụ về máy phân loại tiền xu. Nếu
giá trị phù hợp với hình dạng của mẫu, chúng ta có thể sử dụng các phần đã được đặt tên. Nếu nó
không khớp, mã liên kết với mẫu đó sẽ không chạy.

Chương này là một tài liệu tham khảo về tất cả những thứ liên quan đến mẫu. Chúng ta sẽ đề cập đến
những nơi hợp lệ để sử dụng mẫu, sự khác biệt giữa mẫu refutable và irrefutable,
và các loại cú pháp mẫu khác nhau mà bạn có thể gặp. Đến cuối chương,
bạn sẽ biết cách sử dụng mẫu để diễn đạt nhiều khái niệm theo
một cách rõ ràng.
