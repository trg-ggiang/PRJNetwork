# Giao thức v1 — đề xuất, cần chốt ở phase 1

## Transport

TCP: `[4 byte độ dài big-endian][JSON UTF-8]`. Độ dài chỉ tính payload;
từ 1 đến 65536 byte. Parse nhiều frame trong một recv, giữ phần còn thiếu,
xử lý partial send bằng output queue. Frame quá giới hạn hoặc framing không hợp lệ
đóng kết nối; JSON lỗi trong frame hợp lệ trả error khi có thể.

WebSocket: một **text message** chứa một JSON cùng cấu trúc; không thêm prefix TCP.
Adapter phải ráp fragment WebSocket và giới hạn tổng độ dài trước khi dispatch.
Chọn parser JSON C ở P1-01, ghi dependency/version/license và cách build.

## Envelope

Request:

```json
{"version":1,"type":"LOGIN","request_id":"req-001","data":{"username":"alice","password":"example-only"}}
```

Response:

```json
{"version":1,"type":"RESPONSE","request_id":"req-001","ok":true,"data":{"user_id":1}}
```

Error:

```json
{"version":1,"type":"RESPONSE","request_id":"req-001","ok":false,"error":{"code":"INVALID_CREDENTIALS","message":"Đăng nhập không thành công"}}
```

Event không gắn request:

```json
{"version":1,"type":"MESSAGE_RECEIVED","data":{"message_id":42,"conversation_id":7,"sender_id":1,"body":"Xin chào","created_at":"2026-10-05T10:00:00Z"}}
```

Timestamp server tạo theo UTC; GUI hiển thị theo timezone người dùng.
Giới hạn dự kiến: username 3–32 ký tự ASCII chữ/số/underscore; password 8–128 byte;
body 1–4096 byte UTF-8; tên nhóm 1–128 byte; request_id tối đa 64 byte.
Password không trim hoặc truncate. JS chỉ kiểm tra form hỗ trợ UX; C kiểm tra lại.

## Request/event tối thiểu

| Request | Nội dung `data` dự kiến | Event liên quan |
|---|---|---|
| REGISTER / LOGIN | username, password | ONLINE_USERS sau login |
| LOGOUT | rỗng | USER_OFFLINE, CHAT_ENDED |
| GET_PROFILE / UPDATE_PROFILE | user_id để xem; display_name để sửa hồ sơ của mình | response hồ sơ |
| CHANGE_PASSWORD | old_password, new_password | response; xác thực lại mật khẩu cũ |
| LIST_ONLINE | rỗng | USER_ONLINE / USER_OFFLINE |
| SEND_FRIEND_REQUEST | recipient_id | FRIEND_REQUEST_RECEIVED |
| LIST_FRIEND_REQUESTS | direction, cursor, limit | response lời mời pending |
| ACCEPT_FRIEND_REQUEST / DECLINE_FRIEND_REQUEST | friend_request_id | FRIEND_REQUEST_ACCEPTED / FRIEND_REQUEST_DECLINED |
| REMOVE_FRIEND | friend_id | FRIEND_REMOVED |
| LIST_FRIENDS | cursor, limit | response gồm trạng thái; FRIEND_STATUS_CHANGED |
| OPEN_DIRECT | peer_id | CHAT_STARTED nếu peer online |
| END_CHAT | conversation_id | CHAT_ENDED với reason |
| SEND_DIRECT | conversation_id, body, client_message_id | MESSAGE_RECEIVED |
| CREATE_GROUP | name | GROUP_CREATED |
| INVITE_GROUP | group_id, invitee_id | GROUP_INVITED |
| ACCEPT_INVITE / DECLINE_INVITE | invitation_id | GROUP_MEMBER_JOINED / INVITE_DECLINED |
| LEAVE_GROUP | group_id | GROUP_MEMBER_LEFT |
| REMOVE_GROUP_MEMBER | group_id, member_id | GROUP_MEMBER_REMOVED |
| LIST_GROUPS / LIST_GROUP_MEMBERS | cursor, limit; thêm group_id cho danh sách thành viên | response |
| LIST_GROUP_INVITATIONS | cursor, limit | response lời mời pending khi login lại |
| SEND_GROUP | group_id, body, client_message_id | MESSAGE_RECEIVED tới thành viên |
| FETCH_HISTORY | conversation_id, before_message_id, limit | response phân trang |
| FETCH_OFFLINE | cursor, limit | response các tin chưa xác nhận cho session user |
| ACK_DELIVERED | message_id | MESSAGE_DELIVERED |
| PING | rỗng | response/PONG |

MVP: chỉ người trong nhóm được mời; nhận lời mời không tự trở thành thành viên.
REMOVE_GROUP_MEMBER chỉ owner được gọi; không dùng lời mời kết bạn để tham gia nhóm.
Không JOIN_GROUP tùy ý: tham gia qua ACCEPT_INVITE.
Đổi chat: END_CHAT cũ rồi OPEN_DIRECT mới; lỗi mở mới không làm mất lịch sử.
CHAT_ENDED reason: `USER_REQUEST`, `LOGOUT`, `DISCONNECTED`, `TIMEOUT`.
Mất mạng có thể cần heartbeat, dự kiến ping 15 giây, timeout 45 giây.

## Invariant và kiểm thử phải có

- Request cần login bị từ chối trước xác thực; mọi ID phải được kiểm tra quyền.
- Không gửi tin nhóm cho người đã rời nhóm; không tự nhận lời mời của người khác.
- Không tự kết bạn/accept lời mời gửi cho user khác; hủy bạn là cập nhật hai chiều.
- Logout ngắt session; END_CHAT chỉ dừng chat hiện tại, không logout.
- Tin chưa ACK được query theo người nhận và receipt; cursor chỉ phân trang,
  không coi mọi message có ID nhỏ hơn cursor đã nhận. Nhận event/history không tự ACK.
- Message có ID do server cấp; retry `client_message_id` không tạo bản lưu trùng
  (khóa theo sender + conversation + client_message_id).
- Login lại lấy tin chưa ACK; ACK lặp không gây lỗi hoặc nhân bản trạng thái.
- Test frame chia nhỏ/coalesced, UTF-8, JSON sai, payload quá lớn, client đóng bất ngờ.
- Giới hạn queue/client chậm, phân trang tối đa 100 tin; không tải toàn bộ DB vào RAM.

Chi tiết field/error sẽ bổ sung cùng code ở từng phase; tài liệu này chưa phải
giao thức đã được triển khai hoặc kiểm thử.
