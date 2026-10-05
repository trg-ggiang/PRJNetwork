# Kiến trúc dự kiến — chưa triển khai

## Luồng ứng dụng

```text
Client CLI C -- TCP :9000 ---------+
                                 +--> Server C --> SQLite trên máy server
GUI HTML/CSS/JS -- WebSocket :8080-+
```

Trình duyệt không mở raw TCP socket. Server C cần adapter HTTP/WebSocket
(libwebsockets) để phục vụ `web/` và nhận thông điệp GUI. Cả TCP và WebSocket
gọi cùng dispatcher/service C; không làm hai bộ nghiệp vụ riêng.
JS quản lý form, chọn cuộc trò chuyện, render và vận chuyển request/event.
Server C quyết định danh tính, thành viên nhóm, quyền nhận/gửi và lưu tin.

## Module và trách nhiệm

| Module dự kiến | Trách nhiệm |
|---|---|
| common/net | Winsock trên Windows, POSIX trên Linux; close/init/error thống nhất |
| common/protocol | framing TCP, encode/decode JSON, validate field/giới hạn |
| server/transport | accept, nonblocking I/O, hàng đợi output, disconnect/timeout |
| server/dispatcher | request type → service, response/error/event |
| server/auth | register/login/logout, libsodium password hash, session |
| server/friends | lời mời kết bạn, accept/decline, hủy kết bạn, danh sách bạn |
| server/presence | danh sách online, heartbeat và phát sự kiện trạng thái |
| server/chat | chat 1-1, dừng/chuyển hội thoại, quyền gửi, lưu trước khi gửi |
| server/groups | tạo nhóm, mời, accept/decline, xóa thành viên, rời nhóm, quyền thành viên |
| server/storage | SQLite, prepared statements, transaction, migration |
| server/logging | ghi sự kiện hoạt động có timestamp/user/action/result, che dữ liệu nhạy cảm |
| server/web | HTTP static files, WebSocket adapter, kiểm tra origin |
| client | CLI đọc lệnh, gửi request, nhận event bất đồng bộ |
| web | form đăng nhập, sidebar online/nhóm, lịch sử và composer |

## Quyết định thiết kế

- Chọn client-server để xác thực, trạng thái và tin offline tập trung.
- Phase 1 dùng nonblocking `select` với quy mô demo nhỏ; đặt giới hạn kết nối
  và xử lý giới hạn `FD_SETSIZE`. Mỗi connection có input buffer/output queue.
- Khi thêm WebSocket, tích hợp vòng lặp transport trước; không truy cập trạng thái
  session/database từ hai thread thiếu đồng bộ. Có thể dùng một service queue C.
- SQLite chỉ do server truy cập; tên bảng dự kiến: users, conversations,
  friend_requests, friendships, conversation_members, group_invitations, messages, message_receipts.
  Schema/foreign keys/indexes/transactions sẽ viết ở phase 2–5.
- Dùng libsodium `crypto_pwhash_str`/verify; khởi tạo thư viện và kiểm tra mọi lỗi.
- Một user một session online trong MVP: từ chối lần login thứ hai với lỗi rõ ràng.
- Danh tính người gửi lấy từ session kết nối, không tin `sender_id` do client gửi.
- Kết bạn là quan hệ hai chiều sau khi chấp nhận; pending request không phải friendship.
  Cấm tự kết bạn, lời mời trùng và accept lời mời không gửi cho mình. Lời mời gửi
  ngược khi đang pending trả lỗi rõ ràng, không tự chấp nhận.
- Danh sách bạn bè gồm online/offline; danh sách user online sau login gồm user khác
  đang online, không giới hạn bạn bè. Theo yêu cầu ban đầu, MVP cho chat 1-1 với
  user khác dù chưa kết bạn; hủy kết bạn không xóa lịch sử hoặc tự xóa thành viên nhóm.
- Người tạo nhóm là owner; thành viên có thể mời user khác, owner có quyền xóa
  thành viên. Tham gia qua lời mời được chấp nhận. Owner rời nhóm phải chuyển
  quyền cho thành viên còn lại trong cùng transaction; nếu không còn ai thì đóng nhóm.
- Xóa thành viên phát GROUP_MEMBER_REMOVED cho người bị xóa và các thành viên
  còn lại, thu hồi quyền gửi/nhận/đọc lịch sử nhóm ngay; cấm tự xóa owner qua REMOVE.
- Chat đang mở trên UI khác với lịch sử: dừng hội thoại gửi event cho bên kia nhưng
  không xóa tin; vẫn có thể gửi tin offline qua direct conversation.
- Online/offline là trạng thái kết nối. Delivery/read là trạng thái từng tin nhắn.
- Lưu message thành công rồi ACK người gửi; recipient ACK nhận tin để cập nhật
  delivery. Có thể giao lại sau reconnect; khử trùng bằng message ID/request ID.
- Rời nhóm mất quyền gửi/nhận mới. Chính sách lịch sử MVP: chỉ thành viên hiện tại
  được đọc lịch sử nhóm; nhóm không còn thành viên được đánh dấu đóng.
- TCP/WebSocket ban đầu là demo localhost/LAN; khi triển khai ngoài mạng tin cậy,
  thêm TLS/WSS trước khi truyền thông tin đăng nhập.
- Ghi log từ phase socket: connect/disconnect, lỗi I/O; mở rộng auth, kết bạn,
  nhóm, message ID/trạng thái. Không log mật khẩu/token/nội dung tin nhắn.
  Giới hạn dung lượng/rotation và xử lý lỗi ghi log không làm server crash.

## Folder hỗ trợ

| Folder | Dùng để làm gì | Task liên quan |
|---|---|---|
| include/chat | Header `.h`: khai báo API, kiểu dữ liệu, hằng số dùng chung; triển khai nằm ở src | Hỗ trợ T01–T17, không có điểm riêng |
| tests | Kiểm thử framing và service, nhiều client, mất mạng/restart, quyền truy cập và GUI | Xác minh T01–T19 |
| web | HTML/CSS/JS của giao diện; gọi server C qua WebSocket | T19 và hiển thị T03–T16/T18 |
| data | SQLite database thực tế do server tạo khi chạy; không phải source SQL | T03–T09, T11–T16 |
| logs | Log thực tế do module logging C tạo khi chạy | T17 |
| docs | Setup, kiến trúc, giao thức, bảng yêu cầu; tài liệu demo sẽ bổ sung | Hướng dẫn/handoff mọi task |
| build | CMake/Ninja sinh object/executable/cache trên máy local, không commit | Hỗ trợ build; chưa có sản phẩm |

`src/common/net` là wrapper socket thấp tầng dùng chung; `src/server/transport`
là cơ chế quản lý nhiều kết nối và I/O phía server. `src/common/protocol` ghép/tách
thông điệp; `src/server/dispatcher` đưa thông điệp đã parse tới đúng service.
`src/server/web` là code C phục vụ HTTP/WebSocket, còn `web/` là file giao diện.
`src/server/storage` là code truy cập dữ liệu, còn `data/` là dữ liệu runtime.
`src/server/logging` là code ghi log, còn `logs/` là file log runtime.

Module có thể có header nội bộ đặt cùng source; chỉ API cần chia sẻ mới đặt ở
`include/chat`. Không tạo một header chung khổng lồ cho toàn bộ project.

## Phạm vi bổ sung sau yêu cầu bắt buộc

Ưu tiên lịch sử/pagination, trạng thái delivered/read, typing, tìm tin và đổi mật khẩu.
Gửi file/avatar/emoji nâng cao chỉ làm sau phase 7; không nhận upload vô hạn.
GUI là yêu cầu bắt buộc, không được thay bằng client CLI ở bản nộp.
