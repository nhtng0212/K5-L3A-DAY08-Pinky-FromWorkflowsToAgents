# Báo cáo cá nhân — 2A202603481_NguyenHoangTung

## 1. Thông tin thành viên

- Họ tên: Nguyễn Hoàng Tùng
- MSSV: 2A202603481
- Email: nguyentungpcbv@gmail.com
- GitHub: nhtng0212

## 2. Phần việc phụ trách

- Trưởng nhóm: khởi tạo repo nhóm, phân công vai trò trong `TEAM.md`, điều phối và ghép nhánh (integration).
- Phụ trách Shape A (`src/lab8/shapes/workflow.py`) và bổ sung unit test cho `workflow` (`src/tests/test_workflow_extra.py`).

## 3. Đóng góp cụ thể

- Hoàn thiện `run_claim` cho Shape A (`src/lab8/shapes/workflow.py`): đọc claim qua `get_claim_view`, kiểm tra `order_id` (`needs_info`), tra cứu đơn qua `lookup_order` (`escalated`), tính tiền hoàn bằng `compute_compensation` (không lấy số liệu từ ticket), gọi `durability.issue_refund` với quote đầy đủ và truyền `crash_at` đúng chuẩn không nuốt `InjectedCrash`, ghi trace đúng Trace contract v1 qua `finish()`.
- Thêm 2 unit test trong `src/tests/test_workflow_extra.py` kiểm tra nhánh `unknown_claim` và kiểm tra trace không rò rỉ `CANARY_SECRET`.
- Commit chính: (cập nhật short hash sau khi commit).

## 4. Bằng chứng

- File code: `src/lab8/shapes/workflow.py`
- File test: `src/tests/test_workflow_extra.py` (`test_workflow_unknown_claim_fails_safely_with_valid_trace`, `test_workflow_trace_has_no_canary_secret`)
- Kết quả tự kiểm tra:
  - `python -m pytest -q -m student -o addopts= -k workflow`: `62 passed, 242 deselected`
  - `python -m pytest -q`: `56 passed, 248 deselected`

## 5. Công cụ AI đã dùng

- Sử dụng trợ lý AI (Antigravity) để giải thích yêu cầu bài lab, rà soát luật nghiệp vụ (`docs/business_rules.md`, `docs/contracts.md`), gợi ý khung code `workflow.py` và viết test mẫu `test_workflow_extra.py`.
- Cách tự kiểm tra: đọc hiểu từng bước xử lý (ranh giới code/model, full quote, idempotency/crash, allowlist trace) và chạy trực tiếp bộ test `pytest` trên máy để xác nhận tất cả test đều pass.

## 6. Tự đánh giá

- Hoàn thành đúng phần `workflow` và test bổ sung theo phân công; sẵn sàng hỗ trợ ghép nhánh và kiểm tra tổng thể cùng cả nhóm.
