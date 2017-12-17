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
Đầu tiêu thiết bị A kiểm tra địa chỉ IP đích của gói tin có nằm cùng mạng local của mình hay ko bằng cách gửi ARP request là bản tin broadcast đến tất cả thiết bị trong mạng local. Nếu có thì A sẽ nhận được bản tin unicast trực tiếp từ B. Ta biết răng việc gửi gói tin trong cùn một mạng thông qua switch là dựa vào MAC address. Nếu địa chỉ IP của B nằm trên vùng mạng khác, thì A sẽ gửi  ARP request đến router nằm trên cùng mạng nội bộ. Gói tin được đóng gói sau đó chuyển qua quá trình phân giải địa chỉ ARP và được chuyển đi. Đến router nội bộ thì router sẽ dùng   [proxy ARP](#proxy)  để respone lại cho A.

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

<a name="cache"></a>
### 3.ARP caching 
ARP là một giao thức phân giải địa chỉ động. Quá trình gửi gói tin Request và Reply sẽ tiêu tốn băng thông mạng. Chính vì vậy càng hạn chế tối đa việc gửi gói tin Request và Reply sẽ càng góp phần làm tăng khả năng họat động của mạng. Từ đó sinh ra nhu cầu của ARP Caching.

##### Các thành phần tĩnh và động của ARP cache

ARP Cache có dạng giống như một bảng map giữa địa chỉ hardware và địa chỉ IP. Có hai cách đưa các thành phần tương ứng vào bảng ARP:

**- Các thành phần arp cache tĩnh:** Đây là cách mà các thành phần tương ứng trong bảng ARP được đưa vào lần lượt bởi người quản trị. Công việc được tiến hành một cách thủ công.

<img src="https://i.imgur.com/x4kf5zM.png">

**- Các thành phần arp cache động:**  Đây là quá trình mà các thành phần địa chỉ hardware/IP được đưa vào ARP cache một cách hoàn toàn tự động bằng phần mềm sau khi đã hoàn tất quá trình phân giải địa chỉ. Chúng được lưu trong cache trong một khoảng thời gian và sau đó sẽ được xóa đi.

Dynamic Cache được sử dụng rộng rãi hơn vì tất cả các quá trình diễn ra tự động và không cần đến sự tương tác của người quản trị. Tuy nhiên static cache vẫn có phạm vi ứng dụng nhất định của nó. Đó là trường hợp mà các workstation nên có static ARP entry đến router và file server nằm trong mạng. Điều này sẽ hạn chế việc gửi các gói tin để thực hiện quá trình phân giải địa chỉ.

Tuy nhiên ngoài hạn chế của việc phải nhập bằng tay, static cache còn thêm hạn chế nữa là khi địa chỉ IP của các thiết bị trong mạng thay đổi thì sẽ dẫn đến việc phải thay đổi ARP cache.

#### Xóa thông tin trong cache
Ta xét trường hợp bảng cache của một thiết bị A, trong đó có chứa thông tin về thiết bị B trong mạng. Nếu các thông tin trong cache được lưu mãi mãi, sẽ có một số vấn đề như sau xảy ra :

- Địa chỉ phần cứng thiết vị được thay đổi : Đây là trường hợp khi thiết bị B được thay đổi card mạng hay thiết bị giao tiếp, làm thay đổi địa chỉ MAC của thiết bị. Điều này làm cho các thông tin trong cache của A không còn đúng nữa.

- Địa chỉ IP của thiết bị được thay đổi : Người quản trị hay nhà cung cấp thay đổi địa chỉ IP của B, cũng làm cho thông tin trong cache của A bị sai lệch.

- Thiết bị được rút ra khỏi mạng : Khi B được rút ra khỏi mạng nhưng A không được biết, và gây lãng phí về tài nguyên của A để lưu thông tin không cần thiết và tốn thời gian để tìm kiếm.

Để tránh được những vấn đề này, các thông tin trong dynamic cache sẽ được tự động xóa sau một khoảng thời gian nhất định. Quá trình này được thực hiện một cách hoàn toàn tự động khi sử dụng ARP với khoảng thời gian thường là 10 hoặc 20 phút. Sau một khoảng thời gian nhất định được lưu trong cache , thông tin sẽ được xóa đi. Lần sử dụng sau, thông tin sẽ được update trở lại.

<a name="proxy"></a>
### 4.Proxy ARP
 Tuy nhiên nếu hai thiết bị A và B bị chia cắt bởi 1 router thì chúng sẽ được coi như là không local với nhau nữa. Khi A muốn gửi thông tin đến B, A sẽ không gửi trực tiếp được đến B theo địa chỉ datalink layer, mà phải gửi qua router và được coi là cách nhau 1 hop ở network layer.


**Vì sao cần phải có Proxy ARP?**

Khác với các trường hợp thông thường, nhiều trường hợp hai thiết bị A và B nằm trên 2 segment vật lý khác nhau nhưng được kết nối qua một router và cùng nằm trong một mạng IP hay một IP subnet. Lúc này A và B sẽ coi nhau có quan hệ local.

Giả sử ta có tình huống A muốn gửi thông tin cho B. A nghĩ B trong cùng nội mạng và tìm trong bảng ARP cache. A không lưu địa chỉ MAC của B và bắt đầu tiến hành quá trình phân giải địa chỉ. A broadcast gói ARP request trong nội mạng để tìm địa chỉ MAC của B. Sẽ có vấn đề xảy ra : B không cùng nằm trong mạng và sẽ không nhận được gói tin broadcast cũng như router kết nối sẽ không forward gói broadcasr từ A qua B ( router không truyền các gói broadcast ở lớp datalink ).
<img src="https://i.imgur.com/V4P8JM2.png">
Vì vậy B không bao giờ nhận được request từ A cũng như A sẽ không bao giờ có được địa chỉ MAC của B.

**Hoạt động của Proxy ARP**

Giải pháp cho tình huống này được gọi là ARP proxying hay Proxy ARP. Trong công nghệ này, router nằm giữa 2 mạng local sẽ được cấu hình để đáp ứng các gói tin broadcast gửi từ A thay cho B.

<img src="https://i.imgur.com/n64HeSZ.png">

Router sẽ không gửi cho A địa chỉ MAC của B, vì dù thế nào A và B cũng nằm trên hai mạng khác nhau và không thể gửi trực tiếp đến nhau được. Thay vào đó router sẽ gửi cho A các địa chỉ MAC của chính router.

<img src="https://i.imgur.com/bkdEsst.png">

A sau đó sẽ gửi các gói tin cho router, và router sẽ forward sang cho B. Quá trình cũng hoàn toàn diễn ra tương tự khi B muốn gửi thông tin cho A, hay cho bất cứ thiết bị nào mà đích đến của gói tin là một thiết bị ở một mạng khác.

*****Ví dụ*****

<img src ="https://i.imgur.com/PSyXZRQ.png">

một router kết nối hai mạng LAN 172.16.10.0/24 và 172.16.20.0/24 tuy nhiên chỉ có Host A là có subnet là /16 nên khi mà A muốn liên lạc với C hoặc D nó sẽ nghĩ rằng là đang cùng mạng với C và D lúc này nó sẽ gửi gói tin ARP để xin địa chỉ MAC tương ứng. và điều chắc chắn là không thể nhận được Arp Replay nếu như không thiết lập Proxy Arp trên Router. Lúc này khi nhận được gói tin Arp của A thay vì forward thì router sẽ xem xét nó có đường tời C và D hay không nếu  có nó sẽ trả lời cho A gói tin Arp reply nhưng với địa chỉ Mac là cổng nối trực tiếp với A. `00-00-0c-94-36-ab`

Gói tin A broadcast trên subnet A:
<img src="https://i.imgur.com/IbW2hl3.png">

Gói tin router gửi trả lời cho A:
 <img src="https://i.imgur.com/vR7cwDM.png">

Sau khi nhận được ARP reply, Host A sẽ cập nhật ARP table:
<img src="https://i.imgur.com/WnMTEUf.png)">

ARP cache của Host A:
<img src="https://i.imgur.com/6uWtENn.png)">

#####Ưu điểm và nhược điểm của Proxying
Ưu điểm dễ nhận thấy của Proxy ARP là các router hoạt động nhưng các thiết bị không hề cảm nhận được sự hoạt động của nó. Các hoạt động gửi nhận giữa hai thiết bị thuộc hai LAN khác nhau vẫn diễn ra bình thường.

Hạn chế:

- Thứ nhất, nó làm tăng độ phức tạp của mạng 
- Thứ hai, nếu nhiều hơn một router kết nối tới hai LAN cùng nằm trong một mạng IP, nhiều vấn đề có thể phát sinh. 
- Thứ ba, công nghệ này cũng tạo nên những mối nguy cơ tiềm ẩn về an ninh và bảo mật, khi các router được cấu hình proxy, tạo nguy cơ về giả mạo địa chỉ.

Do vậy, giải pháp tốt nhất là thiết kế lại topo mạng để chỉ một router kết nối tới hai LAN nằm trong một mạng IP.
