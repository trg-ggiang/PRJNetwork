# Tiến trình project và handoff

## Trạng thái hiện tại

- Phase hiện tại: **P0 — bootstrap**.
- Đã có cấu trúc thư mục, cấu hình C11/CMake, setup và tài liệu handoff; chưa có source.
- Chưa có networking, xác thực, kết bạn, nhóm, log runtime, database hay GUI hoạt động.
- Đã đối chiếu đủ 19 mục rubric trong `docs/REQUIREMENTS.md`; 23 điểm cố định + 2–5 bổ sung.
- Toolchain UCRT64 đã cài: GCC 16.2.0, GDB 18.1, CMake 4.4.4, Ninja 1.13.2
  (version/path theo output người dùng; CMake/GCC được kiểm tra thêm bằng configure thực tế).
- CMake configure/generate và build setup đã PASS tại `build/`; chưa có target nên
  Ninja báo no work to do. Đây là kiểm tra môi trường, chưa có tính năng ứng dụng.
- Header SQLite UCRT64 có; header libsodium/libwebsockets chưa có ở vị trí kiểm tra.
- Task tiếp theo: chuẩn bị repo GitHub/chia sẻ nhóm; cài thư viện theo phase.
  Chỉ bắt đầu P1-01 khi được yêu cầu code.
- Người phụ trách task tiếp theo: **chưa nhận**.

## Roadmap và tiêu chí nghiệm thu

| Phase | Phạm vi | Trạng thái | Tiêu chí hoàn thành |
|---|---|---|---|
| P0 | Repo, toolchain, CMake, tài liệu handoff | IN PROGRESS | Toolchain cài xong; CMake configure được trên máy nhóm; repo chia sẻ được |
| P1 | Socket C, framing/JSON, CLI, nhiều client | TODO | Hai client kết nối, echo; partial/coalesced frame; lỗi/đóng socket xử lý đúng |
| P2 | SQLite, tài khoản/hồ sơ/đổi mật khẩu, session, online | TODO | Password hash; đăng ký/login/hồ sơ/đổi mật khẩu; lỗi và online cập nhật đúng |
| P3 | Kết bạn và chat 1-1, dừng/ngắt/chuyển chat | TODO | Mời/accept/decline/hủy bạn, danh sách/trạng thái; A gửi B, dừng/ngắt báo B; chuyển sang C |
| P4 | Tạo nhóm, mời, tham gia/xóa/rời, chat nhóm | TODO | Ba client; accept/decline; owner xóa thành viên; quyền và broadcast đúng |
| P5 | Lưu tin, offline, lịch sử, ACK/retry | TODO | B offline, A gửi, restart server, B login nhận lại; retry không lưu trùng |
| P6 | GUI JS + HTTP/WebSocket C | TODO | Hai trình duyệt thực hiện đủ auth/friends/direct/group/offline qua service C |
| P7 | Kiểm thử LAN, lỗi, tài liệu demo/báo cáo | TODO | Demo nhiều máy, restart/mất mạng, quyền truy cập, build từ clone sạch |
| P8 | Tính năng bổ sung | OPTIONAL | Chọn tính năng sau khi P0–P7 đạt nghiệm thu |

## Task gần nhất

- [x] P0-01: cài UCRT64 GCC/GDB/CMake/Ninja trên máy dev.
- [ ] P0-02: build CMake đã PASS; còn tạo repo GitHub, mời thành viên.
- [ ] P1-01: chốt parser JSON C và framing; tạo API encode/decode, buffer và kiểm thử.
- [ ] P1-02: wrapper Winsock/POSIX, TCP listen/connect, cleanup.
- [ ] P1-03: nonblocking select, output queue, CLI nhận event khi đang nhập lệnh.
- [ ] P1-04: kiểm thử hai client, giới hạn frame/queue, EOF và timeout.
- [ ] P1-05: logging C cơ bản, log connect/disconnect/I/O; bổ sung sự kiện theo P2–P5.

## Task chức năng theo rubric

T01–T19 và acceptance criteria nằm trong `docs/REQUIREMENTS.md`; tất cả TODO.
Task ID Pn-xx là bước triển khai trong phase, Txx là mục chấm điểm, không thay thế nhau.

- [ ] P2-01: storage SQLite + tài khoản/hash password (T03).
- [ ] P2-02: login/logout/session/presence và cleanup (T04, T10).
- [ ] P2-03: hồ sơ/đổi mật khẩu, kiểm tra quyền và log auth (T03, T17).
- [ ] P3-01: lời mời kết bạn pending, lưu bền và list khi login lại (T05).
- [ ] P3-02: accept/decline/hủy bạn (T06, T07).
- [ ] P3-03: danh sách bạn và cập nhật trạng thái (T08).
- [ ] P3-04: direct chat, END_CHAT, chuyển chat, disconnect event (T09, T10).
- [ ] P4-01: tạo nhóm/owner, mời và accept/decline (T11, T12).
- [ ] P4-02: xóa thành viên/rời nhóm, chuyển owner, nhóm rỗng (T13, T14).
- [ ] P4-03: broadcast nhóm và quyền truy cập (T15).
- [ ] P5-01: lưu message/receipts, offline/history, ACK/idempotency (T16).
- [ ] P5-02: hoàn thiện log nghiệp vụ, rotation và lỗi ghi log (T17).
- [ ] P6-01: adapter HTTP/WebSocket C cùng service, giới hạn/origin (T19).
- [ ] P6-02: GUI auth/friends/direct/groups/offline (T19).
- [ ] P7-01: đối chiếu toàn bộ rubric, kiểm thử clone sạch/LAN/restart/mất mạng.
- [ ] P8-01: chọn/ghi rõ phạm vi chức năng bổ sung và tiêu chí demo (T18).

## Quyết định đã chốt

- C11 cho toàn bộ logic và networking; JS chỉ GUI. Không backend Node.js.
- TCP cho CLI và HTTP/WebSocket C cho browser, cùng service C.
- SQLite runtime không commit; libsodium cho password hash; libwebsockets dự kiến cho GUI.
- CMake + Ninja + MSYS2 UCRT64 cho Windows; tài liệu có đường mở rộng POSIX.
- AGENTS.md là rules Codex tự đọc, bắt buộc cập nhật file này sau mỗi phiên.

## Nhật ký phiên (append-only)

### 2026-10-05 — Codex — P0 bootstrap

- Yêu cầu: setup môn Lập trình mạng, C và GUI JS, handoff theo phase qua GitHub.
- Đã kiểm tra thư mục PRJNetwork trống và công cụ trên máy.
- Đã tạo README, AGENTS, PROGRESS, docs/SETUP, ARCHITECTURE, PROTOCOL;
  CMakeLists, config header/common, server/client main, gitignore/gitattributes,
  thư mục web/tests/data.
- Scaffold chỉ in thông báo rồi thoát; không mô tả là server đang hoạt động.
- Kiểm tra: hai scaffold tạm compile bằng GCC Code::Blocks với C11 và warnings,
  chạy exit 0. Sau đó đã bỏ source và executable theo yêu cầu chỉ setup của người dùng.
- Trạng thái cuối: CMake chỉ khai báo project C11, chưa có target; thư mục source trống.
  Chưa kiểm tra CMake/Ninja vì máy chưa có công cụ. Không cài toolchain hay tạo/push repo.
- Blocker: thiếu CMake/Ninja/UCRT64, chưa có URL repo GitHub.
- Bước tiếp: cài UCRT64, configure CMake, tạo repo; chỉ nhận P1-01 khi người dùng yêu cầu code.

### 2026-10-05 17:31 Asia/Saigon — Codex — P0 rà soát setup theo rubric

- Mục tiêu: rà soát và giải thích cấu trúc; vẫn chỉ setup, không viết source tính năng.
- Phát hiện bản trước thiếu kết bạn, xóa thành viên nhóm, log hoạt động và phạm vi
  quản lý tài khoản. Đã bổ sung kế hoạch, quyền, request/event và tiêu chí demo.
- File thay đổi: README, AGENTS, PROGRESS, docs/ARCHITECTURE, docs/PROTOCOL,
  .gitignore; thêm docs/REQUIREMENTS và folder module cùng logs/.gitkeep.
- Kiểm tra thực tế: bảng rubric có 19 dòng T01–T19; tổng điểm cố định 23;
  không có link Markdown local bị thiếu; không có file source C/H/JS/HTML;
  build trống, data/logs chỉ có .gitkeep.
- `git rev-parse --show-toplevel`: báo chưa có repo; CMake/Ninja không tìm thấy
  trong PATH. Chưa chạy build/test tính năng và chưa init/commit/push.
- Đã tra nguồn chính thức Microsoft cho select/Winsock và MSYS2 cho libwebsockets;
  kiến trúc vẫn là đề xuất, chưa có dependency được link hoặc phiên bản khóa.
- Quyết định: bạn bè và lời mời nhóm tách riêng; owner được xóa thành viên;
  online users khác friend list; runtime logs/database không commit.
- Cần chốt theo giảng viên: thêm nhóm qua invitation hay thêm trực tiếp;
  quản lý tài khoản có đòi admin không; lựa chọn hiện tại ghi trong REQUIREMENTS.
- Bước tiếp: thành viên cài toolchain, configure CMake, tạo repo GitHub.
  Chỉ bắt đầu P1 khi người dùng yêu cầu code; chưa có task coding được nhận.

### 2026-10-05 17:35 Asia/Saigon — Codex — P0 kiểm kê công cụ và hướng dẫn cài

- Mục tiêu: xác định phần có/thiếu và giải thích tác dụng, cách setup; không cài hoặc viết source.
- Đã kiểm tra PATH, đường dẫn toolchain phổ biến, thư mục MSYS2, header UCRT64,
  executable editor/browser và extension list của Cursor.
- Có: Git, GCC/GDB Code::Blocks, Cursor, Chrome/Edge. Không có công cụ UCRT64,
  CMake/Ninja/pkgconf hoặc header SQLite/libsodium/libwebsockets ở vị trí kiểm tra.
- C:\msys64 tồn tại nhưng chỉ thấy etc, thiếu launcher/bash/pacman nên chưa dùng được.
- Lệnh code --list-extensions trả danh sách không có C/C++/CMake Tools nhưng có
  lỗi ghi log EPERM do sandbox; chưa xác nhận thêm trong UI. Không sửa cấu hình editor.
- Cập nhật docs/SETUP với bảng kiểm kê, vai trò công cụ/thư viện, cách cài theo phase,
  lệnh kiểm tra và xử lý thư mục MSYS2 có sẵn. Tra package và docs chính thức.
- Chưa chạy CMake/build/link test vì dependency chưa cài. Chưa init/commit/push.
- Bước tiếp: người dùng cài MSYS2 UCRT64 cùng GCC/GDB/CMake/Ninja; chạy kiểm tra
  version/path và configure project. SQLite/libsodium/GUI có thể cài theo phase.

### 2026-10-05 19:14 Asia/Saigon — Codex — P0 xác nhận toolchain và configure

- Người dùng cung cấp output UCRT64 với gcc/gdb/cmake/ninja ở /ucrt64/bin và version.
- Đã kiểm tra C:\msys64\ucrt64\bin có GCC/CMake và chạy trong workspace:
  `cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug` → PASS (GNU 16.2.0),
  `cmake --build build` → PASS (ninja: no work to do), cả hai exit 0.
- PATH UCRT64 chỉ thêm cho process kiểm tra, không sửa PATH hệ thống.
- Header SQLite có; sodium.h và libwebsockets.h chưa có. Chưa test link SQLite.
- File thay đổi: PROGRESS, docs/SETUP; build sinh cache/compiler-check theo CMake,
  đã thuộc gitignore. Không viết source tính năng, không init/commit/push.
- P0-01 DONE; P0-02 còn phần GitHub/chia sẻ nhóm. Bước tiếp: thư viện theo phase
  và repo; chưa nhận task code. P0 vẫn IN PROGRESS.

### 2026-10-05 19:17 Asia/Saigon — Codex — P0 terminal VS Code

- Người dùng hỏi cách chạy thử trực tiếp trong terminal VS Code.
- Cập nhật docs/SETUP với cách PowerShell thêm PATH UCRT64 theo phiên và build;
  cách mở UCRT64 bằng msys2_shell.cmd trong terminal hiện tại, tách cú pháp hai shell.
- Đã xác nhận msys2_shell.cmd và CMake UCRT64 tồn tại; configure/build đã PASS
  ở phiên 19:14. Không chạy lại vì không đổi cấu hình build hoặc source.
- Chưa kiểm tra tương tác UI VS Code, không chỉnh settings editor/global PATH.
- Không viết source/target; hiện chỉ kiểm tra setup, chưa có executable chat.
- Bước tiếp: người dùng chạy các lệnh trong terminal editor; sau đó repo/dependency.

### Mẫu cho phiên tiếp theo (copy và điền)

```text
YYYY-MM-DD HH:mm Asia/Saigon — người/agent — phase/task
Mục tiêu và phạm vi:
File thay đổi:
Hành vi đã hoàn thành:
Lệnh/test + kết quả (pass/fail/chưa chạy):
Quyết định kỹ thuật:
Hạn chế/blocker:
Bước tiếp theo cụ thể + người phụ trách:
```
