# Sử dụng Struct để Cấu trúc Dữ liệu Liên quan

Một _struct_, hay _cấu trúc_, là một kiểu dữ liệu tùy chỉnh cho phép bạn đóng gói
và đặt tên cho nhiều giá trị liên quan tạo nên một nhóm có ý nghĩa. Nếu
bạn đã quen thuộc với một ngôn ngữ hướng đối tượng, một _struct_ giống như
các thuộc tính dữ liệu của một đối tượng. Trong chương này, chúng ta sẽ so sánh và đối chiếu
tuple với struct để xây dựng dựa trên những gì bạn đã biết và minh họa khi nào
struct là cách tốt hơn để nhóm dữ liệu.

Chúng tôi sẽ minh họa cách định nghĩa và khởi tạo struct. Chúng tôi sẽ thảo luận cách
định nghĩa các hàm liên kết, đặc biệt là loại hàm liên kết được gọi là
_phương thức_ (method), để chỉ định hành vi gắn liền với một kiểu struct. Struct và enum
(được thảo luận trong Chương 6) là những viên gạch nền tảng để tạo ra các kiểu mới
trong miền nghiệp vụ chương trình của bạn nhằm tận dụng tối đa khả năng kiểm tra kiểu
tại thời điểm biên dịch của Rust.
