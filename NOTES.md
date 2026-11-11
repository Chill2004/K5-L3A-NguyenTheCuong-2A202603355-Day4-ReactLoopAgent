# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Thế Cường
MSSV: 2A202603355

## 1. Case được chọn

- ID case: H1.
- Câu trả lời của agent: Đã tổng hợp hai incident GitHub và Cloudflare,
  nhưng chỉ dẫn liên kết tới incident tổng kết mà chưa đọc nội dung incident đó.
- Điểm ở lần chạy 1: 60/100.
- Điểm tổng baseline: 74,77/100, chấm đủ 22/22 case.

## 2. Bằng chứng trong trace

Trace baseline có hai tool call trong cùng bước 1:

```text
step 1: get_incident_details(github, h02n9572mx9y)
step 1: get_incident_details(cloudflare, yaa0witbl2me)
step 2: model trả câu trả lời cuối

Update của GitHub dẫn tới incident xr3myzj7ms51,
nhưng trace không có tool call đọc incident này.

grade.mechanical.tool_steps: 1
grade.mechanical.score_cap: 60
```

Nguyên nhân có thể là model coi việc đưa liên kết vào câu trả lời
đã đủ đáp ứng yêu cầu, thay vì tiếp tục gọi tool để đọc bản tổng kết.
Loop ban đầu chưa có cơ chế kiểm tra việc đọc incident liên quan.

## 3. Thay đổi đã thực hiện

- Bổ sung system prompt về thời điểm snapshot, giới hạn dữ liệu
  và việc đọc incident liên quan khi điều tra nguyên nhân hoặc ảnh hưởng.
- Thêm bộ nhớ URL từ các lần đọc incident thành công.
- Nếu câu trả lời không chứa URL incident đã quan sát, ghi event
  validation_missing_source và yêu cầu model bổ sung trong budget hiện có.
- Không yêu cầu URL incident khi tool đọc chi tiết thất bại.
- Chạy lại unit test: 37 test đạt.
- Chạy thử riêng H1: vẫn 60/100, còn thiếu bước đọc incident liên quan.
  Kiểm tra nguồn chưa bảo đảm agent thực hiện đủ quá trình điều tra.

## 4. So sánh hai lần chạy

Hai lần dùng cùng cấu hình: OpenRouter, model openai/gpt-4o-mini
cho cả agent và judge, max_steps=8, dataset core và snapshot có sẵn.

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Case được chấm | 22/22 | 22/22 |
| Điểm tổng | 74,77 | 80,23 |
| Điểm H1 | 60 | 60 |
| Số lần gọi model của agent | 51 | 50 |
| Số lần gọi judge | 22 | 22 |
| Token của agent | 41.877 | 48.245 |
| Token của judge | 40.790 | 35.941 |
| Tổng chi phí agent và judge theo log | 0.01819980 USD | 0.01766805 USD |

Các con số trên chỉ tính hai lần chạy toàn bộ bộ eval,
không bao gồm lần chạy thử riêng hoặc các request kiểm tra kết nối.

- Điểm tổng tăng 5,46 điểm. Một số case cải thiện như
  A5 từ 45 lên 85 và H8 từ 55 lên 100.
- A5 thể hiện rõ hơn rằng dữ liệu 50 incident gần nhất không đủ
  để cung cấp danh sách đầy đủ cho cả năm 2025.
- Cơ chế kiểm tra nguồn đã ghi nhận validation_missing_source
  tại A6, A7 và H6, yêu cầu model sửa trước khi hoàn thành.
- H1 vẫn chưa đọc incident tổng kết được dẫn trong update.
  Hai tool call vẫn nằm trong một model step nên bị áp trần 60 điểm.
- H2 vẫn thiếu get_status để đối chiếu trạng thái component;
  chỉ đọc một incident đã resolved chưa đủ để kết luận trạng thái component.
- Có case giảm điểm: E4 từ 100 xuống 75, H6 từ 60 xuống 45,
  H7 từ 75 xuống 55. Cần đọc câu trả lời và trace để phân biệt
  lỗi thực tế với nhận xét chưa chính xác của judge.
- H7 là lỗi còn lại đáng chú ý: tool Cloudflare timeout nhưng agent
  vẫn làm theo yêu cầu kết luận operational. Khi thiếu dữ liệu,
  agent cần nói không xác minh được thay vì suy đoán trạng thái.
- Số lần gọi agent giảm một lần nhưng token của agent tăng.
  Vì vậy chưa thể kết luận bản cải tiến tiết kiệm tài nguyên hơn.

## 5. Giới hạn

- LLM judge có thể đánh giá sai hoặc không nhất quán. Ví dụ E4
  bị nhận xét chưa làm rõ Actions có liên quan, trong khi câu trả lời
  đã liệt kê Actions và mô tả ảnh hưởng tới workflows.
  Điểm số cần được đối chiếu với trace và dữ liệu tool.
- Agent và judge dùng cùng model nên kết quả có thể có thiên lệch.
- Mỗi phiên bản chỉ chạy toàn bộ eval một lần; mức tăng điểm
  chưa chứng minh hành vi cải thiện ổn định hoặc hoàn toàn do thay đổi code.
- Snapshot bài chính được lưu ngày 09/11/2026, không đại diện
  cho trạng thái dịch vụ tại thời điểm người dùng hỏi.
- Tool chỉ tìm trong 50 incident gần nhất và có thể rút gọn updates.
  Không thể dùng kết quả này để bảo đảm lịch sử incident đầy đủ.
- Kiểm tra nguồn hiện chỉ xác nhận câu trả lời chứa ít nhất một URL
  incident đã quan sát. Nó không xác minh mọi nhận định, không bảo đảm
  nguồn cho từng provider và không bắt buộc model đọc incident liên quan.
- Unit test đạt 37/37 xác nhận các contract được kiểm thử,
  không bảo đảm model luôn trả lời đúng. H1 và H7 cho thấy
  vẫn cần cải tiến khả năng điều tra nhiều bước và xử lý thiếu dữ liệu.
