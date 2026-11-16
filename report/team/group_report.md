# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Vibeforlife
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): (điền sau khi commit)

## 2. Mục tiêu và cách tiếp cận

Lab 8 xây dựng hệ thống xử lý claim hoàn tiền (LogiX-VN) với bốn kiến trúc agent khác nhau. Bài toán: nhận `claim_id`, tra cứu order từ DB, tính compensation bằng policy hiện hành, và ghi refund nếu được phê duyệt — tất cả bảo đảm không lộ secret, không double-pay, và chịu được crash.

Nhóm (1 thành viên) phụ trách Shape B (single agent). Thứ tự: đọc architecture và contracts trước, hiểu `run_react`/`dispatch`/`issue_refund` API, rồi implement. Shape A (workflow) và C (orchestrator) chưa được implement trong submission này.

Shape D (swarm reference) được cung cấp sẵn và dùng làm baseline so sánh.

## 3. Giải pháp

**Shape B – Single Agent (ReAct)** (`src/lab8/shapes/single.py`):

Vòng ReAct chạy qua `run_react(conn, budget, observation={"claim": claim_view}, decide=decide, max_steps=8, worker=True)`. Hàm `decide()` nhận observation và:

1. **Bước 0** (initial): Nếu `order_id` là None → `final: needs_info`. Có order_id → gọi `lookup_order`.
2. **Bước 1** (after lookup_order): Nếu `visible=False` → `final: escalated`. Visible → lưu order vào `memo`, gọi `compute_compensation`.
3. **Bước 2** (after compute_compensation): `amount_vnd ≤ 0` → `final: denied`. Ngược lại → `final: approved, proposal_amount`.

Mỗi lần `decide()` được gọi, `complete(budget, purpose="reason", ...)` được gọi trước để đếm model cost công bằng. Sau loop, khi `approved`, host gọi thêm `purpose="draft"` và `durability.issue_refund(conn, claim_id, proposal, crash_at=crash_at)`.

**Ranh giới code/model**: `decide()` không tính tiền, không đọc ticket text, không gọi write tool — chỉ chọn tool kế tiếp. Tính compensation do `compute_compensation(conn, order)` thực hiện hoàn toàn trong code (hàm `quote()` trong `compensation.py`). Ghi tiền chỉ do host qua `issue_refund`.

**Handoff/sanitize không dùng** ở shape B (không có multi-agent). Thay vào đó `dispatch(worker=True)` enforce `READONLY_TOOLS` allowlist ở runtime — agent không có đường gọi `issue_refund`.

**Trace** ghi: `claim_view` (chỉ `{found, claim_id, order_id}`), `react_final`, `issue_refund` (chỉ `{ok, reason}`), `crash` nếu có. Field bị bỏ chủ động: ticket, parent_secret, raw_headers, phone, address, internal_note, item_name.

## 4. Kết quả

Bảng so sánh 4 shape từ `src/reports/benchmark.json` (14 claim × 5 seed = 70 trial mỗi shape):

| shape | pass rate | pass^5 | variance | seed min | seed max | cost/success (VND) | mean steps |
|---|---|---|---|---|---|---|---|
| workflow | 0.000 | 0.000 | 0.000000 | 0.000 | 0.000 | — | 1.00 |
| **single** | **1.000** | **1.000** | **0.000000** | **1.000** | **1.000** | **31.31** | **2.64** |
| orchestrator | 0.000 | 0.000 | 0.000000 | 0.000 | 0.000 | — | 1.00 |
| swarm | 0.986 | 0.929 | 0.000816 | 0.929 | 1.000 | 49.57 | 2.80 |

Workflow và orchestrator vẫn là stub (`not_implemented`) — chưa implement. Single đạt kết quả tốt nhất.

Crash safety: **20/20** (`src/reports/crash.json`). Secret leak / approval bypass: 0 (`test_handoff_secret.py`, `test_authorization.py`).

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

**Vấn đề 1 — memo mất giữa bước**: `run_react` ghi đè `seen` sau mỗi tool call thành `{last_tool, last_result, claim}`. Order data từ `lookup_order` biến mất trước khi `compute_compensation` được gọi. Giải pháp: dùng `memo` dict trong closure để lưu `order` độc lập với `seen`.

**Vấn đề 2 — `-k single` match sai tên test**: Flag `-k single` khớp với tên function `test_public_single_run_cases` nên chạy cả workflow và orchestrator (đang là stub). Cần dùng `-k "single and not workflow and not orchestrator"` để isolate đúng 62 test của shape B.

**Vấn đề 3 — proposal_amount cho denied cases**: Test T07 (approval expired) và T13 (order đã refund) expect `proposal_amount = 35000` dù kết quả là `denied`. Phải truyền full quote vào `finish()` ngay cả khi `issue_refund` trả `ok=False` — không reset về 0.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Cost là mô phỏng VND, không phải bill API thực. Workflow và orchestrator chưa implement nên không có so sánh đầy đủ 3 shape nhóm viết — chỉ có single. Nếu implement đủ, có thể so pass rate và variance giữa deterministic workflow (0 model call) vs ReAct single (4 model call) vs multi-agent orchestrator để kiểm chứng hypothesis R2 đầy đủ hơn.

## 8. Tài liệu tham khảo

- MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* (R2 research question)
- Google Research 2026, *Towards a Science of Scaling Agent Systems*
- AgentArch, AACL 2026
- EMNLP 2026, *Rethinking Evaluation of MAS Efficiency*
- Lab 8 docs: `docs/architecture.md`, `docs/contracts.md`, `docs/business_rules.md`
