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
| CÂU | R1 / R2 / R3 / R4 | R2 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?*, benchmark trên nhiều hệ thống MAS |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Lỗi phổ biến nhất trong MAS là lỗi xác minh (verification failure): agent chấp nhận đầu vào từ peer mà không kiểm tra lại bằng nguồn gốc (ground truth) |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Kết luận dựa trên các hệ thống MAS dùng LLM thật; MockLLM là seeded deterministic, không có hallucination thực sự — kết quả lab là minh chứng kiến trúc, không là benchmark LLM |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Nếu swarm bật cơ chế peer-trust (seed % 5 == 0 tin delay từ ticket thay vì DB), pass rate sẽ giảm xuống ~0.929 (mất 1 trong 14 claim trên seed 55) vì peer-ack không kiểm chứng lại order row |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ cố định: MockLLM, tools (lookup_order, compute_compensation), corpus 14 claim, seeds [11,22,33,44,55], giá pricing.yaml, evaluator trial_pass(). Biến thay đổi duy nhất: flag `seed % 5 == 0` trong swarm_reference.py dòng 109 bật/tắt trust-ticket path |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: C-002 seed 55 trả `resolved` amount 20000 thay vì `denied` (ticket nói 5 ngày, DB chỉ 2 ngày). Đo bằng pass rate (giảm từ 1.000 xuống 0.929 trên seed 55) và pass^5 (giảm từ 1.000 xuống 0.929) trong benchmark.json |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m pytest -q -m student -o addopts= -k "swarm" 2>&1`
- Run id / file report: `src/reports/benchmark.json` (swarm seed_pass_rates), `R-swarm-C-002-s55`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **ĐÚNG**. Swarm pass rate = 0.986, pass^5 = 0.929, seed_pass_rates cho seed 55 = 0.929 (so với 1.000 ở các seed khác). Claim C-002 seed 55 trả `resolved` (amount 20000) thay vì `denied` như label. Test `test_public_matrix.py` assert miss này (`SWARM_EXPECTED_MISSES = {("C-002", 55)}`). So sánh: `single` cùng claim, cùng seed, pass 100% vì `decide()` chỉ đọc order row qua `compute_compensation`, không tin ticket text. Đây là minh họa cụ thể của MAST R2 verification failure: peer-ack trong swarm thay thế cho kiểm chứng DB.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`. Kết quả phản ánh kiến trúc (trust-ticket path), không phải lỗi LLM thật.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
