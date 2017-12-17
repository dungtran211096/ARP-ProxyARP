# Tổng quan ARP (ARP cache, Proxy ARP...)
- [1. Định nghĩa ARP](#arpdefinition)
- [2. Cơ chế hoạt động](#protocol)
- [3. ARP caching](#cache)
- [4. Proxy ARP](#proxy)

==================================

-  Mạng Lan nhỏ hoạt động dựa trên layer 1 và 2 trong mô hình OSI (Physic và data link). Nhưng Các giao thức liên mạng lại dựa trên địa chỉ layer 3(network).
 
- Có 2 phương pháp phân giải địa chỉ : map trực tiếp và phân giải động.Việc Map trực tiếp gặp nhiều khó khăn do địa chỉ MAC(datalink) có 48bit trong khi địa chỉ IP(network) là 32 bit. Để linh hoat sử dụng nên ARP ra đời, theo chuẩn RFC 826. 
 
<a name="arpdefinition"></a>

### 1. Định nghĩa ARP
 ARP là giao thức phân giải địa chỉ
động giữa địa chỉ network và địa chỉ datalink. Quá trình thực hiện bằng cách: một thiết bị IP trong mạng gửi một gói tin local broadcast đến toàn mạng yêu cầu thiết bị khác gửi trả lại địa chỉ phần cứng ( địa chỉ lớp datalink ) hay còn gọi là Mac Address của mình.

 Ban đầu ARP chỉ **được sử dụng trong mạng Ethernet** để phân giải địa chỉ IP và địa chỉ MAC. Nhưng ngày nay ARP đã được ứng dụng rộng rãi và dùng trong các công nghệ khác dựa trên lớp hai.

<a name="protocol"></a>
### 2. Cơ chế hoạt động

<img src="https://i.imgur.com/QVbDoVR.png">
Quá trình thực hiện ARP được bắt đầu khi một thiết bị nguồn trong một mạng IP có nhu cầu gửi một gói tin IP.
Đầu tiêu thiết bị A kiểm tra địa chỉ IP đích của gói tin có nằm cùng mạng local của mình hay ko bằng cách gửi ARP request là bản tin broadcast đến tất cả thiết bị trong mạng local. Nếu có thì A sẽ nhận được bản tin unicast trực tiếp từ B. Ta biết răng việc gửi gói tin trong cùn một mạng thông qua switch là dựa vào MAC address. Nếu địa chỉ IP của B nằm trên vùng mạng khác, thì A sẽ gửi  ARP request đến router nằm trên cùng mạng nội bộ. Gói tin được đóng gói sau đó chuyển qua quá trình phân giải địa chỉ ARP và được chuyển đi. Đến router nội bộ thì router sẽ dùng [proxy ARP](#proxy)để respone lại cho A.

Về cơ bản, ARP là quá trình 2 chiều request/response giữa các thiết bị cùng mạng nội bộ. Thiết bị source request bằng cách gửi bản tin broadcast đến toàn bộ thiết bị cùng mạng và thiết bị destination response bằng một bản tin unicast cho thiết bị source.

*Định dạng gói tin ARP*

<img src="https://i.imgur.com/dDyXsoW.png">

1. Sender Hardware Address : địa chỉ lớp hai của thiết bị gửi bản tin 
2. Sender Protocol Address : Địa chỉ lớp ba ( hay địa chỉ logic ) của thiết bị gửi bản tin 
3. Target Hardware Address : Địa chỉ lớp hai ( địa chỉ phần cứng ) của thiết bị đích của bản tin 
4. Target Protocol Address : Địa chỉ lớp ba ( hay địa chỉ logic ) của thiết bị đích của bản tin.

#### Các bước hoạt động của ARP:

1. Source Device Checks Cache : Trong bước này, thiết bị sẽ kiểm tra cache ( bộ đệm ) của mình. Nếu đã có địa chỉ IP đích tương ứng với MAC nào đó rồi thì lập tức chuyển sang [bước 9](#9)

2. Source Device Generates ARP Request Message : Bắt đầu khởi tạo gói tin ARP Request với các trường địa chỉ như trên

3. Source Device Broadcasts ARP Request Message : Thiết bị nguồn quảng bá gói tin ARP Request trên toàn mạng

4. Local Devices Process ARP Request Message : Các thiết bị trong mạng đều nhận được gói tin ARP Request. Gói tin được xử lý bằng cách các thiết bị đều nhìn vào trường địa chỉ Target Protocol Address. Nếu trùng với địa chỉ của mình thì tiếp tục xử lý, nếu không thì hủy gói tin

5. Destination Device Generates ARP Reply Message : Thiết bị với IP trùng với IP trong trường Target Protocol Address sẽ bắt đầu quá trình khởi tạo gói tin ARP Reply bằng cách lấy các trường Sender Hardware Address và Sender Protocol Address trong gói tin ARP nhận được đưa vào làm Target trong gói tin gửi đi. Đồng thời thiết bị sẽ lấy địa chỉ datalink của mình để đưa vào trường Sender Hardware Address

6. Destination Device Updates ARP Cache : Thiết bị đích ( thiết bị khởi tạo gói tin ARP Reply ) đồng thời cập nhật bảng ánh xạ địa chỉ IP và MAC của thiết bị nguồn vào bảng ARP cache của mình để giảm bớt thời gian xử lý cho các lần sau

7. Destination Device Sends ARP Reply Message : Thiết bị đích bắt đầu gửi gói tin Reply đã được khởi tạo đến thiết bị nguồn. Gói tin reply là gói tin gửi unicast

8. Source Device Processes ARP Reply Message : Thiết bị nguồn nhận được gói tin reply và xử lý bằng cách lưu trường Sender Hardware Address trong gói reply như địa chỉ phần cứng của thiết bị đích

9. Source Device Updates ARP Cache : Thiết bị nguồn update vào ARP cache của mình giá trị tương ứng giữa địa chỉ network và địa chỉ datalink của thiết bị đích. Lần sau sẽ không còn cần tới request <a name="9"></a>

