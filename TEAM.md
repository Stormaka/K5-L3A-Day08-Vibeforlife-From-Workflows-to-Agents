# Team

## Thông tin nhóm

- Tên nhóm: Vibeforlife
- Lớp: L3A
- Repo nhóm: `K5-L3A-Day08-Vibeforlife-From-Workflows-to-Agents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo: https://github.com/Stormaka/K5-L3A-Day08-Vibeforlife-From-Workflows-to-Agents
- Nhánh nộp, commit chốt: (điền sau khi push lên GitHub)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Hoàng Minh Huy | 2A202603495 | minhhuy108205@gmail.com | Stormaka | Shape B – Single Agent (ReAct) |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Hoàng Minh Huy | Shape B – Single Agent (ReAct) | c52f34b | Triển khai `src/lab8/shapes/single.py`; tự kiểm bằng `pytest -k "single and not workflow and not orchestrator"` (122 passed) |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## MSSV_HoTen

### Phần việc phụ trách

- Shape B – Single Agent (ReAct): `src/lab8/shapes/single.py`

### Đóng góp cụ thể

- Triển khai toàn bộ `single.py`: vòng ReAct dùng `run_react(max_steps=8, worker=True)`, hàm `decide()` với 3 bước (lookup_order → compute_compensation → approved/denied), memo giữ order giữa bước, host là writer duy nhất gọi `issue_refund`.
- Truyền `crash_at` xuyên qua đến `durability.issue_refund`, không bắt `InjectedCrash`.
- Chạy và xác nhận 122 test single-shape xanh, crash suite 20/20, benchmark single pass rate 1.000.

### Bằng chứng

- Run id: `R-single-C-001-s11` (từ `python -m lab8.run --shape single --claim-id C-001 --seed 11`)
- Test: `pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator"` → 122 passed
- Crash: `python -m lab8.crash_test --count 20 --out reports/crash.json` → `crash_safety 20/20`
- Benchmark: `src/reports/benchmark.json`, shape `single`, pass_rate 1.000, pass^5 1.000, variance 0.000000
- Báo cáo cá nhân đầy đủ: `report/team/2A202603495_HoangMinhHuy.md`
