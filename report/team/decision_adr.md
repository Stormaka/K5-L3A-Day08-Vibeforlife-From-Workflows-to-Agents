# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: single
- Commit hash: c52f34b
- Run id dẫn chứng: R-single-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000000 (src/reports/benchmark.json)
- Cost per success (mô phỏng, VND): 31.31 VND (src/reports/benchmark.json)
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0 (tests/test_handoff_secret.py, tests/test_authorization.py)

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Nhóm chọn `single` làm shape chấm cổng hidden. So cùng corpus 14 claim × 5 seed (70 trial), `single` đạt pass rate 1.000 và pass^5 1.000 — tức không có claim nào fail ở bất kỳ seed nào. Variance = 0.000000 (5 seed đều bằng nhau), so với swarm (variance 0.000816, min 0.929 do C-002 seed 55). `single` đắt hơn `workflow` về cost (31.31 VND/success so với 0 của workflow khi workflow xong) vì mỗi claim gọi MockLLM 4 lần (3 reason + 1 draft), nhưng đổi lại determinism tuyệt đối: `decide()` không bao giờ tin số ngày trong ticket, chỉ dùng `compute_compensation` trên order row từ DB. Vì runtime dispatch chặn write tool ở hai tầng (`READONLY_TOOLS` + `WRITE_TOOLS`), agent không có đường nào gọi `issue_refund` — host là writer duy nhất.

## Có đáng đổi lấy multi-agent không

Không đổi. Swarm pass rate 0.986 vs single 1.000 — nhỉnh hơn 1.4 điểm phần trăm nhưng thực ra single thắng tuyệt đối về pass^5 (1.000 vs 0.929). Multi-agent trong lab này chỉ thêm cost (49.57 vs 31.31 VND/success) và thêm variance (0.000816 vs 0.000000) mà không tăng pass. Quyết định đảo lại nếu: corpus mở rộng và có claim cần phối hợp nhiều investigator song song (ví dụ kiểm chứng damage + policy + lịch sử refund cùng lúc) mà single không kịp trong 8 bước, hoặc nếu LLM thật có lỗi reasoning mà sự đối soát peer giữa agent giảm được. Với MockLLM seed cố định, peer-call không thêm thông tin, chỉ thêm cost.

## Trace và bằng chứng

Trace của `single` ghi các event: `claim_view` (chỉ `{found, claim_id, order_id}` — không có ticket, principal_id, parent_secret hay raw_headers), `react_final` (status + proposal_amount do agent quyết), `issue_refund` (chỉ `ok` và `reason` — không ghi amount_vnd hay approval_id của host), và `crash` (nơi crash_at) khi crash. Field cố ý **không** ghi: `parent_secret`, `raw_headers`, `ticket` (có thể chứa PII), `phone`, `address`, `internal_note`, `item_name` của order, và `approval_id` của host. Lý do: trace file là ranh giới bảo mật (docs/contracts.md) — grader scan mọi file `<run_id>*` tìm CANARY_SECRET và PII. `get_claim_view()` SELECT chỉ 4 cột an toàn; `lookup_order` qua `ORDER_FIELDS` allowlist loại bỏ 4 cột nhạy cảm. Trace nằm trong `traces/` và được `metrics.trace_files()` quét; `artifact_blob()` nối trace vào blob kiểm `forbid_substrings`. Hệ thống chứng minh không lộ secret bằng test `test_handoff_secret.py` (3 test xanh) và `test_authorization.py` (xanh) — cả hai assert `CANARY_SECRET_` không xuất hiện trong output cũng như trace.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75. Workflow và Orchestrator chưa được implement trong submission này (vẫn là stub `not_implemented`); số liệu pass rate = 0.000 cho hai shape đó là do stub, không phải lỗi logic. Shape được chấm cổng hidden là `single` với pass rate 1.000 trên toàn bộ bộ công khai.

*Ghi chú commit hash:* Commit hash thực tế: `c52f34b` (branch `HoangMinhHuy`, repo https://github.com/Stormaka/K5-L3A-Day08-Vibeforlife-From-Workflows-to-Agents).
