# 1.  Giới thiệu: 

- Telegraf – Influxdb – Grafana (TIG)  là một stack dùng để giám sát hệ thống, công cụ rất phổ biến.
- **Telegraf** là một agent để thu thập và báo cáo các số liệu và dữ liệu. Nó có thể tích hợp để thu thập nhiều loại nguồn dữ liệu khác nhau của các số liệu, events, và logs trực tiếp từ các container và hệ thống mà nó chạy trên đó.
- **InfluxDB** là một *Time Series Database* (là database được tối ưu hóa để xử lý dữ liệu chuỗi thời gian, các dãy số được lập chỉ mục theo thời gian). Nó cung cấp một ngôn ngữ giống SQL để tương tác với dữ liệu.
- **Grafana** là một công cũ mã nguồn mở  cho phép chúng ta truy vấn, hiển thị, cảnh báo các số liệu thu thập được. Nó có các tính năng nổi bật như hiển thị số liệu theo nhiều dạng biểu đồ khác nhau; cảnh báo qua nhiều kênh như email, telegram, slack, …; Hỗ trợ nhiều loại database như InfluxDB, ElasticSearch, Graphite, …; Cung cấp nhiều plugin, nhiều dashboard template.

- TIG hoạt động như sau : 
    - **Telegraf** đóng vai trò agent thu thập số liệu hệ thống, cũng như dịch vụ trên hệ thống đang chạy
    - Sau khi thu thập số liệu, chúng được đưa vào **InfluxDB** - đóng vai trò làm nơi lưu trữ.
    - Từ **InfluxDB**, dữ liệu được đưa lên **Grafana**, nơi chúng được hiển thị theo dạng các đồ thị trực quan. 
    <img src= "https://github.com/tulha161/TIG/blob/main/picture/1.PNG">

# 2. Cài đặt: 
## Cấu hình cài đặt :
- OS : Ubuntu 18.04
- IP : 10.5.10.98

## 2.1. Cài đặt InfluxDB : 
- thêm Influxdata key : 
`wget -qO- https://repos.influxdata.com/influxdb.key | apt-key add -`
- Thêm repo influxdata và update lại :
```
source /etc/lsb-release

echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | tee /etc/apt/sources.list.d/influxdb.list

apt update -y
```
- Cài đặt gói InfluxDB : 
`apt install influxdb curl -y`
- Khởi động gói : 
`systemctl enable --now influxdb`
- Kiểm tra dịch vụ và version 
```
systemctl status influxdb
influx -version
```
- Kiểm tra và mở port cho InfluxDB : 
```
ufw allow 8086

ufw allow 8088
```

### Cấu hình InfluxDB:
- Như vậy là đã xong phần cài đặt, giờ chuyển sang cấu hình
- File cấu hình đặt tại `/etc/influxdb/influxdb.conf`, chỉnh sửa file này lưu ý các trường trong tag `[http]` : 
```
[http]
# Determines whether HTTP endpoint is enabled.
enabled = true

# Determines whether the Flux query endpoint is enabled.
flux-enabled = true

# The bind address used by the HTTP service.
bind-address = ":8086"

# Determines whether HTTP request logging is enabled.
log-enabled = true
```
- Vào shell InfluxDB bằng cách nhập lệnh `influx`, tại đây tạo db và user cho **telegraf**: 
```
   > create database telegraf
   > create user telegraf with password 'tele.123' with all privileges
```
- Kiểm tra bằng lệnh
``` 
 > show databases
 > show users
```

## 2.2. Cài đặt Telegraf Agent: 
- Download gói cài và cài đặt :
```
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.14.2-1_amd64.deb
sudo dpkg -i telegraf_1.14.2-1_amd64.deb
```
- Khởi động dịch vụ và check :
```
systemctl enable --now telegraf
systemctl status telegraf
```
### Cấu hình telegraf agent : 
- **LƯU Ý** : đây là thao tác tại mỗi host muôn thêm vào để telegraf thu thập thông tin đẩy sang InfluxDB, vì vậy nên khi thêm các host khác, cần fill 4 trường thông tin: *urls, db name, us name, passw* và cấu hình chi tiết các thông tin cần check là được ! 
- file cấu hình đặt tại `/etc/telegraf/telegraf.conf`, chỉnh sửa như sau : 
```
hostname = "tule-TIG"

urls = ["http://10.5.10.98:8086"]

database = "telegraf"

username = "telegraf"

password = "tele.123"

[[inputs.cpu]]

percpu = true

totalcpu = true

collect_cpu_time = false

report_active = false

[[inputs.disk]]

ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

[[inputs.diskio]]

[[inputs.kernel]]

[[inputs.mem]]

[[inputs.processes]]

[[inputs.swap]]

[[inputs.system]]

[[inputs.net]]

[[inputs.netstat]]
```
- Phần này liên quan đến khai báo database cũng như config lại các thông số check của telegraf. Sau khi khai báo xong, restart dịch vụ : ` systemctl restart telegraf`


## 2.3. Cài đặt Grafana : 
- update và cài đặt các gói cần thiết :
```
apt-get install -y apt-transport-https

apt-get install -y software-properties-common wget

apt-get update

apt-get upgrade

reboot
```
- Add key vào các gói cài :
``` 
wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -

add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

- Update repo và cài đặt : 
```
apt-get update

apt-get install -y grafana
```
- Khởi động dịch vụ và kiểm tra : 
```
systemctl enable --now grafana-server
systemctl status grafana-server
```
- Mở port 3000 để chạy dịch vụ : 
```
ufw allow 3000
```
- Cài đặt đã thành công, truy cập vào site bằng `<YourServerIP>:3000` để vào web interface với mật khẩu mặc định admin/admin

<img src= "https://github.com/tulha161/TIG/blob/main/picture/2.PNG">

## 2.4. Cấu hình Grafana : 
### Thêm InfluxDB làm datasource : 

- Trên giao diện Grafana, chọn Configuration -> Data Sources -> Add data source

<img src= "https://github.com/tulha161/TIG/blob/main/picture/3.png">

- Tìm và add **InfluxDB**, fill các trường sau : 
  ```
    - 1. Tên DB
    - 2. URL : ở đây mình cài trên cùng 1 server nên là http://localhost:8086 
    - 3. Cài đặt các phương thức xác thực
    - 4. User xác thực, nhập tài khoản `admin` cùa InfluxDB và mật khẩu
    - 5. DB sẽ lấy, ta dùng telegraf nên điền tên db, user, password đã tạo ở phần trên
  ``` 
- Sau khi fill đủ các trường trên, chọn `Save & Test`


### Thêm Dashboard : 
- Tìm các loại dashboard muốn dùng tại trang : https://grafana.com/grafana/dashboards?orderBy=name&direction=asc
- Ở đây mình dùng : https://grafana.com/grafana/dashboards/61
- Copy ID của Dashboard này để phục vụ việc Import, ID của nó là `61` 

<img src= "https://github.com/tulha161/TIG/blob/main/picture/4.png">

- Chọn import, fill id `61` vào mục `grafana.com dashboard URL or ID` , có thể fill id hoặc url trực tiếp cũng ok. Sau đó chọn `Load`.

<img src= "https://github.com/tulha161/TIG/blob/main/picture/5.png">

- Fill Name, chọn Folder và Source DB ( InfluxDB đã thêm ở step trên) rồi Import.
- Hình ảnh Dashboard đã được thêm thành công : 

<img src= "https://github.com/tulha161/TIG/blob/main/picture/6.png">

- Kiểm tra các collector đã được config tại phần `2.2` ở mục `System`, ngoài ra Dashboard này còn phục vụ hiển thị trạng thái của khá nhiều Service khác như : MYSQL, Mongo, Redis, ... 
<img src= "https://github.com/tulha161/TIG/blob/main/picture/7.png">

- Để thêm Server vào hệ thống cảnh báo, ta cần cài đặt telegraf agent nơi máy trạm, cấu hình hostname, các thông số muốn monitor và trỏ nó tới InfluxDB. Ví dụ ở đây mình add thêm host **db** và monitor vài input về mysql của nó : 

<8.png>
