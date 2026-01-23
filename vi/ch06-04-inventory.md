## Kiểm kê Quyền sở hữu #1

Kiểm kê Quyền sở hữu là một loạt các câu đố để kiểm tra mức độ hiểu biết của bạn về quyền sở hữu (ownership) trong các tình huống thực tế. Những tình huống này được lấy cảm hứng từ các câu hỏi phổ biến trên StackOverflow về Rust. Bạn có thể sử dụng các câu hỏi này để kiểm tra xem bạn hiểu về quyền sở hữu tốt đến mức nào cho đến nay.

### Một công nghệ mới: IDE trong trình duyệt

Các câu hỏi này sẽ liên quan đến các chương trình Rust sử dụng các hàm mà bạn chưa từng thấy trước đây. Do đó, chúng tôi sẽ sử dụng một công nghệ thử nghiệm hỗ trợ các tính năng IDE trong trình duyệt. IDE này cho phép bạn lấy thông tin về các hàm và kiểu dữ liệu lạ. Ví dụ, hãy thử thực hiện các hành động sau trong chương trình bên dưới:

* Di chuột qua `replace` để xem kiểu và mô tả của nó.
* Di chuột qua `s2` để xem kiểu suy luận của nó.

---

---

<pre>
<code class="ide">
/// Biến một chuỗi thành một chuỗi thú vị hơn nhiều
fn make_exciting(s: &str) -> String {
  let s2 = s.replace(".", "!");
  let s3 = s2.replace("?", "‽");
  s3
}
</code>
</pre>

---

Một vài lưu ý quan trọng về công nghệ thử nghiệm này:

**TƯƠNG THÍCH NỀN TẢNG:** IDE trong trình duyệt không hoạt động trên màn hình cảm ứng. IDE trong trình duyệt chỉ mới được thử nghiệm hoạt động trên Google Chrome 109 và Firefox 107. Nó có thể không hoạt động trên các phiên bản Safari cũ hơn.

**SỬ DỤNG BỘ NHỚ:** IDE trong trình duyệt sử dụng bản dựng [WebAssembly](https://rustwasm.github.io/book/) của [rust-analyzer](https://github.com/rust-lang/rust-analyzer), bản dựng này có thể tốn khá nhiều bộ nhớ. Mỗi thực thể của IDE dường như chiếm khoảng ~300 MB. (Lưu ý: chúng tôi cũng nhận được một số báo cáo về mức sử dụng bộ nhớ >10GB.)

**CUỘN TRANG:** IDE trong trình duyệt sẽ "ăn" con trỏ của bạn nếu con trỏ của bạn giao với trình soạn thảo trong khi cuộn. Nếu bạn gặp khó khăn khi cuộn trang, hãy thử di chuyển con trỏ sang thanh cuộn bên phải nhất.

**THỜI GIAN TẢI:** IDE có thể mất tới 15 giây để khởi tạo một chương trình mới. Nó sẽ hiển thị "Loading..." khi bạn tương tác với mã trong trình soạn thảo.

### Câu đố

{{#quiz ../quizzes/ch06-04-inventory.toml}}
