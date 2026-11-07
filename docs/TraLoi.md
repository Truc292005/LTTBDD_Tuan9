# Trả lời câu hỏi bổ sung — Buổi 9

**Nguyễn Mai Trúc – 23633511**

## 1. Giá trị dẫn xuất
`errors` và `valid` suy ra hoàn toàn từ `name`, `phone`, `address` và giỏ hàng, nên tính lại ở mỗi lần render thì luôn khớp với dữ liệu hiện tại và không cần code đồng bộ thêm. Nếu lưu bằng `useState` riêng thì phải nhớ cập nhật ở mọi chỗ gọi `setName`, `setPhone`, `setAddress`, `setItems`...; quên một chỗ là lỗi hiện sai hoặc nút "Xác nhận đặt hàng" mở/khoá sai (state bị lệch với dữ liệu thật).

## 2. Thứ tự gọi
Nếu `clear()` chạy trước `await createOrder(...)` thì khi mất mạng, đơn chưa được tạo trên server mà giỏ đã bị xoá sạch (cả trong store lẫn AsyncStorage), người dùng mất toàn bộ hàng đã chọn và không thể bấm thử lại. Đặt `clear()` ngay sau `createOrder` là đúng vì khi đó server đã nhận đơn thành công; nếu `createOrder` ném lỗi thì code nhảy thẳng vào `catch`, bỏ qua `clear()` và giỏ còn nguyên.

## 3. PUT hay PATCH
`PUT /users/1` với `{ name, phone }` sẽ thay thế cả object, nên user đó trong `db.json` chỉ còn `id`, `name`, `phone`; mất `email` và `password` (không đăng nhập lại được nữa). `PATCH` chỉ cập nhật các trường được gửi và giữ nguyên phần còn lại, nên phù hợp với form chỉ sửa một phần như hồ sơ này.

## 4. Cập nhật lạc quan
Nếu bỏ đoạn hoàn lại `read: false` trong `catch` thì khi mất mạng, giao diện báo "đã đọc" nhưng server vẫn là chưa đọc; người dùng tưởng đã lưu, đến lần mở lại màn thì thông báo bật về chưa đọc, rất khó hiểu. Nên dùng cho thao tác nhẹ, ít khi sai và dễ hoàn lại như thả tim/bỏ yêu thích sách hoặc đánh dấu đã đọc. Không nên dùng cho thanh toán/đặt hàng, vì phải chắc server đã nhận đơn mới được báo thành công và xoá giỏ.

## 5. Code bổ sung: bộ lọc trạng thái ở màn Đơn hàng
Code nằm trong `src/screens/OrdersScreen.tsx`:

```tsx
// state bộ lọc
const [status, setStatus] = useState<"all" | OrderStatus>("all");

// một chỗ sửa trong dữ liệu truyền cho FlatList
const visibleOrders = orders.filter((o) => status === "all" || o.status === status);
<FlatList data={visibleOrders} ... />
```

Hàng nút lọc (Tất cả / Chờ xác nhận / Đang giao / Hoàn tất / Đã huỷ) đặt phía trên FlatList, bấm nút nào thì `setStatus(...)` nút đó. Ảnh chụp màn hình: xem `docs/screenshots/02c_orders_filter_completed.png`.
