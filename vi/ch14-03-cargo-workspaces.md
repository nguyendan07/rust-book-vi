## Không gian làm việc Cargo (Cargo Workspaces)

Trong Chương 12, chúng ta đã xây dựng một gói bao gồm một crate nhị phân và một crate thư viện.
Khi dự án của bạn phát triển, bạn có thể thấy rằng crate thư viện
tiếp tục lớn dần và bạn muốn chia nhỏ gói của mình hơn nữa thành
nhiều crate thư viện. Cargo cung cấp một tính năng gọi là _không gian làm việc_ (workspaces) có thể
giúp quản lý nhiều gói liên quan được phát triển song song.

### Tạo một Không gian làm việc

Một _không gian làm việc_ là một tập hợp các gói dùng chung cùng một tệp _Cargo.lock_ và thư mục
đầu ra. Hãy cùng tạo một dự án sử dụng không gian làm việc—chúng ta sẽ sử dụng mã nguồn đơn giản để
có thể tập trung vào cấu trúc của không gian làm việc. Có nhiều cách để
cấu trúc một không gian làm việc, vì vậy chúng ta sẽ chỉ trình bày một cách phổ biến. Chúng ta sẽ có một
không gian làm việc chứa một tệp nhị phân và hai thư viện. Tệp nhị phân, cái sẽ cung cấp
chức năng chính, sẽ phụ thuộc vào hai thư viện. Một thư viện sẽ
cung cấp một hàm `add_one` và thư viện còn lại cung cấp một hàm `add_two`.
Ba crate này sẽ là một phần của cùng một không gian làm việc. Chúng ta sẽ bắt đầu bằng cách tạo
một thư mục mới cho không gian làm việc:

```console
$ mkdir add
$ cd add
```

Tiếp theo, trong thư mục _add_, chúng ta tạo tệp _Cargo.toml_ để
cấu hình toàn bộ không gian làm việc. Tệp này sẽ không có phần `[package]`.
Thay vào đó, nó sẽ bắt đầu với một phần `[workspace]` cho phép chúng ta thêm
các thành viên vào không gian làm việc. Chúng ta cũng lưu ý sử dụng phiên bản mới nhất và tốt nhất
của thuật toán giải quyết (resolver) của Cargo trong không gian làm việc của mình bằng cách đặt
`resolver` thành `"3"`.

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Tiếp theo, chúng ta sẽ tạo crate nhị phân `adder` bằng cách chạy lệnh `cargo new` bên trong
thư mục _add_:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
    Creating binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Chạy `cargo new` bên trong một không gian làm việc cũng tự động thêm gói vừa được tạo
vào khóa `members` trong định nghĩa `[workspace]` trong tệp `Cargo.toml` của không gian làm việc,
như thế này:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Tại thời điểm này, chúng ta có thể xây dựng không gian làm việc bằng cách chạy `cargo build`. Các tệp
trong thư mục _add_ của bạn sẽ trông như thế này:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Không gian làm việc có một thư mục _target_ ở cấp cao nhất để các thành phần đã biên dịch
được đặt vào đó; gói `adder` không có thư mục _target_ riêng của nó. Ngay cả khi chúng ta chạy
`cargo build` từ bên trong thư mục
_adder_, các thành phần đã biên dịch vẫn sẽ nằm trong _add/target_
thay vì _add/adder/target_. Cargo cấu trúc thư mục _target_ trong một
không gian làm việc như thế này vì các crate trong một không gian làm việc được thiết kế để phụ thuộc vào
nhau. Nếu mỗi crate có thư mục _target_ riêng, mỗi crate sẽ phải
biên dịch lại từng crate khác trong không gian làm việc để đặt các thành phần
vào thư mục _target_ của riêng nó. Bằng cách dùng chung một thư mục _target_, các crate
có thể tránh việc xây dựng lại không cần thiết.

### Tạo Gói Thứ Hai trong Không gian làm việc

Tiếp theo, hãy tạo một gói thành viên khác trong không gian làm việc và gọi nó là
`add_one`. Tạo một crate thư viện mới tên là `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
    Creating library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

Tệp _Cargo.toml_ ở cấp cao nhất bây giờ sẽ bao gồm đường dẫn _add_one_ trong danh sách
`members`:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Thư mục _add_ của bạn bây giờ sẽ có các thư mục và tệp này:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Trong tệp _add_one/src/lib.rs_, hãy thêm một hàm `add_one`:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Bây giờ chúng ta có thể làm cho gói `adder` với tệp nhị phân của mình phụ thuộc vào gói `add_one`
chứa thư viện của chúng ta. Trước tiên chúng ta sẽ cần thêm một phụ thuộc đường dẫn (path dependency) vào
`add_one` trong tệp _adder/Cargo.toml_.

<span class="filename">Filename: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo không giả định rằng các crate trong một không gian làm việc sẽ phụ thuộc vào nhau, vì vậy
chúng ta cần phải chỉ rõ các mối quan hệ phụ thuộc.

Tiếp theo, hãy sử dụng hàm `add_one` (từ crate `add_one`) trong
crate `adder`. Mở tệp _adder/src/main.rs_ và thay đổi hàm `main`
để gọi hàm `add_one`, như trong Liệt kê 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Sử dụng crate thư viện `add_one` trong crate `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Hãy xây dựng không gian làm việc bằng cách chạy `cargo build` trong thư mục _add_
ở cấp cao nhất!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

Để chạy crate nhị phân từ thư mục _add_, chúng ta có thể chỉ định
gói nào trong không gian làm việc mà chúng ta muốn chạy bằng cách sử dụng đối số `-p` và
tên gói với lệnh `cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Lệnh này chạy mã trong _adder/src/main.rs_, cái phụ thuộc vào crate `add_one`.

#### Phụ thuộc vào một Gói Bên ngoài trong một Không gian làm việc

Lưu ý rằng không gian làm việc chỉ có một tệp _Cargo.lock_ duy nhất ở cấp cao nhất,
thay vì có một tệp _Cargo.lock_ trong thư mục của mỗi crate. Điều này đảm bảo rằng
tất cả các crate đang sử dụng cùng một phiên bản của tất cả các phụ thuộc. Nếu chúng ta thêm gói `rand`
vào các tệp _adder/Cargo.toml_ và _add_one/Cargo.toml_, Cargo sẽ
giải quyết cả hai thành một phiên bản của `rand` và ghi lại điều đó trong một tệp
_Cargo.lock_ duy nhất. Việc làm cho tất cả các crate trong không gian làm việc sử dụng cùng các phụ thuộc
có nghĩa là các crate sẽ luôn tương thích với nhau. Hãy thêm
crate `rand` vào phần `[dependencies]` trong tệp _add_one/Cargo.toml_
để chúng ta có thể sử dụng crate `rand` trong crate `add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Bây giờ chúng ta có thể thêm `use rand;` vào tệp _add_one/src/lib.rs_, và việc xây dựng
toàn bộ không gian làm việc bằng cách chạy `cargo build` trong thư mục _add_ sẽ tải về
và biên dịch crate `rand`. Chúng ta sẽ nhận được một cảnh báo vì chúng ta không
tham chiếu đến `rand` mà chúng ta đã đưa vào phạm vi:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

Tệp _Cargo.lock_ ở cấp cao nhất bây giờ chứa thông tin về phụ thuộc của
`add_one` vào `rand`. Tuy nhiên, mặc dù `rand` được sử dụng ở đâu đó trong
không gian làm việc, chúng ta không thể sử dụng nó trong các crate khác trong không gian làm việc trừ khi chúng ta thêm
`rand` vào các tệp _Cargo.toml_ của chúng nữa. Ví dụ, nếu chúng ta thêm `use rand;`
vào tệp _adder/src/main.rs_ cho gói `adder`, chúng ta sẽ gặp lỗi:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Để khắc phục điều này, hãy chỉnh sửa tệp _Cargo.toml_ cho gói `adder` và chỉ ra
rằng `rand` cũng là một phụ thuộc của nó. Việc xây dựng gói `adder` sẽ
thêm `rand` vào danh sách các phụ thuộc cho `adder` trong _Cargo.lock_, nhưng không có
bản sao bổ sung nào của `rand` được tải xuống. Cargo sẽ đảm bảo rằng mọi
crate trong mọi gói trong không gian làm việc sử dụng gói `rand` sẽ sử dụng
cùng một phiên bản miễn là chúng chỉ định các phiên bản tương thích của `rand`, giúp chúng ta tiết kiệm
không gian và đảm bảo rằng các crate trong không gian làm việc sẽ tương thích với
nhau.

Nếu các crate trong không gian làm việc chỉ định các phiên bản không tương thích của cùng một phụ thuộc,
Cargo sẽ giải quyết từng phiên bản trong số chúng, nhưng vẫn sẽ cố gắng giải quyết ít phiên bản nhất
có thể.

Lưu ý rằng Cargo chỉ đảm bảo tính tương thích trong các quy tắc của [Semantic Versioning].
Ví dụ, giả sử một không gian làm việc có một crate phụ thuộc vào `rand` 0.8.0, và một crate khác
phụ thuộc vào `rand` 0.8.1. Các quy tắc semver nói rằng 0.8.1 tương thích với 0.8.0,
so với cả hai crate sẽ phụ thuộc vào 0.8.1 (hoặc có khả năng là một bản vá gần đây hơn, như 0.8.2). Nhưng nếu
một crate phụ thuộc vào `rand` 0.7.0 và crate khác vào `rand` 0.8.0, những phiên bản đó không tương thích semver.
Do đó, Cargo sẽ sử dụng một phiên bản `rand` khác nhau cho mỗi crate.

#### Thêm một Bài kiểm tra vào một Không gian làm việc

Để thực hiện một cải tiến khác, hãy thêm một bài kiểm tra cho hàm `add_one::add_one`
bên trong crate `add_one`:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Bây giờ hãy chạy `cargo test` trong thư mục _add_ ở cấp cao nhất. Chạy `cargo test` trong
một không gian làm việc được cấu trúc như thế này sẽ chạy các bài kiểm tra cho tất cả các crate trong
không gian làm việc:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Phần đầu tiên của đầu ra cho thấy bài kiểm tra `it_works` trong crate `add_one`
đã vượt qua. Phần tiếp theo cho thấy không có bài kiểm tra nào được tìm thấy trong crate `adder`,
và sau đó phần cuối cùng cho thấy không có bài kiểm tra tài liệu nào được tìm thấy trong
crate `add_one`.

Chúng ta cũng có thể chạy các bài kiểm tra cho một crate cụ thể trong một không gian làm việc từ
thư mục cấp cao nhất bằng cách sử dụng cờ `-p` và chỉ định tên của crate
mà chúng ta muốn kiểm tra:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Đầu ra này cho thấy `cargo test` chỉ chạy các bài kiểm tra cho crate `add_one` và
không chạy các bài kiểm tra của crate `adder`.

Nếu bạn xuất bản các crate trong không gian làm việc lên [crates.io](https://crates.io/),
mỗi crate trong không gian làm việc sẽ cần được xuất bản riêng biệt. Giống như `cargo`
test, chúng ta có thể xuất bản một crate cụ thể trong không gian làm việc của mình bằng cách sử dụng cờ `-p`
và chỉ định tên của crate mà chúng ta muốn xuất bản.

Để thực hành thêm, hãy thêm một crate `add_two` vào không gian làm việc này theo cách tương tự
như crate `add_one`!

Khi dự án của bạn phát triển, hãy cân nhắc sử dụng một không gian làm việc: nó cho phép bạn làm việc với
các thành phần nhỏ hơn, dễ hiểu hơn thay vì một khối mã lớn. Hơn nữa,
việc giữ các crate trong một không gian làm việc có thể giúp việc phối hợp giữa các crate dễ dàng hơn nếu
chúng thường xuyên được thay đổi cùng một lúc.

{{#quiz ../quizzes/ch14-03-cargo-workspaces.toml}}

[Semantic Versioning]: https://semver.org/---
