# Quy tắc làm việc cho Codex và thành viên nhóm

## Bắt đầu mỗi phiên

1. Đọc `PROGRESS.md`, `README.md`, `docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`, `docs/PROTOCOL.md`.
2. Kiểm tra `git status`, diff và trạng thái code thực tế; không coi nhật ký là bằng chứng tính năng đã chạy.
3. Chọn một task trong phase hiện tại, ghi người phụ trách và phạm vi vào `PROGRESS.md` trước khi sửa.
4. Không ghi đè thay đổi của người khác; nếu cùng sửa một module, thống nhất trước.

## Ràng buộc

- C11 cho server, client CLI, networking, xác thực, phân quyền, lưu trữ và nghiệp vụ.
- JS chỉ phục vụ giao diện trình duyệt, gửi yêu cầu và hiển thị sự kiện. Không tạo backend Node.js hoặc database trong JS.
- HTML/CSS được dùng cho giao diện. Thư viện native phải có API C; không chuyển project sang C++.
- Hàm/biến camelCase; hằng UPPER_SNAKE_CASE; thụt 4 spaces, dấu `{` cùng dòng. Comment giải thích lý do.
- Kiểm tra lỗi API và giải phóng socket/buffer/database ở mọi nhánh lỗi.
- Không giả định một `send`/`recv` tương ứng một thông điệp. Giới hạn kích thước trước khi cấp phát.
- Không lưu mật khẩu plaintext, không ghi password/token vào log, dùng prepared statements.
- Mỗi thay đổi phải có kiểm tra phù hợp; không đánh dấu hoàn tất nếu chưa đạt tiêu chí nghiệm thu.
- Đối chiếu task T01–T19 trong `docs/REQUIREMENTS.md`; lời mời kết bạn và lời mời nhóm là hai nghiệp vụ riêng.
- Hiện người dùng chỉ yêu cầu setup/tài liệu. Không viết source hoặc tính năng cho đến khi được yêu cầu bắt đầu code.
- Không tự commit/push khi người dùng chưa yêu cầu. Không đưa build, database thật hay secret vào Git.

## BẮT BUỘC trước khi kết thúc mỗi phiên

**Cập nhật `PROGRESS.md` dù phiên hoàn tất, bị lỗi hay dừng giữa chừng.**

Ghi ngày giờ Asia/Saigon, người/agent, phase/task, file đã sửa, kết quả chạy lệnh/test
(pass/fail/chưa chạy và lý do), quyết định kỹ thuật, hạn chế/blocker và bước tiếp theo cụ thể.
Cập nhật bảng phase và task hiện tại đúng thực tế. Không ghi “đã test” nếu chỉ đọc code.
Không xóa lịch sử phiên cũ. Nếu đổi giao thức/kiến trúc, cập nhật tài liệu tương ứng cùng phiên.
Cuối câu trả lời nêu kết quả kiểm tra và việc còn thiếu.

## Handoff và Git

- Mỗi nhánh tập trung một task: `feat/<task>`, `fix/<task>`, `docs/<task>`.
- Pull trước khi nhận task; commit code cùng bản cập nhật tiến trình.
- PR ghi hành vi thay đổi và cách kiểm tra. Không đưa dữ liệu demo nhạy cảm vào PR.
- Sau merge, người tiếp theo đọc nhật ký và kiểm tra lại môi trường trước khi tiếp tục.
