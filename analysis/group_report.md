# Group Report - Lab 18: Production RAG

**Nhóm/Sinh viên:** Hoang Anh Thu
**Ngày:** 22/06/2026

## Thành viên & Phân công

| Tên | Module | Hoàn thành | Ghi chú |
|-----|--------|-----------|--------|
| Hoang Anh Thu | M1: Chunking | x | Semantic, hierarchical, structure-aware; production dùng hierarchical parent+child |
| Hoang Anh Thu | M2: Hybrid Search | x | BM25 tiếng Việt + Dense/Qdrant fallback + RRF + query expansion |
| Hoang Anh Thu | M3: Reranking | x | CrossEncoder wrapper, fallback lexical rerank để chạy nhẹ |
| Hoang Anh Thu | M4: Evaluation | x | RAGAS/fallback metrics, report, failure analysis |
| Hoang Anh Thu | M5: Enrichment | x | Summary/questions/contextual prepend/metadata với fallback heuristic |

## Kết quả RAGAS

| Metric | Naive | Production | Delta |
|--------|------:|-----------:|------:|
| Faithfulness | 0.9000 | 0.8200 | -0.0800 |
| Answer Relevancy | 0.7632 | 0.7725 | +0.0093 |
| Context Precision | 0.9250 | 0.7988 | -0.1262 |
| Context Recall | 0.9250 | 0.9000 | -0.0250 |

## Key Findings

1. **Biggest improvement:** Answer Relevancy tăng nhẹ từ 0.7632 lên 0.7725. Prompt mới trả lời trực tiếp hơn cho câu có context rõ và câu phủ định như KHÔNG/chưa được.
2. **Biggest challenge:** Context Precision giảm từ 0.9250 xuống 0.7988 vì production lấy nhiều parent/enriched chunks để bảo toàn context cho câu multi-hop. Đây là trade-off giữa recall và precision.
3. **Surprise finding:** Naive baseline có precision cao hơn do context ngắn hơn, nhưng production xử lý tốt hơn các case conflict version như chính sách 2024 thay thế 2023 và các câu cần nối nhiều nguồn.

## Presentation Notes (5 phút)

1. **RAGAS scores:** Production đạt faithfulness 0.82, answer relevancy 0.77, context precision 0.80, context recall 0.90.
2. **Biggest win:** Kết hợp hierarchical chunking + query expansion giúp kéo được parent context cho các câu bị cắt mất ý như nghỉ không lương 16-30 ngày cần CEO.
3. **Case study:** Câu hoàn chi đào tạo 25 triệu vẫn fail do retrieval chưa ưu tiên đúng chunk cam kết hoàn chi; cần query expansion/metadata boost theo domain đào tạo.
4. **Next optimization:** Giảm top context cho câu single-hop, giữ parent context cho multi-hop, thêm template trả lời yes/no và calculation.

## Files Generated

- `ragas_report.json`: kết quả production RAGAS.
- `naive_baseline_report.json`: kết quả baseline để so sánh.
- `analysis/failure_analysis.md`: bottom failure analysis.
- `analysis/reflections/reflection_HoangAnhThu.md`: reflection cá nhân.
