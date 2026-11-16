# Báo cáo cá nhân — 2A202603436_PhamNguyenVu

## 1. Thông tin thành viên

- Họ tên: Phạm Nguyên Vũ
- MSSV: 2A202603436
- Email: phamvu65cs@gmail.com
- GitHub: `phamvu65`

## 2. Phần việc phụ trách

Shape B (`single`) trong `src/lab8/shapes/single.py`, gồm vòng ReAct, xử lý kết quả và kiểm thử hành vi hoàn tiền.

## 3. Đóng góp cụ thể

- Commit `22387b7` triển khai `run_claim`. Hàm `decide` mô phỏng chọn hành động theo quan sát từ tool, còn MockLLM ghi nhận chi phí từng lượt vào `Budget`. Vòng ReAct chạy tối đa 8 bước với các tool chỉ đọc.
- `compute_compensation` tính quote từ đơn hàng và chính sách hiện hành. Shape chuyển nguyên quote cho `issue_refund` để host kiểm tra approval, trần theo đơn và ghi ledger. `crash_at` được chuyển tới host để kiểm tra retry sau crash.
- Trace ghi tên tool và kết quả quyết định. Test `test_single_keeps_untrusted_ticket_out_of_trace` chèn chỉ dẫn và một canary vào ticket, sau đó kiểm tra số tiền, giới hạn gọi model và nội dung trace/stdout.

## 4. Bằng chứng

- Ma trận student riêng cho `single`: **62 passed**, gồm T11 (retry) và T12 (crash rồi chạy lại).
- Bộ test host: **55 passed**, gồm test mới trong `src/tests/test_single_agent.py`.
- Benchmark riêng `single` trên 14 claim public × 5 seed (`11,22,33,44,55`): **70/70** trial pass, `pass^5 = 1.0`, `seed_variance = 0`, chi phí mô phỏng **29,46 VND/lượt thành công**. Run ID ví dụ: `R-single-C-014-s11`.

Để tái hiện kết quả trên PowerShell, chạy các lệnh sau trong `src` với môi trường `.venv` đã kích hoạt. Mỗi lần chạy pytest dùng một thư mục tạm mới trong `.pytest_cache` để tránh lỗi quyền truy cập thư mục tạm mặc định.

```powershell
$studentTemp = Join-Path '.pytest_cache' ('student-single-' + [guid]::NewGuid().ToString('N'))
python -m pytest -q -m student -o 'addopts=' -k 'single and not (workflow or orchestrator or swarm)' --basetemp $studentTemp

$hostTemp = Join-Path '.pytest_cache' ('host-single-' + [guid]::NewGuid().ToString('N'))
python -m pytest -q --basetemp $hostTemp

python -m lab8.benchmark --shapes single --cases data/public_claims.jsonl --seeds '11,22,33,44,55' --out .pytest_cache/single-benchmark.json
```

## 5. Công cụ AI đã dùng

Codex hỗ trợ triển khai và kiểm thử Shape B (single).

## 6. Tự đánh giá

Điểm quan trọng của `single` là tách bước điều tra khỏi quyền ghi giao dịch. Vòng ReAct có thể đọc dữ liệu để lấy quote, còn host kiểm tra approval và ghi ledger. T14 cho thấy cần chuyển quote đầy đủ **150.000 VND** để host trừ **30.000 VND** đã hoàn và ghi đúng **120.000 VND**. T12 kiểm tra trường hợp crash ngay sau khi ghi giao dịch, khi lần chạy lại phải giữ nguyên khoản đã ghi.

Kết quả 70/70 cho thấy `single` ổn định trên 14 claim public và 5 seed đã chạy. `decide` chọn hành động bằng logic mô phỏng dựa trên kết quả tool, còn MockLLM ghi nhận số lượt gọi và chi phí mô phỏng. Kết quả vì vậy giới hạn ở bộ public và cách mô phỏng này. Khả năng tổng quát trên bộ hidden và hiệu năng với model thật vẫn cần được đánh giá riêng.
