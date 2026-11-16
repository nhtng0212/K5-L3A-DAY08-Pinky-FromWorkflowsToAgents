# Team

## Thông tin nhóm

- Tên nhóm: Pinky
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-Pinky-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/nhtng0212/K5-L3A-DAY08-Pinky-FromWorkflowsToAgents, main, (điền khi chốt)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Nguyễn Hoàng Tùng | 2A202603481 | nguyentungpcbv@gmail.com | nhtng0212 | workflow + integration (Trưởng nhóm) |
|   2 | Phạm Nguyên Vũ | 2A202603436 | phamvu65cs@gmail.com | phamvu65 | single |
|   3 | Phạm Việt Hưng | 2A202603370 | hungphamm1507@gmail.com | rachel157 | orchestrator + handoff |
|   4 | Chu Mai Hồng | 2A202603377 | chumaihongabc@gamil.com | maihong1208 | ledger + evaluation |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Hoàng Tùng | workflow + integration | | |
| Phạm Việt Hưng | orchestrator + handoff | dfcf899, 4ce6c4c, d678215 | src/tests/test_orchestrator_handoff.py (21 tests; 62 student orchestrator tests) |
| Phạm Nguyên Vũ | single | `22387b7` | `test_single_keeps_untrusted_ticket_out_of_trace` |
| Chu Mai Hồng | ledger + evaluation | | |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-

## 2A202603370_PhamVietHung

### Phần việc phụ trách

- Người 3 — Shape C (`orchestrator` + Handoff): `src/lab8/shapes/orchestrator.py` và `src/tests/test_orchestrator_handoff.py`.

### Đóng góp cụ thể

- `dfcf899`: xây dựng `DEFAULT_INVESTIGATOR_TOOLS` và `build_handoff` dùng `lab8.handoff.make_handoff` lọc theo allowlist, chặn write tool và secret.
- `4ce6c4c`: triển khai `run_claim` / `_run` tách vai Orchestrator & read-only Investigator (`run_readonly_investigators`), tính quote bằng `lab8.compensation.quote`, truyền quote đầy đủ và `crash_at` vào `lab8.durability.issue_refund`, không nuốt `InjectedCrash`, ghi Trace Contract v1 sạch secret/PII trên mọi nhánh.
- `d678215`: bổ sung test T11 idempotency và T12 crash/resume cho cả 3 điểm crash.

### Bằng chứng

- Các commit trên nhánh `feat/shape-c-orchestrator`: `dfcf899`, `4ce6c4c`, `d678215`.
- Báo cáo cá nhân: `report/team/2A202603370_PhamVietHung.md`.
- `python -m lab8.smoke` → `lab8.smoke pass`; `python -m pytest -q` → `75 passed, 248 deselected`; `python -m pytest -q -m student -o addopts= -k orchestrator` → `62 passed, 261 deselected`.
