#Tổng Quan giao thức SSH
## Mục lục
[1.Khái Niệm] (#1)

[2.Tổng Quan các đặc điểm của SSH] (#2)

[3.Các thuộc tính] (#3)

[4.Cơ chế hoạt động] (#4)

[5.Các thuật toán sử dụng trong ssh] (#5)

[6.Tham Khảo] (#6)

<a name="1"></a>
### 1.Khái niệm
- SSH (tiếng Anh: Secure Shell) là 1 giao thức mạng dùng để thiết lập kết nối mạng 1 cách bảo mật.
- SSH hoạt động ở lớp trên trong mô hình phân lớp TCP/IP.
- Các công cụ SSH (như là OpenSSH, PuTTy,…) cung cấp cho người dùng cách thức để thiết lập kết nối mạng được mã hoá để tạo 1 kênh kết nối riêng tư.
- Hơn nữa tính năng tunneling (hoặc còn gọi là port forwarding) của các công cụ này cho phép chuyển tải các giao vận theo các giao thức khác.
- Mỗi khi dữ liệu được gửi bởi 1 máy tính vào mạng, SSH tự động mã hoá nó. Khi dữ liệu được nhận vào, SSH tự động giải mã nó.
- Kết quả là việc mã hoá được thực hiện trong suốt.

<a name="2"></a>  
### 2.Tổng Quan các đặc điểm của SSH
- Các đặc điểm chính của giao thức SSH là:
<ul>
	<li>Tính bảo mật (Privacy): mã hoá dữ liệu dựa trên khoá ngẫu nhiên. (AES, DES, 3DES, ARCFOUR ...)</li>
	<li>Tính toàn vẹn (integrity): đảm bảo thông tin không bị biến đổi bằng phương pháp kiểm tra toàn vẹn mật mã (MD5 và SHA-1.).</li>
	<li>Chứng minh xác thực (authentication): kiểm tra định danh của ai đó để xác định đúng là người đó hay không</li>
	<li>Giấy phép (authorization) :dùng để điều khiển truy cập đến tài khoản.Quyết định ai đó có thể hoặc không thể làm gì.</li>
	<li>Chuyển tiếp (forwarding) hoặc tạo đường hầm (tunneling) để mã hoá những phiên khác dựa trên giao thức TCP/IP.</li>
</ul>

<a name="3"></a>
### 3.Các thuộc tính 
- Server: 1 chương trình cho phép đi vào kết nối SSH với 1 bộ máy, trình bày xác thực, cấp phép, … Trong hầu hết SSH bổ sung của Unix thì server thường là sshd.
- Client: 1 chương trình kết nối đến SSH server và đưa ra yêu cầu như là “log me in” hoặc “copy this file”. Trong SSH1, SSH2 và OpenSSH, client chủ yếu là ssh và scp.
- Session: phiên kết nối giữa 1 client - 1 server. Nó bắt đầu sau khi client xác thực thành công đến 1 server và kết thúc khi kết nối chấm dứt. Session có thể được tương tác với nhau hoặc có thể là 1 chuyến riêng.
<img src="http://i.imgur.com/N68V7YY.jpg" />

- Key
<ul>
	<li>Trong mã hóa: Nó chắc chắn rằng chỉ người nào đó giữ khoá.(hoặc 1 ai có liên quan) có thể giải mã thông điệp.</li>
	<li>Có hai loại khóa: khoá đối xứng hoặc khoá bí mật và khoá bất đối xứng hoặc khóa công khai.</li>
	<li>1 khoá bất đối xứng hoặc khoá công khai có hai phần: thành phần công khai và thàn phần bí mật.</li>
	<li>Các loại khóa thành phần</li>
	<ul>
		<li>User key: tồn tại lâu dài, là khóa bất đối xứng dành cho client,chứng minh nhận dạng của client (1 client có thể có nhiều khóa).</li>
		<li>Host key: tồn tại lâu dài, là khóa bất đối xứng dành cho server,chứng minh nhận dạng của server.Mỗi server có thể có host key riêng hoặc dùng chung.</li>
		<li>Server key: tồn tại tạm thời, là khóa bất đối xứng được tạo bởi server theo chu kỳ thường xuyên	và bảo vệ session key.</li>
		<li>Session key: phát sinh ngẫu nhiên, là khóa đối xứng cho việc mã hóa truyền thông giữa 1 SSH client và 1 SSH server.Nó được chia ra làm 2 thành phần cho client và server. </li>
	</ul>
</ul>

- Key generator: chương trình tạo ra những loại khoá lâu dài( user key và host key) cho SSH. SSH1, SSH2 và OpenSSH có chương trình ssh-keygen.
- Known hosts database: 1 chồng host key. Client và server dựa vào cơ sở dữ liệu này để xác thực lẫn nhau.
- Agent: chương trình lưu user key trong bộ nhớ.
- Configuration file: chứa các tham số cấu hình cho ssh server cũng như client. Các tham số bạn có thể tham khảo ở đây (https://www.freebsd.org/cgi/man.cgi?query=sshd_config&sektion=5).

<a name="4"></a>
### 4.Cơ chế hoạt động
<img src="http://i.imgur.com/8kob10F.jpg" />

- 1 phiên làm việc của SSH trải qua 4 bước chủ yếu như sau:
<ul>
	<li>Thiết lập kết nối ban đầu (SSH-TRANS).</li>
	<li>Tiến hành xác thực lẫn nhau (SSH-AUTH).</li>
	<li>Mở phiên kết nối để thực hiện các dịch vụ (SSH-CONN).</li>
	<li>Chạy các dịch vụ ứng dụng SSH ( có thể là SSH-SFTP).</li>
</ul>
- SSH-TRANS là khối xây dựng cơ bản cung cấp kết nối ban đầu, ghi chép giao thức, xác thực server, mã hóa cơ bản và các dịch vụ bảo toàn.
- Sau khi thiết lập SSH-TRANS, Client có 1 kết nối đơn, đảm bảo, luồng truyền byte full-duplex đến 1 xác nhận tương đương.
- Tiếp theo, client dùng SSH-AUTH thông qua kết nối SSH-TRANS đến xác thực của chính nó với server.
- SSH-AUTH định nghĩa một cái khung đối với việc nhiều cơ chế xác thực có thể được sử dụng.	
- SSH-AUTH chỉ yêu cầu một phương thức: khoá công cộng với thuật toán DSS.
- Sau khi xác thực, SSH client yêu cầu giao thức SSH-CONN để cung cấp một sự đa dạng của các dịch vụ thông qua một kênh đơn cung cấp bởi SSH-TRANS.
- 1 kênh thường bao gồm: những luồng đa thành phần khác nhau (hoặc channel) ngang qua kết nối bên dưới, quản lý X, TCP và agent forwarding, truyền tín hiệu thông qua kết nối, nén dữ liệu và thực thi chương trình từ xa.
- Cuối cùng, một ứng dụng có thể sử dụng SSH-SFTP qua một kênh SSH-CONN để cung cấp truyền file và các chức năng thao tác hệ thống tập tin từ xa.

<a name="5"></a>
### 5.Các thuật toán sử dụng trong ssh
- Những thuật toán khóa công khai
<ul>
	<li>RSA: kiểu tính toán không đối xứng, dùng cả cho việc mã hóa và chữ ký.</li>
	<li>Thuật toán chữ kí số (DSA): là 1 chữ ký không dùng cho mã hóa.</li>
	<li>Thuật toán thoả thuận khoá Diffie-Hellman: cho phép 2 bên lấy được 1 khóa chia sẻ an toàn trên 1 kênh mở.</li>
</ul>
- Những thuật toán khóa bí mật
<ul>
	<li>Chuẩn mã hoá tiên tiến (AES):1 thuật toán mã hoá khối bí mật với chiều dài khoá có thể là 128, 192, hoăc 256 bits.</li>	 
	<li>Chuẩn mã hoá dữ liệu (DES):là bản cũ của thuật toán mã hóa bảo mật và giờ được thay thế bằng AES.</li>
	<li>Trible-DES (3DES):1 dạng khác của DES dùng để tăng tính bảo mật bằng việc tăng chiều dài khoá.</li>
	<li>ARCFFOUR (RC4):mã hóa rất nhanh nhưng không bảo mật.Nó sử dụng kích thước khóa biến đổi.</li>
	<li>Blowfish: 1 thuật toán mã hóa miễn phí, nhanh hơn DES chậm hơn RC4.Chiều dài khóa thay đổi từ 32 bits-448 bits.</li>
</ul>
- Những hàm băm
<ul>
	<li>CRC-32: 1 hàm băm không mã hóa đối với việc dò tìm lỗi thay đổi dữ liệu.Nó được sử dụng trong việc kiểm tra toàn vẹn và chống tấn công "insertion attack - chèn mã độc".</li>
	<li>MD5: 1 thuật toán free, mã hóa mạnh, là thuật toán 128 bits.</li>
	<li>SHA-1: giống MD5, cung cấp hàm băm 160 bits, bảo mật hơn MD5 vì giá trị hàm băm dài hơn.</li>
	<li>Zlib: thuật toán nén dữ kiệu duy nhất được định nghĩa cho SSH.</li>
</ul>

<a name="6"><a>
### 6.Tham Khảo
- http://sinhvienit.net/forum/chuyen-de-tim-hieu-ssh.10058.html
- http://aperiodic.net/phil/ssh/
