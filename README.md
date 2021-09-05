# The Simplified Integritee BOOK with NOTES
made by my blockchain passion :hearts:.

[The Integritee Book](https://book.integritee.network) included my notes & under my understanding.


### Hướng dẫn đọc sách ở local
- Install Rust từ [official guide](https://www.rust-lang.org/tools/install)
- Sau khi Rust install thành công, thì cài [mdBook](https://github.com/rust-lang/mdBook) (1 tool bằng Rust cho phép tạo sách từ markdown files) từ cargo với command sau:
```
cargo install mdbook
```
- Clone repository này về máy bạn, sau đó cd vào directory và run command:
```
mdbook build
```
- Build thành công rồi thì giờ launch sách lên đọc thôi nào:
```
mdbook serve -p 3001
```
Mặc định nếu chỉ chạy `mdbook serve` port được sử dụng là 3000.

### TODO list:
- [ ] page [What is Integritee Good For?](./what_for.md) cần được vẽ sequence diagram tóm tắt cho từng use cases.
- [ ] các page khác trong quá trình update kèm notes.