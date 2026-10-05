# Setup chi tiết trên Windows

## 0. Kiểm kê máy ngày 2026-10-05

**Cập nhật 19:14 Asia/Saigon:** UCRT64 đã cài, output người dùng xác nhận
GCC 16.2.0/GDB 18.1/CMake 4.4.4/Ninja 1.13.2 ở `/ucrt64/bin`.
CMake configure/generate và build setup đã chạy PASS trong project; chưa có target
nên Ninja báo `no work to do`. Header SQLite đã có; libsodium/libwebsockets còn thiếu.
Bảng dưới là kiểm kê trước khi cài, giữ lại để tham khảo lịch sử.

Kiểm tra PATH và vị trí cài phổ biến, không khẳng định đã quét toàn bộ ổ đĩa.

| Phần | Trạng thái kiểm tra | Vai trò / cần làm |
|---|---|---|
| Git | Có tại D:\Git\cmd | Quản lý phiên bản, clone/pull/push; không cần cài lại |
| GCC/GDB Code::Blocks | Có trong C:\Program Files\CodeBlocks\MinGW\bin | Compiler/debugger có sẵn; dùng toolchain UCRT64 riêng cho project để đồng bộ thư viện |
| Cursor | Có; lệnh code trỏ tới Cursor | Có thể dùng làm editor, không bắt buộc cài VS Code |
| Chrome/Edge | Có executable tại vị trí cài phổ biến | Chạy GUI và DevTools sau phase 6 |
| MSYS2 UCRT64 | C:\msys64 chỉ thấy etc; thiếu ucrt64.exe, bash.exe, pacman.exe | Chưa có bản cài dùng được; cài bằng installer chính thức |
| CMake/Ninja | Không có trong PATH; CMake không có ở vị trí standalone phổ biến | Cài package UCRT64 |
| pkgconf | Không có trong PATH/UCRT64 | Giúp build tìm header/library và flags |
| SQLite/libsodium/libwebsockets UCRT64 | Thiếu header trong C:\msys64\ucrt64\include | Cài trước phase sử dụng |
| Extension C/C++ và CMake Tools | Chưa thấy trong kết quả code --list-extensions của Cursor | Tùy chọn, hỗ trợ editor; CLI báo lỗi ghi log do quyền sandbox nên kiểm tra thêm trong UI |

Không cần cài backend Node.js, MySQL/PostgreSQL server, XAMPP hay WSL theo
kiến trúc đang chọn. SQLite chạy trong process C và lưu vào file, không cần dịch vụ DB riêng.
Winsock thuộc Windows; toolchain MinGW-w64 cung cấp header/import library, không
cần tải một phần mềm socket riêng. JSON parser C chọn ở P1-01, không phải package JS.

## 1. Toolchain thống nhất cả nhóm

Dùng VS Code hoặc Codex để sửa code, MSYS2 **UCRT64** để build C11, CMake và Ninja.
Máy hiện đã có toolchain UCRT64; các bước dưới dành cho máy mới hoặc thành viên khác.
Không trộn compiler Code::Blocks với thư viện UCRT64 vì chúng có thể khác ABI/runtime.

MSYS2 quản lý việc cài/cập nhật package; UCRT64 là môi trường compiler/thư viện
native Windows thống nhất. GCC biến source C thành object/executable; GDB dùng
breakpoint và kiểm tra biến khi lỗi. CMake đọc CMakeLists.txt, cấu hình target và
sinh hướng dẫn build; Ninja thực thi các bước compile/link đó. CMake không thay GCC.

1. Tải MSYS2 từ https://www.msys2.org/ và cài ở `C:\msys64`.
   Nếu installer từ chối thư mục có sẵn, giữ nguyên thư mục đó và chọn đường dẫn
   khác, ví dụ `C:\msys64-chat`; thay đường dẫn trong các ví dụ PATH bên dưới tương ứng.
2. Mở **MSYS2 UCRT64** trong Start Menu.
3. Cập nhật:

```sh
pacman -Syu
```

Nếu chương trình yêu cầu đóng terminal, đóng rồi mở lại UCRT64, chạy lại lệnh
đến khi không còn bản cập nhật. Sau đó cài công cụ:

```sh
pacman -S --needed git mingw-w64-ucrt-x86_64-gcc mingw-w64-ucrt-x86_64-gdb mingw-w64-ucrt-x86_64-cmake mingw-w64-ucrt-x86_64-ninja
```

Kiểm tra:

```sh
echo "$MSYSTEM"
which gcc cmake ninja git
gcc --version
cmake --version
ninja --version
git --version
```

`MSYSTEM` phải là `UCRT64`; gcc/cmake/ninja phải ở `/ucrt64/bin`.
Git có thể ở `/usr/bin`. Không dùng terminal MSYS mặc định để build project này.

## 2. Mở project và build

Trong terminal UCRT64:

```sh
cd '/d/Lập trình mạng/PRJNetwork'
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Kết quả mong đợi: configure/generate thành công; Ninja báo không có việc cần build.
Chưa có source/target nên chưa tạo executable. Đây chỉ là kiểm tra toolchain.

Nếu toolchain gặp lỗi đường dẫn Unicode, clone repo mới vào `D:\network-chat`.
Không cần đổi tên thư mục bài tập cũ. Nếu đã configure bằng compiler khác,
dùng thư mục build mới, ví dụ `-B build-ucrt64`; không tái sử dụng cache cũ.

## 3. VS Code (tùy chọn)

Có thể tiếp tục dùng Cursor hiện có; không bắt buộc cài thêm editor.
Cài extensions **C/C++** và **CMake Tools** của Microsoft.
Trong editor mở Extensions (`Ctrl+Shift+X`), tìm C/C++ (`ms-vscode.cpptools`)
và CMake Tools (`ms-vscode.cmake-tools`). Nếu editor không cung cấp extension
tương ứng, vẫn build trong terminal UCRT64; VS Code chính thức là lựa chọn khác.
Extension giúp autocomplete/diagnostics/debug và thao tác CMake, không tự cài compiler.
Mở `PRJNetwork` bằng File → Open Folder. Cách đơn giản nhất là build trong UCRT64.
Nếu build bằng PowerShell, thêm `C:\msys64\ucrt64\bin` vào PATH của phiên hiện tại:

```powershell
$env:Path = 'C:\msys64\ucrt64\bin;' + $env:Path
Get-Command gcc, cmake, ninja
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Chọn compiler UCRT64 trong CMake Tools; mở terminal có PATH UCRT64 khi chạy để
tìm được DLL của compiler/thư viện. Debug bằng GDB UCRT64 khi phase socket đã có code.

### Chạy thử ngay trong terminal VS Code

Mở folder `D:\Lập trình mạng\PRJNetwork` bằng File → Open Folder, chọn Terminal →
New Terminal. Nếu terminal là PowerShell (prompt bắt đầu bằng `PS`), chạy:

```powershell
cd 'D:\Lập trình mạng\PRJNetwork'
$env:Path = 'C:\msys64\ucrt64\bin;' + $env:Path
Get-Command gcc, gdb, cmake, ninja
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Get-Command cần trỏ tới `C:\msys64\ucrt64\bin`; cấu hình PATH chỉ áp dụng cho
terminal này, cần chạy lại nếu mở terminal mới. Project hiện chưa có target nên
Ninja báo `no work to do`, chưa có executable server/client để chạy.

Nếu muốn chuyển terminal PowerShell đó sang UCRT64 đầy đủ (để dùng pacman/which):

```powershell
& 'C:\msys64\msys2_shell.cmd' -defterm -here -no-start -ucrt64
```

Sau khi prompt đổi sang Bash, dùng cú pháp Bash:

```sh
echo "$MSYSTEM"
which gcc cmake ninja
cd '/d/Lập trình mạng/PRJNetwork'
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Không dùng `$env:Path` trong Bash; không dùng `echo "$MSYSTEM"` để kiểm tra
UCRT64 trong PowerShell. Launcher MSYS2 tồn tại trên máy; chưa kiểm tra tương tác
UI terminal VS Code trong phiên này.
Nguồn: [MSYS2 terminal launcher](https://www.msys2.org/docs/terminals/),
[VS Code terminal profiles](https://code.visualstudio.com/docs/terminal/profiles).

## 4. Thư viện cài khi bắt đầu phase liên quan

Phase 2 dùng SQLite (database) và libsodium (hash mật khẩu); phase 6 dùng
libwebsockets (HTTP/WebSocket bằng C). Thư viện cung cấp API C:

| Thư viện/công cụ | Tác dụng | Task |
|---|---|---|
| SQLite | Lưu bền tài khoản, bạn bè, nhóm, message và receipt trong file DB | T03–T09, T11–T16 |
| libsodium | Hash/verify mật khẩu với salt và password-hashing API | T03, T04 |
| libwebsockets | Server HTTP/WebSocket C giao tiếp với GUI trình duyệt | T19 |
| pkgconf | Cung cấp pkg-config để tìm version/header/library/flags cho build | Hỗ trợ link dependency |

Cài SQLite/libsodium/pkgconf khi chuẩn bị phase 2:

```sh
pacman -S --needed mingw-w64-ucrt-x86_64-sqlite3 mingw-w64-ucrt-x86_64-libsodium mingw-w64-ucrt-x86_64-pkgconf
```

Cài libwebsockets khi chuẩn bị phase 6:

```sh
pacman -S --needed mingw-w64-ucrt-x86_64-libwebsockets
```

Hoặc cài tất cả từ đầu nếu muốn chuẩn bị sẵn:

```sh
pacman -S --needed mingw-w64-ucrt-x86_64-sqlite3 mingw-w64-ucrt-x86_64-libsodium mingw-w64-ucrt-x86_64-libwebsockets mingw-w64-ucrt-x86_64-pkgconf
```

Không cần Node.js, npm, Express hoặc Electron để chạy thiết kế này.
JSON parser C sẽ được chọn và khóa phiên bản trong phase 1; không tự viết parser JSON.
Có thể chọn cJSON; package dự kiến là `mingw-w64-ucrt-x86_64-cjson` nhưng chưa
chốt dependency trong repo. Chỉ cài/nối nó khi nhận task P1-01.
CMake hiện chưa có target: khi dùng thư viện mới, thêm dependency vào CMake
và ghi rõ cách tìm/link chúng trong tài liệu. Không chỉ include header rồi coi là xong.

Sau khi cài các package ở trên, kiểm tra trong UCRT64:

```sh
sqlite3 --version
pkg-config --version
pkg-config --modversion sqlite3 libsodium libwebsockets
```

Nếu chưa cài phase 6 thì bỏ `libwebsockets` trong lệnh cuối. Các lệnh này kiểm tra
công cụ/metadata dependency, chưa chứng minh code ứng dụng đã link/chạy đúng.
Không tạo DB thật trong repo trước khi có schema. DB Browser for SQLite và
Wireshark chỉ là công cụ hỗ trợ tùy chọn, không cần để bắt đầu.

## 5. GitHub và làm việc nhóm

Tạo repo GitHub trống bằng tài khoản của bạn. Trước khi init, kiểm tra:

```sh
git rev-parse --show-toplevel
```

Nếu không có repo, tại `PRJNetwork`:

```sh
git init -b main
git add .
git commit -m "chore: bootstrap C chat project and handoff docs"
git remote add origin https://github.com/<owner>/<repo>.git
git push -u origin main
```

Thay `<owner>/<repo>` bằng repo thật. Nếu lệnh kiểm tra trả về repo cha,
quyết định dùng repo cha hay repo riêng trước khi init; tránh repo lồng ngoài ý muốn.
Nếu repo đã tồn tại, kiểm tra branch/remote trước, không chạy lại toàn bộ lệnh init.
Không push token/password; dùng đăng nhập GitHub hoặc credential manager.

Người cùng nhóm:

```sh
git clone https://github.com/<owner>/<repo>.git
cd <repo>
git switch -c feat/tcp-framing
```

Cài toolchain, đọc `AGENTS.md` và `PROGRESS.md`, rồi mở repo trong Codex.
Prompt gợi ý:

> Đọc AGENTS.md, PROGRESS.md và docs. Kiểm tra trạng thái code thực tế.
> Tiếp tục task P1-01, giữ toàn bộ networking/nghiệp vụ bằng C.
> Trước khi kết thúc phiên, cập nhật PROGRESS.md với kiểm tra và bước tiếp theo.

Sau một phiên: kiểm tra diff → cập nhật tiến trình → commit → push branch → mở PR.
Để đồng bộ main khi không có thay đổi chưa commit:

```sh
git switch main
git pull --ff-only
```

Không cùng nhận một task; người làm TCP framing và người làm auth phải thống nhất
header/giao thức trước. Chỉ ghi task DONE khi đạt acceptance criteria.

## 6. Chạy mạng khi phase 1 đã hoàn thành

- Ban đầu dùng `127.0.0.1:9000`, hai terminal client và một server.
- GUI sau này dùng `http://127.0.0.1:8080`, endpoint `/ws` trên cùng server C.
- Khi thử LAN: server bind địa chỉ phù hợp, client dùng IPv4 LAN của máy server
  (`ipconfig`), cho phép executable/port qua Windows Firewall trên mạng Private.
- Không cần mở port router cho demo LAN. Không tắt toàn bộ firewall.
- Khi rút mạng bất ngờ, heartbeat/timeout phải phát hiện; không chỉ dựa vào `recv == 0`.

## 7. Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| `cmake`/`ninja` không được nhận diện | Dùng UCRT64 hoặc PATH phiên PowerShell ở trên |
| Compiler là Code::Blocks | Kiểm tra `which gcc`/`Get-Command gcc`, dùng build dir mới |
| Thiếu DLL khi chạy executable | Chạy trong UCRT64; lúc đóng gói mới gom DLL cần thiết |
| `address already in use` sau phase socket | Kiểm tra server khác đang chạy; đổi port cấu hình |
| Máy khác không kết nối được | Kiểm tra IP/bind/firewall và cùng mạng LAN |
| Codex không biết làm tiếp | Mở đúng root repo, yêu cầu đọc AGENTS.md/PROGRESS.md |

Nguồn chính thức: [MSYS2 setup](https://www.msys2.org/),
[UCRT64 environment](https://www.msys2.org/docs/environments/),
[mingw-w64 với MSYS2](https://www.mingw-w64.org/getting-started/msys2/).
Package chính thức: [SQLite](https://packages.msys2.org/packages/mingw-w64-ucrt-x86_64-sqlite3),
[libsodium](https://packages.msys2.org/packages/mingw-w64-ucrt-x86_64-libsodium),
[libwebsockets](https://packages.msys2.org/packages/mingw-w64-ucrt-x86_64-libwebsockets),
[pkgconf](https://packages.msys2.org/packages/mingw-w64-ucrt-x86_64-pkgconf).
