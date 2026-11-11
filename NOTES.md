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

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng |  |  |
| Điểm case đã chọn |  |  |
| Số model calls |  |  |
| Token, chi phí nếu có |  |  |

- Lỗi đã giải quyết:
- Lỗi còn lại hoặc mới xuất hiện:

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.
