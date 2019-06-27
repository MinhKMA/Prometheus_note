# Querying in Prometheus 

Prometheus cung cấp ngôn ngữ truy vấn được gọi là PromQL( viết tắt bởi Promtheus Query Language) cho phép người dụng select và tổng hợp time series data theo thời gian thực. Kết quả của một biểu thức có thể được hiển thị dưới dạng biểu đồ  được xem dưới dạng dữ liệu bảng trên dashboard của Prom hoặc từ các ứng dụng thứ ba thông qua HTTP API. 

Ví dụ: 

- Truy vấn : `{job="node_exporter2"}`
- Kết quả thu được: 

    <img src="https://i.imgur.com/JgZGoOh.png">

## 1. Basics 

### Expression language data types

- **Instant vector**: Truy vấn một chuỗi các time series có dùng một timestamp 

    + VD: 
        + Nếu query `http_request_count` thì kết quả có dạng như sau:
            + http_request_count{status=“200”} 20 
            + http_request_count{status=“404”} 3
            + http_request_count{status=“500”} 5
- **Range Vector**: Truy vấn một chuỗi các time series trong khoảng thời gian

    + VD: 
        + Nếu query `http_request_count[5m]` thì kết quả:
            + http_request_count{status=“200”} 

- **Scalar** : as a literal and as result of an expression 
- **String** : only currently as a literal in an expression

### Literals

- String literals

    + Có thể được chỉ định là chuỗi kí tự trong dấu ngoặc képm ngoặc đơn hoặc or backticks.
    + Đối với những kí tự đặc biệt cần thêm dấu \. Vd như \n, \t. Đọc thêm ở <a href='https://golang.org/ref/spec#String_literals'>đây</a>. 

- Float literals

    + Các giá trị float được viết dưới dạng chữ số  `[-](digits)[.(digits)].`
    + VD: `-2.43`

### Time series Selectors

- Instant vector selectors

    ```
    http_requests_total
    ```
    
    + Select tất cả time series của metric name  `http_requests_total`

    hoặc 

    ```
    http_requests_total{job="prometheus",group="canary"}
    ```

    + Select time series metric name là `http_requests_total` có job lable là `prometheus` và group lable là `canary`

    Ngòai ra ta còn có thể sử dụng các biểu thức chính quy: 

    + `=`: Chọn nhãn chính xác so với string khai báo.
    + `!=`: Ngược lại với `=`
    + `=~`: Select lable regex-match với chuỗi khai báo.
    + `!~`: Ngược lại với `=~`

- Range Vector Selectors

    + Thời lượng được chỉ định là một số được biểu diễn trong `[]`, theo sau là một trong các đơn vị sau:

        + s - seconds
        + m - minutes
        + h - hours
        + d - days
        + w - weeks
        + y - years

    + Trong ví dụ dưới đây, select tất cả các giá giá trị trong 5 phút gần đây của tất cả time series metric name `http_requests_total` và job lable là `prometheus`

        ````
        http_requests_total{job="prometheus"}[5m]
        ````
- Offset modifier

    + Ví dụ: biểu thức sau đây trả về giá trị của   `http_quests_total` 5 phút trong quá khứ so với thời gian truy vấn hiện tại:

        ```
        http_requests_total offset 5m
        ```

## 2. Operators 

- Phần này bạn nên đọc tài liệu trên trang chủ Prometheus để giữ nguyên được từ khóa toán học 

- https://prometheus.io/docs/prometheus/latest/querying/operators/

## 3. Functions

### abs()

`abs(v instant-vector)` trả về giá trị tuyệt đối của input

### absent(): 

### ceil()

`ceil(v instant-vector)` làm tròn các giá trị của các phần tử v lên số nguyên gần nhất

### changes()

to be continue 