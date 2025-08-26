# Giới thiệu

> Ghi chú: Phiên bản này của cuốn sách giống với [The Rust Programming
> Language][nsprust] có sẵn ở dạng bản in và ebook từ [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-2nd-edition
[nsp]: https://nostarch.com/

Chào mừng bạn đến với _Ngôn ngữ lập trình Rust_, một cuốn sách giới thiệu về Rust.
Ngôn ngữ lập trình Rust giúp bạn viết phần mềm nhanh hơn, đáng tin cậy hơn.
Tính tiện dụng cấp cao và khả năng kiểm soát cấp thấp thường mâu thuẫn với nhau trong thiết kế ngôn ngữ lập trình; Rust thách thức sự xung đột đó. Bằng cách cân bằng giữa năng lực kỹ thuật mạnh mẽ và trải nghiệm tuyệt vời cho nhà phát triển, Rust cho bạn tùy chọn kiểm soát các chi tiết cấp thấp (như sử dụng bộ nhớ) mà không gặp phải những rắc rối thường đi kèm với việc kiểm soát đó.

## Rust dành cho ai

Rust là lựa chọn lý tưởng cho nhiều người vì nhiều lý do khác nhau. Hãy cùng xem xét một vài nhóm quan trọng nhất.

### Các nhóm phát triển

Rust đang chứng tỏ là một công cụ hiệu quả cho việc hợp tác giữa các nhóm lớn các nhà phát triển với các trình độ kiến thức lập trình hệ thống khác nhau. Mã nguồn cấp thấp dễ gặp phải nhiều lỗi tinh vi, mà ở hầu hết các ngôn ngữ khác chỉ có thể phát hiện được thông qua kiểm thử sâu rộng và đánh giá mã nguồn cẩn thận bởi các nhà phát triển có kinh nghiệm. Trong Rust, trình biên dịch đóng vai trò như một người gác cổng bằng cách từ chối biên dịch mã nguồn có chứa những lỗi khó phát hiện này, bao gồm cả lỗi tương tranh (concurrency bugs). Bằng cách làm việc cùng với trình biên dịch, nhóm có thể dành thời gian tập trung vào logic của chương trình thay vì đuổi theo các lỗi.

Rust cũng mang đến các công cụ phát triển hiện đại cho thế giới lập trình hệ thống:

- Cargo, trình quản lý phụ thuộc và công cụ xây dựng đi kèm, giúp việc thêm, biên dịch và quản lý các phụ thuộc trở nên dễ dàng và nhất quán trong toàn bộ hệ sinh thái Rust.
- Công cụ định dạng Rustfmt đảm bảo một phong cách viết mã nhất quán giữa các nhà phát triển.
- rust-analyzer cung cấp sức mạnh cho việc tích hợp Môi trường phát triển tích hợp (IDE) để hoàn thành mã và hiển thị thông báo lỗi nội tuyến.

Bằng cách sử dụng những công cụ này và các công cụ khác trong hệ sinh thái Rust, các nhà phát triển có thể làm việc hiệu quả khi viết mã ở cấp độ hệ thống.

### Sinh viên

Rust dành cho sinh viên và những người quan tâm đến việc học các khái niệm về hệ thống. Sử dụng Rust, nhiều người đã học về các chủ đề như phát triển hệ điều hành. Cộng đồng rất chào đón và sẵn lòng trả lời các câu hỏi của sinh viên. Thông qua những nỗ lực như cuốn sách này, các nhóm Rust muốn làm cho các khái niệm hệ thống trở nên dễ tiếp cận hơn với nhiều người hơn, đặc biệt là những người mới bắt đầu lập trình.

### Các công ty

Hàng trăm công ty, lớn và nhỏ, sử dụng Rust trong môi trường sản phẩm cho nhiều tác vụ khác nhau, bao gồm các công cụ dòng lệnh, dịch vụ web, công cụ DevOps, thiết bị nhúng, phân tích và chuyển mã âm thanh và video, tiền điện tử, tin sinh học, công cụ tìm kiếm, ứng dụng Internet of Things, học máy, và thậm chí cả các phần chính của trình duyệt web Firefox.

### Các nhà phát triển mã nguồn mở

Rust dành cho những người muốn xây dựng ngôn ngữ lập trình Rust, cộng đồng, công cụ cho nhà phát triển và các thư viện. Chúng tôi rất mong bạn đóng góp cho ngôn ngữ Rust.

### Những người coi trọng tốc độ và sự ổn định

Rust dành cho những người khao khát tốc độ và sự ổn định trong một ngôn ngữ. Nói về tốc độ, chúng tôi muốn nói đến cả tốc độ chạy của mã Rust và tốc độ mà Rust cho phép bạn viết chương trình. Các kiểm tra của trình biên dịch Rust đảm bảo sự ổn định thông qua việc thêm tính năng và tái cấu trúc mã. Điều này trái ngược với mã nguồn cũ dễ vỡ trong các ngôn ngữ không có những kiểm tra này, mà các nhà phát triển thường ngại sửa đổi. Bằng cách phấn đấu cho các trừu tượng hóa không chi phí (zero-cost abstractions)—các tính năng cấp cao biên dịch thành mã cấp thấp nhanh như mã được viết thủ công—Rust nỗ lực để làm cho mã an toàn cũng là mã nhanh.

Ngôn ngữ Rust cũng hy vọng sẽ hỗ trợ nhiều người dùng khác; những người được đề cập ở đây chỉ là một vài trong số các bên liên quan lớn nhất. Nhìn chung, tham vọng lớn nhất của Rust là loại bỏ những đánh đổi mà các lập trình viên đã chấp nhận trong nhiều thập kỷ bằng cách cung cấp sự an toàn _và_ năng suất, tốc độ _và_ tính tiện dụng. Hãy thử Rust và xem liệu các lựa chọn của nó có phù hợp với bạn không.

## Cuốn sách này dành cho ai

Cuốn sách này giả định rằng bạn đã từng viết mã bằng một ngôn ngữ lập trình khác nhưng không đưa ra bất kỳ giả định nào về đó là ngôn ngữ nào. Chúng tôi đã cố gắng làm cho tài liệu có thể tiếp cận rộng rãi với những người có nền tảng lập trình đa dạng. Chúng tôi không dành nhiều thời gian để nói về lập trình _là_ gì hay cách suy nghĩ về nó. Nếu bạn hoàn toàn mới với lập trình, bạn nên đọc một cuốn sách chuyên về giới thiệu lập trình.

## Cách sử dụng cuốn sách này

Nhìn chung, cuốn sách này giả định rằng bạn đang đọc nó theo thứ tự từ đầu đến cuối. Các chương sau sẽ xây dựng dựa trên các khái niệm trong các chương trước, và các chương trước có thể không đi sâu vào chi tiết về một chủ đề cụ thể nhưng sẽ quay lại chủ đề đó trong một chương sau.

Bạn sẽ tìm thấy hai loại chương trong cuốn sách này: chương khái niệm và chương dự án. Trong các chương khái niệm, bạn sẽ học về một khía cạnh của Rust. Trong các chương dự án, chúng ta sẽ cùng nhau xây dựng các chương trình nhỏ, áp dụng những gì bạn đã học được cho đến nay. Chương 2, 12 và 21 là các chương dự án; phần còn lại là các chương khái niệm.

Chương 1 giải thích cách cài đặt Rust, cách viết chương trình “Hello, world!” và cách sử dụng Cargo, trình quản lý gói và công cụ xây dựng của Rust. Chương 2 là phần giới thiệu thực hành về việc viết một chương trình bằng Rust, yêu cầu bạn xây dựng một trò chơi đoán số. Ở đây chúng ta sẽ đề cập đến các khái niệm ở mức độ cao, và các chương sau sẽ cung cấp thêm chi tiết. Nếu bạn muốn bắt tay vào làm ngay, Chương 2 là nơi dành cho bạn. Chương 3 đề cập đến các tính năng của Rust tương tự như các ngôn ngữ lập trình khác, và trong Chương 4 bạn sẽ học về hệ thống sở hữu của Rust. Nếu bạn là một người học đặc biệt tỉ mỉ, thích học mọi chi tiết trước khi chuyển sang phần tiếp theo, bạn có thể muốn bỏ qua Chương 2 và đi thẳng đến Chương 3, sau đó quay lại Chương 2 khi bạn muốn làm một dự án áp dụng các chi tiết đã học.

Chương 5 thảo luận về struct và phương thức, và Chương 6 đề cập đến enum, biểu thức `match`, và cấu trúc điều khiển luồng `if let`. Bạn sẽ sử dụng struct và enum để tạo các kiểu tùy chỉnh trong Rust.

Trong Chương 7, bạn sẽ học về hệ thống mô-đun của Rust và về các quy tắc riêng tư để tổ chức mã nguồn và Giao diện lập trình ứng dụng (API) công khai của nó. Chương 8 thảo luận về một số cấu trúc dữ liệu tập hợp phổ biến mà thư viện chuẩn cung cấp, chẳng hạn như vector, chuỗi và hash map. Chương 9 khám phá triết lý và kỹ thuật xử lý lỗi của Rust.

Chương 10 đi sâu vào generics, traits và lifetimes, những thứ cho bạn sức mạnh để định nghĩa mã áp dụng cho nhiều kiểu. Chương 11 hoàn toàn về kiểm thử, mà ngay cả với các đảm bảo an toàn của Rust, vẫn cần thiết để đảm bảo logic chương trình của bạn là chính xác. Trong Chương 12, chúng ta sẽ xây dựng một triển khai riêng của một tập hợp con chức năng từ công cụ dòng lệnh `grep` dùng để tìm kiếm văn bản trong các tệp. Để làm điều này, chúng ta sẽ sử dụng nhiều khái niệm đã thảo luận trong các chương trước.

Chương 13 khám phá closure và iterator: các tính năng của Rust đến từ các ngôn ngữ lập trình chức năng. Trong Chương 14, chúng ta sẽ xem xét Cargo sâu hơn và nói về các thực hành tốt nhất để chia sẻ thư viện của bạn với người khác. Chương 15 thảo luận về các con trỏ thông minh mà thư viện chuẩn cung cấp và các trait cho phép chức năng của chúng.

Trong Chương 16, chúng ta sẽ đi qua các mô hình lập trình tương tranh khác nhau và nói về cách Rust giúp bạn lập trình đa luồng một cách không sợ hãi. Trong Chương 17, chúng ta sẽ xây dựng trên đó bằng cách khám phá cú pháp async và await của Rust, cùng với các task, future và stream, và mô hình tương tranh nhẹ mà chúng cho phép.

Chương 18 xem xét cách các thành ngữ Rust so sánh với các nguyên tắc lập trình hướng đối tượng mà bạn có thể đã quen thuộc. Chương 19 là một tài liệu tham khảo về các mẫu và khớp mẫu, là những cách mạnh mẽ để thể hiện ý tưởng trong các chương trình Rust. Chương 20 chứa một tập hợp các chủ đề nâng cao đáng quan tâm, bao gồm Rust không an toàn, macro, và nhiều hơn nữa về lifetimes, traits, types, functions, và closures.

Trong Chương 21, chúng ta sẽ hoàn thành một dự án trong đó chúng ta sẽ triển khai một máy chủ web đa luồng cấp thấp!

Cuối cùng, một số phụ lục chứa thông tin hữu ích về ngôn ngữ ở định dạng giống như tài liệu tham khảo hơn. **Phụ lục A** bao gồm các từ khóa của Rust, **Phụ lục B** bao gồm các toán tử và ký hiệu của Rust, **Phụ lục C** bao gồm các trait có thể dẫn xuất được cung cấp bởi thư viện chuẩn, **Phụ lục D** bao gồm một số công cụ phát triển hữu ích, và **Phụ lục E** giải thích về các phiên bản Rust (editions). Trong **Phụ lục F**, bạn có thể tìm thấy các bản dịch của cuốn sách, và trong **Phụ lục G** chúng ta sẽ đề cập đến cách Rust được tạo ra và nightly Rust là gì.

Không có cách nào sai để đọc cuốn sách này: nếu bạn muốn bỏ qua, cứ tự nhiên! Bạn có thể phải quay lại các chương trước nếu gặp bất kỳ sự nhầm lẫn nào. Nhưng hãy làm bất cứ điều gì phù hợp với bạn.

<span id="ferris"></span>

Một phần quan trọng của quá trình học Rust là học cách đọc các thông báo lỗi mà trình biên dịch hiển thị: chúng sẽ hướng dẫn bạn đến mã nguồn hoạt động được. Do đó, chúng tôi sẽ cung cấp nhiều ví dụ không biên dịch được cùng với thông báo lỗi mà trình biên dịch sẽ hiển thị cho bạn trong mỗi tình huống. Hãy biết rằng nếu bạn nhập và chạy một ví dụ ngẫu nhiên, nó có thể không biên dịch được! Hãy chắc chắn rằng bạn đọc văn bản xung quanh để xem liệu ví dụ bạn đang cố chạy có chủ đích gây ra lỗi hay không. Ferris cũng sẽ giúp bạn phân biệt mã không được thiết kế để hoạt động:

| Ferris                                                                                                           | Ý nghĩa                                          |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris with a question mark"/>            | Mã này không biên dịch được!                      |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris throwing up their hands"/>                   | Mã này panic!                                |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris with one claw up, shrugging"/> | Mã này không tạo ra hành vi mong muốn. |

Trong hầu hết các tình huống, chúng tôi sẽ dẫn bạn đến phiên bản đúng của bất kỳ mã nào không biên dịch được.

## Mã nguồn

Các tệp nguồn mà từ đó cuốn sách này được tạo ra có thể được tìm thấy trên [GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src

