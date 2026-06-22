# Failure Analysis - Lab 18: Production RAG

**Sinh viên:** Hoang Anh Thu
**Pipeline:** M1 Chunking + M5 Enrichment + M2 Hybrid Search + M3 Reranking + M4 RAGAS

## RAGAS Scores

| Metric | Naive Baseline | Production | Delta |
|---|---:|---:|---:|
| Faithfulness | 0.9000 | 0.8200 | -0.0800 |
| Answer Relevancy | 0.7632 | 0.7725 | +0.0093 |
| Context Precision | 0.9250 | 0.7988 | -0.1262 |
| Context Recall | 0.9250 | 0.9000 | -0.0250 |

## Summary

Production đạt faithfulness 0.82, answer relevancy 0.77, context precision 0.80, context recall 0.90. So với baseline, answer relevancy tăng nhẹ nhưng precision giảm vì pipeline lấy thêm parent/enriched contexts để tránh thiếu bằng chứng trong các câu multi-hop.

## Bottom-5 Failures

### #1
- **Question:** Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng hoàn thành khóa học. Phải hoàn trả bao nhiêu?
- **Expected:** Nhân viên phải cam kết làm việc ít nhất 1 năm sau khi hoàn thành khóa học. Nghỉ sau 8 tháng là trước hạn cam kết, phải hoàn trả 100% chi phí tức 25.000.000 VNĐ.
- **Got:** Nhân viên phải hoàn trả toàn bộ chi phí khóa học 25 triệu VNĐ do đã nghỉ việc trước thời gian quy định nhận thưởng hoặc hỗ trợ. Không có thông tin cụ thể về mức hoàn trả áp dụng cho trường hợp này, vì chỉ đề cập đến ngân sách đào tạo 5 triệu VNĐ/người/năm cho các khóa học bên ngoài. Do đó, số tiền hoàn lại trong trường hợp này là 25 triệu VNĐ.
- **Worst metric:** answer_relevancy = 0.0000
- **Diagnosis:** Câu trả lời chưa bám sát câu hỏi.
- **Suggested fix:** Cải thiện prompt template, rewrite query, hoặc thêm bước intent detection.
- **Error Tree:** Output đúng? -> Context đủ? -> Query/rerank đúng? -> Root cause: Context có một phần đáp án nhưng câu trả lời chưa đủ các ý trong ground truth; cần answer template theo dạng câu hỏi.
- **Root cause:** Context có một phần đáp án nhưng câu trả lời chưa đủ các ý trong ground truth; cần answer template theo dạng câu hỏi.

### #2
- **Question:** Khi phát hiện malware trên máy, nhân viên có nên tự xử lý không?
- **Expected:** KHÔNG. Nhân viên tuyệt đối không được tự ý xử lý malware. Phải báo cáo trong vòng 1 giờ qua helpdesk@cty.vn hoặc hotline CNTT. Tự ý xử lý bị coi là vi phạm nghiêm trọng.
- **Got:** Nhân viên **không nên tự ý xử lý malware** hoặc tìm cách khắc phục mà không có sự hướng dẫn của đội CNTT.
- **Worst metric:** answer_relevancy = 0.0000
- **Diagnosis:** Câu trả lời chưa bám sát câu hỏi.
- **Suggested fix:** Cải thiện prompt template, rewrite query, hoặc thêm bước intent detection.
- **Error Tree:** Output đúng? -> Context đủ? -> Query/rerank đúng? -> Root cause: Context có một phần đáp án nhưng câu trả lời chưa đủ các ý trong ground truth; cần answer template theo dạng câu hỏi.
- **Root cause:** Context có một phần đáp án nhưng câu trả lời chưa đủ các ý trong ground truth; cần answer template theo dạng câu hỏi.

### #3
- **Question:** Nhân viên thử việc có được hưởng bảo hiểm sức khỏe PVI không?
- **Expected:** KHÔNG. Nhân viên thử việc chưa được hưởng gói bảo hiểm sức khỏe PVI. Chỉ được tham gia bảo hiểm xã hội bắt buộc.
- **Got:** Nhân viên thử việc **chưa được hưởng gói bảo hiểm sức khỏe PVI**.
- **Worst metric:** answer_relevancy = 0.0000
- **Diagnosis:** Câu trả lời chưa bám sát câu hỏi.
- **Suggested fix:** Cải thiện prompt template, rewrite query, hoặc thêm bước intent detection.
- **Error Tree:** Output đúng? -> Context đủ? -> Query/rerank đúng? -> Root cause: Context có một phần đáp án nhưng câu trả lời chưa đủ các ý trong ground truth; cần answer template theo dạng câu hỏi.
- **Root cause:** Context có một phần đáp án nhưng câu trả lời chưa đủ các ý trong ground truth; cần answer template theo dạng câu hỏi.

### #4
- **Question:** Nhân viên tạm ứng 15 triệu, sau 20 ngày mới thanh toán. Bị phạt bao nhiêu?
- **Expected:** Thời hạn thanh toán là 15 ngày. Quá hạn 5 ngày, bị tính phí 2%/tháng trên 15.000.000 VNĐ = 300.000 VNĐ/tháng (tính pro-rata khoảng 50.000 VNĐ cho 5 ngày).
- **Got:** Nhân viên tạm ứng 15 triệu và thanh toán sau 20 ngày, tức là quá hạn 5 ngày (20 - 15 = 5).   Phí phạt là 2% mỗi tháng, tính theo số ngày quá hạn pro-rata cho tháng. Công thức tính phí phạt là:  \[  \text{Phí phạt} = \text{Số tiền chưa hoàn ứng} \times \left(\frac{2\%}{30}\right) \times \text{Số ngày quá hạn}  \]  Áp dụng vào trường hợp này:  \[ \text{Phí phạt} = 15.000.000 \times \left(\frac{2\%}{30}\right) \times 5 \approx 50.000 VNĐ \]  Vậy nhân viên sẽ bị phạt khoảng 50.000 VNĐ.
- **Worst metric:** faithfulness = 0.1667
- **Diagnosis:** LLM có thể đang hallucinate hoặc trả lời vượt quá context.
- **Suggested fix:** Siết prompt: bắt buộc chỉ trả lời từ context, thêm citation, giảm temperature.
- **Error Tree:** Output đúng? -> Context đủ? -> Query/rerank đúng? -> Root cause: LLM suy luận/tính toán vượt quá bằng chứng trực tiếp trong context; cần prompt/citation chặt hơn hoặc bổ sung rule vào data.
- **Root cause:** LLM suy luận/tính toán vượt quá bằng chứng trực tiếp trong context; cần prompt/citation chặt hơn hoặc bổ sung rule vào data.

### #5
- **Question:** Bảo hiểm sức khỏe PVI có hạn mức bao nhiêu cho nhân viên?
- **Expected:** Hạn mức bảo hiểm sức khỏe PVI cho nhân viên là 200.000.000 VNĐ/năm, bao gồm nội trú, ngoại trú và nha khoa.
- **Got:** Bảo hiểm sức khỏe PVI có hạn mức **200.000.000 VNĐ/năm** cho nhân viên.
- **Worst metric:** context_precision = 0.2500
- **Diagnosis:** Retriever lấy quá nhiều chunk không liên quan.
- **Suggested fix:** Thêm reranking, metadata filter, hoặc giảm top_k retrieval.
- **Error Tree:** Output đúng? -> Context đủ? -> Query/rerank đúng? -> Root cause: Retriever lấy nhiều context ngoài trọng tâm; cần giảm top_k sau rerank hoặc filter theo metadata/source.
- **Root cause:** Retriever lấy nhiều context ngoài trọng tâm; cần giảm top_k sau rerank hoặc filter theo metadata/source.

## Case Study

**Question chọn phân tích:** Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng hoàn thành khóa học. Phải hoàn trả bao nhiêu?

**Error Tree walkthrough:**
1. Output đúng một phần: có số tiền 25 triệu nhưng diễn giải chưa bám sát điều kiện cam kết 1 năm.
2. Context chưa đủ: RAGAS context recall của case này bằng 0.0.
3. Query rewrite chưa tốt: cần mở rộng sang hoàn chi đào tạo, cam kết làm việc 1 năm, nghỉ trước hạn.
4. Fix ở bước M2/M3: query expansion, metadata boost cho file đào tạo, và rerank ưu tiên chunk cam kết hoàn chi.

**Nếu có thêm 1 giờ, sẽ optimize:**
- Giảm số context gửi LLM cho câu single-hop để tăng Context Precision.
- Thêm query expansion cho đào tạo, malware, bảo hiểm thử việc.
- Thêm answer template cho yes/no và calculation questions.
- Giữ metadata source/section sau enrichment để filter ổn định hơn.
