# Dự Án Cuối Cùng: Xây Dựng Một Web Server Đa Luồng

Đó là một hành trình dài, nhưng chúng ta đã đi đến phần cuối của cuốn sách. Trong
chương này, chúng ta sẽ cùng nhau xây dựng thêm một dự án nữa để minh họa một số
khái niệm đã được đề cập trong các chương cuối, cũng như điểm lại một số bài học
trước đó.

Đối với dự án cuối cùng này, chúng ta sẽ tạo một web server gửi lời chào “hello” và
trông giống như Hình 21-1 trong trình duyệt web.

![hello from rust](img/trpl21-01.png)

<span class="caption">Hình 21-1: Dự án chung cuối cùng của chúng ta</span>

Dưới đây là kế hoạch xây dựng web server của chúng ta:

1. Tìm hiểu một chút về TCP và HTTP.
2. Lắng nghe các kết nối TCP trên một socket.
3. Phân tích cú pháp (parse) một số lượng nhỏ các HTTP request.
4. Tạo một HTTP response thích hợp.
5. Cải thiện thông lượng (throughput) của server với một thread pool (nhóm luồng).

Trước khi bắt đầu, chúng ta nên lưu ý hai chi tiết. Thứ nhất, phương pháp chúng ta
sử dụng sẽ không phải là cách tốt nhất để xây dựng một web server với Rust. Các
thành viên trong cộng đồng đã xuất bản một số crate sẵn sàng cho môi trường production
trên [crates.io](https://crates.io/) cung cấp các bản triển khai web server và thread
pool hoàn chỉnh hơn những gì chúng ta sẽ xây dựng. Tuy nhiên, mục đích của chúng ta trong
chương này là giúp bạn học hỏi, chứ không phải đi con đường dễ dàng. Vì Rust là một ngôn
ngữ lập trình hệ thống, chúng ta có thể chọn mức độ trừu tượng mà mình muốn làm việc
và có thể đi xuống mức thấp hơn mức có thể hoặc thực tế trong các ngôn ngữ khác.

Thứ hai, chúng ta sẽ không sử dụng async và await ở đây. Bản thân việc xây dựng một
thread pool đã là một thử thách đủ lớn, mà không cần phải thêm vào việc xây dựng một
async runtime! Tuy nhiên, chúng ta sẽ lưu ý cách async và await có thể áp dụng cho một số
vấn đề tương tự mà chúng ta sẽ thấy trong chương này. Cuối cùng, như chúng ta đã lưu ý
trong Chương 17, nhiều async runtime sử dụng các thread pool để quản lý công việc của chúng.

Do đó, chúng ta sẽ tự tay viết HTTP server cơ bản và thread pool theo cách thủ công để bạn
có thể học được các ý tưởng và kỹ thuật chung đằng sau các crate mà bạn có thể sử dụng
trong tương lai.
