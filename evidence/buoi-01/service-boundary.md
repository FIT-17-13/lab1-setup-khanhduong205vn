# Service Boundary của nhóm

## 1. Thông tin nhóm

- Tên nhóm: Nhóm 9
- Lớp: CNTT17-13
- Thành viên: Hà Huy Khánh Dương, Nguyễn Hà Phương, Dương Thị Hoài, Đặng Thị Huyền
- Service nhóm phụ trách: B1 + B7 
- Sản phẩm tổng thể của lớp: Product B

## 2. Actor
- **Thiết bị/Cảm biến IoT:** Gửi dữ liệu thông số môi trường, hình ảnh camera về hệ thống.
- **Người dùng cuối(sinh viên, bảo vệ):** Nhận thông báo, cảnh báo qua các kênh liên lạc (Email, SMS, App).
- **Hệ thống quản trị:** Theo dõi trạng thái hoạt động của các service.

## 3. System Boundary
Nhóm xây dựng hai thành phần quan trọng ở hai đầu của luồng dữ liệu: tiếp nhận dữ liệu từ phần cứng và truyền tải thông tin đến người dùng.

**Phần nhóm kiểm soát:**
- Tiếp nhận, chuẩn hóa dữ liệu từ các thiết bị biên (IoT Ingestion).
- Quản lý logic gửi thông báo, chọn kênh gửi và cơ chế gửi lại (Notification).
- Quản lý các Endpoint truy vấn dữ liệu gần nhất và nhật ký thông báo.

**Phần nhóm chỉ tích hợp:**
- **Phần cứng:** Các sensor, camera thu thập dữ liệu tại campus.
- **Core Business:** Hệ thống trung tâm xử lý logic nghiệp vụ và ra quyết định.
- **Dịch vụ bên thứ 3:** Các Gateway gửi tin nhắn (FCM, Twilio, SMTP Server).

## 4. Service Boundary
**Service của nhóm có trách nhiệm gì?**
- Đảm bảo dữ liệu từ thiết bị được đưa vào hệ thống một cách chuẩn xác về định dạng và thời gian.
- Cung cấp các API để hệ thống khác lấy dữ liệu cảm biến mới nhất.
- Đảm bảo mọi cảnh báo từ hệ thống lõi được chuyển đến đúng người dùng qua đúng kênh yêu cầu.
- Ghi log và thực hiện Retry nếu việc gửi thông báo gặp sự cố.

**Service KHÔNG làm gì?**
- Không phân tích dữ liệu chuyên sâu (đây là nhiệm vụ của Analytics Service).
- Không tự ý sinh ra cảnh báo (Notification chỉ gửi khi có yêu cầu từ Core Business).

## 5. Input / Output
**Input:**
- Dữ liệu thô từ cảm biến (IoT Ingestion).
- Yêu cầu gửi alert từ Core Business (Notification).

**Output:**
- Dữ liệu đã chuẩn hóa (JSON) cung cấp cho Core Business và Analytics.
- Thông báo hiển thị trên thiết bị người dùng (Email, SMS, Chat).

## 6. API dự kiến

| Method | Endpoint | Mục đích |
|---|---|---|
| **GET** | `/health` | Kiểm tra trạng thái hoạt động của các service |
| **POST** | `/readings` | Tiếp nhận dữ liệu từ cảm biến IoT |
| **GET** | `/readings/latest` | Lấy dữ liệu cảm biến mới nhất |
| **POST** | `/notifications` | Tiếp nhận yêu cầu và thực hiện gửi thông báo |
| **GET** | `/notifications/{id}` | Kiểm tra trạng thái của một thông báo cụ thể |
| **POST** | `/notifications/{id}/retry` | Gửi lại thông báo bị lỗi |

## 7. Phụ thuộc service khác
- **Service này gọi đến:** Core Business (để đẩy dữ liệu), Analytics (để đẩy dữ liệu), và các Provider bên thứ 3 (để gửi tin).
- **Service nào gọi đến service này:** Các thiết bị IoT biên gửi dữ liệu vào, Core Business gọi để yêu cầu gửi thông báo.

## 8. Sơ đồ minh họa

```mermaid
flowchart TD
    subgraph "Input Layer"
        IoT[Thiết bị / Cảm biến IoT]
    end

    subgraph "Nhóm 9 Services"
        Ingestion[IoT Ingestion Service]
        Noti[Notification Service]
        DB[(Database / Log)]
    end

    subgraph "Internal System"
        Core[Core Business]
        Analytic[Analytics Service]
    end

    subgraph "Output Layer"
        User((Người dùng))
        ThirdParty[Email/SMS/App Services]
    end

    IoT -- POST /readings --> Ingestion
    Ingestion -- Chuẩn hóa --> Core
    Ingestion -- Chuẩn hóa --> Analytic
    Ingestion -.-> DB

    Core -- Yêu cầu Alert --> Noti
    Noti -- Gửi tin --> ThirdParty
    ThirdParty --> User
    Noti -.-> DB
