## Xây dựng concurrency linh hoạt nhờ các trait `Send` và `Sync`

<!-- Old link, do not remove -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>

Thật thú vị, hầu như mọi tính năng concurrency mà chúng ta đã nói đến cho đến nay trong
chương này đều là một phần của thư viện tiêu chuẩn, không phải của ngôn ngữ. Các
tùy chọn của bạn để xử lý concurrency không bị giới hạn trong ngôn ngữ hoặc thư viện tiêu chuẩn;
bạn có thể viết các tính năng concurrency của riêng mình hoặc sử dụng những tính năng do
người khác viết.

Tuy nhiên, trong số các khái niệm concurrency chính được nhúng vào ngôn ngữ
thay vì thư viện tiêu chuẩn là các trait `std::marker` gồm `Send` và
`Sync`.

### Cho phép chuyển giao quyền sở hữu giữa các luồng với `Send`

Trait đánh dấu (marker trait) `Send` chỉ ra rằng quyền sở hữu các giá trị của kiểu
triển khai `Send` có thể được chuyển giao giữa các luồng. Hầu hết mọi kiểu dữ liệu của Rust
đều là `Send`, nhưng có một số ngoại lệ, bao gồm `Rc<T>`: kiểu này không thể
triển khai `Send` bởi vì nếu bạn nhân bản một giá trị `Rc<T>` và cố gắng chuyển
quyền sở hữu của bản sao sang một luồng khác, cả hai luồng có thể cập nhật
số lượng tham chiếu cùng một lúc. Vì lý do này, `Rc<T>` được triển khai để
sử dụng trong các tình huống đơn luồng nơi bạn không muốn trả giá cho hình phạt hiệu suất
an toàn luồng.

Do đó, hệ thống kiểu và các ràng buộc trait của Rust đảm bảo rằng bạn không bao giờ có thể
vô tình gửi một giá trị `Rc<T>` qua các luồng một cách không an toàn. Khi chúng ta cố gắng làm
điều này trong Listing 16-14, chúng ta đã nhận được lỗi `the trait Send is not implemented for
Rc<Mutex<i32>>`. Khi chúng ta chuyển sang `Arc<T>`, kiểu có triển khai `Send`,
mã đã biên dịch được.

Bất kỳ kiểu dữ liệu nào được cấu thành hoàn toàn từ các kiểu `Send` cũng sẽ tự động được đánh dấu là `Send`.
Hầu như tất cả các kiểu nguyên thủy đều là `Send`, ngoại trừ các con trỏ thô (raw pointers), mà
chúng ta sẽ thảo luận trong Chương 20.

### Cho phép truy cập từ nhiều luồng với `Sync`

Trait đánh dấu `Sync` chỉ ra rằng một kiểu dữ liệu triển khai `Sync` là an toàn
để được tham chiếu từ nhiều luồng. Nói cách khác, bất kỳ kiểu `T` nào
triển khai `Sync` nếu `&T` (một tham chiếu bất biến đến `T`) triển khai `Send`,
nghĩa là tham chiếu đó có thể được gửi an toàn đến một luồng khác. Tương tự như `Send`,
các kiểu nguyên thủy đều triển khai `Sync`, và các kiểu được cấu thành hoàn toàn từ các kiểu
triển khai `Sync` cũng triển khai `Sync`.

<!-- BEGIN INTERVENTION: 43081862-aac8-4e18-9c55-1107ea4c7cc1 -->

`Sync` là khái niệm trong Rust gần giống nhất với ý nghĩa thông thường của cụm từ "thread-safe" (an toàn luồng), tức là một mẩu dữ liệu cụ thể có thể được sử dụng an toàn bởi nhiều luồng đồng thời. Lý do cho việc có các trait `Send` và `Sync` riêng biệt là vì một kiểu đôi khi có thể là một trong hai, hoặc cả hai, hoặc không cái nào. Ví dụ:

- Con trỏ thông minh `Rc<T>` cũng không phải `Send` cũng không phải `Sync`, vì những lý do đã mô tả ở trên.
- Kiểu `RefCell<T>` (mà chúng ta đã nói ở Chương 15) và
  gia đình các kiểu `Cell<T>` liên quan là `Send` (nếu `T: Send`), nhưng chúng không phải `Sync`. Một `RefCell` có thể được gửi qua ranh giới luồng, nhưng không thể được truy cập đồng thời vì việc triển khai kiểm tra mượn (borrow checking) mà `RefCell<T>` thực hiện tại thời điểm chạy không an toàn cho luồng.
- Con trỏ thông minh `Mutex<T>` là `Send` và `Sync`, và có thể được sử dụng để chia sẻ quyền truy cập với nhiều luồng như bạn đã thấy trong phần [“Chia sẻ một `Mutex<T>` giữa nhiều luồng”][sharing-a-mutext-between-multiple-threads]<!-- ignore -->.
- Kiểu `MutexGuard<'a, T>` được trả về bởi `Mutex::lock` là `Sync` (nếu `T: Sync`) nhưng không phải `Send`. Nó cụ thể không phải là `Send` vì [một số nền tảng yêu cầu rằng các mutex phải được mở khóa bởi chính luồng đã khóa chúng][mutex-guards-are-not-send].

<!-- END INTERVENTION: 43081862-aac8-4e18-9c55-1107ea4c7cc1 -->

### Việc triển khai `Send` và `Sync` thủ công là không an toàn (Unsafe)

Bởi vì các kiểu được cấu thành hoàn toàn từ các kiểu khác đã triển khai các trait `Send` và
`Sync` cũng tự động triển khai `Send` và `Sync`, nên chúng ta không cần phải
triển khai các trait đó một cách thủ công. Là các trait đánh dấu, chúng thậm chí không có bất kỳ
phương thức nào để triển khai. Chúng chỉ hữu ích cho việc thực thi các bất biến (invariants) liên quan đến
concurrency.

Việc triển khai thủ công các trait này liên quan đến việc triển khai mã Rust không an toàn (unsafe).
Chúng ta sẽ nói về việc sử dụng mã Rust không an toàn trong Chương 20; hiện tại, thông tin
quan trọng là việc xây dựng các kiểu đồng thời mới không được tạo thành từ các phần `Send` và
`Sync` đòi hỏi sự suy nghĩ cẩn thận để duy trì các đảm bảo an toàn. [“The
Rustonomicon”][nomicon] có thêm thông tin về các đảm bảo này và cách
duy trì chúng.

## Tóm tắt

Đây không phải là lần cuối cùng bạn thấy concurrency trong cuốn sách này: chương tiếp theo
tập trung vào lập trình async, và dự án trong Chương 21 sẽ sử dụng các
khái niệm trong chương này trong một tình huống thực tế hơn so với các ví dụ nhỏ
được thảo luận ở đây.

Như đã đề cập trước đó, vì rất ít cách Rust xử lý concurrency là
một phần của ngôn ngữ, nên nhiều giải pháp concurrency được triển khai dưới dạng các crate.
Chúng tiến hóa nhanh hơn thư viện tiêu chuẩn, vì vậy hãy nhớ tìm kiếm
trực tuyến các crate hiện đại, tiên tiến nhất để sử dụng trong các tình huống đa luồng.

Thư viện tiêu chuẩn của Rust cung cấp các kênh cho việc truyền thông điệp và các kiểu
con trỏ thông minh, chẳng hạn như `Mutex<T>` và `Arc<T>`, an toàn để sử dụng trong
các ngữ cảnh đồng thời. Hệ thống kiểu và trình kiểm tra mượn (borrow checker) đảm bảo rằng
mã sử dụng các giải pháp này sẽ không kết thúc với các lỗi tranh đua dữ liệu (data races) hoặc các tham chiếu không hợp lệ.
Một khi bạn làm cho mã của mình biên dịch được, bạn có thể yên tâm rằng nó sẽ chạy vui vẻ
trên nhiều luồng mà không gặp phải các loại lỗi khó theo dõi vốn phổ biến trong
các ngôn ngữ khác. Lập trình đồng thời không còn là một khái niệm đáng sợ nữa:
hãy tiến lên và làm cho các chương trình của bạn trở nên đồng thời, một cách không sợ hãi!

{{#quiz ../quizzes/ch16-04-extensible-concurrency-send-and-sync.toml}}

[sharing-a-mutext-between-multiple-threads]: ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
[mutex-guards-are-not-send]: https://github.com/rust-lang/rust/issues/23465#issuecomment-82730326
