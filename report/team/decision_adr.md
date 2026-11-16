# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: db1b020
- Run id dẫn chứng: R-workflow-C-001-s11 (src/reports/benchmark.json)
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000-1.000 / 0.000000
- Cost per success (mô phỏng, VND): 28.79
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0 (78 host tests + 248 student tests passed)

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

So trên cùng tập 14 claim public × 5 seed (`11,22,33,44,55`) với cùng MockLLM và bảng giá mô phỏng, `workflow` đạt pass rate `1.000`, `pass^5 = 1.000`, `variance = 0.000000`, và `cost/success = 28.79` VND (thấp nhất trong 4 shape, với `mean_steps = 1.86`). Bài toán xử lý claim LogiX-VN có luồng nghiệp vụ xác định (đọc claim, kiểm tra `order_id`, tra cứu order, tính tiền bằng `compute_compensation`, gọi `durability.issue_refund`), nên chọn kiến trúc ít tự chủ nhất (`workflow`) vừa đủ giải quyết triệt để vừa tránh dao động giữa các seed.

## Có đáng đổi lấy multi-agent không

Không đáng đổi trong bài toán này. `orchestrator` đạt cùng pass rate `1.000` nhưng `cost/success` tăng gần gấp đôi (`57.10` so với `28.79` VND), còn `swarm` có chi phí cao hơn (`49.57` VND) và pass rate giảm xuống `0.986` (`pass^5 = 0.929`, `variance = 0.000816`) do lỗi tin vào số ngày trễ trong ticket ở `seed % 5 == 0`. Quyết định chỉ đảo lại nếu dữ liệu thực tế có nhiều nhánh điều tra không cấu trúc trước được (ví dụ phải đọc nhiều nguồn ngoài DB) khiến workflow cố định không đạt ngưỡng pass rate ≥ 0.75.

## Trace và bằng chứng

Trace ghi các event theo allowlist: `lookup_order` (chỉ `visible`, `reason`), `quote`, `decision`, `error`, `crash` thông qua `lab8.trace.Trace` và `lab8.shapes.common.finish()`. Nhóm cố ý **không** ghi claim row thô (`parent_secret`, `CANARY_SECRET_*`), nội dung `ticket` thô hay thông tin cá nhân trên order row (SĐT, địa chỉ, tên hàng, ghi chú nội bộ) vào trace hoặc `stdout`. Để chứng minh không lộ secret kể cả với file phụ, mọi log đều nằm cùng thư mục trace với tiền tố `<run_id>` để `check_trace` / `artifact_blob` quét fail-closed.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
