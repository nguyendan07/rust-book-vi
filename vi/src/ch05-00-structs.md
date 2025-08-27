# Using Structs to Structure Related Data

Một _struct_, hoặc _structure_, là một kiểu dữ liệu tùy chỉnh cho phép bạn
gói gọn và đặt tên cho nhiều giá trị liên quan tạo thành một nhóm có ý nghĩa.
Nếu bạn quen thuộc với ngôn ngữ hướng đối tượng, một _struct_ giống như các
thuộc tính dữ liệu của một đối tượng. Trong chương này, chúng ta sẽ so sánh
và đối chiếu giữa tuples và structs để xây dựng trên những gì bạn đã biết
và minh họa khi nào structs là cách tốt hơn để nhóm dữ liệu.

Chúng ta sẽ minh họa cách định nghĩa và khởi tạo structs. Chúng ta sẽ thảo
luận về cách định nghĩa các hàm liên kết, đặc biệt là loại hàm liên kết được
gọi là _methods_, để chỉ định hành vi liên kết với kiểu struct. Structs và
enums (được thảo luận trong Chương 6) là các khối xây dựng để tạo ra các
kiểu mới trong miền của chương trình của bạn nhằm tận dụng tối đa việc kiểm
tra kiểu tại thời điểm biên dịch của Rust.
