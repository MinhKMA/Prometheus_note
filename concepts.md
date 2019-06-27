# Các khái niệm được đề cập trong Promtheus 

## 1. Data model 

Prometheus về cơ bản lưu trữ tất cả dữ liệu dưới dạng time series

### Metric names and labels

Mỗi time series được xác định duy nhất bởi tên metric name và một bộ các cặp key-value, còn được gọi là labels(nhãn).

- Metric name chỉ định các thông số của của hệ thống (Vd: `http_requests_total` - tổng số request HTTP đã nhận). Nó có thể chứa các chữ cái và chữ số ASCII cũng như dấu gạch dưới và dấu hai chấm. Nó phải được match với `[a-zA-Z_:][a-zA-Z0-9_:]*`.

    ***Note: Dấu hai chấm được dành riêng cho quy tắc ghi do người dùng xác định. Chúng không được sử dụng bởi exporters hoặc direct instrumentation.***

- Labels: bất kỳ sự kết hợp nhãn đã cho nào cho cùng một metric name xác định particular dimensional instantiation of that metric( Vd: tất cả HTTP requests đã sử dụng phương thức POST tới /api/tracks xử lý) Ngôn ngữ truy vấn cho phép lọc và tổng hợp dựa trên các labels này. Khi thay đổi giá tri label, bao gồm cả thêm hoặc xóa label sẽ tạo ta một time series mới. 

### Ví dụ minh họa

`api_http_requests_total{method="POST", handler="/messages"}`

- Một time series có metric name là `api_http_requests_total` và lable `method="POST"`, `handler="/messages`

## 2. Metric types 

Có 4 kiểu metric được sử dụng trong prometheus: 

- **counter**: là một bộ đếm tích lũy, được đặt về 0 khi restart. Ví dụ, có thể dùng counter để đếm số request được phục vụ, số lỗi, số task hoàn thành,... Không sử dụng cho gía trị có thể giảm như số tiến trình đang chạy. Trong trường hợp này, ta có thể sử dụng gauge

- **gauge**: đại diện cho số liệu duy nhất, nó có thể lên hoặc xuống. Nó thường được sử dụng cho các giá trị đo

- **histogram**: lấy mẫu quan sát (thường là những thứ như là thời lượng yêu cầu, kích thước phản hồi). Nó cũng cung cấp tổng của các giá trị đó.

- **summary**: tương tự histogram, nó cung cấp tổng số các quan sát và tổng các giá trị đó, nó tính toán số lượng có thể cấu hình qua sliding time window.

## 3. Jobs và Instances

Theo như thuật ngữ trong prometheus quy định một endpoint mà bạn có thế thể scrape được gọi là một instance. 

Tập hợp của các instances có cùng một mục đích được gọi là một job 

Ví dụ: 

- job: `api-server`
    + instance 1: `1.2.3.4:5670`
    + instance 2: `1.2.3.4:5671`
    + instance 3: `5.6.7.8:5670`
    + instance 4: `5.6.7.8:5671`

