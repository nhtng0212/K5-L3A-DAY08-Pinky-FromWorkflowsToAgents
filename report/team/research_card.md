# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R1 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Google Research 2026, *Towards a Science of Scaling Agent Systems* (slide 68–69, benchmark/empirical study) |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Với các tác vụ có quy trình tuần tự xác định, việc thêm nhiều agent hoặc tầng điều phối làm tăng chi phí gọi model/tool mà không tự động cải thiện độ chính xác |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Áp dụng trong phạm vi đánh giá trên MockLLM, 14 public claims × 5 seed (`11,22,33,44,55`), bảng giá mô phỏng VND và môi trường SQLite cô lập từng trial |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Khi chuyển từ `workflow` (A) sang `orchestrator` (C) hoặc `swarm` (D) trên cùng corpus và seed, `cost/success` sẽ tăng mạnh (từ ~29 VND lên ~50–57 VND) và `steps` tăng, trong khi `pass_rate` không cao hơn `1.000` |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ cố định: MockLLM, bộ tool read-only + host `issue_refund`, 5 seed (`11,22,33,44,55`), `data/public_claims.jsonl`, bảng giá VND, evaluator `metrics.py`. Biến duy nhất thay đổi: `shape` (`workflow` vs `single` vs `orchestrator` vs `swarm`) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: `swarm` lệch `status resolved != denied` khi tin số ngày trễ trong ticket ở `seed % 5 == 0`. Đo bằng `pass_rate`, `pass^5`, `seed_variance`, `cost_per_success_vnd`, `mean_steps` |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `R-workflow-C-001-s11` / `src/reports/benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **Đúng với hypothesis.** `workflow` đạt `pass=1.000`, `pass^5=1.000`, `var=0.000000`, `cost/success=28.79` VND, `steps=1.86`. `single` giữ `pass=1.000` nhưng `cost/success=29.46`, `steps=5.29`. `orchestrator` giữ `pass=1.000` nhưng `cost/success=57.10` VND (gần gấp đôi), `steps=3.43`. `swarm` có `cost/success=49.57` VND và `pass=0.986`, `pass^5=0.929`, `var=0.000816` (1 lỗi `status resolved != denied`).
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
