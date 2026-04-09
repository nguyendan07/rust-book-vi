# Các đánh đổi trong thiết kế

Phần này nói về **các đánh đổi trong thiết kế** trong Rust. Để trở thành một kỹ sư Rust hiệu quả, chỉ biết Rust hoạt động như thế nào là chưa đủ. Bạn phải quyết định công cụ nào trong số nhiều công cụ của Rust là phù hợp cho một công việc nhất định. Trong phần này, chúng tôi sẽ cung cấp cho bạn một chuỗi các câu đố về sự hiểu biết của bạn về các đánh đổi trong thiết kế trong Rust. Sau mỗi câu đố, chúng tôi sẽ giải thích chuyên sâu lý do của mình cho mỗi câu hỏi.

Dưới đây là một ví dụ về một câu hỏi sẽ trông như thế nào. Nó sẽ bắt đầu bằng cách mô tả một nghiên cứu điển hình về phần mềm với một không gian các thiết kế:

> **Ngữ cảnh:** Bạn đang thiết kế một ứng dụng với một cấu hình toàn cục, ví dụ: chứa các cờ dòng lệnh.
>
> **Chức năng:** Ứng dụng cần truyền các tham chiếu không thay đổi (immutable references) đến cấu hình này trong toàn bộ ứng dụng.
>
> **Thiết kế:** Dưới đây là một số thiết kế được đề xuất để triển khai chức năng này.
>
> ```rust,ignore
> use std::rc::Rc;
> use std::sync::Arc;
>
> struct Config {
>     flags: Flags,
>     // .. thêm các trường khác ..
> }
>
> // Tùy chọn 1: sử dụng một tham chiếu
> struct ConfigRef<'a>(&'a Config);
>
> // Tùy chọn 2: sử dụng một con trỏ đếm tham chiếu
> struct ConfigRef(Rc<Config>);
>
> // Tùy chọn 3: sử dụng một con trỏ đếm tham chiếu nguyên tử
> struct ConfigRef(Arc<Config>);
> ```

Nếu chỉ dựa vào ngữ cảnh và chức năng chính, cả ba thiết kế đều là những ứng cử viên tiềm năng.
Chúng ta cần thêm thông tin về các mục tiêu hệ thống để quyết định cái nào là hợp lý nhất.
Do đó, chúng tôi đưa ra một yêu cầu mới:

> Chọn mỗi tùy chọn thiết kế đáp ứng yêu cầu sau:
>
> **Yêu cầu:** Tham chiếu cấu hình phải có thể chia sẻ được giữa nhiều luồng (threads).
>
> **Câu trả lời:**
>
> <input type="checkbox" checked disabled> Tùy chọn 1 <br>
> <input type="checkbox" disabled> Tùy chọn 2 <br>
> <input type="checkbox" checked disabled> Tùy chọn 3 <br>

Về mặt kỹ thuật, điều này có nghĩa là `ConfigRef` triển khai [`Send`] và [`Sync`].
Giả sử `Config: Send + Sync`, thì cả `&Config` và `Arc<Config>` đều đáp ứng yêu cầu này,
nhưng [`Rc`] thì không (vì các con trỏ đếm tham chiếu không nguyên tử không an toàn luồng). Vì vậy Tùy chọn 2 không đáp ứng yêu cầu, trong khi Tùy chọn 3 thì có.

Chúng ta cũng có thể bị cám dỗ để kết luận rằng Tùy chọn 1 không đáp ứng yêu cầu vì các hàm như [`thread::spawn`] yêu cầu tất cả dữ liệu được di chuyển vào một luồng chỉ có thể chứa các tham chiếu với thời gian sống `'static`. Tuy nhiên, điều đó không loại trừ Tùy chọn 1 vì hai lý do:

1. `Config` có thể được lưu trữ dưới dạng một biến tĩnh toàn cục (ví dụ: sử dụng [`OnceLock`]), vì vậy người ta có thể xây dựng các tham chiếu `&'static Config`.
2. Không phải tất cả các cơ chế đồng thời đều yêu cầu thời gian sống `'static`, chẳng hạn như [`thread::scope`].

Do đó, yêu cầu như đã nêu chỉ loại trừ các kiểu không phải-[`Send`], và chúng tôi coi Tùy chọn 1 và 3 là các câu trả lời đúng.

[`thread::spawn`]: https://doc.rust-lang.org/std/thread/fn.spawn.html
[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`OnceLock`]: https://doc.rust-lang.org/std/sync/struct.OnceLock.html
[`thread::scope`]: https://doc.rust-lang.org/std/thread/fn.scope.html

<hr>

Bây giờ hãy thử với các câu hỏi bên dưới! Mỗi phần chứa một câu đố tập trung vào một tình huống duy nhất. Hoàn thành câu đố và đảm bảo đọc ngữ cảnh câu trả lời sau mỗi câu đố.

 <!-- Những câu hỏi này mang tính thử nghiệm và chủ quan &mdash; vui lòng để lại phản hồi cho chúng tôi qua nút báo lỗi 🐞 nếu bạn không đồng ý với câu trả lời của chúng tôi. -->

Cùng với mỗi câu đố, chúng tôi cũng cung cấp các liên kết đến các crate Rust phổ biến đóng vai trò là nguồn cảm hứng cho câu đố.

## Tham chiếu (References)

_Nguồn cảm hứng:_ [Bevy assets], [Petgraph node indices], [Cargo units]

{{#quiz ../quizzes/ch17-05-design-challenge-references.toml}}

[Bevy assets]: https://docs.rs/bevy/0.11.2/bevy/asset/struct.Assets.html
[Petgraph node indices]: https://docs.rs/petgraph/0.6.4/petgraph/graph/struct.NodeIndex.html
[Cargo units]: https://docs.rs/cargo/0.73.1/cargo/core/compiler/struct.Unit.html

## Cây Trait (Trait Trees)

_Nguồn cảm hứng:_ [Yew components], [Druid widgets]

{{#quiz ../quizzes/ch17-05-design-challenge-trait-trees.toml}}

[Yew components]: https://docs.rs/yew/0.20.0/yew/html/trait.Component.html
[Druid widgets]: https://docs.rs/druid/0.8.3/druid/trait.Widget.html

## Điều phối (Dispatch)

_Nguồn cảm hứng:_ [Bevy systems], [Diesel queries], [Axum handlers]

{{#quiz ../quizzes/ch17-05-design-challenge-dispatch.toml}}

[Bevy systems]: https://docs.rs/bevy_ecs/0.11.2/bevy_ecs/system/trait.IntoSystem.html
[Diesel queries]: https://docs.diesel.rs/2.1.x/diesel/query_dsl/trait.BelongingToDsl.html
[Axum handlers]: https://docs.rs/axum/0.6.20/axum/handler/trait.Handler.html

## Các kiểu trung gian (Intermediates)

_Nguồn cảm hứng:_ [Serde] và [miniserde]

{{#quiz ../quizzes/ch17-05-design-challenge-intermediates.toml}}

[Serde]: https://docs.rs/serde/1.0.188/serde/trait.Serialize.html
[miniserde]: https://docs.rs/miniserde/0.1.34/miniserde/trait.Serialize.html
