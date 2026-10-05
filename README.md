# PRJNetwork — ứng dụng chat môn Lập trình mạng

Project dùng C11 cho phần xử lý; HTML/CSS/JavaScript cho GUI trình duyệt.
Hiện tại chỉ **setup project**, chưa viết source C hay tính năng ứng dụng.

## Đọc trước khi code

- [Hướng dẫn setup Windows](docs/SETUP.md)
- [19 task chấm điểm và tiêu chí nghiệm thu](docs/REQUIREMENTS.md)
- [Kiến trúc và phân chia module](docs/ARCHITECTURE.md)
- [Giao thức dự kiến](docs/PROTOCOL.md)
- [Tiến trình theo phase và nhật ký handoff](PROGRESS.md)
- [Quy tắc bắt buộc cho mỗi phiên](AGENTS.md)

## Build sau khi cài toolchain

Trong MSYS2 UCRT64, tại thư mục project:

```sh
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Chưa khai báo executable nên build chưa tạo chương trình. CMake chỉ cấu hình
ngôn ngữ C11; sẽ thêm target và dependency khi bắt đầu code.

## Cấu trúc

```text
src/server/       server C: transport/auth/friends/chat/groups/storage/logging/...
src/client/       client CLI C dùng kiểm thử TCP
src/common/       protocol và networking dùng chung
include/chat/    header C dùng chung
web/             HTML/CSS/JS giao diện (chưa triển khai)
tests/           kiểm thử giao thức và tích hợp (chưa triển khai)
data/            database runtime, không commit
logs/            log hoạt động runtime, không commit
docs/            setup, kiến trúc, giao thức
```

Chỉ bắt đầu GUI sau khi luồng chat C đã kiểm thử được. Mở riêng thư mục này
bằng Codex để `AGENTS.md` áp dụng cho toàn bộ project.

Giải thích đầy đủ từng module và folder: [ARCHITECTURE.md](docs/ARCHITECTURE.md).
Các folder module hiện chỉ có `.gitkeep`, không có logic đã triển khai.
