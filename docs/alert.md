# Hướng dẫn cài đặt Alertmanager 

## 0. Sơ bộ về  Alertmanager

Alertmanager xử lý cảnh báo được gửi bởi ứng dụng như là Prometheus server. Nó có các cơ chế ***Grouping, Inhibition, Silience***

- **Grouping** :
    Phân loại cảnh báo có những tính chất tương tự. Điều này thực sự hữu ích trong một hệ thống lớn với nhiều thông báo được gửi đông thời.

    Ví dụ: một hệ thống với nhiều server mất kết nối đến cơ sở dữ liệu, thay vì rất nhiều cảnh báo được gửi về Alertmanager thì Grouping giúp cho việc giảm số lượng cảnh báo trùng lặp, thay vào đó là một cảnh báo để chúng ta có thể biết được chuyện gì đang xảy ra với hệ thống của bạn.

- **Inhibition** :
    Inhibition là một khái niệm về việc chặn thông báo cho một số cảnh báo nhất định nếu các cảnh báo khác đã được kích hoạt.

    Ví dụ: Một cảnh báo đang kích hoạt, thông báo là cluster không thể truy cập (not reachable). Alertmanager có thể được cấu hình là tắt các cảnh báo khác liên quan đến cluster này nếu cảnh báo đó đang kích hoạt. Điều này lọc bớt những cảnh báo không liên quan đến vấn đề hiện tại.

- **Silience** :
    Silience là tắt cảnh báo trong một thời gian nhất định. Nó được cấu hình dựa trên các match, nếu nó match với các điều kiện thì sẽ không có cảnh báo nào được gửi khi đó.

- **High avability** :
    Alertmanager hỗ trợ cấu hình để tạo một cluster với độ khả dụng cao.

## 1. Cài đặt Alertmanager cảnh báo qua mail và slack 

### Bước 1: Tạo ra user 

```
useradd --no-create-home --shell /bin/false alertmanager
```

### Bước 2: Tải source code 

```
wget https://github.com/prometheus/alertmanager/releases/download/v0.17.0/alertmanager-0.17.0.linux-amd64.tar.gz

tar xvf alertmanager-0.17.0.linux-amd64.tar.gz

mv alertmanager-0.17.0.linux-amd64/alertmanager /usr/local/bin/
mv alertmanager-0.17.0.linux-amd64/amtool /usr/local/bin/

chown alertmanager:alertmanager /usr/local/bin/alertmanager
chown alertmanager:alertmanager /usr/local/bin/amtool

rm -rf alertmanager-0.17.0.linux-amd64*
```

### Bước 3: Cấu hình để alertmanager gửi cảnh báo qua email và slack

Tạo ra đường dẫn và file khai báo thông tin 

```
mkdir /etc/alertmanager
chown alertmanager:alertmanager /etc/alertmanager
```

Chỉnh sửa file

```
vim /etc/alertmanager/alertmanager.yml
```

Như sau 

```
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'thuoclaoping@gmail.com'
  smtp_auth_username: 'username'
  smtp_auth_password: 'password'

  slack_api_url: 'web_hooks_api'

route:
  group_by: [alertname, datacenter, app]
  receiver: 'team-1'

receivers:
  - name: 'team-1'
    email_configs:
    - to: 'admin@gmail.com'
    slack_configs:
    - channel: '#prometheus'
      text: "<!channel> \nsummary: {{ .CommonAnnotations.summary }}\ndescription: {{ .CommonAnnotations.description }}"
```

Bạn cần thay đổi `smtp_auth_username`, `smtp_auth_password` và `slack_api_url` phù hợp với bạn 

Phân quyền cho file cấu hình 

```
chown alertmanager:alertmanager /etc/alertmanager/alertmanager.yml
```

### Bước 4: Tạo rule alert

Tạo file khai báo rule 

```
touch /etc/prometheus/alert.rules.yml
chown prometheus:prometheus /etc/prometheus/alert.rules.yml
```

Sau đó sửa file ``alert.rules.yml`

```
vim /etc/prometheus/alert.rules.yml
```

như sau 

```
groups:
- name: Instances
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 10s
    labels:
      severity: page
    # Prometheus templates apply here in the annotation and label fields of the alert.
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 10 s.'
      summary: 'Instance {{ $labels.instance }} down'
```

Cảnh báo được match khi máy client bị mất kết nối trong vòng 10s =) 

Phân quyền

```
chown prometheus:prometheus /etc/prometheus/alert.rules.yml
```

sử dụng `promtool` check syntax 

```
[root@prometheus ~]# promtool check rules /etc/prometheus/alert.rules.yml
Checking /etc/prometheus/alert.rules.yml
  SUCCESS: 1 rules found
```

### Bước 5: Khai báo service alertmanager với prometheus 

```
vim /etc/prometheus/prometheus.yml
```

Khai báo thêm các dòng sau vào file config có sẵn 

```
rule_files:
  - alert.rules.yml

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
```

Sửa lại file `/etc/systemd/system/prometheus.service`

```
vim /etc/systemd/system/prometheus.service
```

Sửa lại mỗi `ExecStart` cũ sang 

```
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
    --web.external-url=http://10.10.10.173
```

Ở đây `web.external-url` : URL mà Prometheus có thể truy cập được từ bên ngoài

### Bước 7: Chạy alertmanager dưới systemd 

```
vim /etc/systemd/system/alertmanager.service
```

như sau 

```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml --web.external-url http://10.10.10.173:9093

[Install]
WantedBy=multi-user.target
```

***Lưu ý thay đổi địa chỉ ip trong `--web.external-url` phù hợp với url của alertmanager của bạn***

Sau đó 

```
systemctl daemon-reload
systemctl restart prometheus
systemctl start alertmanager
systemctl enable alertmanager
```

### Kết quả 

- init 0 máy client 

- Một lúc sau kiểm tra mail và slack 

<img src="https://i.imgur.com/laY9gcz.png">

<img src="https://i.imgur.com/tDlphrb.png">



