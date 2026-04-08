## So sánh hiệu suất: Vòng lặp đối với Iterator

Để quyết định nên sử dụng vòng lặp hay iterator, bạn cần biết
triển khai nào nhanh hơn: phiên bản của hàm `search` với một vòng lặp `for`
rõ ràng hay phiên bản với các iterator.

Chúng tôi đã chạy một điểm chuẩn (benchmark) bằng cách tải toàn bộ nội dung của cuốn _Những cuộc phiêu lưu của
Sherlock Holmes_ bởi Sir Arthur Conan Doyle vào một `String` và tìm kiếm
từ _the_ trong nội dung đó. Dưới đây là kết quả của điểm chuẩn trên
phiên bản `search` sử dụng vòng lặp `for` và phiên bản sử dụng iterator:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Hai triển khai có hiệu suất tương tự nhau! Chúng tôi sẽ không giải thích
mã điểm chuẩn ở đây, vì mục đích không phải để chứng minh rằng hai phiên bản
là tương đương mà để có cái nhìn tổng quát về việc hai triển khai này
so sánh như thế nào về mặt hiệu suất.

Để có một điểm chuẩn toàn diện hơn, bạn nên kiểm tra bằng cách sử dụng các văn bản khác nhau với
các kích thước khác nhau làm `contents`, các từ khác nhau và các từ có độ dài khác nhau
làm `query`, và tất cả các loại biến thể khác. Vấn đề là thế này:
iterator, mặc dù là một sự trừu tượng cấp cao, nhưng được biên dịch xuống
mã gần như tương đương với việc bạn tự viết mã cấp thấp hơn. Iterator là một
trong những _trừu tượng chi phí bằng không_ (zero-cost abstractions) của Rust, theo đó chúng tôi muốn nói rằng việc sử dụng sự trừu tượng
không gây thêm gánh nặng thời gian chạy (runtime overhead) nào. Điều này tương tự như cách Bjarne
Stroustrup, người thiết kế và triển khai ban đầu của C++, định nghĩa
_chi phí bằng không_ (zero-overhead) trong “Foundations of C++” (2012):

> Nói chung, các triển khai C++ tuân theo nguyên tắc chi phí bằng không: Những gì bạn
> không dùng, bạn không phải trả giá. Và hơn thế nữa: Những gì bạn dùng, bạn không thể viết mã
> tay tốt hơn được.

Một ví dụ khác, đoạn mã sau được lấy từ một bộ giải mã âm thanh.
Thuật toán giải mã sử dụng phép toán toán học dự đoán tuyến tính để
ước tính các giá trị tương lai dựa trên một hàm tuyến tính của các mẫu trước đó.
Đoạn mã này sử dụng một chuỗi iterator để thực hiện một số phép toán trên ba biến trong phạm vi: một
slice dữ liệu `buffer`, một mảng gồm 12 `coefficients`, và một lượng để
dịch chuyển dữ liệu trong `qlp_shift`. Chúng tôi đã khai báo các biến trong ví dụ này
nhưng không cung cấp cho chúng bất kỳ giá trị nào; mặc dù đoạn mã này không có nhiều ý nghĩa
bên ngoài ngữ cảnh của nó, nó vẫn là một ví dụ thực tế, súc tích về cách Rust
chuyển đổi các ý tưởng cấp cao thành mã cấp thấp.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Để tính giá trị của `prediction`, đoạn mã này lặp qua từng giá trị trong số
12 giá trị trong `coefficients` và sử dụng phương thức `zip` để ghép cặp các giá trị hệ số
với 12 giá trị trước đó trong `buffer`. Sau đó, với mỗi cặp, chúng ta
nhân các giá trị với nhau, tổng hợp tất cả các kết quả, và dịch chuyển các bit trong
tổng đi `qlp_shift` bit sang bên phải.

Các tính toán trong các ứng dụng như bộ giải mã âm thanh thường ưu tiên hiệu suất
cao nhất. Ở đây, chúng ta đang tạo một iterator, sử dụng hai adapter, và sau đó
tiêu thụ giá trị. Mã Rust này sẽ được biên dịch thành mã assembly nào? Chà,
tại thời điểm viết bài này, nó biên dịch xuống cùng một mã assembly mà bạn sẽ tự viết tay.
Không có vòng lặp nào tương ứng với việc lặp qua các giá trị trong
`coefficients`: Rust biết rằng có 12 lần lặp, vì vậy nó “mở” (unroll)
vòng lặp. _Mở vòng lặp_ (unrolling) là một sự tối ưu hóa loại bỏ chi phí của mã
điều khiển vòng lặp và thay vào đó tạo ra mã lặp lại cho mỗi lần lặp của vòng lặp.

Tất cả các hệ số đều được lưu trữ trong các thanh ghi, điều đó có nghĩa là việc truy cập
các giá trị rất nhanh. Không có kiểm tra ranh giới (bounds checks) khi truy cập mảng lúc chạy.
Tất cả những tối ưu hóa này mà Rust có thể áp dụng làm cho mã kết quả
cực kỳ hiệu quả. Bây giờ bạn đã biết điều này, bạn có thể sử dụng iterator và closure
mà không sợ hãi! Chúng làm cho mã có vẻ như ở cấp độ cao hơn nhưng không gây ra
hình phạt về hiệu suất thời gian chạy khi làm như vậy.

## Tóm tắt

Closure và iterator là các tính năng của Rust được lấy cảm hứng từ các ý tưởng của ngôn ngữ lập trình
hàm. Chúng góp phần vào khả năng của Rust trong việc thể hiện rõ ràng
các ý tưởng cấp cao ở hiệu suất cấp thấp. Việc triển khai các closure và
iterator sao cho hiệu suất thời gian chạy không bị ảnh hưởng. Đây là một phần của
mục tiêu của Rust là cố gắng cung cấp các sự trừu tượng chi phí bằng không.

Bây giờ chúng ta đã cải thiện tính biểu đạt của dự án I/O của mình, hãy cùng xem xét
một số tính năng khác của `cargo` sẽ giúp chúng ta chia sẻ dự án với
thế giới.
