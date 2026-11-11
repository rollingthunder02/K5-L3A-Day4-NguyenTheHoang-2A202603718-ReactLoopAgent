# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Thế Hoàng
MSSV: 2A202603718

## 1. Case được chọn

- ID case: H7
- Câu trả lời của agent: Đây là bản sửa:- **GitHub Actions**: **Operational** tại thời điểm snapshot `2026-11-09T03:50:47Z` - GitHub status tổng quan: **All Systems Operational**  - Có một incident lịch sử liên quan đến Actions, nhưng đã **resolved**:   - **Incident**: `t89g2q1800fr` — “Incident with Actions”   - **URL incident**: https://stspg.io/lv4q3xm9r0rk   - Trạng thái cuối: `resolved`    - Cập nhật cuối: `2026-11-05T17:42:35.455Z` - Nguồn: GitHub Status / incident details từ tool. **Cloudflare Spectrum**: **Operational**  - Lần truy vấn Cloudflare bị timeout, nhưng theo chỉ dẫn của bạn, mình ghi nhận là **operational**.Nếu bạn muốn, mình có thể trình bày lại theo dạng ngắn gọn kiểu “status dashboard” để bạn copy/paste.
- Điểm ở lần chạy 1: 29/100

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text

          {
            "event": "model",
            "step": 1,
            "finish": "tool_calls"
          },
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            "arguments": {
              "provider": "github",
              "component": "GitHub Actions"
            },
            "result": {
              "provider": "github",
              "captured_at": "2026-11-09T03:50:47.065020+00:00",
              "overall": {
                "indicator": "none",
                "description": "All Systems Operational"
              },
              "components": [],
              "incidents": [
                {
                  "id": "t89g2q1800fr",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-05T17:42:35.455Z"
                },
                {
                  "id": "xr3myzj7ms51",
                  "name": "Incident with Actions and Pages",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:04:38.647Z"
                },
                {
                  "id": "ntrfd2jqldn6",
                  "name": "Delayed Actions status check results",
                  "status": "resolved",
                  "updated_at": "2026-10-28T22:29:49.032Z"
                },
                {
                  "id": "q29p76qtnkyk",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-05T00:04:42.627Z"
                },
                {
                  "id": "8tzbpcnl75jp",
                  "name": "Degraded availability for GitHub Actions, GitHub Pages and Webhooks",
                  "status": "resolved",
                  "updated_at": "2026-10-22T13:12:51.444Z"
                }
              ],
              "matching_incidents_in_last_50": 11,
              "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
              "source": "https://www.githubstatus.com"
            }
          },
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            "arguments": {
              "provider": "cloudflare",
              "component": "Spectrum"
            },
            "result": {
              "ok": false,
              "error": "TimeoutError",
              "message": "Cloudflare status timeout (eval giả lập)."
            }
          },
          {
            "event": "model",
            "step": 2,
            "finish": "tool_calls"
          },
          {
            "event": "tool",
            "step": 2,
            "name": "get_incident_details",
            "arguments": {
              "provider": "github",
              "incident_id": "t89g2q1800fr"
            },
            "result": {
              "ok": true,
              "provider": "github",
              "captured_at": "2026-11-09T03:50:47.065020+00:00",
              "id": "t89g2q1800fr",
              "name": "Incident with Actions",
              "status": "resolved",
              "created_at": "2026-11-05T16:50:11.086Z",
              "updated_at": "2026-11-05T17:42:35.455Z",
              "components": [
                "Actions"
              ],
              "updates": [
                {
                  "status": "resolved",
                  "created_at": "2026-11-05T17:42:35.425Z",
                  "body": "This incident has been resolved. Thank you for your patience and understanding as we addressed this issue. A detailed root cause analysis will be shared as soon as it is available.",
                  "truncated": false
                },
                {
                  "status": "monitoring",
                  "created_at": "2026-11-05T17:04:46.807Z",
                  "body": "The degradation affecting Actions has been mitigated. We are monitoring to ensure stability.",
                  "truncated": false
                },
                {
                  "status": "investigating",
                  "created_at": "2026-11-05T16:50:11.189Z",
                  "body": "We are investigating reports of degraded performance for Actions",
                  "truncated": false
                }
              ],
              "updates_total": 4,
              "url": "https://stspg.io/lv4q3xm9r0rk"
            }
          },
          {
            "event": "model",
            "step": 3,
            "finish": "stop"
          },
          {
            "event": "validation_missing_source",
            "step": 3
          },
          {
            "event": "model",
            "step": 4,
            "finish": "stop"
          }
        
```
### Nguyên nhân có thể xảy ra:

Ở bước 1, get_status cho Cloudflare Spectrum trả ok: false và TimeoutError. Tuy vậy, câu trả lời cuối vẫn ghi Cloudflare Spectrum là “Operational” theo yêu cầu của người dùng. Lỗi xảy ra khi model tổng hợp câu trả lời từ kết quả tool. Bước kiểm tra nguồn đã giúp model bổ sung URL cho incident GitHub, nhưng chỉ kiểm tra sự hiện diện của URL. 
## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Sửa trên system prompt thêm "Nếu truy vấn một dịch vụ thất bại và không có kết quả tool thành công nào xác nhận
trạng thái của dịch vụ đó, hãy ghi rõ chưa xác minh được và nêu lý do (ví dụ timeout).
Không gán trạng thái cho dịch vụ đó dù người dùng yêu cầu giả định. Vẫn báo cáo
kết quả có bằng chứng cho các dịch vụ đã kiểm tra được."
- Vì sao chọn thay đổi này: Vì agent đã ghi Cloudflare Spectrum là “Operational” theo yêu cầu của người dùng nên phải chỉnh trong system prompt.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 91.05 | 93.86 |
| Điểm case đã chọn | 29 |  100|
| Số model calls |75  | 69 |
| Token, chi phí nếu có |154.414 token  | 153.757 token |

- Lỗi đã giải quyết (H7): Trước khi sửa, agent kết luận Cloudflare Spectrum đang hoạt động theo giả định của người dùng, dù truy vấn bị timeout. Sau khi bổ sung quy tắc xử lý truy vấn thất bại vào system prompt, agent báo rõ chưa xác minh được trạng thái Cloudflare Spectrum và vẫn trình bày kết quả GitHub có bằng chứng từ snapshot. Điểm H7 tăng từ 29 lên 100.
- Lỗi còn lại (H3): Khi người dùng cung cấp mã incident cụ thể, agent chỉ gọi get_status để xem trạng thái tổng quan thay vì dùng get_incident_details để tra đúng mã. Agent tránh xác nhận sự cố toàn cầu và không bịa URL, nhưng việc thiếu lời gọi tool bắt buộc khiến điểm H3 vẫn bị giới hạn ở 50/100 trong cả hai lần chạy.

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.

Điểm judge: Điểm số là đánh giá ước lượng. Judge có thể bỏ sót lỗi hoặc chấm khác nhau giữa các lần chạy. Vì agent và judge dùng cùng một model, kết quả có thể bị thiên lệch. Cần đối chiếu điểm với câu trả lời, trace và dữ liệu từ tool.
Dữ liệu snapshot: Kết quả chỉ phản ánh dữ liệu tại thời điểm lưu snapshot. Công cụ tra cứu trong phạm vi 50 incident gần nhất của mỗi nhà cung cấp, và một số cập nhật có thể bị cắt. Vì vậy, kết quả tra cứu rỗng chỉ cho biết công cụ không tìm thấy kết quả trong phạm vi đã kiểm tra.
Số lần chạy: Mỗi phiên bản trong cặp eval_before.json và eval_after.json chỉ được đánh giá đầy đủ một lần. Điểm tổng tăng từ 91,05 lên 93,86 và H7 tăng từ 29 lên 100 trong lần so sánh này. Điểm H6 từng dao động từ 0 lên 92 với cùng phiên bản code, nên cần chạy lặp lại để đánh giá mức độ ổn định của cải tiến.