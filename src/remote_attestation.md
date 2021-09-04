# Remote Attestation

Mục tiêu của Attestation (chứng thực) là thuyết phục bên thứ 3 1 đoạn code cụ thể đang chạy trên 1 phần cứng Intel SGX chính hãng.

## Thuyết phục người sử dụng Integritee


Cơ sở đặt ra là một user khi tương tác với Integritee mong muốn chắc chắn là **public key bọc khiên** (mình hình tượng hóa term gốc chút) mà cô ấy sử dụng để mã hóa cuộc gọi của cô ấy đến STF bắt nguồn định hình thành một vỏ bọc (enclave):
1. đang chạy trên một phần cứng Intel SGX chính hãng.
2. chạy code official
3. truy cập trạng thái STF chính xác

Vậy chúng ta có các giải pháp nào ? **How-to convince?**

### Giải pháp Chứng thực từ xa cổ điển - **Classical Remote Attestation Solution** 

Use case chuẩn chỉnh cho Remote Attestation liên quan đến 1 nhà cung cấp dịch vụ (Service Provider - SP) như là dịch vụ video streaming muốn chắc chắn là app viewer của họ chạy trên 1 phần cứng SGX chính hãng và tôn trọng DRM. Do đó SP yêu cầu 1 bản báo giá từ app và gửi bản đó đến ***Intel Attestation Services (IAS)** - dịch vụ ký vào bản báo giá đó nếu nó chứng thực được bản này là chính cống.

Vấn đề ở đây là IAS chỉ nói chuyện với các clients đã đăng ký rồi. Bạn cần phải thực hiện đăng ký để lấy được SPID mà cần được cung cấp cùng với các yêu cầu.

### Đăng ký chứng thực On-Chain - **Attestation Registry On-Chain**
Okay nhìn vào vấn đề ở giải pháp thứ nhất, giải pháp này như sau:

Khá là không thực tế khi yêu cầu mỗi client đăng ký với Intel và thực hiện RA trước mỗi request. Vì thế chúng ta muốn để **SubstraTEE worker operators** chứng thực các vỏ bọc (enclaves) của họ với IAS và ghi vào bản báo giá đã ký cũng như chứng chỉ của họ vào blockchain để mọi người đều có thể xác minh.

Điều này thực sự làm thay đổi giao thức chứng thực (attestation protocol). Bây giờ SP và vỏ bọc trong phác thảo ở trên đều đang chạy trên cùng 1 cỗ máy. SubstraTEE-worker sẽ tự thực hiện một giao thức chứng thực với chính vỏ bọc của nó và nhận báo giá do IAS ký. Giống như vậy, chỉ Integritee operators mới cần đăng ký với IAS.

#### **Sequence Diagram**

Báo cáo chứng thực được ghi vào một on-chain registry chứa:

- enclave quote
    - report body
        - MRENCLAVE (mã hash của vỏ bọc tạo ra)
        - Product ID (hard-coded bên trong Integritee mã nguồn)
        - Security Version (hard-coded bên trong Integritee mã nguồn)
        - dữ liệu người dùng là mã hash trong ngữ cảnh:
            - enclave-individual signing pubkey
            - mã hash khối mới nhất
    - ...
- IAS response
    - `body`
        - status (OK||CONFIGURATION_NEEDED|GROUP_REVOKED....)
        - timestamp
        - enclave quote
        ...
    - IAS certificate
    - Chữ ký IAS trên `body` attribute

Bây giờ bất kỳ user nào có thể xác minh chữ ký IAS và MRENCLAVE (được cung cấp bởi vỏ bọc Integritee có thể được xây dựng một cách xác định). Xem qua [ví dụ]((https://github.com/rodolfoams/sgx-retrieve-identity/blob/5be913b96b2a6e5a0e1158ad169b977507291faa/Makefile#L253)) về cách mà bạn có thể tách MRENCLAVE sau khi xây dựng vỏ bọc bao quanh.

Còn về worker, bây giờ cos thể xuất bản cái public key bọc khiên của anh ấy mà đã được ký với chính khóa cá nhân được nêu trong bản báo giá (quote).

Các workers sẽ lặp lại hành động chứng thực từ xa này (remote attestation) trong những khoảng thời gian đều đặn hợp lý. ( ví dụ như là mỗi tháng 1 lần)


### **Enclave Registry On-Chain**
Để chain validator có thể xác minh MRENCLAVE, phải có một đồng thuận về MRENCLAVE của phiên bản Integritee hợp lệ.

Các nhà phát triển của Integritee sẽ đề xuất các bản mã cập nhật để được bỏ phiếu. Các validators kiểm tra các bản mã này và bỏ phiếu chọn dưới tư cách thay mặt hoặc chống lại mỗi bản đề xuất đó. MRENCLAVE có thể được sao chép bằng cách sao chép substraTEE-woker repo, build nó và sau đó:
```
sgx_sign dump -enclave enclave.signed.so -dumpfile out.log
```

**TODO**: chúng ta có lẽ cần cung cấp một docker env để đạt được các bản builds xác định.

## Secret Provisioning - Cung cấp tính bảo mật

Để thiết lập được việc chia sẻ các secrets giữa các workers, chúng cần tự thuyết phục lẫn nhau rằng chúng là hàng thật (chính cống) trước khi bước vào một số giao thức Tạo khóa phân tán - **Distributed Key Generation (DKG)**.

## Sealing - Ấn ký

Các secrets được đóng dấu với Intel's [SGX Sealing](https://software.intel.com/en-us/blogs/2016/05/04/introduction-to-intel-sgx-sealing). Có 2 loại niêm dấu khác nhau tồn tại.
- MRENCLAVE là duy nhất cho mỗi bản build và mỗi phần của phần cứng.
- Trong khi đó MRSIGNER thì được đựa trên thẩm quyền của 1 nhà cung cấp phần mềm.

Loại niêm dấu thứ 2 là thực tế cho phần mềm độc quyền bởi vì các nhà cung cấp có thể cập nhật phần mềm của họ mà *không phải tái cấp phép* (re-provisioning) lại cho các secrets.

Tuy nhiên, đối với các dự án open source phi tập trung, MRSIGNER không thể áp dụng vì không có quyền hạn nào có thể ký các bản builds.

Vì thế, *định danh vỏ bọc - enclave identity* MRSIGNER phải được áp dụng.

### SW updates - Các cập nhật phần mềm

Vì các bản cập nhật phần mềm sẽ có các đo lường khác nhau, bản build mới không thể đọc trạng thái mà đã được mã hóa bởi bản build cũ. Chứng thực cục bộ cho phép phien bản mới request các secrets được cung cấp.

Chúng ta giả sử các bản builds có thể tái tạo lại cho các vỏ bọc mà có thể khả thi với các Rust subject tùy theo một vài giả định. Hiện tại, hãy check [issue này](https://github.com/rust-lang/rust/issues/34902) và [cargo-repro](https://github.com/rust-secure-code/cargo-repro) - build và xác minh từng byte cho các Rust packages sử dụng quy trình dựa trên Cargo (Rust package manager).

**My Notes**:
- Ở thời điểm viết sách này, issue [Bit-for-bit deterministic / reproducible builds](https://github.com/rust-lang/rust/issues/34902) chưa được giải quyết, nhưng hiện tại theo mình thấy issue đã closed rồi. Vậy nên việc này đồng nghĩa với giả định ở trên đã được hiện thực hóa. Bạn có thể đọc các commits fixed issue ngay trong issue luôn nếu hứng thú.
- Cargo-repro đã được archived bởi chính tác giả và anh ấy cũng link tới 1 repo mới dựa trên cùng concept ở [đây](https://github.com/kpcyrd/rebuilderd).

Simplified Protocol - Giao thức đơn giản hóa

1. mã hash TCB của phiên bản mới được bỏ phiếu bởi sự đồng thuận on-chain.
2. phiên bản mới đăng ký chứng thực của nó on-chain.
3. các phiên bản cũ chia sẻ secret được cấp phép với phiên bản mới đang chạy trên cùng cỗ máy bằng chứng thực cục bộ (intra-platform) nếu TCB của phiên bản mới tuơng ứng với bản đăng ký on-chain.

Tham khảo [Intel's sealing paper](https://software.intel.com/en-us/articles/innovative-technology-for-cpu-based-attestation-and-sealing)

## IAS - Intel Attestation Services (Các dịch vụ chứng thực của Intel)

Intel định nghĩa [các mode (chế độ)](https://software.intel.com/en-us/blogs/2016/01/07/intel-sgx-debug-production-prelease-whats-the-difference) khác nhau cho việc chạy các vỏ bọc.

- **compilation modes** - các chế độ biên dịch: *Debug*, *Release*, *Pre-Release*, *Simulation*
- **launching modes** - các chế độ khởi động: *Debug*, *Production*

### EPID - Enhanced Privacy ID (Đinh danh của quyền riêng tư nâng cao)

Một khóa chữ ký nhóm chỉ được biết đến với vỏ bọc định giá. Chỉ được sử dụng cho remote attestation.

### SPID - Service Provider ID (Định danh của 1 nhà cung cấp dịch vụ)

Để giao tiếp được với IAS cần phải có SPID. Các developers có thể đạt được SPID của họ thông qua việc [đăng ký với Intel](https://software.intel.com/en-us/form/sgx-onboarding) (chỉ cho phép chứng thực các vỏ bọc ở chế độ DEBUG!)

Bạn có thể yêu cầu định giá dạng *linkable* hoặc *unlinkable*.

**tl;dr**: chọn UNLINKABLE là 1 lựa chọn an toàn. Nhưng đừng mong đợi bạn đợi ẩn danh.

Trong cả hai trường hợp, vỏ bọc định giá sử dụng một chữ ký nhóm cho một định giá. Bạn chỉ có thể quyết định nếu bạn mong muốn 2 chữ ký tiếp theo có thể liên kết với nhau hay không (một người quan sát học được rằng "định giá được ký bởi cùng 1 platform").

Trong bất kỳ trường hợp nào, Intel có thể nhận dạng BẠN bởi SSID vì bạn sử dụng SSID của bạn cho remote attestation với IAS. Nó chỉ không biết về nền tảng phần cứng nào mà bản định giá request từ đó.


*Giờ chúng ta sẽ tìm hiểu thêm về 2 mode khởi động chạy vỏ bọc*

### Production vs. Debug Mode

Do chính sách của Intel, các developers chỉ có thể biên dịch các vỏ bọc trong các mode *Debug*, *Pre-Release* hoặc *Simulation*. Điều này có nghĩa là vỏ bọc sẽ luôn được khởi động ở chế độ *Debug*, chế độ này không cung cấp tính bảo mật bởi vì bộ nhớ của vỏ bọc không được mã hóa.

Để biên dịch các vỏ bọc ở mode *Release* (và chạy chúng ở mode *Production*), các nhà sản xuất phần mềm phải gắn cho nó một 1 cái giấy phép SGX Production.

Hơn thế nữa, remote attestion ở *Production* mode chỉ có thể được thực hiện khi có giấy phép như vậy.

*SCS đang xem xét các lựa chọn làm thế nào để áp dụng chính sach đó đến một hệ thống phi tập trung với Intel.*

**Các giải pháp ứng cử**

1. 1 tập hợp *các công ty* (vd như SCS, web3 foundation) đăng ký 1 cái giấy phép production với Intel.
2. substraTEE-workers gửi bản định giá RA của chúng đến chain.
3. *công ty* lắng nghe các định giá RA mới và gửi chúng đến IAS với SPID của *công ty*.
4. *công ty* gửi báo cáo IAS đến chain.

## Literature

danh sách các pointers hỗn loạn:

* [bản đánh giá SGX PSE (Platform Service/Security Enclave) cho các bộ đếm đơn (monotonic counter) và thời gian đáng tin cậy](https://davejingtian.org/2017/11/10/some-notes-on-the-monotonic-counter-in-intel-sgx-and-me/)


Vậy đâu là nơi Integritee được sử dụng? Đi tiếp chương kế nhé, chúng ta sẽ cùng khảo sát các use cases nơi mà Integritee được áp dụng. 