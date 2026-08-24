# Evidence — Day 22: LangSmith + Prompt Versioning

## Danh sách tệp

| Tệp | Nội dung |
|---|---|
| `01_langsmith_traces.png` | Ảnh chụp LangSmith dashboard, project `day22-nguyenviethung` |
| `02_prompt_hub.png` | Ảnh chụp Prompt Hub với 2 phiên bản prompt |
| `02_ab_routing_log.txt` | Log console Bước 2 — 50 câu hỏi, routing V1=19 / V2=31 |
| `03_ragas_scores.png` | Ảnh chụp terminal — bảng so sánh điểm RAGAS V1 vs V2 |
| `03_ragas_report.json` | Bản sao `data/ragas_report.json` |
| `04_pii_demo_log.txt` | Log console demo PII Detector — 6 test case |
| `04_json_demo_log.txt` | Log console demo JSON Formatter — 5 test case |

## Kết quả RAGAS — V1 vs V2

| Chỉ số | V1 (ngắn gọn) | V2 (có cấu trúc) | Chênh lệch |
|---|---:|---:|---:|
| faithfulness | **0.9653** ⭐ | 0.8188 ⭐ | +0.147 nghiêng V1 |
| answer_relevancy | **0.9113** | 0.8989 | +0.012 nghiêng V1 |
| context_recall | 1.0000 | 1.0000 | hòa |
| context_precision | 0.9417 | 0.9450 | +0.003 nghiêng V2 |

Cả 2 phiên bản đều đạt mục tiêu faithfulness ≥ 0.8. `context_recall`/`context_precision` gần như bằng nhau vì cả 2 chain dùng chung một retriever (k=3) — chỉ khác nhau ở system prompt cho bước sinh câu trả lời, nên chênh lệch chủ yếu nằm ở `faithfulness` và `answer_relevancy`.

### Vì sao V1 có faithfulness cao hơn V2?

V1 yêu cầu trả lời ngắn (2-4 câu) và nói thẳng "không biết" khi thiếu thông tin, khiến mô hình bám sát context được truy xuất. V2 yêu cầu cấu trúc 3 phần (tóm tắt → trích dẫn → **mức độ chắc chắn**) — phần "nêu mức độ chắc chắn" thường buộc mô hình thêm các câu bình luận/đánh giá mang tính diễn giải mà không được nêu trực tiếp trong context nguồn. RAGAS faithfulness tính theo tỷ lệ claim trong câu trả lời có thể suy ra được từ context, nên càng nhiều câu diễn giải thêm (dù hợp lý) càng kéo điểm xuống — đúng như số liệu cho thấy: khoảng cách faithfulness (+0.147) lớn hơn nhiều so với khoảng cách answer_relevancy (+0.012), tức nội dung V2 vẫn liên quan đến câu hỏi tốt gần như V1, chỉ là "nói nhiều hơn những gì context xác nhận được".

Kết quả này nhất quán qua 2 lần chạy độc lập toàn bộ 50 câu hỏi (RAGAS gọi LLM-judge nên có dao động nhỏ giữa các lần chạy): lần 1 V1=0.9419/V2=0.8142, lần 2 (số liệu chính thức ở trên) V1=0.9653/V2=0.8188 — cả 2 lần V1 đều vượt V2 khoảng 0.13-0.15 điểm faithfulness, và cả 2 phiên bản đều luôn ≥ 0.8.

## LangSmith Project

- Project: `day22-nguyenviethung`
- Tổng root traces: 618 (`rag-query` ≥ 50, `ab-rag-query` ≥ 50, phần còn lại từ các lệnh gọi LLM trực tiếp trong Bước 3)
