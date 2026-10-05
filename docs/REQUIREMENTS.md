# Đối chiếu yêu cầu chấm điểm

Nguồn: danh sách người dùng cung cấp ngày 2026-10-05. Đây là kế hoạch thực hiện,
**tất cả tính năng vẫn TODO**; có folder/tài liệu không có nghĩa đã đạt điểm.
Tổng các mục ngoài bổ sung: **23 điểm**; cộng T18 từ 2–5 thành **25–28 điểm** theo
phép cộng danh sách cung cấp, không suy ra cách quy đổi điểm môn học.

| ID | Task | Điểm | Phase | Module C chính | Tiêu chí demo/kiểm tra |
|---|---|---:|---|---|---|
| T01 | Xử lý truyền dòng | 1 | P1 | common/protocol | Frame chia nhỏ hoặc nhiều frame trong một recv vẫn parse đúng; partial send không mất dữ liệu |
| T02 | Cơ chế vào/ra socket trên server | 2 | P1 | common/net, server/transport | Nhiều client cùng kết nối; nonblocking select/output queue; client chậm không chặn toàn server |
| T03 | Đăng ký và quản lý tài khoản | 2 | P2 | server/auth, storage | Register; username unique; hash password; xem/sửa hồ sơ, đổi mật khẩu sau xác thực; dữ liệu còn sau restart |
| T04 | Đăng nhập và quản lý phiên | 2 | P2 | server/auth, presence | Login đúng/sai; một session/user; logout/timeout thu hồi session; request chưa login bị từ chối |
| T05 | Gửi lời mời kết bạn | 1 | P3 | server/friends, storage | A mời B; B nhận/lấy được lời mời pending, kể cả sau khi login lại; cấm tự mời/trùng |
| T06 | Chấp nhận/Từ chối lời mời kết bạn | 1 | P3 | server/friends | Chỉ người nhận được accept/decline; accept tạo bạn hai chiều, decline không tạo bạn |
| T07 | Hủy kết bạn | 1 | P3 | server/friends | A hủy B; cả hai mất quan hệ bạn bè; không xóa lịch sử chat |
| T08 | Danh sách bạn bè và trạng thái | 1 | P3 | server/friends, presence | Danh sách gồm bạn online/offline; trạng thái cập nhật khi login/logout/mất mạng |
| T09 | Tin nhắn giữa hai người dùng | 1 | P3 | server/chat, transport | A gửi B; chỉ đúng conversation nhận; C không đọc được; chuyển/dừng chat theo yêu cầu gốc |
| T10 | Ngắt kết nối | 1 | P1/P2/P3 | server/transport, auth, presence, chat | Đóng client/EOF/reset/timeout giải phóng tài nguyên và session; bên đang chat nhận reason phù hợp |
| T11 | Tạo nhóm chat | 1 | P4 | server/groups, storage | Tạo nhóm có ID, owner và thành viên ban đầu là người tạo |
| T12 | Thêm người dùng vào nhóm | 1 | P4 | server/groups | Gửi lời mời; người nhận accept mới thành thành viên; lời mời nhóm độc lập lời mời bạn |
| T13 | Xóa người dùng khỏi nhóm | 1 | P4 | server/groups | Owner xóa thành viên; người thường không được xóa; người bị xóa được báo và mất quyền |
| T14 | Rời nhóm chat | 1 | P4 | server/groups | Thành viên rời, không nhận/gửi tin mới; xử lý owner và nhóm rỗng nhất quán |
| T15 | Tin nhắn trong nhóm | 1 | P4 | server/groups, chat | Broadcast tới thành viên hiện tại; người ngoài/đã rời/đã bị xóa không có quyền |
| T16 | Gửi tin nhắn offline | 1 | P5 | server/chat, storage | B offline, A gửi; restart server, B login nhận tin; ACK/retry không mất hoặc lưu trùng |
| T17 | Ghi log hoạt động | 1 | P1–P5 | server/logging | File log có thời gian, actor/action/result, connect/auth/friends/groups/messages; không secret; rotation |
| T18 | Chức năng bổ sung | 2–5 | P8 | tùy tính năng | Chọn sau: read receipts, typing, tìm tin; có demo/test riêng, không tự gán điểm |
| T19 | Giao diện đồ họa | 3 | P6 | web + server/web | Form auth, online/bạn bè, lời mời, direct chat, quản lý nhóm, offline và thông báo lỗi hoạt động qua C |

## Yêu cầu ban đầu vẫn phải giữ

- Sau login trả danh sách các user khác online (khác danh sách bạn bè).
- Dừng cuộc trò chuyện hoặc ngắt kết nối phải báo cho bên còn lại.
- Có thể chuyển sang cuộc trò chuyện với user khác.
- Tham gia/rời nhóm qua flow rõ ràng; lưu lịch sử và tin offline.
- C cho toàn bộ nghiệp vụ/networking; JS chỉ giao diện.

## Các lựa chọn cần đối chiếu với giảng viên khi chốt bản nộp

- T12 hiện triển khai theo yêu cầu gốc: mời → người nhận chấp nhận → thêm vào nhóm.
  Nếu rubric đòi owner thêm trực tiếp, điều chỉnh flow và tiêu chí trước phase P4.
- “Quản lý tài khoản” hiện gồm hồ sơ và đổi mật khẩu; chưa mặc định phải có admin
  quản trị/xóa/khóa mọi user vì danh sách chưa chỉ rõ vai trò admin.
- Lịch sử/pagination phục vụ yêu cầu lưu tin, không tự tính lại là điểm bổ sung.
