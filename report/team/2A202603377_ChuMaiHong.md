# Báo cáo cá nhân — Chu Mai Hồng

## 1. Thông tin thành viên

- Họ tên: Chu Mai Hồng
- MSSV: 2A202603377
- Email: chumaihongabc@gmail.com
- GitHub: maihong1208

## 2. Phần việc phụ trách

Phụ trách ledger/durability và evaluation: kiểm tra idempotency, crash-resume, trần hoàn theo đơn, rồi chạy và tổng hợp benchmark/crash. Không sửa các module host bị cấm (`durability.py`, `policy.py`, `metrics.py`, cấu hình hoặc dữ liệu).

## 3. Đóng góp cụ thể

- Bổ sung `test_p95_steps_uses_nearest_rank_for_twenty_trials` trong `src/tests/test_metrics.py` để xác nhận p95 95th-percentile theo nearest-rank trên 20 trial.
- Chạy benchmark public 4 shape × 14 claim × 5 seed và crash suite 20 điểm; JSON trong `src/reports/` được công cụ lab sinh ra, không chỉnh tay.
- Commit chính: sẽ điền hash commit cục bộ trước khi push.

## 4. Bằng chứng

- `python -m pytest -q tests/test_metrics.py`: 2 passed.
- `python -m pytest -q`: 79 passed, 248 deselected.
- `python -m pytest -q -m student -o addopts=`: 248 passed, 79 deselected; bao gồm T11 retry, T12 crash/resume và T13/T14 trần hoàn theo đơn trên các shape.
- `python -m lab8.crash_test --count 20 --out reports/crash.json`: crash safety 20/20.
- `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`: 4 shape × 14 claim × 5 seed. Run ID dẫn chứng: `R-workflow-C-001-s11`.
- Kết quả: workflow `1.000 / 1.000`, variance `0`, cost/success `28.79` VND; single `1.000 / 1.000`, `0`, `29.46` VND; orchestrator `1.000 / 1.000`, `0`, `57.10` VND; swarm `0.9857 / 0.9286`, variance `0.0008163`, `49.57` VND. Mỗi cặp số là pass rate / pass^5.
- `python -m lab8.check_submission`: 0 FAIL, 1 WARN (chưa có commit cho các thay đổi cục bộ).

## 5. Công cụ AI đã dùng

- Copilot trong VS Code: đối chiếu hướng dẫn lab, rà yêu cầu metric và hỗ trợ đề xuất test p95.
- Tự kiểm tra bằng pytest, crash suite, benchmark và submission checker; không tự tạo hoặc sửa tay số liệu.

## 6. Tự đánh giá

Test metric mới, toàn bộ 248 student tests, bộ test host và crash suite đều đạt. Benchmark xác nhận workflow đạt ngưỡng public; swarm có một trial fail như điểm yếu tham chiếu đã định trước. Các kết quả dùng chung corpus, seed và bảng giá, không suy rộng sang model sản xuất.
