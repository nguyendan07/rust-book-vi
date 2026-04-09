# Concurrency không sợ hãi (Fearless Concurrency)

Xử lý lập trình đồng thời (concurrent programming) một cách an toàn và hiệu quả là một trong những
mục tiêu chính khác của Rust. _Lập trình đồng thời_, trong đó các phần khác nhau của một chương trình
thực thi độc lập, và _lập trình song song_ (parallel programming), trong đó các phần khác nhau của
một chương trình thực thi cùng một lúc, đang ngày càng trở nên quan trọng khi nhiều
máy tính tận dụng lợi thế của nhiều bộ vi xử lý của chúng. Trong lịch sử,
việc lập trình trong những ngữ cảnh này đã từng gặp nhiều khó khăn và dễ xảy ra lỗi. Rust hy vọng
thay đổi điều đó.

Ban đầu, đội ngũ Rust đã nghĩ rằng việc đảm bảo an toàn bộ nhớ và ngăn chặn
các vấn đề về đồng thời là hai thử thách riêng biệt cần được giải quyết bằng các phương pháp
khác nhau. Theo thời gian, đội ngũ đã phát hiện ra rằng hệ thống sở hữu (ownership) và hệ thống kiểu (type system) là
một bộ công cụ mạnh mẽ giúp quản lý an toàn bộ nhớ _và_ các vấn đề
về đồng thời! Bằng cách tận dụng quyền sở hữu và kiểm tra kiểu, nhiều lỗi đồng thời
là các lỗi thời điểm biên dịch (compile-time) trong Rust thay vì các lỗi thời điểm chạy (runtime). Do đó, thay vì
khiến bạn mất nhiều thời gian để cố gắng tái hiện lại các tình huống chính xác
mà một lỗi đồng thời lúc thực thi xảy ra, mã không chính xác sẽ bị từ chối
biên dịch và hiển thị một lỗi giải thích vấn đề. Kết quả là, bạn có thể sửa
mã của mình trong khi đang làm việc thay vì có khả năng là sau khi nó đã được
đưa vào vận hành (production). Chúng tôi đã đặt biệt danh cho khía cạnh này của Rust là _concurrency không_
_sợ hãi_. Concurrency không sợ hãi cho phép bạn viết mã không có các lỗi
tinh vi và dễ dàng cấu trúc lại (refactor) mà không gây ra các lỗi mới.

> Ghi chú: Để cho đơn giản, chúng tôi sẽ gọi nhiều vấn đề là
> _đồng thời_ thay vì nói chính xác hơn là _đồng thời và/hoặc
> song song_. Trong chương này, vui lòng tự hiểu _đồng thời
> và/hoặc song song_ bất cứ khi nào chúng tôi sử dụng _đồng thời_. Trong chương tiếp theo, nơi mà
> sự phân biệt quan trọng hơn, chúng tôi sẽ cụ thể hơn.

Nhiều ngôn ngữ rất giáo điều về các giải pháp mà chúng cung cấp để xử lý
các vấn đề đồng thời. Ví dụ, Erlang có chức năng thanh thoát cho
concurrency truyền thông điệp (message-passing) nhưng chỉ có những cách mơ hồ để chia sẻ trạng thái giữa
các luồng (threads). Chỉ hỗ trợ một tập hợp con các giải pháp khả thi là một chiến lược
hợp lý cho các ngôn ngữ bậc cao, bởi vì một ngôn ngữ bậc cao hứa hẹn mang lại
lợi ích từ việc từ bỏ một số quyền kiểm soát để đạt được các sự trừu tượng hóa. Tuy nhiên, các ngôn ngữ
bậc thấp được kỳ vọng sẽ cung cấp giải pháp với hiệu suất tốt nhất trong bất kỳ
tình huống nhất định nào và có ít sự trừu tượng hóa hơn đối với phần cứng. Do đó, Rust
cung cấp nhiều công cụ khác nhau để mô hình hóa các vấn đề theo bất kỳ cách nào phù hợp
với tình huống và yêu cầu của bạn.

Dưới đây là các chủ đề chúng ta sẽ đề cập trong chương này:

- Cách tạo các luồng để chạy nhiều đoạn mã cùng một lúc
- Concurrency _truyền thông điệp_, nơi các kênh (channels) gửi thông điệp giữa các luồng
- Concurrency _chia sẻ trạng thái_ (shared-state), nơi nhiều luồng có quyền truy cập vào một số mẩu
  dữ liệu
- Các trait `Sync` và `Send`, giúp mở rộng các đảm bảo concurrency của Rust cho
  các kiểu do người dùng định nghĩa cũng như các kiểu được cung cấp bởi thư viện tiêu chuẩn
