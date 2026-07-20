# PET Shop Multi-store Microservices

Project bán sản phẩm thú cưng cho nhiều cửa hàng, triển khai theo mô hình Microservice bằng ASP.NET Core Web API + ASP.NET Core MVC.

## Phạm vi

Project bao gồm toàn bộ phạm vi mở rộng:

- Đăng ký, đăng nhập, refresh token, đăng xuất và phân quyền JWT.
- Admin quản lý tài khoản; tạo Staff; khóa/mở tài khoản.
- Customer gửi yêu cầu mở Shop; Staff duyệt hoặc từ chối.
- Admin/Staff quản lý Shop; Chủ Shop cập nhật thông tin Shop.
- Chủ Shop CRUD danh mục, sản phẩm và phân loại sản phẩm.
- Catalog công khai, tìm kiếm/lọc sản phẩm, xem chi tiết.
- Quản lý tồn kho, giữ hàng, trừ kho, hoàn kho và lịch sử giao dịch kho.
- Giỏ hàng và tách đơn theo từng Shop khi checkout.
- Luồng đơn: Pending → Confirmed → Preparing → Shipping → Completed; hỗ trợ Cancelled.
- Thanh toán COD và chuyển khoản mô phỏng; hoàn tiền.
- Thông báo trong hệ thống.
- Đánh giá sản phẩm dành cho khách đã mua và hoàn thành đơn.
- Báo cáo doanh thu theo Shop và toàn hệ thống.
- YARP API Gateway và giao diện MVC cơ bản.

Không có Docker, Kubernetes, CI/CD hay cấu hình cloud vì yêu cầu không bao gồm Deployment.

## Công nghệ

- .NET 8 / ASP.NET Core Web API
- ASP.NET Core MVC
- Entity Framework Core 8 + SQL Server
- JWT Bearer Authentication
- YARP Reverse Proxy
- Swagger/OpenAPI

## Cấu trúc

```text
src/
├── BuildingBlocks/
│   ├── PetShop.Contracts
│   └── PetShop.ServiceDefaults
├── Gateway/PetShop.ApiGateway
├── Services/
│   ├── Identity/PetShop.Identity.Api
│   ├── Shops/PetShop.Shops.Api
│   ├── Catalog/PetShop.Catalog.Api
│   ├── Inventory/PetShop.Inventory.Api
│   ├── Orders/PetShop.Orders.Api
│   ├── Payments/PetShop.Payments.Api
│   └── Notifications/PetShop.Notifications.Api
└── Web/PetShop.Web
```

Mỗi service sở hữu database riêng và không truy cập trực tiếp database của service khác.

## Database

| Service | Database |
|---|---|
| Identity | PetShopIdentityDb |
| Shops | PetShopShopsDb |
| Catalog | PetShopCatalogDb |
| Inventory | PetShopInventoryDb |
| Orders | PetShopOrdersDb |
| Payments | PetShopPaymentsDb |
| Notifications | PetShopNotificationsDb |

Các service sử dụng `Database.EnsureCreatedAsync()` để tự tạo database/bảng khi chạy lần đầu. Tài khoản SQL Server hiện tại phải có quyền tạo database. Khi làm production có thể đổi sang EF Core Migrations.

## Chuẩn bị môi trường local

1. Cài .NET 8 SDK.
2. Cài SQL Server hoặc SQL Server Express.
3. Mở solution `PetShopMicroservices.sln` bằng Visual Studio 2022.
4. Kiểm tra `ConnectionStrings` trong từng file `appsettings.json`.
5. Nếu SQL Server của bạn là SQL Express, có thể đổi `Server=localhost` thành `Server=.\\SQLEXPRESS`.
6. Giữ `Jwt:Key` và `InternalApiKey` giống nhau giữa tất cả service.

## Port

| Thành phần | URL |
|---|---|
| API Gateway | http://localhost:7000 |
| Identity API | http://localhost:7001/swagger |
| Shops API | http://localhost:7002/swagger |
| Catalog API | http://localhost:7003/swagger |
| Inventory API | http://localhost:7004/swagger |
| Orders API | http://localhost:7005/swagger |
| Payments API | http://localhost:7006/swagger |
| Notifications API | http://localhost:7007/swagger |
| MVC Web | http://localhost:7010 |

## Chạy project

### Cách 1: Visual Studio

Thiết lập Multiple Startup Projects và chọn `Start` cho:

1. PetShop.Notifications.Api
2. PetShop.Identity.Api
3. PetShop.Shops.Api
4. PetShop.Catalog.Api
5. PetShop.Inventory.Api
6. PetShop.Orders.Api
7. PetShop.Payments.Api
8. PetShop.ApiGateway
9. PetShop.Web

### Cách 2: PowerShell

Chạy:

```powershell
./scripts/run-all.ps1
```

## Tài khoản seed

| Role | Email | Password |
|---|---|---|
| Admin | admin@petshop.local | Admin@123 |
| Staff | staff@petshop.local | Staff@123 |
| Customer | customer@petshop.local | Customer@123 |

Luồng tạo Chủ Shop:

1. Đăng nhập Customer.
2. Gửi yêu cầu mở Shop.
3. Đăng nhập Staff và duyệt yêu cầu.
4. Identity Service tự gán thêm role `ShopOwner` cho Customer.
5. Đăng nhập lại để JWT mới chứa role `ShopOwner`.

## Thứ tự test nghiệp vụ

1. Customer đăng nhập và gửi yêu cầu mở Shop.
2. Staff duyệt Shop.
3. Chủ Shop tạo danh mục và sản phẩm.
4. Chủ Shop nhập số lượng tồn kho theo `ProductId`.
5. Customer khác thêm sản phẩm vào giỏ và checkout.
6. Chủ Shop xác nhận → chuẩn bị → giao → hoàn thành.
7. Customer tạo thanh toán hoặc dùng COD.
8. Sau khi đơn Completed, Customer có thể đánh giá sản phẩm.
9. Chủ Shop/Admin xem báo cáo doanh thu.

## Lưu ý kỹ thuật

- Checkout tự chia giỏ hàng thành nhiều đơn nếu có sản phẩm từ nhiều Shop.
- Inventory sử dụng `ReservedQuantity` để tránh bán vượt tồn kho.
- Khi Customer hủy đơn Pending, số lượng giữ được giải phóng.
- Khi Shop hủy đơn sau khi đã xác nhận, số lượng được hoàn lại kho.
- `InternalApiKey` chỉ dùng cho giao tiếp service-to-service và không được đưa ra frontend.
- Ảnh sản phẩm hiện lưu dưới dạng URL. Có thể thay bằng Cloudinary/MinIO sau này mà không ảnh hưởng nghiệp vụ chính.

## Phạm vi giao diện MVC

MVC có các màn hình chính cho đăng ký/đăng nhập, catalog, giỏ hàng, checkout, đơn hàng, yêu cầu mở Shop, duyệt Shop, quản lý tài khoản, danh mục, sản phẩm, tồn kho, đơn Shop và thông báo. Toàn bộ API nâng cao vẫn có thể kiểm thử trực tiếp bằng Swagger.
