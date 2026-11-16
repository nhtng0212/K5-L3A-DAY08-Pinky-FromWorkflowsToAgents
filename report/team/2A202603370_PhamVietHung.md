# Báo cáo cá nhân — 2A202603370_PhamVietHung

## 1. Thông tin thành viên

- Họ tên: Phạm Việt Hưng
- MSSV: 2A202603370
- Email: hungphamm1507@gmail.com
- GitHub: rachel157

## 2. Phần việc phụ trách

- Phụ trách **Người 3 — Shape C (`orchestrator` + Handoff)** trong hệ thống xử lý claim LogiX-VN:
  - Triển khai [src/lab8/shapes/orchestrator.py](../../src/lab8/shapes/orchestrator.py) với kiến trúc tách 2 vai Orchestrator và Investigator chỉ đọc.
  - Bổ sung bộ test kiểm thử ranh giới allowlist handoff, bảo mật trace không lộ secret/PII, chống prompt injection, idempotency và crash-recovery tại [src/tests/test_orchestrator_handoff.py](../../src/tests/test_orchestrator_handoff.py).

## 3. Đóng góp cụ thể

- **`dfcf899` — `feat(orchestrator): add allowlisted handoff builder and boundary tests`**
  - Định nghĩa `DEFAULT_INVESTIGATOR_TOOLS = ("lookup_order", "lookup_policy", "lookup_refund_history", "compute_compensation")` đảm bảo toàn bộ tool chỉ đọc và `lookup_order` đứng trước `compute_compensation` theo yêu cầu của `lab8.runtime.run_readonly_investigators`.
  - Xây dựng `build_handoff(...)` gọi `lab8.handoff.make_handoff` để lọc `facts` qua allowlist (`FACT_KEYS`), chặn `parent_secret`, `raw_headers`, `CANARY_SECRET_*` và các write tool (`issue_refund`, `append_audit`).
- **`4ce6c4c` — `feat(orchestrator): implement Shape C orchestrator with read-only investigators`**
  - Triển khai `run_claim(claim_id, *, seed, store_path, crash_at=None) -> RunResult` và `_run(...)` với vòng đời `connect()`/`close()` an toàn trong `try/finally`.
  - Orchestrator đọc `get_claim_view` (không `SELECT *` toàn bộ `claims`), xử lý sớm `unknown_claim` (`failed`) và thiếu `order_id` (`needs_info`), gọi `MockLLM` (`complete`) ở vai trò lập kế hoạch và soạn câu trả lời nhưng **không** tin dữ liệu tự khai trong `ticket` (bỏ qua delay giả, mã đơn giả, clause cũ hay prompt injection).
  - Giao việc qua `build_handoff` và thực thi bằng `lab8.runtime.run_readonly_investigators`; nếu đơn không thuộc principal/không tồn tại thì trả `escalated` không lộ PII (`phone`, `address`, `internal_note`, `item_name`).
  - Tính quote bằng code (`lab8.compensation.quote`) trên dữ liệu order/policy hợp lệ; nếu `proposal <= 0` trả `denied` không gọi ledger; nếu `proposal > 0` truyền **quote đầy đủ** vào `lab8.durability.issue_refund(..., crash_at=crash_at)` (không tự trừ phần đã hoàn trước đó) và không bắt/nuốt `InjectedCrash`.
  - Ghi Trace Contract v1 (`lab8.trace.Trace`) trên mọi nhánh return (bao gồm early return và nhánh crash) với chỉ dữ liệu đã qua allowlist (`safe_dump(handoff)`, `ORDER_FIELDS`, `policy_id`).
- **`d678215` — `test(orchestrator): add T11 idempotency and crash-resume tests`**
  - Bổ sung kiểm thử T11 (gọi lại cùng claim giữ đúng 1 dòng `refund_ledger`) và T12 cho cả 3 điểm crash (`before_effect`, `after_effect_before_checkpoint`, `after_checkpoint`) đảm bảo `InjectedCrash` được truyền ra và resume không ghi trùng refund.

## 4. Bằng chứng

- Các commit trên nhánh `feat/shape-c-orchestrator`: `dfcf899`, `4ce6c4c`, `d678215`.
- Bộ test tự thêm: `src/tests/test_orchestrator_handoff.py` (21 tests: handoff allowlist/rejection, 12 single-run claims C-001..C-010/C-013/C-014, trace hygiene, unknown claim, ticket prompt injection, T11 retry idempotency, 3 điểm crash T12).
- Kết quả chạy kiểm chứng trong thư mục `src/`:
  - `python -m lab8.smoke` → `lab8.smoke pass`
  - `python -m pytest -q` → `75 passed, 248 deselected` (bao gồm 54 test host sẵn có + 21 test mới cho orchestrator/handoff)
  - `python -m pytest -q -m student -o addopts= -k orchestrator` → `62 passed, 261 deselected` (pass toàn bộ T01–T14 trên 5 seed `11,22,33,44,55` + T11 + T12)

## 5. Công cụ AI đã dùng

- Công cụ: Antigravity AI Assistant.
- Mục đích sử dụng: phân tích kiến trúc repo và contract grader, lập kế hoạch triển khai theo TDD, gợi ý code cho `orchestrator.py` và bộ test `test_orchestrator_handoff.py`, rà soát ranh giới bảo mật/durability.
- Cách tự kiểm tra kết quả: đối chiếu trực tiếp diff với [docs/contracts.md](../../docs/contracts.md), [docs/business_rules.md](../../docs/business_rules.md), [RULES.md](../../RULES.md); chạy thực tế `lab8.smoke`, `python -m pytest -q`, và `python -m pytest -q -m student -o addopts= -k orchestrator` để xác nhận 100% pass và không sửa bất kỳ file thước đo nào (`metrics.py`, `durability.py`, `policy.py`, `config/*.yaml`, `data/*`).

## 6. Tự đánh giá

- Hoàn thành đầy đủ 4 đầu việc của Người 3 cho Shape C (`orchestrator` + Handoff).
- Đáp ứng cả 3 điều kiện must-pass trên Shape C: không refund khi thiếu approval hợp lệ, không lộ `CANARY_SECRET_*`/PII trong handoff/trace/kết quả, và crash/retry chỉ tạo tối đa 1 refund.
- Chưa chạy `lab8.benchmark` đầy đủ 4 shape và `lab8.check_submission` ở bước này vì còn phụ thuộc Shape A (`workflow`), Shape B (`single`) và phần tổng hợp báo cáo nhóm của các thành viên khác.
