# Yêu cầu chức năng mới — Quản lý linh hoạt danh sách chờ (waiting list)

> Đây là **đầu vào thô** của buổi đầu tiên. Chưa phải đặc tả: còn mơ hồ và thiếu có chủ ý, việc của bạn (cùng Spec-Kit) là làm rõ.

## Bối cảnh

MedBook là nền tảng web quản lý lịch khám bệnh viện, đã chạy được (xem `boi-canh-chung/`).

Phía bệnh nhân đã có: tìm bác sĩ theo tên/chuyên khoa; xem lịch làm việc và slot còn trống; đặt lịch khám (trực tiếp hoặc online); hủy lịch hẹn.

Phía nhân viên đã có: quản lý lịch làm việc (thêm, điều chỉnh slot); xem lịch hẹn theo ngày; xác nhận hoặc hủy lịch hẹn.

## Vấn đề

Hệ thống chưa có cơ chế nào xử lý khi một slot khám bị trống đột ngột (ví dụ bệnh nhân hủy lịch). Nhân viên đang xử lý thủ công hoàn toàn: gọi điện hoặc nhắn tin cho từng bệnh nhân trong danh sách chờ.

## Yêu cầu mới

Bệnh viện muốn phát triển tính năng **Quản lý linh hoạt danh sách chờ**:

- Khi một slot khám trở nên khả dụng, hệ thống tự động xác định bệnh nhân phù hợp nhất trong danh sách chờ.
- Tiêu chí chọn: mức độ ưu tiên y tế, thời gian chờ, sự phù hợp chuyên khoa / bác sĩ.
- Hệ thống gửi đề xuất cho bệnh nhân → bệnh nhân phản hồi trong X phút.
- Nếu từ chối hoặc không phản hồi → chuyển sang bệnh nhân tiếp theo.
- Xử lý xung đột lịch, thông báo đầy đủ cho tất cả các bên.
