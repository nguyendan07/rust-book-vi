# Enum và Khớp mẫu (Pattern Matching)

Trong chương này, chúng ta sẽ tìm hiểu về _phép liệt kê (enumerations)_, hay còn được gọi là _enum_.
Enum cho phép bạn định nghĩa một kiểu dữ liệu bằng cách liệt kê các _biến thể (variants)_ khả thi của nó. Đầu tiên,
chúng ta sẽ định nghĩa và sử dụng một enum để chỉ ra cách một enum có thể mã hóa ý nghĩa cùng với
dữ liệu. Tiếp theo, chúng ta sẽ khám phá một enum đặc biệt hữu ích, được gọi là `Option`, dùng để
diễn đạt rằng một giá trị có thể là một thứ gì đó hoặc không là gì cả. Sau đó, chúng ta sẽ xem xét
cách khớp mẫu (pattern matching) trong biểu thức `match` giúp dễ dàng chạy các mã khác nhau cho các
giá trị khác nhau của một enum. Cuối cùng, chúng ta sẽ tìm hiểu về cấu trúc `if let`, một cách diễn đạt
thuận tiện và súc tích khác để xử lý các enum trong mã của bạn.
