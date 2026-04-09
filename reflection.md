# Individual reflection — Nguyễn Hồ Bảo Thiên (2A202600163)

## 1. Role
Xây dựng Guardrails + Rule Engine. Phụ trách xây dựng các node tiền xử lý (Pre-LLM logic), kiểm tra an toàn và phân loại mức độ nghiêm trọng trước khi đưa dữ liệu cho AI.

## 2. Đóng góp cụ thể
- Tạo `guard_node`: Quét file JSON kết quả xét nghiệm của bệnh nhân để đối chiếu với `CRITICAL_THRESHOLDS`, giúp hệ thống chặn và phát cảnh báo cấp cứu ngay lập tức.
- Tạo `severity_node`: Khớp dữ liệu thô vào các Pydantic model (`LabResult`), tích hợp với hàm `classify_severity` để dán nhãn mức độ nghiêm trọng (NORMAL, WATCH, SEE_DOCTOR, CRITICAL) cho từng chỉ số.
- Thiết kế luồng output khớp với cấu trúc `AgentState` (dạng list of dicts) để dễ dàng ráp vào LangGraph.

## 3. SPEC mạnh/yếu
- **Mạnh nhất:** Kiến trúc luồng xử lý (Data Flow) — việc tách bạch rõ ràng giữa Deterministic Rule-based (`guard_node`) và Probabilistic AI (`explain_node`). Điều này đảm bảo AI không bao giờ được phép tự đánh giá độ nguy kịch bằng cảm tính mà phải tuân theo hard-code y tế.
- **Yếu nhất:** Xử lý Edge Cases trong dữ liệu đầu vào. Do phụ thuộc hoàn toàn vào file JSON từ bệnh nhân, nếu thiếu key hoặc đổi cấu trúc (ví dụ: bị rớt mất key `test_name` hoặc `unit`), hệ thống dễ báo lỗi hoặc hiển thị không trọn vẹn ở giao diện. 

## 4. Đóng góp khác
- Viết file `demo_nodes.py` để test luồng độc lập với các JSON mock data (`P001.json`) trước khi hand-off.
- Hỗ trợ debug và fix các lỗi crash liên quan đến `TypeError` và `Pydantic ValidationError` (ví dụ: lỗi thiếu field required khi map dữ liệu từ Dict lồng nhau sang Object).

## 5. Điều học được
Trước hackathon, tôi nghĩ cốt lõi của một ứng dụng AI là nằm ở LLM và Prompt Engineering. Sau khi làm phân hệ Triage & Lab Results, tôi mới hiểu: LLM rất dễ "ảo giác" (hallucinate). Trong các lĩnh vực sinh tử như y tế, AI chỉ là "người phát ngôn", còn bộ não quyết định sự an toàn phải nằm ở các "Guardrails" — những logic rule-based truyền thống chạy ngầm. Safety > Smart.

## 6. AI giúp gì / AI sai gì
- **Giúp:** AI xử lý rất nhanh việc refactor các vòng lặp Dictionary/JSON lồng nhau, giúp tôi tìm và sửa lỗi Pydantic (`Field required [type=missing]`) chỉ trong chớp mắt thông qua việc đối chiếu traceback error.
- **Sai/mislead:** AI (trong một số lần prompt) đã xui tôi tự viết lại luôn các hàm tính toán % chênh lệch (deviation) bằng các thuật toán phức tạp, hoặc khuyên tôi bỏ qua tham số bắt buộc. Nếu nghe theo, tôi sẽ tự phá vỡ cấu trúc OOP và bỏ qua việc tái sử dụng hàm (`classify_severity`) cực tốt mà Team Lead đã viết sẵn, dẫn đến lặp code (DRY violation) không cần thiết. Trải nghiệm này dạy tôi phải luôn soi kỹ source code gốc trước khi tin code AI sinh ra.

