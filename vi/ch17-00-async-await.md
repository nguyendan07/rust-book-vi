# Cơ bản về Lập trình Bất đồng bộ: Async, Await, Futures, và Streams

Nhiều thao tác mà chúng ta yêu cầu máy tính thực hiện có thể mất một lúc để hoàn thành. Sẽ thật
tốt nếu chúng ta có thể làm việc khác trong khi chờ đợi những
tiến trình chạy lâu đó kết thúc. Máy tính hiện đại cung cấp hai kỹ thuật để
làm việc trên nhiều thao tác cùng một lúc: tính song song (parallelism) và tính đồng thời (concurrency). Tuy nhiên,
một khi chúng ta bắt đầu viết các chương trình liên quan đến các thao tác song song hoặc đồng thời,
chúng ta sẽ nhanh chóng gặp phải những thử thách mới vốn có của _lập trình bất đồng bộ_ (asynchronous programming),
nơi các thao tác có thể không kết thúc theo đúng thứ tự mà chúng đã bắt đầu. Chương này được xây dựng dựa trên
việc sử dụng các luồng (threads) trong Chương 16 cho tính song song và đồng thời bằng cách giới thiệu một phương pháp thay thế
cho lập trình bất đồng bộ: Futures, Streams của Rust, cú pháp `async` và `await` hỗ trợ
chúng, và các công cụ để quản lý và điều phối giữa các thao tác bất đồng bộ.

Hãy xem xét một ví dụ. Giả sử bạn đang xuất một video bạn đã tạo về một buổi kỷ niệm của gia đình,
một thao tác có thể mất từ vài phút đến vài giờ.
Việc xuất video sẽ sử dụng nhiều sức mạnh CPU và GPU nhất có thể. Nếu bạn chỉ có
một lõi CPU và hệ điều hành của bạn không tạm dừng việc xuất đó cho đến khi nó hoàn thành—nghĩa là,
nếu nó thực thi việc xuất một cách _đồng bộ_ (synchronously)—bạn không thể làm bất cứ việc gì khác
trên máy tính của mình trong khi tác vụ đó đang chạy. Đó sẽ là một trải nghiệm khá bực bội.
May mắn thay, hệ điều hành của máy tính có thể, và thực tế là có, ngắt quãng việc xuất một cách âm thầm
để cho phép bạn hoàn thành các công việc khác cùng lúc.

Bây giờ giả sử bạn đang tải xuống một video do người khác chia sẻ, việc này cũng có thể mất một lúc
nhưng không chiếm nhiều thời gian CPU. Trong trường hợp này, CPU phải đợi dữ liệu
đến từ mạng. Mặc dù bạn có thể bắt đầu đọc dữ liệu khi nó bắt đầu đến, nhưng có thể mất một thời gian
để tất cả dữ liệu xuất hiện. Ngay cả khi dữ liệu đã hiện diện đầy đủ, nếu video
khá lớn, có thể mất ít nhất một hoặc hai giây để tải tất cả. Nghe có vẻ không nhiều,
nhưng đó là một khoảng thời gian rất dài đối với một bộ vi xử lý hiện đại, vốn có thể thực hiện hàng tỷ thao tác
mỗi giây. Một lần nữa, hệ điều hành của bạn sẽ âm thầm ngắt chương trình của bạn để
cho phép CPU thực hiện công việc khác trong khi đợi lệnh gọi mạng kết thúc.

Việc xuất video là một ví dụ về thao tác _ràng buộc bởi CPU_ (CPU-bound) hoặc _ràng buộc bởi tính toán_ (compute-bound).
Nó bị giới hạn bởi tốc độ xử lý dữ liệu tiềm năng của máy tính bên trong CPU hoặc GPU,
và bao nhiêu phần trăm tốc độ đó nó có thể dành cho thao tác. Tải video là một ví dụ về
thao tác _ràng buộc bởi IO_ (IO-bound), vì nó bị giới hạn bởi tốc độ _đầu vào và đầu ra_ (input and output) của máy tính;
nó chỉ có thể chạy nhanh bằng tốc độ dữ liệu có thể được gửi qua mạng.

Trong cả hai ví dụ này, các lần ngắt quãng âm thầm của hệ điều hành cung cấp một dạng
của tính đồng thời. Tuy nhiên, tính đồng thời đó chỉ xảy ra ở cấp độ của toàn bộ chương trình:
hệ điều hành ngắt quãng một chương trình để cho phép các chương trình khác làm việc.
Trong nhiều trường hợp, vì chúng ta hiểu chương trình của mình ở mức độ chi tiết hơn nhiều
so với hệ điều hành, chúng ta có thể nhận ra các cơ hội cho tính đồng thời mà hệ điều hành không thể thấy.

Ví dụ, nếu chúng ta đang xây dựng một công cụ để quản lý việc tải tệp xuống, chúng ta nên có thể
viết chương trình của mình sao cho việc bắt đầu một lượt tải xuống sẽ không làm khóa giao diện người dùng (UI),
và người dùng có thể bắt đầu nhiều lượt tải xuống cùng một lúc. Tuy nhiên, nhiều API của hệ điều hành
để tương tác với mạng là _chặn_ (blocking); nghĩa là chúng chặn tiến trình của chương trình cho đến khi
dữ liệu mà chúng đang xử lý hoàn toàn sẵn sàng.

> Ghi chú: Đây là cách mà _hầu hết_ các lệnh gọi hàm hoạt động, nếu bạn suy nghĩ về nó. Tuy nhiên,
> thuật ngữ _blocking_ thường được dành riêng cho các lệnh gọi hàm tương tác với
> tệp, mạng hoặc các tài nguyên khác trên máy tính, bởi vì đó là những trường hợp mà
> một chương trình cá nhân sẽ được hưởng lợi từ việc thao tác đó là _không_ chặn (non-blocking).

Chúng ta có thể tránh việc chặn luồng chính của mình bằng cách tạo ra một luồng chuyên dụng để
tải xuống mỗi tệp. Tuy nhiên, chi phí overhead của những luồng đó cuối cùng sẽ trở thành một vấn đề.
Sẽ tốt hơn nếu lệnh gọi không chặn ngay từ đầu. Sẽ tốt hơn nữa nếu chúng ta có thể viết theo cùng một phong cách
trực tiếp mà chúng ta sử dụng trong mã chặn, tương tự như thế này:

```rust,ignore,does_not_compile
let data = fetch_data_from(url).await;
println!("{data}");
```

Đó chính xác là những gì trừu tượng _async_ (viết tắt của _asynchronous_ - bất đồng bộ) của Rust mang lại cho chúng ta.
Trong chương này, bạn sẽ tìm hiểu tất cả về async khi chúng ta đề cập đến các chủ đề sau:

- Cách sử dụng cú pháp `async` và `await` của Rust
- Cách sử dụng mô hình async để giải quyết một số thử thách tương tự mà chúng ta đã xem xét
  ở Chương 16
- Cách đa luồng (multithreading) và async cung cấp các giải pháp bổ trợ, mà bạn có thể
  kết hợp trong nhiều trường hợp

Tuy nhiên, trước khi chúng ta xem async hoạt động như thế nào trong thực tế, chúng ta cần thực hiện một chuyến đi ngắn
để thảo luận về sự khác biệt giữa tính song song và tính đồng thời.

### Tính song song và Tính đồng thời

Chúng ta đã coi tính song song và tính đồng thời hầu như có thể thay thế cho nhau cho đến nay. Bây giờ
chúng ta cần phân biệt chúng một cách chính xác hơn, vì những điểm khác biệt sẽ
xuất hiện khi chúng ta bắt đầu làm việc.

Hãy xem xét các cách khác nhau mà một nhóm có thể chia nhỏ công việc trong một dự án phần mềm.
Bạn có thể giao cho một thành viên duy nhất nhiều tác vụ, giao cho mỗi thành viên một tác vụ, hoặc
sử dụng sự kết hợp của cả hai phương pháp.

Khi một cá nhân làm việc trên nhiều tác vụ khác nhau trước khi bất kỳ tác vụ nào trong số đó hoàn thành,
đây là _tính đồng thời_ (concurrency). Có lẽ bạn đang mở hai dự án khác nhau trên máy tính của mình,
và khi bạn cảm thấy chán hoặc bị bế tắc ở một dự án, bạn chuyển sang dự án kia.
Bạn chỉ là một người, vì vậy bạn không thể thực hiện tiến độ trên cả hai tác vụ cùng một thời điểm chính xác,
nhưng bạn có thể làm đa nhiệm, tạo ra tiến độ cho từng cái một bằng cách chuyển đổi qua lại giữa chúng (xem Hình 17-1).

<figure>

<img src="img/trpl17-01.svg" class="center" alt="Một sơ đồ với các hộp được gắn nhãn Task A và Task B, với các hình kim cương bên trong chúng đại diện cho các tác vụ con. Có các mũi tên chỉ từ A1 đến B1, B1 đến A2, A2 đến B2, B2 đến A3, A3 đến A4, và A4 đến B3. Các mũi tên giữa các tác vụ con đi xuyên qua các hộp giữa Task A và Task B." />

<figcaption>Hình 17-1: Một quy trình làm việc đồng thời, chuyển đổi giữa Tác vụ A và Tác vụ B</figcaption>

</figure>

Khi nhóm chia nhỏ một nhóm các tác vụ bằng cách để mỗi thành viên nhận một tác vụ và
làm việc trên nó một mình, đây là _tính song song_ (parallelism). Mỗi người trong nhóm có thể tạo ra tiến độ
tại cùng một thời điểm chính xác (xem Hình 17-2).

<figure>

<img src="img/trpl17-02.svg" class="center" alt="Một sơ đồ với các hộp được gắn nhãn Task A và Task B, với các hình kim cương bên trong chúng đại diện cho các tác vụ con. Có các mũi tên chỉ từ A1 đến A2, A2 đến A3, A3 đến A4, B1 đến B2, và B2 đến B3. Không có mũi tên nào đi xuyên giữa các hộp cho Task A và Task B." />

<figcaption>Hình 17-2: Một quy trình làm việc song song, nơi công việc diễn ra trên Tác vụ A và Tác vụ B một cách độc lập</figcaption>

</figure>

Trong cả hai quy trình làm việc này, bạn có thể phải điều phối giữa các
tác vụ khác nhau. Có thể bạn _đã nghĩ_ tác vụ được giao cho một người là hoàn toàn
độc lập với công việc của những người khác, nhưng thực tế nó yêu cầu một người khác
trong nhóm phải hoàn thành tác vụ của họ trước. Một số công việc có thể được thực hiện song song,
nhưng một số trong đó thực sự là _tuần tự_ (serial): nó chỉ có thể xảy ra theo một chuỗi,
tác vụ này tiếp sau tác vụ kia, như trong Hình 17-3.

<figure>

<img src="img/trpl17-03.svg" class="center" alt="Một sơ đồ với các hộp được gắn nhãn Task A và Task B, với các hình kim cương bên trong chúng đại diện cho các tác vụ con. Có các mũi tên chỉ từ A1 đến A2, A2 đến một cặp vạch thẳng đứng dày giống như biểu tượng “tạm dừng”, từ biểu tượng đó đến A3, B1 đến B2, B2 đến B3 nằm dưới biểu tượng đó, B3 đến A3, và B3 đến B4." />

<figcaption>Hình 17-3: Một quy trình làm việc song song một phần, nơi công việc diễn ra trên Tác vụ A và Tác vụ B độc lập cho đến khi Tác vụ A3 bị chặn bởi kết quả của Tác vụ B3.</figcaption>

</figure>

Tương tự, bạn có thể nhận ra rằng một trong các tác vụ của chính mình phụ thuộc vào một tác vụ khác của bạn.
Bây giờ công việc đồng thời của bạn cũng trở thành tuần tự.

Tính song song và tính đồng thời cũng có thể giao thoa với nhau. Nếu bạn biết rằng
một đồng nghiệp đang bị bế tắc cho đến khi bạn hoàn thành một trong các tác vụ của mình, bạn có thể sẽ
tập trung toàn bộ nỗ lực vào tác vụ đó để "khai thông" cho đồng nghiệp của mình. Bạn và
đồng nghiệp của mình không còn có thể làm việc song song, và bạn cũng không còn có thể làm việc
đồng thời trên các tác vụ của riêng mình.

Các động lực cơ bản tương tự cũng diễn ra với phần mềm và phần cứng. Trên một máy tính
có một lõi CPU duy nhất, CPU chỉ có thể thực hiện một thao tác tại một thời điểm, nhưng nó
vẫn có thể làm việc đồng thời. Sử dụng các công cụ như luồng (threads), tiến trình (processes) và async,
máy tính có thể tạm dừng một hoạt động và chuyển sang các hoạt động khác trước khi cuối cùng
quay trở lại hoạt động đầu tiên đó. Trên một máy tính có nhiều lõi CPU, nó cũng có thể thực hiện
công việc song song. Một lõi có thể đang thực hiện một tác vụ trong khi một lõi khác thực hiện
một tác vụ hoàn toàn không liên quan, và những thao tác đó thực sự xảy ra cùng một lúc.

Khi làm việc với async trong Rust, chúng ta luôn đối mặt với tính đồng thời.
Tùy thuộc vào phần cứng, hệ điều hành và môi trường thực thi (runtime) bất đồng bộ mà chúng ta đang sử dụng
(chúng ta sẽ nói thêm về async runtime ngay sau đây), tính đồng thời đó cũng có thể sử dụng tính song song
ở bên dưới.

Bây giờ, hãy cùng tìm hiểu xem lập trình bất đồng bộ trong Rust thực sự hoạt động như thế nào.
