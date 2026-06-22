# Individual Reflection - Lab 18

**Tên:** Hoang Anh Thu
**Module phụ trách:** M1, M2, M3, M4, M5

---

## 1. Mapping bài giảng

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | Nhóm câu theo similarity để tránh cắt ngang ý; fallback lexical giúp chạy được khi chưa tải embedding model. |
| Hierarchical chunking | M1 | `chunk_hierarchical()` | Parent giữ context rộng, child giúp search chính xác; sau khi chạy report thấy parent chunk cải thiện recall nhưng làm precision giảm. |
| Structure-aware chunking | M1 | `chunk_structure_aware()` | Markdown headers và section metadata giúp giữ cấu trúc chính sách như Phiên bản, Quy trình, Điều kiện. |
| BM25 + Dense fusion | M2 | `HybridSearch`, `reciprocal_rank_fusion()` | BM25 bắt keyword/số liệu tốt, Dense hỗ trợ paraphrase; RRF hợp nhất hai nguồn nhưng cần rerank/filter để bớt context nhiễu. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Rerank cải thiện thứ tự context, nhưng fallback lexical vẫn cần domain boost cho case version/conflict và câu multi-hop. |
| RAGAS 4 metrics | M4 | `evaluate_ragas()`, `failure_analysis()` | Faithfulness/recall cho biết có đủ bằng chứng, precision cho thấy retrieval có nhiều nhiễu, answer relevancy cho thấy câu trả lời có đủ ý không. |
| Contextual embeddings / enrichment | M5 | `contextual_prepend()`, `enrich_chunks()` | Enrichment giúp chunk tự mô tả tốt hơn, nhưng nếu prepend quá nhiều có thể làm context dài và giảm precision. |

## 2. Khó khăn & Cách giải quyết

- **Lỗi môi trường:** `pytest` trong venv cũ bị lỗi launcher vì trỏ sai Python. Cách xử lý là tạo `venv311` bằng Python 3.11 và chạy bằng đường dẫn trực tiếp `D:\AIA\venv311\Scripts\python.exe`.
- **Qdrant:** Khi Docker/Qdrant chưa chạy có lỗi `[WinError 10061] No connection could be made because the target machine actively refused it`. Pipeline có fallback in-memory để vẫn chạy được, còn muốn dùng Qdrant thì chạy `docker compose up -d` và kiểm tra `curl http://localhost:6333/collections`.
- **Hugging Face model:** Lần đầu tải embedding/reranker có warning HF_TOKEN và tải model chậm. Giải pháp là dùng fallback cho reranker trong test/dev, chỉ bật model nặng khi cần.
- **Conflict version:** Với chính sách nghỉ phép năm v2023/v2024, retriever ban đầu kéo bản cũ hoặc chunk lân cận. Đã thêm version-aware context augmentation để ưu tiên `2024`, `hiện hành`, `thay thế`, `v2.0`.
- **Chunk bị cắt:** Câu nghỉ không lương 20 ngày bị cắt trước đoạn CEO. Đã index thêm parent chunk để giữ đủ câu điều kiện 16-30 ngày.
- **Answer quá ngắn:** Một số câu yes/no đúng nhưng thiếu vế bổ sung. Prompt cần yêu cầu trả lời đủ điều kiện và ngoại lệ trong context.

## 3. Action Plan cho project

## Project: Legal/HR Policy RAG Assistant

### Hiện tại
- RAG pipeline hiện tại đã có document loader, chunking nâng cao, hybrid search, reranking, enrichment và RAGAS evaluation.
- Known issues: context precision giảm khi lấy quá nhiều parent/enriched chunks; một số câu calculation và yes/no cần answer template rõ hơn.

### Plan áp dụng
1. [ ] **Chunking strategy:** Dùng hierarchical chunking làm chính; child để search, parent để answer. Với tài liệu markdown/chính sách, bổ sung structure-aware metadata theo header.
2. [ ] **Search:** Kết hợp BM25 + Dense + RRF. BM25 ưu tiên số liệu, điều kiện, tên chính sách; Dense dùng cho câu hỏi paraphrase.
3. [ ] **Reranking:** Dùng CrossEncoder khi chạy production, fallback lexical khi test. Thêm metadata boost cho version hiện hành, source đúng domain, và section liên quan.
4. [ ] **Evaluation:** Dùng RAGAS cho faithfulness, answer relevancy, context precision, context recall; thêm custom checks cho calculation và yes/no completeness.
5. [ ] **Enrichment:** Dùng contextual prepend ngắn, giữ source/section/version trong metadata để filter. Tránh prepend quá dài làm giảm precision.

### Timeline
- Tuần 1: Chuẩn hóa loader, chunking, metadata version/source/section.
- Tuần 2: Tối ưu hybrid retrieval, query expansion theo domain, Qdrant collection ổn định.
- Tuần 3: Thêm reranker thật và answer templates cho yes/no, conflict version, calculation.
- Tuần 4: Chạy RAGAS regression, phân tích bottom failures, viết báo cáo cuối.

## 4. Nếu làm lại

- Sẽ tách rõ chế độ single-hop và multi-hop: single-hop chỉ gửi top 3 context để precision cao hơn; multi-hop mới gửi parent và context bổ sung.
- Sẽ giữ metadata sau enrichment tốt hơn để filter theo `source`, `section`, `version` thay vì chỉ dựa vào text marker.
- Sẽ thêm bộ test nhỏ cho các case conflict version, tính toán pro-rata, và câu hỏi phủ định.

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------|
| Hiểu bài giảng | 4 |
| Code quality | 4 |
| Teamwork/tự quản lý task | 4 |
| Problem solving | 4 |
| Debug môi trường | 4 |
