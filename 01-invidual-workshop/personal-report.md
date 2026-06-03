# Báo cáo cá nhân — Mổ App AI Thật

- Tên: Phạm Triều DƯơng
- Mã số: 2A202600833
- Ngày: 03/06/2026
- Sản phẩm: Vietnam Airlines — NEO (Chatbot)

Tóm tắt (1 đoạn)

Trong phiên thử nghiệm, tôi đóng vai hành khách muốn biết "Cao Bằng → TP. HCM" giá vé thế nào. NEO thường yêu cầu user cung cấp mã sân bay thay vì tự suy luận sân bay gần nhất; kết quả là người dùng phải tương tác nhiều lượt để có giá tiền, dẫn tới trải nghiệm kém. Báo cáo này tóm tắt bằng chứng, phân tích failure mode, và đưa ra các quyết định SPEC rõ ràng để giảm thiểu số lượt tương tác và cải thiện UX.

1. Context & mục tiêu

- Vai trò: end-user, task: tra cứu giá vé nhanh từ địa danh tự nhiên.
- Mục tiêu: kiểm tra promise vs reality của NEO với địa danh không có sân bay (Cao Bằng).

2. Phát hiện chính (evidence)

- Observation: Khi user nhập "Cao Bằng đi HCM giá thế nào", NEO trả lời rằng "Cao Bằng không có mã sân bay phù hợp" và yêu cầu người dùng chỉ định sân bay/thành phố khởi hành, thay vì tự đề xuất sân bay gần nhất hoặc trả giá tham khảo.
- Quote tiêu biểu: "Cao Bằng không có mã sân bay phù hợp. Vui lòng chỉ định một sân bay hoặc thành phố khởi hành."
- File ảnh chứng cứ: xem [01-invidual-workshop/screenshots](01-invidual-workshop/screenshots) (tôi đã chuẩn hoá thư mục ảnh; chèn `neo_1.png`, `neo_2.png`, ... vào đó).

3. Four paths — trạng thái hiện tại

- Happy: Khi user biết mã IATA (ví dụ HAN → SGN), NEO gọi API và trả giá nhanh.
- Low-confidence: Không có path rõ ràng — NEO re-ask thay vì đưa giả định hoặc gợi ý.
- Failure: Luồng slot-filling lặp nhiều lượt; user không nhận được giá nhanh và dễ bỏ cuộc.
- Correction: Không có cơ chế lưu/cập nhật từ phản hồi tiêu cực trong luồng này.

4. Product decision (clear, actionable)

- Vấn đề: NEO thiếu Data/Mapping + thiếu low-confidence UX → ép user hoàn thiện slot thủ công.
- Tác động: tăng thời gian tương tác, giảm tỉ lệ chuyển đổi, trải nghiệm người dùng xấu (negative sentiment).
- Yêu cầu SPEC:
  - Geographic Clustering Middleware: nếu `Has_Airport = False` cho địa danh nhập vào, middleware sẽ map tới `default_hub` (ví dụ `HAN`) và system sẽ trả một "giá tham khảo" trong cùng response.
  - Low-confidence behavior: NEO phải đưa ra tối đa 2 giả định (ví dụ: "Ý bạn là Hà Nội (HAN) hay Sơn La?"), hoặc tự chọn hub mặc định + ghi rõ nguồn/độ tin cậy.
  - Sentiment-aware fallback: detect negative sentiment → rút ngắn quy trình, hiển thị fallback (best-guess) và offer handoff tới agent/phone.

5. Kết quả mong muốn & tiêu chí chấp nhận (Definition of Done)

- Người dùng nhận được một giá tham khảo trong tối đa 2 lượt hội thoại cho các truy vấn địa danh (không biết mã sân bay).
- NEO hiển thị nguồn/độ tin cậy cho giá tham khảo (ví dụ: "Giá 1.200.000 VND — ước tính từ HAN → SGN, ngày 05/06/2026").
- Khi detect negative sentiment, NEO tự động rút gọn slot-filling và đề xuất handoff.

6. Concise implementation hint (developer-facing)

- Middleware (pre-NLU): lookup `place -> has_airport` and `place -> nearest_hub(distance, population)`.
- NLU: thêm flag `low_confidence_intent` khi place không map trực tiếp.
- Response builder: if low_confidence then include `[assumption options]` + `best_guess_price` + `confidence_label`.

7. Appendix — Evidence

- Screenshots & logs: [01-invidual-workshop/screenshots](01-invidual-workshop/screenshots)
- Raw quote: "Cao Bằng không có mã sân bay phù hợp. Vui lòng chỉ định một sân bay hoặc thành phố khởi hành."

--
_Báo cáo hoàn thiện để nộp; nếu cần tôi có thể chuyển phần "Product decision" thành một trang SPEC kỹ thuật chi tiết hơn._
