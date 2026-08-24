## Xây Dựng Một Web Server Đơn Luồng

Chúng ta sẽ bắt đầu bằng việc làm cho một web server đơn luồng (single-threaded)
hoạt động. Trước khi bắt đầu, hãy xem qua tổng quan nhanh về các giao thức liên quan
đến việc xây dựng web server. Chi tiết về các giao thức này nằm ngoài phạm vi của cuốn
sách, nhưng một cái nhìn tổng quan ngắn gọn sẽ cung cấp cho bạn thông tin cần thiết.

Hai giao thức chính liên quan đến web server là _Hypertext Transfer Protocol_
_(HTTP)_ và _Transmission Control Protocol_ _(TCP)_. Cả hai giao thức đều là giao
thức _yêu cầu - phản hồi_ (_request-response_), nghĩa là một _client_ (phía máy khách)
khởi tạo các yêu cầu (request) và một _server_ (máy chủ) lắng nghe các yêu cầu và
cung cấp phản hồi (response) cho client. Nội dung của các request và response đó được
định nghĩa bởi các giao thức.

TCP là giao thức ở tầng thấp hơn, mô tả chi tiết cách thông tin truyền từ máy chủ này
sang máy chủ khác nhưng không chỉ định thông tin đó là gì. HTTP được xây dựng trên
nền tảng TCP bằng cách định nghĩa nội dung của các request và response. Về mặt kỹ
thuật, có thể sử dụng HTTP với các giao thức khác, nhưng trong phần lớn các trường
hợp, HTTP gửi dữ liệu của nó qua TCP. Chúng ta sẽ làm việc với các byte thô của TCP
cùng các request và response HTTP.

### Lắng Nghe Kết Nối TCP

Web server của chúng ta cần lắng nghe một kết nối TCP, vì vậy đó là phần đầu tiên
chúng ta sẽ thực hiện. Thư viện chuẩn cung cấp module `std::net` cho phép chúng ta
làm điều này. Hãy tạo một dự án mới theo cách thông thường:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Bây giờ, hãy nhập mã trong Listing 21-1 vào _src/main.rs_ để bắt đầu. Mã này sẽ lắng
nghe tại địa chỉ cục bộ `127.0.0.1:7878` cho các TCP stream đến. Khi nhận được một
stream đến, nó sẽ in ra `Connection established!`.

<Listing number="21-1" file-name="src/main.rs" caption="Lắng nghe các stream đến và in một thông báo khi nhận được một stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

Sử dụng `TcpListener`, chúng ta có thể lắng nghe các kết nối TCP tại địa chỉ
`127.0.0.1:7878`. Trong địa chỉ này, phần trước dấu hai chấm là địa chỉ IP đại diện
cho máy tính của bạn (địa chỉ này giống nhau trên mọi máy tính và không đại diện
riêng cho máy tính của tác giả), và `7878` là cổng (port). Chúng ta chọn cổng này vì
hai lý do: HTTP thường không được chấp nhận trên cổng này nên server của chúng ta khó
có khả năng xung đột với bất kỳ web server nào khác có thể đang chạy trên máy của
bạn, và 7878 là các phím gõ chữ _rust_ trên bàn phím điện thoại.

Hàm `bind` trong kịch bản này hoạt động giống như hàm `new` ở chỗ nó sẽ trả về một
instance `TcpListener` mới. Hàm được gọi là `bind` vì trong mạng máy tính, việc kết
nối tới một cổng để lắng nghe được gọi là “ràng buộc vào một cổng” (binding to a port).

Hàm `bind` trả về một `Result<T, E>`, cho biết việc binding có thể thất bại. Ví dụ:
kết nối đến cổng 80 yêu cầu quyền quản trị viên (người dùng không phải quản trị viên
chỉ có thể lắng nghe trên các cổng lớn hơn 1023), vì vậy nếu chúng ta cố gắng kết
nối đến cổng 80 mà không có quyền quản trị, binding sẽ không hoạt động. Binding cũng
sẽ không hoạt động nếu, chẳng hạn, chúng ta chạy hai phiên bản của chương trình và có
hai chương trình cùng lắng nghe trên một cổng. Vì chúng ta chỉ đang viết một server cơ
bản cho mục đích học tập, chúng ta sẽ không bận tâm đến việc xử lý các loại lỗi này;
thay vào đó, chúng ta sử dụng `unwrap` để dừng chương trình nếu xảy ra lỗi.

Phương thức `incoming` trên `TcpListener` trả về một iterator cung cấp cho chúng ta
một chuỗi các stream (cụ thể hơn là các stream có kiểu `TcpStream`). Một _stream_ đơn
lẻ đại diện cho một kết nối mở giữa client và server. Một _kết nối_ (connection) là
tên gọi cho toàn bộ quá trình yêu cầu và phản hồi, trong đó client kết nối đến server,
server tạo một phản hồi, và server đóng kết nối. Do đó, chúng ta sẽ đọc từ `TcpStream`
để xem client đã gửi những gì và sau đó ghi phản hồi của chúng ta vào stream để gửi
dữ liệu trở lại cho client. Nhìn chung, vòng lặp `for` này sẽ xử lý lần lượt từng kết
nối và tạo ra một loạt các stream để chúng ta xử lý.

Hiện tại, việc xử lý stream của chúng ta bao gồm việc gọi `unwrap` để chấm dứt chương
trình nếu stream có bất kỳ lỗi nào; nếu không có lỗi, chương trình sẽ in một thông
báo. Chúng ta sẽ thêm nhiều chức năng hơn cho trường hợp thành công trong listing tiếp
theo. Lý do chúng ta có thể nhận được lỗi từ phương thức `incoming` khi một client
kết nối đến server là vì chúng ta không thực sự đang lặp qua các kết nối. Thay vào đó,
chúng ta đang lặp qua _các nỗ lực kết nối_ (connection attempts). Kết nối có thể không
thành công vì nhiều lý do, nhiều lý do trong số đó mang tính đặc thù của hệ điều hành.
Ví dụ, nhiều hệ điều hành có giới hạn về số lượng kết nối mở đồng thời mà chúng có
thể hỗ trợ; các nỗ lực kết nối mới vượt quá con số đó sẽ tạo ra lỗi cho đến khi một
số kết nối đang mở được đóng lại.

Hãy thử chạy mã này! Chạy `cargo run` trong terminal và sau đó mở _127.0.0.1:7878_ trong
trình duyệt web. Trình duyệt sẽ hiển thị một thông báo lỗi như “Connection reset” vì
server hiện không gửi lại bất kỳ dữ liệu nào. Nhưng khi nhìn vào terminal, bạn sẽ
thấy một vài thông báo đã được in ra khi trình duyệt kết nối với server!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Đôi khi bạn sẽ thấy nhiều thông báo được in ra cho một yêu cầu từ trình duyệt; lý do
có thể là trình duyệt đang gửi một yêu cầu cho trang cũng như một yêu cầu cho các tài
nguyên khác, chẳng hạn như biểu tượng _favicon.ico_ xuất hiện trên tab trình duyệt.

Nó cũng có thể là do trình duyệt đang cố gắng kết nối với server nhiều lần vì server
không phản hồi bất kỳ dữ liệu nào. Khi `stream` đi ra ngoài phạm vi và bị drop ở cuối
vòng lặp, kết nối sẽ bị đóng như một phần của quá trình triển khai `drop`. Các trình
duyệt đôi khi xử lý các kết nối bị đóng bằng cách thử lại, vì sự cố có thể chỉ là tạm
thời.

Các trình duyệt đôi khi cũng mở nhiều kết nối tới server mà không gửi bất kỳ yêu cầu
nào, để nếu sau đó chúng *thực sự* gửi yêu cầu, chúng có thể diễn ra nhanh hơn. Khi
điều này xảy ra, server của chúng ta sẽ nhìn thấy từng kết nối, bất kể có bất kỳ yêu
cầu nào qua kết nối đó hay không. Ví dụ, nhiều phiên bản của các trình duyệt dựa trên
Chrome thực hiện điều này; bạn có thể vô hiệu hóa tối ưu hóa đó bằng cách sử dụng chế
độ duyệt web riêng tư hoặc sử dụng một trình duyệt khác.

Yếu tố quan trọng là chúng ta đã nắm giữ thành công một kết nối TCP!

Hãy nhớ dừng chương trình bằng cách nhấn <kbd>ctrl</kbd>-<kbd>c</kbd> khi bạn chạy
xong một phiên bản mã cụ thể. Sau đó khởi động lại chương trình bằng cách gọi lệnh
`cargo run` sau mỗi lần bạn thực hiện thay đổi mã để đảm bảo bạn đang chạy mã mới
nhất.

### Đọc Request

Hãy triển khai chức năng đọc request từ trình duyệt! Để tách biệt các mối quan tâm
giữa việc nhận kết nối trước rồi sau đó thực hiện một số hành động với kết nối, chúng
ta sẽ bắt đầu một hàm mới để xử lý các kết nối. Trong hàm `handle_connection` mới
này, chúng ta sẽ đọc dữ liệu từ TCP stream và in ra để có thể xem dữ liệu được gửi từ
trình duyệt. Hãy thay đổi mã để trông giống như Listing 21-2.

<Listing number="21-2" file-name="src/main.rs" caption="Đọc từ `TcpStream` và in dữ liệu ra">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Chúng ta đưa `std::io::prelude` và `std::io::BufReader` vào phạm vi để có quyền truy
cập vào các trait và kiểu cho phép chúng ta đọc và ghi vào stream. Trong vòng lặp
`for` ở hàm `main`, thay vì in một thông báo cho biết chúng ta đã tạo kết nối, giờ
đây chúng ta gọi hàm `handle_connection` mới và truyền `stream` cho nó.

Trong hàm `handle_connection`, chúng ta tạo một instance `BufReader` mới bao bọc
một tham chiếu đến `stream`. `BufReader` thêm bộ đệm bằng cách thay chúng ta quản lý
các lệnh gọi phương thức của trait `std::io::Read`.

Chúng ta tạo một biến tên là `http_request` để thu thập các dòng của request mà trình
duyệt gửi đến server của chúng ta. Chúng ta chỉ định rằng mình muốn thu thập các dòng
này vào một vector bằng cách thêm chú thích kiểu `Vec<_>`.

`BufReader` triển khai trait `std::io::BufRead`, trait này cung cấp phương thức
`lines`. Phương thức `lines` trả về một iterator của `Result<String, std::io::Error>`
bằng cách tách luồng dữ liệu bất cứ khi nào nó thấy một byte dòng mới (newline byte).
Để lấy từng `String`, chúng ta map và `unwrap` từng `Result`. `Result` có thể là một
lỗi nếu dữ liệu không phải là UTF-8 hợp lệ hoặc nếu có vấn đề khi đọc từ stream. Một
lần nữa, một chương trình production nên xử lý các lỗi này một cách khéo léo hơn, nhưng
chúng ta chọn dừng chương trình trong trường hợp có lỗi để cho đơn giản.

Trình duyệt báo hiệu kết thúc một HTTP request bằng cách gửi hai ký tự dòng mới liên
tiếp, vì vậy để lấy một request từ stream, chúng ta lấy các dòng cho đến khi gặp một
dòng là chuỗi rỗng. Khi đã thu thập các dòng vào vector, chúng ta in chúng ra bằng định
dạng debug đẹp mắt để có thể xem các chỉ dẫn mà trình duyệt web đang gửi tới server
của chúng ta.

Hãy thử mã này! Khởi động chương trình và thực hiện lại một yêu cầu trong trình duyệt
web. Lưu ý rằng chúng ta vẫn sẽ nhận được một trang lỗi trong trình duyệt, nhưng đầu
ra của chương trình trong terminal giờ đây sẽ trông tương tự như thế này:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Tùy thuộc vào trình duyệt của bạn, bạn có thể nhận được đầu ra hơi khác một chút.
Bây giờ chúng ta đang in dữ liệu request, chúng ta có thể thấy lý do tại sao chúng
ta nhận được nhiều kết nối từ một yêu cầu của trình duyệt bằng cách nhìn vào đường
dẫn sau `GET` ở dòng đầu tiên của request. Nếu các kết nối lặp lại đều đang yêu cầu
_/_, chúng ta biết trình duyệt đang cố gắng lấy _/_ lặp đi lặp lại vì nó không nhận
được phản hồi từ chương trình của chúng ta.

Hãy phân tích dữ liệu request này để hiểu những gì trình duyệt đang yêu cầu từ chương
trình của chúng ta.

### Tìm Hiểu Kỹ Hơn Về Một HTTP Request

HTTP là một giao thức dựa trên văn bản, và một request có định dạng như sau:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Dòng đầu tiên là _request line_ chứa thông tin về những gì client đang yêu cầu. Phần
đầu tiên của dòng request cho biết _method_ (phương thức) đang được sử dụng, chẳng
hạn như `GET` hoặc `POST`, mô tả cách client thực hiện yêu cầu này. Client của chúng
ta đã sử dụng một request `GET`, có nghĩa là nó đang yêu cầu thông tin.

Phần tiếp theo của dòng request là _/_, biểu thị _uniform resource identifier_
_(URI)_ mà client đang yêu cầu: URI gần như, nhưng không hoàn toàn, giống với
_uniform resource locator_ _(URL)_. Sự khác biệt giữa URI và URL không quan trọng
đối với mục đích của chúng ta trong chương này, nhưng đặc tả HTTP sử dụng thuật ngữ
URI, vì vậy chúng ta có thể ngầm hiểu _URL_ thay cho _URI_ ở đây.

Phần cuối cùng là phiên bản HTTP mà client sử dụng, và sau đó dòng request kết thúc
bằng một chuỗi CRLF. (CRLF là viết tắt của _carriage return_ và _line feed_, là những
thuật ngữ từ thời máy đánh chữ!) Chuỗi CRLF cũng có thể được viết là `\r\n`, trong
đó `\r` là carriage return (về đầu dòng) và `\n` là line feed (xuống dòng). _Chuỗi
CRLF_ phân tách dòng request với phần còn lại của dữ liệu request. Lưu ý rằng khi CRLF
được in ra, chúng ta thấy một dòng mới bắt đầu thay vì nhìn thấy `\r\n`.

Nhìn vào dữ liệu dòng request mà chúng ta nhận được từ việc chạy chương trình cho đến
nay, chúng ta thấy rằng `GET` là method, _/_ là URI được yêu cầu, và `HTTP/1.1` là
phiên bản.

Sau dòng request, các dòng còn lại bắt đầu từ `Host:` trở đi là các header. Các
request `GET` không có body.

Hãy thử thực hiện một yêu cầu từ một trình duyệt khác hoặc yêu cầu một địa chỉ khác,
chẳng hạn như _127.0.0.1:7878/test_, để xem dữ liệu request thay đổi như thế nào.

Bây giờ chúng ta đã biết trình duyệt đang yêu cầu những gì, hãy gửi lại một số dữ
liệu!

### Viết Một Response

Chúng ta sẽ triển khai việc gửi dữ liệu để phản hồi một yêu cầu của client. Các
response có định dạng như sau:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Dòng đầu tiên là _status line_ (dòng trạng thái) chứa phiên bản HTTP được sử dụng
trong response, một mã trạng thái (status code) bằng số tóm tắt kết quả của request,
và một cụm từ lý do (reason phrase) cung cấp mô tả dạng văn bản cho mã trạng thái đó.
Sau chuỗi CRLF là các header (nếu có), một chuỗi CRLF khác, và phần body của response.

Dưới đây là một ví dụ về response sử dụng HTTP phiên bản 1.1, có status code là 200,
reason phrase là OK, không có header và không có body:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Mã trạng thái 200 là phản hồi thành công tiêu chuẩn. Văn bản này là một HTTP response
thành công nhỏ gọn. Hãy ghi nội dung này vào stream làm phản hồi của chúng ta cho một
request thành công! Từ hàm `handle_connection`, hãy xóa lệnh `println!` dùng để in
dữ liệu request và thay thế bằng mã trong Listing 21-3.

<Listing number="21-3" file-name="src/main.rs" caption="Ghi một HTTP response thành công nhỏ gọn vào stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

Dòng mới đầu tiên định nghĩa biến `response` chứa dữ liệu của thông báo thành công.
Sau đó, chúng ta gọi `as_bytes` trên `response` để chuyển đổi dữ liệu chuỗi thành các
byte. Phương thức `write_all` trên `stream` nhận một `&[u8]` và gửi các byte đó trực
tiếp qua kết nối. Vì thao tác `write_all` có thể thất bại, chúng ta sử dụng `unwrap`
trên mọi kết quả lỗi như trước. Một lần nữa, trong một ứng dụng thực tế, bạn sẽ thêm
phần xử lý lỗi ở đây.

Với những thay đổi này, hãy chạy mã của chúng ta và thực hiện một request. Chúng ta
không còn in bất kỳ dữ liệu nào ra terminal nữa, vì vậy chúng ta sẽ không thấy đầu ra
nào khác ngoài đầu ra từ Cargo. Khi bạn mở _127.0.0.1:7878_ trong trình duyệt web,
bạn sẽ nhận được một trang trắng thay vì một trang lỗi. Bạn vừa tự tay viết mã nhận
một HTTP request và gửi một response!

### Trả Về HTML Thực Sự

Hãy triển khai chức năng trả về nhiều hơn một trang trắng. Tạo tệp mới _hello.html_
trong thư mục gốc của dự án, không phải trong thư mục _src_. Bạn có thể nhập bất kỳ
mã HTML nào bạn muốn; Listing 21-4 cho thấy một ví dụ khả dĩ.

<Listing number="21-4" file-name="hello.html" caption="Một tệp HTML mẫu để trả về trong response">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Đây là một tài liệu HTML5 tối giản với một tiêu đề và một số văn bản. Để server trả
về nội dung này khi nhận được một request, chúng ta sẽ sửa đổi `handle_connection`
như trong Listing 21-5 để đọc tệp HTML, thêm nó vào response dưới dạng body và gửi đi.

<Listing number="21-5" file-name="src/main.rs" caption="Gửi nội dung của *hello.html* dưới dạng body của response">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm `fs` vào câu lệnh `use` để đưa module hệ thống tệp của thư viện
chuẩn vào phạm vi. Mã để đọc nội dung của một tệp thành chuỗi hẳn trông quen thuộc;
chúng ta đã sử dụng nó khi đọc nội dung của một tệp cho dự án I/O của chúng ta trong
Listing 12-4.

Tiếp theo, chúng ta sử dụng `format!` để thêm nội dung của tệp vào làm body của
response thành công. Để đảm bảo một HTTP response hợp lệ, chúng ta thêm header
`Content-Length` được đặt bằng kích thước của body phản hồi, trong trường hợp này là
kích thước của `hello.html`.

Chạy mã này bằng `cargo run` và mở _127.0.0.1:7878_ trong trình duyệt của bạn; bạn
sẽ thấy HTML của mình được hiển thị!

Hiện tại, chúng ta đang bỏ qua dữ liệu request trong `http_request` và chỉ gửi lại
nội dung của tệp HTML một cách vô điều kiện. Điều đó có nghĩa là nếu bạn thử yêu cầu
_127.0.0.1:7878/something-else_ trong trình duyệt, bạn vẫn sẽ nhận lại cùng phản hồi
HTML này. Hiện tại, server của chúng ta rất hạn chế và không làm những gì mà hầu hết
các web server thường làm. Chúng ta muốn tùy chỉnh các phản hồi tùy thuộc vào request
và chỉ gửi lại tệp HTML cho một request đúng định dạng tới _/_.

### Xác Thực Request và Phản Hồi Có Chọn Lọc

Ngay lúc này, web server của chúng ta sẽ trả về mã HTML trong tệp bất kể client đã
yêu cầu điều gì. Hãy thêm chức năng để kiểm tra xem trình duyệt có đang yêu cầu _/_
hay không trước khi trả về tệp HTML và trả về lỗi nếu trình duyệt yêu cầu bất kỳ thứ
gì khác. Để làm điều này, chúng ta cần sửa đổi `handle_connection`, như được hiển thị
trong Listing 21-6. Mã mới này kiểm tra nội dung của request nhận được so với những
gì chúng ta biết về một request cho _/_ và thêm các khối `if` và `else` để xử lý các
request khác nhau.

<Listing number="21-6" file-name="src/main.rs" caption="Xử lý các request đến */* khác biệt so với các request khác">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Chúng ta sẽ chỉ xem xét dòng đầu tiên của HTTP request, vì vậy thay vì đọc toàn bộ
request vào một vector, chúng ta gọi `next` để lấy mục đầu tiên từ iterator. Lệnh
`unwrap` đầu tiên xử lý `Option` và dừng chương trình nếu iterator không có phần tử
nào. Lệnh `unwrap` thứ hai xử lý `Result` và có tác dụng tương tự như `unwrap` nằm
trong `map` được thêm vào trong Listing 21-2.

Tiếp theo, chúng ta kiểm tra `request_line` xem nó có bằng với request line của một
GET request đến đường dẫn _/_ hay không. Nếu bằng, khối `if` sẽ trả về nội dung của
tệp HTML của chúng ta.

Nếu `request_line` _không_ bằng GET request đến đường dẫn _/_, điều đó có nghĩa là
chúng ta đã nhận được một request khác. Chúng ta sẽ thêm mã vào khối `else` ngay sau
đây để phản hồi tất cả các request khác.

Hãy chạy mã này ngay bây giờ và yêu cầu _127.0.0.1:7878_; bạn sẽ nhận được HTML trong
_hello.html_. Nếu bạn thực hiện bất kỳ request nào khác, chẳng hạn như
_127.0.0.1:7878/something-else_, bạn sẽ nhận được lỗi kết nối giống như những lỗi bạn
đã thấy khi chạy mã trong Listing 21-1 và Listing 21-2.

Bây giờ hãy thêm mã trong Listing 21-7 vào khối `else` để trả về một response với
status code 404, báo hiệu rằng nội dung cho request không được tìm thấy. Chúng ta
cũng sẽ trả về một số mã HTML để một trang hiển thị trong trình duyệt cho người dùng
cuối thấy phản hồi.

<Listing number="21-7" file-name="src/main.rs" caption="Phản hồi với status code 404 và một trang lỗi nếu có bất kỳ thứ gì khác */* được yêu cầu">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Ở đây, response của chúng ta có một status line với status code 404 và reason phrase
`NOT FOUND`. Body của response sẽ là HTML trong tệp _404.html_. Bạn sẽ cần tạo một
tệp _404.html_ cạnh _hello.html_ cho trang lỗi; một lần nữa, bạn có thể thoải mái sử
dụng bất kỳ mã HTML nào bạn muốn hoặc sử dụng mã HTML mẫu trong Listing 21-8.

<Listing number="21-8" file-name="404.html" caption="Nội dung mẫu cho trang gửi lại với bất kỳ response 404 nào">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Với những thay đổi này, hãy chạy lại server của bạn. Yêu cầu _127.0.0.1:7878_ sẽ trả
về nội dung của _hello.html_, và bất kỳ yêu cầu nào khác, chẳng hạn như
_127.0.0.1:7878/foo_, sẽ trả về HTML báo lỗi từ _404.html_.

### Tinh Chỉnh Mã Nguồn (Refactoring)

Hiện tại, các khối `if` và `else` có rất nhiều sự lặp lại: cả hai đều đang đọc các
tệp và ghi nội dung của các tệp vào stream. Điểm khác biệt duy nhất là status line và
tên tệp. Hãy làm cho mã súc tích hơn bằng cách tách những điểm khác biệt đó ra thành
các dòng `if` và `else` riêng biệt nhằm gán giá trị của status line và tên tệp cho các
biến; sau đó chúng ta có thể sử dụng các biến đó một cách vô điều kiện trong mã để đọc
tệp và ghi response. Listing 21-9 hiển thị mã kết quả sau khi thay thế các khối `if`
và `else` lớn.

<Listing number="21-9" file-name="src/main.rs" caption="Refactor các khối `if` và `else` để chỉ chứa mã khác biệt giữa hai trường hợp">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Bây giờ các khối `if` và `else` chỉ trả về các giá trị thích hợp cho status line và
tên tệp trong một tuple; sau đó chúng ta sử dụng phân rã (destructuring) để gán hai
giá trị này cho `status_line` và `filename` bằng cách sử dụng một pattern trong câu
lệnh `let`, như đã thảo luận trong Chương 19.

Mã bị trùng lặp trước đó giờ đã nằm ngoài các khối `if` và `else` và sử dụng các biến
`status_line` cùng `filename`. Điều này giúp dễ dàng nhận thấy sự khác biệt giữa hai
trường hợp, và có nghĩa là chúng ta chỉ có một nơi duy nhất để cập nhật mã nếu muốn
thay đổi cách hoạt động của việc đọc tệp và ghi response. Hành vi của mã trong Listing
21-9 sẽ giống với mã trong Listing 21-7.

Tuyệt vời! Bây giờ chúng ta đã có một web server đơn giản với khoảng 40 dòng mã Rust,
phản hồi một yêu cầu bằng một trang nội dung và phản hồi tất cả các yêu cầu khác bằng
một phản hồi 404.

Hiện tại, server của chúng ta chạy trong một luồng đơn (single thread), nghĩa là nó
chỉ có thể phục vụ một request tại một thời điểm. Hãy xem xét việc đó có thể gây ra
vấn đề như thế nào bằng cách mô phỏng một số request chậm. Sau đó, chúng ta sẽ khắc
phục để server của chúng ta có thể xử lý nhiều request cùng một lúc.
