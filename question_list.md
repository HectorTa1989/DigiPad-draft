# 1. Các câu hỏi quan trọng nhất cần hỏi trước
SHB muốn hệ thống giải quyết quy trình cụ thể nào nhất?
Ví dụ: tín dụng SME, kiểm tra hồ sơ vay cá nhân, KYC/AML, xử lý khiếu nại hay vận hành nội bộ?


Một “complex request” hợp lệ trong tiêu chí chấm điểm trông như thế nào?
Có bắt buộc cả Credit, Compliance và Operations cùng tham gia không?


Agent phải thực hiện hành động thật đến mức nào?
Gọi mock API và cập nhật database demo có được tính là “execute actions within operational systems” không?


Những quyết định/hành động nào bắt buộc phải có human approval?
Agent có được đề xuất phê duyệt tín dụng, tạo case, chuyển trạng thái hồ sơ hay gửi yêu cầu bổ sung tài liệu không?


Ban tổ chức có cung cấp dữ liệu, policy, API hoặc schema nghiệp vụ mẫu không?


Có được sử dụng dữ liệu tổng hợp và quy định giả lập nếu không được cung cấp dữ liệu SHB không?


Kết quả của Compliance Agent cần dẫn nguồn ở mức nào?
Tên tài liệu, phiên bản, điều khoản hay chỉ cần đoạn văn liên quan?


Dashboard cần hiển thị loại trace nào?
Task, agent handoff, tool calls, evidence, approval và audit log có đủ không?


Phần so sánh single-agent và multi-agent được chấm theo tiêu chí nào?
Độ chính xác, thời gian, chi phí, khả năng giải thích hay tỷ lệ hoàn thành workflow?


Tiêu chí nào có trọng số cao nhất: nghiệp vụ, độ hoàn thiện demo, AI innovation, system architecture hay khả năng áp dụng tại SHB?


# 2. Câu hỏi về nghiệp vụ ngân hàng
## 2.1 Chọn đúng use case
Nên hỏi:

SHB đang gặp pain point lớn nhất ở bước nào của quy trình tín dụng?
Quy trình nào hiện yêu cầu nhiều bộ phận trao đổi thủ công nhất?
Hồ sơ SME có phù hợp để làm MVP không, hay SHB ưu tiên tín dụng cá nhân?
Một hồ sơ thực tế thường đi qua những phòng ban và vai trò nào?
SLA hiện tại của quy trình là bao lâu?
Bước nào chiếm nhiều thời gian nhất?
Bước nào thường xảy ra lỗi, thiếu chứng từ hoặc phải làm lại?
Trong demo, nên tối ưu tốc độ xử lý, độ chính xác hay khả năng kiểm soát?

Mục tiêu cần chốt
Team phải có được một workflow rõ ràng như:
```
Nhân viên tạo yêu cầu
→ Kiểm tra hồ sơ
→ Phân tích tài chính
→ Kiểm tra chính sách tín dụng
→ Kiểm tra KYC/AML
→ Phát hiện tài liệu thiếu
→ Tổng hợp đề xuất
→ Human review
→ Tạo hoặc cập nhật case
```

## 2.2 Ranh giới giữa “đề xuất” và “quyết định”
Đây là câu hỏi nghiệp vụ quan trọng nhất.
Nên hỏi thẳng:

Credit Agent chỉ được đưa ra phân tích hay được đề xuất hạn mức?
Agent có được đề xuất approve, conditional approve hoặc reject không?
Ai là người chịu trách nhiệm cuối cùng?
Những trường hợp nào bắt buộc chuyển chuyên viên?
Nếu Credit Agent và Compliance Agent đưa ra kết luận khác nhau, quy tắc nào được ưu tiên?
Một compliance finding mức high có tự động dừng workflow không?
Có cần mô hình “four-eyes principle” — ít nhất hai người kiểm tra/phê duyệt — trong demo không?

Khuyến nghị đề xuất với mentor
Agent chỉ nên tạo:
```
Recommendation:
- Eligible
- Conditionally eligible
- Insufficient information
- Escalation required
```

Không nên để LLM tự đưa ra quyết định pháp lý/tín dụng cuối cùng. Quyết định cuối thuộc về nhân sự có thẩm quyền.

## 2.3 Quy tắc tín dụng và tính toán
Nên hỏi rõ:

SHB có cung cấp công thức hoặc rule scoring mẫu không?
Những chỉ số tài chính nào cần xuất hiện trong demo?
Có threshold mẫu cho DSCR, leverage hoặc khả năng trả nợ không?
Credit score nên là rule-based, ML-based hay LLM-generated?
Có cần đánh giá tài sản bảo đảm không?
Có cần kiểm tra lịch sử tín dụng/CIC không?
Nếu không có CIC API, mock response có được chấp nhận không?
Có cần giải thích rõ mỗi yếu tố đóng góp vào mức rủi ro không?

Điểm cần thống nhất
LLM không nên tự tính toán hoặc tự sáng tạo quy tắc tín dụng.
Nên chốt với mentor kiến trúc:
```
LLM/Agent:
- Lập kế hoạch
- Chọn tool
- Giải thích kết quả
- Tổng hợp evidence

Deterministic service:
- Tính financial ratios
- Áp dụng threshold
- Tính risk score
- Kiểm tra document rules
```

## 2.4 Legal, KYC, AML và Compliance
Nên hỏi:

Compliance Agent nên kiểm tra quy định nội bộ, pháp luật hay cả hai?
Ban tổ chức có cung cấp một bộ policy mẫu không?
Policy có nhiều phiên bản và ngày hiệu lực không?
Có cần xử lý trường hợp quy định xung đột hoặc hết hiệu lực không?
Có cần sanctions screening thật hay chỉ cần service giả lập?
Có cần xác định UBO — chủ sở hữu hưởng lợi cuối cùng — không?
Mức độ explainability được yêu cầu là gì?
Finding phải trích dẫn đến section, clause hay trang tài liệu?
Trường hợp không đủ evidence, output mong muốn là gì?
Có cần người thuộc bộ phận pháp chế xác nhận mọi kết luận không?

Output nên đề xuất cho mentor
```
{
  "status": "escalation_required",
  "finding": "Missing UBO declaration",
  "severity": "high",
  "evidence": [
    {
      "document": "KYC Policy",
      "version": "2026.1",
      "clause": "5.3.2"
    }
  ],
  "recommended_action": "Request additional documentation"
}
```

Hỏi mentor xem cấu trúc như vậy có đủ gần nghiệp vụ SHB hay không.

# 3. Câu hỏi về dữ liệu và RAG
## 3.1 Dữ liệu do ban tổ chức cung cấp
Nên hỏi:

Có sample customer profile không?
Có báo cáo tài chính mẫu không?
Có bộ hồ sơ tín dụng mẫu không?
Có checklist chứng từ không?
Có policy nội bộ đã được ẩn thông tin nhạy cảm không?
Dữ liệu ở định dạng PDF, Word, Excel, JSON hay API?
Có tài liệu scan cần OCR không?
Có bộ ground truth để đánh giá không?
Dữ liệu được phép gửi tới cloud LLM không?
Có yêu cầu lưu trữ dữ liệu tại Việt Nam hoặc trong môi trường riêng không?

## 3.2 Nếu không có dữ liệu thật
Hỏi mentor:

Nếu SHB không cung cấp dữ liệu nội bộ, team có thể dùng synthetic data nhưng mô phỏng đầy đủ customer profile, financial statements, KYC, policy version và workflow state không?

Cũng cần hỏi:

Synthetic policy có được chấp nhận không?
Có cần đánh dấu rõ dữ liệu giả lập trên dashboard không?
Có cần công cụ để sinh nhiều bộ hồ sơ test không?
Giám khảo đánh giá kiến trúc hay độ chính xác trên dữ liệu SHB?

## 3.3 Thiết kế RAG
Nên hỏi:

Mỗi agent có cần knowledge base riêng không?
Có cho phép một agent truy cập dữ liệu của domain khác không?
Retrieval có phải áp dụng quyền truy cập theo user không?
Có cần versioning và effective date không?
Có yêu cầu citations bắt buộc không?
Có cần hiển thị đoạn evidence gốc trên dashboard không?
Có cần benchmark retrieval quality riêng không?
Có cần hỗ trợ tiếng Việt và tiếng Anh đồng thời không?

Điểm nên xin mentor xác nhận
```
Credit Agent     → chỉ tìm credit policies
Compliance Agent → chỉ tìm KYC/AML/legal policies
Operations Agent → chỉ tìm SOP và checklist
Supervisor       → nhận kết quả có cấu trúc, không truy cập toàn bộ tài liệu
```

# 4. Câu hỏi về kiến trúc multi-agent
## 4.1 Multi-agent đến mức nào mới đạt yêu cầu?
Nên hỏi:

Có bắt buộc các agent giao tiếp trực tiếp với nhau không?
Supervisor-mediated communication có đủ không?
Agent có cần tự thương lượng hoặc phản biện lẫn nhau không?
Workflow cố định nhưng có conditional routing có được tính là agentic không?
Planner có bắt buộc tạo dynamic plan bằng LLM không?
Có cần hỗ trợ re-planning khi tool lỗi hoặc thiếu dữ liệu không?
Tối thiểu bao nhiêu agent chuyên môn?
Validator/Critic có được tính là một specialist agent không?
Một request có bắt buộc gọi tất cả agent không, hay planner được quyền bỏ qua agent không liên quan?

## 4.2 Câu hỏi đặc biệt quan trọng
Hãy hỏi mentor:

Ban giám khảo ưu tiên một workflow rõ ràng, kiểm soát được bằng LangGraph hay ưu tiên agent tự do trao đổi và tự lập kế hoạch?

Hai hướng này khác nhau:
Workflow có kiểm soát
```
Ưu điểm:
- Dễ audit
- Dễ test
- Dễ giới hạn quyền
- Phù hợp ngân hàng
- Demo ổn định
```
Agent tự do trao đổi
```
Ưu điểm:
- Trông “agentic” hơn
- Linh hoạt hơn

Rủi ro:
- Khó dự đoán
- Tốn token
- Có thể lặp vô hạn
- Khó giải thích trách nhiệm
- Demo dễ lỗi
```

Đối với banking, team nên nghiêng về bounded autonomy: agent được tự lập kế hoạch trong một graph có giới hạn.

# 5. Câu hỏi về tool use và tích hợp hệ thống
Nên hỏi cụ thể:

Mock API có đủ để chứng minh integration không?
API cần tích hợp trực tiếp với hệ thống nào của SHB?
Có sandbox hoặc OpenAPI specification không?
Có cần dùng MCP hay MCP chỉ là công nghệ gợi ý?
Tool call có cần chạy qua approval gateway không?
Các thao tác nào cần idempotency?
Demo có cần thể hiện rollback không?
Nếu API lỗi, agent phải retry, re-plan hay chuyển cho con người?
Có yêu cầu transaction management không?
Agent được phép cập nhật database trực tiếp hay phải đi qua service API?

Cần chốt định nghĩa “action”
Hỏi mentor liệu ba mức sau có được công nhận không:

Read action: truy vấn profile hoặc policy.
Draft mutation: tạo case/checklist trong database.
Approved mutation: người dùng phê duyệt rồi agent mới submit hoặc gửi request.

Nếu chỉ gọi API đọc dữ liệu, giám khảo có thể đánh giá hệ thống vẫn thiên về hỏi đáp. Demo nên có ít nhất một state-changing action.



# 6. Câu hỏi về human-in-the-loop
Nên hỏi:

Hành động nào bắt buộc phê duyệt?
Ai có quyền phê duyệt từng loại hành động?
Có cần nhiều cấp phê duyệt không?
Khi request bị reject, agent có được tự sửa và submit lại không?
Có cần timeout cho approval không?
Có cần phân biệt maker và checker không?
Có cần lưu lý do approve/reject không?
Có cần chứng minh payload không bị thay đổi sau phê duyệt không?

Workflow đề xuất để mentor review
```
Agent tạo action draft
→ Validator kiểm tra
→ Dashboard hiển thị payload + evidence
→ Nhân viên approve/reject
→ Backend kiểm tra role
→ Phát hành approval token
→ Tool thực thi đúng payload đã được approve
→ Ghi audit event
```

# 7. Câu hỏi về security và audit
Nên hỏi:

Có security checklist chính thức không?
Có yêu cầu agent identity riêng không?
Có yêu cầu RBAC/ABAC không?
Có cần mask CCCD, số tài khoản và thông tin khách hàng?
Có được gửi dữ liệu lên API LLM công cộng không?
Có cần hỗ trợ private/on-premise model không?
Log được phép chứa những trường dữ liệu nào?
Audit log cần append-only hay hash-chain không?
