# Hiểu về Quyền sở hữu (Ownership)

Quyền sở hữu (Ownership) là tính năng độc đáo nhất của Rust và có ảnh hưởng sâu
sắc đến toàn bộ ngôn ngữ. Nó cho phép Rust đảm bảo an toàn bộ nhớ mà
không cần bộ thu gom rác (garbage collector - GC), vì vậy việc hiểu cách ownership
hoạt động là nền tảng tối quan trọng. Trong chương này, chúng ta sẽ tìm hiểu về ownership cũng
như các tính năng liên quan chặt chẽ: mượn (borrowing), lát cắt/phân đoạn (slices), và cách Rust
tổ chức dữ liệu trong bộ nhớ (Stack & Heap).

> [!NOTE]
> **Khác biệt cốt lõi với Python:**
> Trong Python, bộ nhớ được quản lý tự động bởi Garbage Collector (kết hợp Reference Counting và Generational GC) tại thời gian chạy (runtime). Điều này giúp lập trình viên không phải quản lý bộ nhớ nhưng đánh đổi bằng việc tiêu tốn nhiều RAM hơn và có thể phát sinh độ trễ lúc chạy.
> Ngược lại, Rust sử dụng hệ thống **Ownership**: quyền sở hữu và thời gian giải phóng bộ nhớ được kiểm tra và xác định chính xác ngay tại thời điểm biên dịch (compile-time), giúp chương trình đạt hiệu năng tối đa và không tốn chi phí runtime.
