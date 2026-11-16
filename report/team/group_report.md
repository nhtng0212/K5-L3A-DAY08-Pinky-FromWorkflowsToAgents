# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Pinky
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): e3a22b6

## 2. Mục tiêu và cách tiếp cận

Bài toán yêu cầu xử lý claim hoàn phí vận chuyển cho công ty giả lập LogiX-VN bằng 4 kiến trúc (shape) có mức độ tự chủ tăng dần:
1. Shape A (`workflow`): Quy trình code tuần tự, xác định.
2. Shape B (`single`): 1 Agent chạy vòng lặp ReAct tối đa 8 bước.
3. Shape C (`orchestrator`): Mô hình Orchestrator điều phối các Investigator chỉ đọc.
4. Shape D (`swarm`): Bầy agent ngang hàng tự trao đổi (shape đối chứng có sẵn).

Mục tiêu cốt lõi là đo lường thực nghiệm (pass rate, độ ổn định qua các seed, chi phí mô phỏng VND, độ bền khi crash) để chọn ra kiến trúc **ít tự chủ nhất mà vẫn giải quyết triệt để bài toán**, thay vì mặc định chọn multi-agent phức tạp.

Nhóm phân công 4 thành viên độc lập theo đúng cấu trúc bài:
- Nguyễn Hoàng Tùng (Trưởng nhóm): Shape A (`workflow`) + integration & quản lý repo.
- Phạm Nguyên Vũ: Shape B (`single` agent).
- Phạm Việt Hưng: Shape C (`orchestrator` + handoff).
- Chu Mai Hồng: Kiểm tra durability/crash + evaluation & báo cáo.

Thứ tự thực hiện: Cài đặt và kiểm tra cổng host (CP1) -> Hoàn thiện 3 shape theo TDD (CP2) -> Rà soát durability, crash safety và allowlist ranh giới bảo mật (CP3, CP4) -> Chạy benchmark 4 shape × 5 seed và đối chiếu quyết định kiến trúc (CP5).

## 3. Giải pháp

### 3.1. Ranh giới Code và Model
Ở cả ba shape nhóm tự triển khai:
- **Tiền do code tính, không do MockLLM quyết định:** Số tiền hoàn được tính bằng hàm nghiệp vụ `compute_compensation` dựa trên dữ liệu order và policy hiện hành. MockLLM chỉ đóng vai trò phân tích ticket hoặc soạn văn bản trả lời.
- **Cửa kiểm soát Host:** Shape không ghi trực tiếp vào SQLite mà bắt buộc gọi qua `durability.issue_refund(conn, claim_id, proposal, crash_at=crash_at)`. Host kiểm tra approval hợp lệ, trần hoàn theo đơn và idempotency trước khi ghi ledger.
- **Nguyên tắc Quote đầy đủ:** Shape luôn truyền nguyên vẹn `proposal` (full quote) vào `issue_refund`, để Host tự trừ các khoản hoàn trước đó của cùng một đơn hàng (xử lý case T13, T14). Shape tự trừ sẽ dẫn đến lỗi trừ hai lần.

### 3.2. Ranh giới bảo mật và Handoff
- **Chống Prompt Injection:** Dữ liệu trong ticket của khách chỉ được coi là dữ liệu thô, không được tin cậy để ghi đè số ngày trễ hay clause chính sách từ database.
- **Handoff qua Allowlist:** Trong Shape C (`orchestrator`), dữ liệu chuyển sang investigator được lọc nghiêm ngặt qua `lab8.handoff.make_handoff` với allowlist facts. Tuyệt đối không chuyển `parent_secret`, raw headers hay toàn bộ dòng claim vào context của agent con.
- **Công cụ chỉ đọc:** Investigator chỉ được cấp quyền sử dụng các tool chỉ đọc (`lookup_order`, `lookup_policy`, `lookup_refund_history`, `compute_compensation`), không có bất kỳ tool ghi hay quyền gọi ledger nào.

### 3.3. Trace Contract v1
- Mọi nhánh kết thúc (kể cả `needs_info`, `escalated`, `denied`, `failed`, `crash`) đều sinh file trace JSON đúng `run_id` thông qua `lab8.shapes.common.finish()`.
- Trace chỉ ghi các trường an toàn (`visible`, `reason`, `amount_vnd`), loại bỏ hoàn toàn thông tin cá nhân (SĐT, địa chỉ, tên hàng hóa) và secret (`CANARY_SECRET_*`).

## 4. Kết quả

### 4.1. Bảng so sánh 4 kiến trúc trên tập Public
Số liệu trích xuất trực tiếp từ `src/reports/benchmark.json` (chạy trên 14 claim public × 5 seed: 11, 22, 33, 44, 55; tổng cộng 70 trial/shape):

| Shape | Pass rate | Pass^5 | Seed min-max | Seed variance | Cost/success (VND) | Mean steps | P95 steps |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **workflow** | **1.000** | **1.000** | 1.000 - 1.000 | 0.000000 | **28.79** | **1.86** | 2.0 |
| **single** | **1.000** | **1.000** | 1.000 - 1.000 | 0.000000 | 29.46 | 5.29 | 6.0 |
| **orchestrator** | **1.000** | **1.000** | 1.000 - 1.000 | 0.000000 | 57.10 | 3.43 | 4.0 |
| **swarm** | 0.986 | 0.929 | 0.929 - 1.000 | 0.000816 | 49.57 | 2.80 | 3.0 |

*Ghi chú:* Swarm gặp 1 lỗi `status resolved != denied` ở seed 55 do điểm yếu tham chiếu (tin vào số ngày trễ ghi trong ticket thay vì dữ liệu đơn hàng trong DB).

### 4.2. Độ an toàn khi sập nguồn (Crash Safety)
- Kết quả chạy `lab8.crash_test --count 20 --out reports/crash.json`: Đạt **20/20** (`crash_safety: 20/20`).
- Cả 3 shape đều xử lý chính xác `InjectedCrash` mà không nuốt lỗi, bảo đảm khi resume lại thì idempotency key `refund:{claim_id}` ngăn chặn hoàn toàn việc ghi trùng refund thứ hai vào ledger.

### 4.3. Tự kiểm tra trên tập ẩn (`student_drop/`)
- Nhóm đã chạy tự kiểm trên 40 claim không nhãn × 5 seed (200 trial/shape) lưu tại `src/reports/hidden_selfcheck.json`.
- Cả 3 shape đều chạy thông suốt 200 trial không phát sinh exception, format trace hợp lệ và **`leak_suspect: None`** (không rò rỉ secret).

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

1. **Xử lý trần hoàn theo đơn (Per-order Cap - T13, T14):**
   - *Vấn đề:* Ban đầu dễ nhầm lẫn việc tự trừ số tiền đã hoàn trước đó trong code của shape trước khi gửi sang Host.
   - *Xử lý:* Đọc kỹ contract Host và thống nhất chỉ gửi **quote đầy đủ** (`proposal`) vào `issue_refund`, để Host tự trừ và lưu ledger đúng phần chênh lệch (T14: quote 150k, đã hoàn 30k -> ghi ledger 120k, `proposal_amount` vẫn là 150k).
2. **Crash Injection & Trace ranh giới:**
   - *Vấn đề:* Khối `try/except` bắt `InjectedCrash` nếu không cẩn thận sẽ nuốt mất exception khiến test T12 đánh giá sai.
   - *Xử lý:* Cả 3 shape khi bắt `InjectedCrash` đều ghi sự kiện `crash` vào trace, gọi `trace.write()` rồi lập tức `raise` lại exception ra ngoài.
3. **Đồng bộ hóa Git khi 4 thành viên cùng làm:**
   - Nhóm chia tách rõ ranh giới các file `workflow.py`, `single.py`, `orchestrator.py` và test riêng trong `src/tests/`, giúp hạn chế xung đột merge.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.
- **Nguyễn Hoàng Tùng:** Phụ trách Shape A (`workflow.py`), test an toàn trace `test_workflow_extra.py`, đóng vai trò Integration ghép nối nhánh và điều phối nộp bài.
- **Phạm Nguyên Vũ:** Phụ trách Shape B (`single.py`), thiết kế ReAct 8 bước, test `test_single_keeps_untrusted_ticket_out_of_trace`.
- **Phạm Việt Hưng:** Phụ trách Shape C (`orchestrator.py`), xây dựng builder allowlist handoff, viết bộ 21 test trong `test_orchestrator_handoff.py`.
- **Chu Mai Hồng:** Phụ trách ledger durability và evaluation, bổ sung test `test_p95_steps_uses_nearest_rank_for_twenty_trials`, rà soát benchmark và crash suite.

## 7. Hạn chế và hướng cải thiện

- **Giới hạn môi trường giả lập:** Mô hình sử dụng là `MockLLM`, chưa phản ánh độ trễ thực tế (`latency: unavailable`) và độ bất định của LLM sản xuất khi gặp ngôn ngữ tự nhiên phức tạp hoặc văn bản prompt injection tinh vi.
- **Điều kiện quyết định kiến trúc thay đổi:** Trong bài lab, `workflow` tối ưu nhất vì bài toán có luồng nghiệp vụ rõ ràng, dữ liệu cấu trúc sẵn trong SQLite. Tuy nhiên, nếu nghiệp vụ mở rộng sang các dạng bán cấu trúc (đọc hóa đơn scan, tra cứu API đối tác bên thứ ba có schema thay đổi, hoặc phân loại khiếu nại mơ hồ), kiến trúc `single` hoặc `orchestrator` sẽ cần thiết để xử lý tính linh hoạt, chấp nhận chi phí cao hơn.

## 8. Tài liệu tham khảo

1. Tài liệu môn học Lab 8 — LogiX-VN (`docs/business_rules.md`, `docs/contracts.md`, `docs/architecture.md`, `docs/metrics.md`).
2. Google Research (2026), *Towards a Science of Scaling Agent Systems*.
3. MAST, NeurIPS (2025), *Why Do Multi-Agent LLM Systems Fail?*
4. AgentArch, AACL (2026).
