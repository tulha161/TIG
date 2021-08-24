# 1.  Giới thiệu: 

- Telegraf – Influxdb – Grafana (TIG)  là một stack dùng để giám sát hệ thống, công cụ rất phổ biến.
- **Telegraf** là một agent để thu thập và báo cáo các số liệu và dữ liệu. Nó có thể tích hợp để thu thập nhiều loại nguồn dữ liệu khác nhau của các số liệu, events, và logs trực tiếp từ các container và hệ thống mà nó chạy trên đó.
- **InfluxDB** là một *Time Series Database* (là database được tối ưu hóa để xử lý dữ liệu chuỗi thời gian, các dãy số được lập chỉ mục theo thời gian). Nó cung cấp một ngôn ngữ giống SQL để tương tác với dữ liệu.
- **Grafana** là một công cũ mã nguồn mở  cho phép chúng ta truy vấn, hiển thị, cảnh báo các số liệu thu thập được. Nó có các tính năng nổi bật như hiển thị số liệu theo nhiều dạng biểu đồ khác nhau; cảnh báo qua nhiều kênh như email, telegram, slack, …; Hỗ trợ nhiều loại database như InfluxDB, ElasticSearch, Graphite, …; Cung cấp nhiều plugin, nhiều dashboard template.

- TIG hoạt động như sau : 
    - **Telegraf** đóng vai trò agent thu thập số liệu hệ thống, cũng như dịch vụ trên hệ thống đang chạy
    - Sau khi thu thập số liệu, chúng được đưa vào **InfluxDB** - đóng vai trò làm nơi lưu trữ.
    - Từ **InfluxDB**, dữ liệu được đưa lên **Grafana**, nơi chúng được hiển thị theo dạng các đồ thị trực quan. 
    <img src= "https://imgur.com/a/lOIuMNC">