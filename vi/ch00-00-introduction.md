# Giới thiệu

> Lưu ý: Ấn bản này của cuốn sách giống với ấn bản [The Rust Programming
> Language][nsprust] được phát hành dưới dạng sách in và ebook bởi [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-2nd-edition
[nsp]: https://nostarch.com/

Chào mừng bạn đến với _The Rust Programming Language_, cuốn sách nhập môn về Rust.
Ngôn ngữ lập trình Rust giúp bạn viết phần mềm nhanh hơn, đáng tin cậy hơn.
Tính tiện dụng cấp cao (high-level ergonomics) và khả năng kiểm soát cấp thấp (low-level control) thường mâu thuẫn với nhau trong thiết kế ngôn ngữ lập trình; Rust thách thức mâu thuẫn đó. Bằng cách cân bằng giữa năng lực kỹ thuật mạnh mẽ và trải nghiệm lập trình viên tuyệt vời, Rust mang lại cho bạn tùy chọn kiểm soát các chi tiết cấp thấp (chẳng hạn như việc sử dụng bộ nhớ) mà không gặp phải tất cả những phiền toái vốn thường gắn liền với sự kiểm soát đó.

## Rust Dành Cho Ai

Rust lý tưởng cho rất nhiều người vì nhiều lý do khác nhau. Hãy cùng điểm qua một vài nhóm đối tượng quan trọng nhất.

### Các Đội Ngũ Lập Trình Viên

Rust đang chứng minh là một công cụ làm việc hiệu quả để cộng tác giữa các nhóm lập trình viên lớn với mức độ hiểu biết khác nhau về lập trình hệ thống (systems programming). Mã nguồn cấp thấp thường dễ gặp nhiều lỗi tinh vi, mà trong hầu hết các ngôn ngữ khác chỉ có thể phát hiện được thông qua việc kiểm thử diện rộng và rà soát mã nguồn (code review) cẩn thận bởi các lập trình viên giàu kinh nghiệm. Trong Rust, trình biên dịch đóng vai trò như một người gác cổng khi từ chối biên dịch mã có các lỗi khó nắm bắt này, bao gồm cả các lỗi đồng thời (concurrency bugs). Bằng cách làm việc cùng với trình biên dịch, đội ngũ có thể dành thời gian tập trung vào logic của chương trình thay vì phải tốn công săn lùng các lỗi bộ nhớ.

Rust cũng mang các công cụ phát triển hiện đại đến với thế giới lập trình hệ thống:

- Cargo, công cụ quản lý gói phụ thuộc (dependency manager) và build tool đi kèm, giúp việc thêm, biên dịch và quản lý các phụ thuộc trở nên nhẹ nhàng và nhất quán trên toàn bộ hệ sinh thái Rust.
- Công cụ định dạng Rustfmt đảm bảo phong cách viết mã nhất quán giữa các lập trình viên.
- rust-analyzer hỗ trợ tích hợp với Môi trường Phát triển Tích hợp (IDE) để tự động hoàn thành mã (code completion) và hiển thị thông báo lỗi trực tiếp (inline error messages).

Bằng cách sử dụng các công cụ này và các công cụ khác trong hệ sinh thái Rust, các lập trình viên có thể làm việc hiệu quả trong khi viết mã nguồn ở cấp độ hệ thống.

### Học Sinh, Sinh Viên

Rust dành cho sinh viên và những ai quan tâm đến việc tìm hiểu các khái niệm hệ thống. Thông qua Rust, nhiều người đã học được các chủ đề như phát triển hệ điều hành. Cộng đồng Rust rất cởi mở và luôn sẵn lòng giải đáp thắc mắc của người học. Qua những nỗ lực như cuốn sách này, đội ngũ phát triển Rust mong muốn đưa các khái niệm hệ thống đến gần hơn với nhiều người, đặc biệt là những người mới bắt đầu lập trình.

### Các Doanh Nghiệp

Hàng trăm công ty, lớn và nhỏ, sử dụng Rust trong môi trường thực tế (production) cho nhiều tác vụ khác nhau, bao gồm các công cụ dòng lệnh (CLI tools), dịch vụ web, công cụ DevOps, thiết bị nhúng, phân tích và chuyển mã âm thanh/video, tiền mã hóa, tin sinh học, công cụ tìm kiếm, ứng dụng Internet of Things (IoT), học máy (machine learning), và thậm chí cả các phần cốt lõi của trình duyệt web Firefox.

### Các Nhà Phát Triển Mã Nguồn Mở

Rust dành cho những ai muốn xây dựng ngôn ngữ lập trình Rust, cộng đồng, các công cụ phát triển và các thư viện. Chúng tôi rất hoan nghênh sự đóng góp của bạn cho ngôn ngữ Rust.

### Những Người Đề Cao Tốc Độ và Tính Ổn Định

Rust dành cho những ai khao khát tốc độ và tính ổn định trong một ngôn ngữ. Về tốc độ, chúng tôi muốn nói đến cả tốc độ thực thi của mã Rust lẫn tốc độ mà Rust cho phép bạn viết chương trình. Các kiểm tra tĩnh của trình biên dịch Rust đảm bảo tính ổn định xuyên suốt quá trình bổ sung tính năng và tái cấu trúc mã nguồn (refactoring). Điều này hoàn toàn trái ngược với mã nguồn cũ kỹ, dễ vỡ trong các ngôn ngữ không có các kiểm tra này, nơi mà các lập trình viên thường e ngại khi phải chỉnh sửa. Bằng cách hướng tới các trừu tượng hóa không chi phí (zero-cost abstractions) — các tính năng cấp cao được biên dịch thành mã máy cấp thấp nhanh như mã viết tay — Rust nỗ lực để mã an toàn cũng đồng thời là mã có tốc độ nhanh nhất.

Ngôn ngữ Rust hy vọng sẽ hỗ trợ nhiều đối tượng người dùng khác nữa; những nhóm được đề cập ở đây chỉ là một vài đối tượng tiêu biểu. Nhìn chung, tham vọng lớn nhất của Rust là xóa bỏ sự đánh đổi mà các lập trình viên đã phải chấp nhận trong nhiều thập kỷ bằng cách mang lại sự an toàn _và_ năng suất, tốc độ _và_ tính tiện dụng. Hãy thử trải nghiệm Rust và xem liệu những lựa chọn thiết kế của nó có phù hợp với bạn không.

## Cuốn Sách Này Dành Cho Ai

Cuốn sách này giả định rằng bạn đã từng viết mã bằng một ngôn ngữ lập trình khác, nhưng không đưa ra giả định cụ thể về ngôn ngữ nào. Chúng tôi đã cố gắng làm cho tài liệu dễ tiếp cận với những người có nền tảng lập trình đa dạng. Chúng tôi không dành nhiều thời gian để nói về lập trình _là gì_ hay cách tư duy lập trình căn bản. Nếu bạn hoàn toàn mới bắt đầu học lập trình, bạn nên đọc một cuốn sách chuyên biệt về nhập môn lập trình trước.

## Cách Sử Dụng Cuốn Sách Này

Nhìn chung, cuốn sách này được thiết kế để bạn đọc tuần tự từ đầu đến cuối. Các chương sau được xây dựng dựa trên các khái niệm ở các chương trước, và các chương trước có thể chưa đi sâu vào chi tiết của một chủ đề cụ thể nhưng sẽ xem xét lại chủ đề đó trong một chương sau.

Bạn sẽ tìm thấy hai loại chương trong cuốn sách này: chương khái niệm và chương dự án. Trong các chương khái niệm, bạn sẽ tìm hiểu về một khía cạnh của Rust. Trong các chương dự án, chúng ta sẽ cùng nhau xây dựng các chương trình nhỏ, áp dụng những gì bạn đã học được. Chương 2, 12, và 21 là các chương dự án; các chương còn lại là các chương khái niệm.

Chương 1 giải thích cách cài đặt Rust, cách viết chương trình “Hello, world!”, và cách sử dụng Cargo, công cụ quản lý gói và build tool của Rust. Chương 2 là phần giới thiệu thực hành về việc viết chương trình trong Rust bằng cách xây dựng một trò chơi đoán số. Tại đây, chúng tôi giới thiệu các khái niệm ở mức tổng quan, và các chương sau sẽ cung cấp thêm chi tiết. Nếu bạn muốn bắt tay vào viết mã ngay lập tức, Chương 2 là nơi dành cho bạn. Chương 3 đề cập đến các tính năng của Rust tương tự như các ngôn ngữ lập trình khác, và trong Chương 4 bạn sẽ tìm hiểu về hệ thống quyền sở hữu (ownership) độc đáo của Rust. Nếu bạn là người học tỉ mỉ và muốn nắm rõ từng chi tiết trước khi chuyển sang phần tiếp theo, bạn có thể bỏ qua Chương 2 và đi thẳng vào Chương 3, sau đó quay lại Chương 2 khi bạn muốn thực hiện một dự án áp dụng các chi tiết đã học.

Chương 5 thảo luận về struct và method (phương thức), và Chương 6 đề cập đến enum, biểu thức `match`, và cấu trúc điều khiển `if let`. Bạn sẽ sử dụng struct và enum để tạo các kiểu dữ liệu tùy chỉnh trong Rust.

Trong Chương 7, bạn sẽ tìm hiểu về hệ thống module của Rust và các quy tắc phạm vi/tính riêng tư (privacy rules) để tổ chức mã nguồn và Giao diện Lập trình Ứng dụng (API) công khai của bạn. Chương 8 thảo luận về một số cấu trúc dữ liệu tập hợp (collection) phổ biến mà thư viện chuẩn cung cấp, chẳng hạn như vector, string và hash map. Chương 9 khám phá triết lý và kỹ thuật xử lý lỗi trong Rust.

Chương 10 đào sâu vào generic, trait, và lifetime (vòng đời tham chiếu), mang lại cho bạn khả năng định nghĩa mã áp dụng cho nhiều kiểu dữ liệu. Chương 11 là tất cả về kiểm thử (testing), điều cần thiết để đảm bảo logic chương trình của bạn là chính xác ngay cả khi Rust đã đảm bảo an toàn bộ nhớ. Trong Chương 12, chúng ta sẽ tự xây dựng một bản triển khai cho một tập hợp con chức năng của công cụ dòng lệnh `grep` để tìm kiếm văn bản trong các tệp. Dự án này sẽ sử dụng nhiều khái niệm đã thảo luận trong các chương trước.

Chương 13 khám phá closure và iterator: các tính năng của Rust bắt nguồn từ các ngôn ngữ lập trình hàm (functional programming). Trong Chương 14, chúng ta sẽ xem xét Cargo sâu hơn và nói về các phương pháp hay nhất để chia sẻ thư viện của bạn với người khác. Chương 15 thảo luận về con trỏ thông minh (smart pointers) mà thư viện chuẩn cung cấp và các trait hỗ trợ chức năng của chúng.

Trong Chương 16, chúng ta sẽ đi qua các mô hình lập trình đồng thời khác nhau và nói về cách Rust giúp bạn lập trình đa luồng (multithreading) mà không phải lo sợ lỗi bộ nhớ (fearless concurrency). Trong Chương 17, chúng ta tiếp tục khám phá cú pháp async/await của Rust, cùng với task, future, và stream, cũng như mô hình đồng thời bất đồng bộ gọn nhẹ mà chúng mang lại.

Chương 18 xem xét cách các quy ước của Rust so sánh với các nguyên tắc lập trình hướng đối tượng (OOP) mà bạn có thể đã quen thuộc. Chương 19 là tài liệu tham khảo về pattern và khớp mẫu (pattern matching), là những cách diễn đạt ý tưởng mạnh mẽ xuyên suốt các chương trình Rust. Chương 20 chứa một tập hợp phong phú các chủ đề nâng cao đáng chú ý, bao gồm Unsafe Rust, macro, và tìm hiểu sâu hơn về lifetime, trait, type, function, và closure.

Trong Chương 21, chúng ta sẽ hoàn thành một dự án trọn vẹn: triển khai một web server đa luồng cấp thấp!

Cuối cùng, một số phụ lục chứa thông tin hữu ích về ngôn ngữ theo định dạng tham khảo nhanh. **Phụ lục A** bao gồm các từ khóa của Rust, **Phụ lục B** bao gồm các toán tử và ký hiệu của Rust, **Phụ lục C** bao gồm các derivable trait do thư viện chuẩn cung cấp, **Phụ lục D** bao gồm một số công cụ phát triển hữu ích, và **Phụ lục E** giải thích các ấn bản (editions) của Rust. Trong **Phụ lục F**, bạn có thể tìm thấy các bản dịch của cuốn sách, và trong **Phụ lục G** chúng ta sẽ tìm hiểu cách Rust được phát triển và Rust nightly là gì.

Không có cách nào là sai khi đọc cuốn sách này: nếu bạn muốn đọc lướt hoặc nhảy cóc, hãy cứ tự nhiên! Bạn có thể quay lại các chương trước nếu gặp phải bất kỳ sự nhầm lẫn nào. Hãy chọn bất cứ cách nào hiệu quả nhất đối với bạn.

<span id="ferris"></span>

Một phần quan trọng của quá trình học Rust là học cách đọc các thông báo lỗi mà trình biên dịch hiển thị: chúng sẽ hướng dẫn bạn cách sửa mã hoạt động đúng. Do đó, chúng tôi sẽ cung cấp nhiều ví dụ không biên dịch được cùng với thông báo lỗi mà trình biên dịch sẽ hiển thị trong từng tình huống. Lưu ý rằng nếu bạn sao chép và chạy một ví dụ ngẫu nhiên, ví dụ đó có thể không biên dịch được! Hãy đảm bảo bạn đọc phần văn bản xung quanh để xem liệu ví dụ bạn đang cố gắng chạy có chủ đích tạo ra lỗi hay không. Chú cua Ferris cũng sẽ giúp bạn phân biệt mã nào không hoạt động theo thiết kế:

| Ferris                                                                                                           | Ý nghĩa                                          |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris với một dấu hỏi"/>            | Mã này không biên dịch được!                      |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris giơ hai càng lên trời"/>                   | Mã này gây ra lỗi panic!                                |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris giơ một càng, nhún vai"/> | Mã này không tạo ra hành vi mong muốn. |

Trong hầu hết các tình huống, chúng tôi sẽ dẫn dắt bạn đến phiên bản chính xác của bất kỳ đoạn mã nào không biên dịch được.

## Mã Nguồn

Các tệp mã nguồn từ đó cuốn sách này được tạo ra có thể được tìm thấy trên [GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src
