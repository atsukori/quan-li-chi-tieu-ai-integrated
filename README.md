# Hệ thống Quản lý Chi tiêu Tập thể / Tổ chức

Quản lí bởi ChatGPT và Codex

Ứng dụng web Flask giúp một tập thể hoặc tổ chức ghi nhận thu chi, theo dõi ngân sách và xem báo cáo tài chính. Ứng dụng dùng SQLite, hỗ trợ chạy trên Windows và Raspberry Pi/Linux; giao diện Bootstrap được lưu cục bộ để sử dụng trong mạng nội bộ không cần CDN.

## Tính năng

- Đăng nhập, phân quyền quản trị viên và thành viên; mật khẩu được lưu dưới dạng băm.
- Quản lý tài khoản thành viên và danh mục thu/chi.
- Thêm, sửa, xóa giao dịch; hỗ trợ nguồn `personal` (cá nhân), `common` (quỹ chung) và `unit` (ngân sách đơn vị).
- Hạn mức theo tuần, tháng hoặc quý; cảnh báo khi sử dụng từ 80% hoặc vượt mức.
- Bảng tổng quan, lọc báo cáo theo thời gian, nguồn tiền và thành viên; xuất Excel hoặc CSV UTF-8.
- Nhập giao dịch từ tệp `.xlsx` theo mẫu sau bước xem trước.
- Nhật ký hoạt động quản trị, sao lưu SQLite từ giao diện hoặc dòng lệnh.

## Công nghệ và cấu trúc

- Python 3.11, Flask, Flask-SQLAlchemy, SQLAlchemy, Flask-Login và Flask-WTF.
- SQLite làm cơ sở dữ liệu mặc định; Waitress chạy ứng dụng khi `FLASK_DEBUG=0`.
- Cấu trúc Flask Application Factory + Blueprint; các phần chính nằm trong `app/`.

Các bảng nghiệp vụ gồm `User`, `Role`, `Category`, `Expense` và `Budget`. `AuditLog` lưu dấu vết thao tác quản trị và tài chính, không lưu giá trị mật khẩu. SQLite được lưu mặc định tại `instance/expense.db`. Một số trường bổ sung phục vụ khóa tài khoản, ẩn danh mục, nguồn ngân sách và chu kỳ hạn mức.

## Chạy trên Windows

Cài Python 3.11, mở PowerShell tại thư mục dự án rồi chạy:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
```

Mở `.env`, đổi `SECRET_KEY` thành chuỗi ngẫu nhiên riêng. Tạo dữ liệu mẫu và chạy ứng dụng:

```powershell
python seed.py
python run.py
```

Mở `http://127.0.0.1:5000`. Dừng ứng dụng bằng `Ctrl+C` trong terminal. Để chạy trên LAN, đặt `FLASK_HOST=0.0.0.0` trong môi trường chạy ứng dụng và giới hạn truy cập bằng firewall.

## Chạy trên Raspberry Pi / Linux

Tạo môi trường bằng Python 3.11 và cài dependencies:

```bash
python3.11 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

Sau khi tạo `.env` với `SECRET_KEY` riêng, dùng `./run_admin.sh` để khởi tạo tài khoản quản trị nếu CSDL còn trống và chạy server trên LAN. Script `./run_demo.sh` tạo lại dữ liệu demo và yêu cầu xác nhận trước khi xóa dữ liệu cũ. Hướng dẫn cài Python 3.11 trên Debian 13 Trixie có trong [docs/python311-trixie.md](docs/python311-trixie.md).

## Tài khoản và dữ liệu demo

`python seed.py` tạo dữ liệu demo nếu CSDL chưa có tài khoản. Tài khoản mẫu:

| Vai trò | Tên đăng nhập | Mật khẩu ban đầu |
|---|---|---|
| Quản trị viên | `admin` | `admin12345` |
| Thành viên | `member1` đến `member4` | `member12345` |

Đây là thông tin thử nghiệm; hãy đổi mật khẩu trước khi nhập dữ liệu thật. Không dùng `python seed.py --reset` trên CSDL cần giữ vì lệnh này xóa dữ liệu hiện có.

## Sao lưu và khôi phục

Quản trị viên có thể tạo và tải bản sao lưu từ **Quản trị → Sao lưu database**. Có thể tạo bản sao từ dòng lệnh:

```powershell
python backup_db.py
```

Bản sao được kiểm tra toàn vẹn và lưu trong `instance/backups/`. Khôi phục qua giao diện chưa được hỗ trợ; hãy dừng ứng dụng và làm theo hướng dẫn trong phần sao lưu và khôi phục của [docs/description.md](docs/description.md).

## Tài liệu dự án

- [Mô tả tính năng, cấu trúc, cài đặt và phạm vi dự án](docs/description.md)
- [Hồ sơ dự án](docs/quan-li-chi-tieu-refresh.txt)
- [Cấu trúc thư mục](docs/project-structure.txt)
- [Hướng dẫn Raspberry Pi OS / Debian 13 Trixie](docs/python311-trixie.md)

## Giới hạn hiện tại

Ứng dụng chưa kết nối ngân hàng hoặc ví điện tử, chưa có ứng dụng di động độc lập, chưa hỗ trợ đa tiền tệ hay nhiều đơn vị độc lập trong cùng một CSDL. Chức năng khôi phục bản sao lưu cần thực hiện thủ công.

## Bàn giao ngữ cảnh cho phiên phát triển AI

### Mục tiêu dự án

Đây là ứng dụng web quản lý thu chi cho tập thể / tổ chức. Mục tiêu là minh bạch hóa các khoản thu chi, tách tiền cá nhân khỏi quỹ chung và ngân sách đơn vị, theo dõi hạn mức định kỳ và hỗ trợ báo cáo. Dự án dùng Python 3.11, Flask, SQLite và kiến trúc Application Factory + Blueprint; cần tiếp tục chạy trên Windows và Raspberry Pi Zero 2 W / Linux.

### Trạng thái mã nguồn tại thời điểm sao chép

- Mã nguồn được sao chép từ `atsukori/quan-li-chi-tieu-refresh`, branch `codex`, commit `bb057a4bf4c9111fe13cfe673e5efce81a5f746e`.
- Ứng dụng hiện có đăng nhập và phân quyền admin/user, quản lý tài khoản và danh mục, CRUD giao dịch, import Excel có xem trước, hạn mức tuần/tháng/quý, cảnh báo, báo cáo và xuất CSV/XLSX, audit log và sao lưu SQLite.
- Nguồn tiền hiện có `personal`, `common` và `unit`. Giữ các giá trị nghiệp vụ này khi bổ sung tính năng.
- CSDL mặc định trong mã hiện tại là `instance/expense.db`.
- Chưa có thành phần hoặc tính năng AI trong mã nguồn ban đầu; yêu cầu tích hợp AI là mục tiêu mới do giáo viên yêu cầu.
- Branch `main` là bản sao snapshot của branch nguồn. Branch `codex` cũng bắt đầu từ snapshot đó; README này có thêm ghi chú bàn giao riêng cho hướng tích hợp AI.

### Hướng tích hợp AI đang được cân nhắc

Chưa chốt nhà cung cấp AI hay tính năng cụ thể. Các ý tưởng đã thảo luận:

1. **Đọc hóa đơn/biên lai**: trích ngày, số tiền và nội dung từ ảnh để điền sẵn biểu mẫu giao dịch; gợi ý danh mục nếu đủ tin cậy. Người dùng phải xem lại và xác nhận trước khi lưu. Hướng này đã được nêu trong tài liệu dự án.
2. **Tóm tắt báo cáo bằng AI**: dựa trên bộ lọc báo cáo hiện tại để giải thích xu hướng thu chi, biến động bất thường và nguy cơ vượt hạn mức.
3. **Gợi ý danh mục giao dịch**: đề xuất danh mục theo nội dung ghi chú, người dùng tự xác nhận.

Nhận diện hóa đơn là hướng demo được đề xuất vì gắn với quy trình nhập giao dịch và đã có trong định hướng tài liệu, nhưng đây mới là đề xuất, chưa phải quyết định của người dùng. Hãy thống nhất phạm vi và cách chạy (dịch vụ trực tuyến hay mô hình xử lý tại máy) trước khi triển khai tính năng đó.

### Ràng buộc khi phát triển

- Tài liệu trong `docs/` là tài liệu tham khảo do người dùng tự quản lý: **không sửa file nào trong `docs/`** nếu chưa được yêu cầu rõ.
- Một số tài liệu có thể còn tham chiếu tên database cũ `family_expenses.db` hoặc branch `chatgpt-work`; kiểm tra mã nguồn hiện hành và branch thực tế trước khi dựa vào các chi tiết này.
- Giữ nguyên cấu trúc App Factory + Blueprint, ưu tiên thay đổi nhỏ, không thêm dependency nếu chưa cần.
- Giao diện hiện phục vụ offline sau khi cài dependencies. Tích hợp AI qua API sẽ cần Internet, khóa API và cân nhắc quyền riêng tư dữ liệu tài chính; nếu có thể, gửi dữ liệu tối thiểu/tổng hợp.
- Raspberry Pi Zero 2 W có tài nguyên hạn chế. Đừng giả định mô hình AI lớn có thể chạy ổn định tại máy.
- AI chỉ nên tạo gợi ý/phân tích. Không tự động ghi hoặc sửa giao dịch, không để kết quả AI thay thế kiểm tra số tiền, ngày, danh mục và quyền hạn hiện có.
- Làm việc trong repo mới này theo branch `codex`; branch nguồn `codex` của repo `quan-li-chi-tieu-refresh` cần được giữ độc lập.

