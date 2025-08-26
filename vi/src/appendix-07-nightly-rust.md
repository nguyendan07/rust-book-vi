## Phụ lục G - Cách Rust được tạo ra và “Rust Nightly”

Phụ lục này nói về cách Rust được tạo ra và điều đó ảnh hưởng đến bạn như thế nào với tư cách là một nhà phát triển Rust.

### Ổn định không trì trệ

Là một ngôn ngữ, Rust quan tâm _rất nhiều_ đến sự ổn định của mã của bạn. Chúng tôi muốn Rust là một nền tảng vững chắc mà bạn có thể xây dựng, và nếu mọi thứ liên tục thay đổi, điều đó sẽ là không thể. Đồng thời, nếu chúng ta không thể thử nghiệm các tính năng mới, chúng ta có thể không phát hiện ra những sai sót quan trọng cho đến sau khi chúng được phát hành, khi chúng ta không còn có thể thay đổi mọi thứ nữa.

Giải pháp của chúng tôi cho vấn đề này là cái mà chúng tôi gọi là “ổn định không trì trệ”, và nguyên tắc chỉ đạo của chúng tôi là: bạn không bao giờ phải sợ hãi khi nâng cấp lên phiên bản Rust ổn định mới. Mỗi lần nâng cấp đều phải dễ dàng, nhưng cũng mang lại cho bạn các tính năng mới, ít lỗi hơn và thời gian biên dịch nhanh hơn.

### Choo, Choo! Các kênh phát hành và đi theo đoàn tàu

Việc phát triển Rust hoạt động theo một _lịch trình tàu hỏa_. Tức là, tất cả việc phát triển đều được thực hiện trên nhánh `master` của kho mã Rust. Các bản phát hành tuân theo mô hình tàu phát hành phần mềm, đã được sử dụng bởi Cisco IOS và các dự án phần mềm khác. Có ba _kênh phát hành_ cho Rust:

- Nightly
- Beta
- Stable

Hầu hết các nhà phát triển Rust chủ yếu sử dụng kênh stable, nhưng những người muốn thử các tính năng mới thử nghiệm có thể sử dụng nightly hoặc beta.

Đây là một ví dụ về cách quy trình phát triển và phát hành hoạt động: giả sử rằng nhóm Rust đang làm việc trên bản phát hành Rust 1.5. Bản phát hành đó đã xảy ra vào tháng 12 năm 2015, nhưng nó sẽ cung cấp cho chúng ta các số phiên bản thực tế. Một tính năng mới được thêm vào Rust: một commit mới được đưa vào nhánh `master`. Mỗi đêm, một phiên bản nightly mới của Rust được tạo ra. Mỗi ngày là một ngày phát hành, và các bản phát hành này được tạo ra tự động bởi cơ sở hạ tầng phát hành của chúng tôi. Vì vậy, theo thời gian, các bản phát hành của chúng tôi trông như thế này, mỗi đêm một lần:

```text
nightly: * - - * - - *
```

Cứ sáu tuần một lần, đã đến lúc chuẩn bị một bản phát hành mới! Nhánh `beta` của kho mã Rust được tách ra từ nhánh `master` được sử dụng bởi nightly. Bây giờ, có hai bản phát hành:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Hầu hết người dùng Rust không sử dụng các bản phát hành beta một cách tích cực, nhưng họ kiểm thử với beta trong hệ thống CI của mình để giúp Rust phát hiện các lỗi hồi quy có thể xảy ra. Trong khi đó, vẫn có một bản phát hành nightly mỗi đêm:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Giả sử một lỗi hồi quy được tìm thấy. Thật may là chúng ta đã có thời gian để kiểm thử bản phát hành beta trước khi lỗi hồi quy lọt vào một bản phát hành ổn định! Bản vá được áp dụng cho `master`, để nightly được sửa, và sau đó bản vá được đưa ngược lại vào nhánh `beta`, và một bản phát hành beta mới được tạo ra:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Sáu tuần sau khi bản beta đầu tiên được tạo ra, đã đến lúc cho một bản phát hành ổn định! Nhánh `stable` được tạo ra từ nhánh `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Hoan hô! Rust 1.5 đã hoàn thành! Tuy nhiên, chúng ta đã quên một điều: vì sáu tuần đã trôi qua, chúng ta cũng cần một bản beta mới của phiên bản Rust _tiếp theo_, 1.6. Vì vậy, sau khi `stable` tách ra từ `beta`, phiên bản `beta` tiếp theo lại tách ra từ `nightly`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Đây được gọi là “mô hình tàu hỏa” vì cứ sáu tuần một lần, một bản phát hành “rời ga”, nhưng vẫn phải đi qua kênh beta trước khi đến được dưới dạng một bản phát hành ổn định.

Rust phát hành sáu tuần một lần, như một chiếc đồng hồ. Nếu bạn biết ngày của một bản phát hành Rust, bạn có thể biết ngày của bản phát hành tiếp theo: đó là sáu tuần sau. Một khía cạnh hay của việc có các bản phát hành được lên lịch sáu tuần một lần là chuyến tàu tiếp theo sẽ sớm đến. Nếu một tính năng tình cờ bỏ lỡ một bản phát hành cụ thể, không cần phải lo lắng: một bản phát hành khác sẽ diễn ra trong một thời gian ngắn! Điều này giúp giảm áp lực phải đưa các tính năng có thể chưa được hoàn thiện vào gần thời hạn phát hành.

Nhờ quy trình này, bạn luôn có thể kiểm tra bản dựng tiếp theo của Rust và tự mình xác minh rằng việc nâng cấp rất dễ dàng: nếu một bản phát hành beta không hoạt động như mong đợi, bạn có thể báo cáo cho nhóm và được sửa chữa trước khi bản phát hành ổn định tiếp theo diễn ra! Sự cố trong một bản phát hành beta tương đối hiếm, nhưng `rustc` vẫn là một phần mềm, và lỗi vẫn tồn tại.

### Thời gian bảo trì

Dự án Rust hỗ trợ phiên bản ổn định gần đây nhất. Khi một phiên bản ổn định mới được phát hành, phiên bản cũ sẽ hết vòng đời (EOL). Điều này có nghĩa là mỗi phiên bản được hỗ trợ trong sáu tuần.

### Các tính năng không ổn định

Có một điểm cần lưu ý nữa với mô hình phát hành này: các tính năng không ổn định. Rust sử dụng một kỹ thuật gọi là “cờ tính năng” (feature flags) để xác định những tính năng nào được bật trong một bản phát hành nhất định. Nếu một tính năng mới đang được phát triển tích cực, nó sẽ được đưa vào `master`, và do đó, vào nightly, nhưng đằng sau một _cờ tính năng_. Nếu bạn, với tư cách là người dùng, muốn thử tính năng đang trong quá trình hoàn thiện, bạn có thể, nhưng bạn phải sử dụng bản phát hành nightly của Rust và chú thích mã nguồn của mình bằng cờ thích hợp để chọn tham gia.

Nếu bạn đang sử dụng bản phát hành beta hoặc stable của Rust, bạn không thể sử dụng bất kỳ cờ tính năng nào. Đây là chìa khóa cho phép chúng tôi có được kinh nghiệm thực tế với các tính năng mới trước khi chúng tôi tuyên bố chúng ổn định mãi mãi. Những người muốn chọn tham gia vào các tính năng tiên tiến nhất có thể làm như vậy, và những người muốn có một trải nghiệm vững chắc có thể gắn bó với stable và biết rằng mã của họ sẽ không bị hỏng. Ổn định không trì trệ.

Cuốn sách này chỉ chứa thông tin về các tính năng ổn định, vì các tính năng đang trong quá trình hoàn thiện vẫn đang thay đổi, và chắc chắn chúng sẽ khác nhau giữa thời điểm cuốn sách này được viết và khi chúng được bật trong các bản dựng ổn định. Bạn có thể tìm thấy tài liệu cho các tính năng chỉ có trên nightly trực tuyến.

### Rustup và vai trò của Rust Nightly

Rustup giúp dễ dàng thay đổi giữa các kênh phát hành khác nhau của Rust, trên cơ sở toàn cục hoặc theo từng dự án. Theo mặc định, bạn sẽ có Rust stable được cài đặt. Để cài đặt nightly, ví dụ:

```console
$ rustup toolchain install nightly
```

Bạn cũng có thể xem tất cả các _toolchains_ (các bản phát hành của Rust và các thành phần liên quan) mà bạn đã cài đặt với `rustup`. Đây là một ví dụ trên máy tính Windows của một trong những tác giả của bạn:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Như bạn có thể thấy, toolchain stable là mặc định. Hầu hết người dùng Rust sử dụng stable trong phần lớn thời gian. Bạn có thể muốn sử dụng stable trong phần lớn thời gian, nhưng sử dụng nightly cho một dự án cụ thể, vì bạn quan tâm đến một tính năng tiên tiến. Để làm như vậy, bạn có thể sử dụng `rustup override` trong thư mục của dự án đó để đặt toolchain nightly làm toolchain mà `rustup` nên sử dụng khi bạn ở trong thư mục đó:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Bây giờ, mỗi khi bạn gọi `rustc` hoặc `cargo` bên trong _~/projects/needs-nightly_, `rustup` sẽ đảm bảo rằng bạn đang sử dụng Rust nightly, thay vì mặc định là Rust stable. Điều này rất hữu ích khi bạn có nhiều dự án Rust!

### Quy trình RFC và các Nhóm

Vậy làm thế nào để bạn tìm hiểu về những tính năng mới này? Mô hình phát triển của Rust tuân theo một quy trình _Yêu cầu Bình luận (Request For Comments - RFC)_. Nếu bạn muốn một cải tiến trong Rust, bạn có thể viết một đề xuất, được gọi là RFC.

Bất kỳ ai cũng có thể viết RFC để cải thiện Rust, và các đề xuất được xem xét và thảo luận bởi nhóm Rust, bao gồm nhiều nhóm nhỏ theo chủ đề. Có một danh sách đầy đủ các nhóm [trên trang web của Rust](https://www.rust-lang.org/governance), bao gồm các nhóm cho từng lĩnh vực của dự án: thiết kế ngôn ngữ, triển khai trình biên dịch, cơ sở hạ tầng, tài liệu, và nhiều hơn nữa. Nhóm thích hợp sẽ đọc đề xuất và các bình luận, viết một số bình luận của riêng họ, và cuối cùng, có sự đồng thuận để chấp nhận hoặc từ chối tính năng.

Nếu tính năng được chấp nhận, một issue sẽ được mở trên kho mã Rust, và ai đó có thể triển khai nó. Người triển khai nó rất có thể không phải là người đã đề xuất tính năng ngay từ đầu! Khi việc triển khai sẵn sàng, nó sẽ được đưa vào nhánh `master` đằng sau một cổng tính năng (feature gate), như chúng ta đã thảo luận trong phần [“Các tính năng không ổn định”](#unstable-features)<!-- ignore -->.

Sau một thời gian, một khi các nhà phát triển Rust sử dụng các bản phát hành nightly đã có thể thử tính năng mới, các thành viên trong nhóm sẽ thảo luận về tính năng đó, cách nó hoạt động trên nightly, và quyết định xem nó có nên được đưa vào Rust stable hay không. Nếu quyết định là tiếp tục, cổng tính năng sẽ được gỡ bỏ, và tính năng đó bây giờ được coi là ổn định! Nó đi theo các chuyến tàu vào một bản phát hành ổn định mới của Rust.
