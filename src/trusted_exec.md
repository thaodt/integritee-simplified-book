# Trusted Execution - Thực thi đáng tin cậy là gì nhỉ?

Cách dẫn dắt của quyển sách khá hay, nên mình sẽ giữ nguyên vấn đề đặt ra như sau:

Chúng ta đã quen với sự thật là bằng phép thần kỳ nào đó chúng ta sống với lòng tin chân chính, phải tin tưởng vào IT admintrators (quản trị viên). Trong khi những quản trị viên này từng là nhân viên nội bộ tại các công ty của chúng ta, nhưng ngày nay chúng ta thường làm việc trên các cloud platforms.

Các admins có thể đọc và chỉnh sửa tất cả dữ liệu được xử lý trên bất kỳ cỗ máy này mà họ quản lý. Không may thay quyền năng tối thượng này không chỉ dành cho các quản trị viên mà chúng ta tin tưởng mà còn vô tình tạo điều kiện cho các hackers chiếm đặc quyền quản trị. Bất kể trình độ như thế nào, bất kể công ty ra sao cũng khó có thể miễn nhiễm với những cuộc tấn công như vậy.

Okay, so TEEs can shine here, true or false? 


Bây giờ bạn thử hình dung TEE như là 1 bộ đồng xử lý mà có thể quản lý chính các key mật mã của riêng nó và chỉ thực thi những program có mã hash hoặc fingerprint, tương ứng với mã gốc. Nhà sản xuất của bộ xử lý này đảm bảo rằng bằng thiết kế phần cứng của họ !? (mình tạm đặt nghi vấn ở đây trước, vì không gì có tính tuyệt đối cả), thì *không ai* (nhấn mạnh **KHÔNG MỘT AI**) có quyền truy cập vào các khóa nội hàm của TEE hoặc có thể đọc được bộ nhớ của nó.

Hơn thế nữa, nhà sản xuất có thể xác thực từng TEE và cung cấp remote attestation đến 1 người dùng để xác nhận program chưa được thẩm định đang thực sự chạy trên 1 TEE **chính cống** (ở đây họ dùng từ **genuine** TEE - giống như bạn hay gặp từ này khi máy tính đang sử dụng bản Windows lậu/dùng thử, trên Windows ở góc hay hiện term này để khuyên bạn verify bản Windows đó), ngay cả khi trên thực tế máy tính đó đang được đặt ở 1 off-site data center.

Nói 1 cách ngắn gọn, TEEs hứa hẹn là tính toàn vẹn (integrity) và tính bảo mật của việc tính toán (remote). Tuy nhiên, bạn nên nhận thức về các **mối đe dọa security** vẫn có thể xảy ra (as always!).

Giả sử chúng ta tin vào độ toàn vẹn cũng như năng lực thiết kế của các nhà sản xuất TEE, TEEs cho phép chúng ta thực thi bất kỳ update trạng thái nào mà *không phải chia sẻ* dữ liệu của chúng ta với blockchain validator hoặc những người dùng khác. Nhờ đó, việc chuyển tokens riêng tư, các smart contracts riêng tự và các channel trạng thái riêng tư trở nên khả thi và tương đối rẻ (?).

Đến đây chúng ta đã hiểu WH-at is Trusted Execution, trong đoạn trên TEE có nói đến term [**Remote Attestion**](./remote_attestation.md), vậy giờ chúng ta sẽ đi sâu hơn 1 chút về kỹ thuật này nhé. Lets go!