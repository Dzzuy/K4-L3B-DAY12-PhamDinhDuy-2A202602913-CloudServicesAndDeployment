# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng mẫu trả lời bằng nội dung của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phạm Đình Duy; Mã học viên: 2A202602913

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy, nếu quên khai báo `AGENT_API_KEY` mà ứng dụng dùng mặc định `changeme`, service vẫn hoạt động nhưng người biết khóa mặc định có thể gọi `/ask` và làm phát sinh chi phí. Để trường này bắt buộc, ứng dụng dừng ngay khi khởi động. Lỗi được thấy trong log deploy trước khi service mở ra Internet.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log mình thu được là `{"event":"ask_completed","level":"info","timestamp":"2026-09-29T04:58:44.526490+00:00","user_id":"exercise-proof","tokens_in":3,"tokens_out":41,"cost_usd":2.505e-05}`. Có thể lọc hoặc đếm số request từ log này theo `user_id`, và theo dõi token, chi phí để phát hiện user hoặc thời điểm có mức tiêu thụ bất thường. Dòng `print("đã trả lời xong")` không có dữ liệu cấu trúc cho hai việc đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | Chưa đo lại sau khi Dockerfile gốc đã được thay |
| Multi-stage | 267 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi đã đo image multi-stage là 267 MB. Dockerfile một stage ban đầu không còn được giữ như một file độc lập nên không ghi số liệu suy đoán. Về nguyên tắc, stage builder giữ dependency/công cụ cài đặt chỉ trong lúc build; runtime chỉ nhận `/install` cùng source cần chạy, nên giảm các layer và thành phần không cần khi vận hành.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Nếu chỉ sửa `app/main.py`, layer copy `requirements.txt` và layer `pip install` vẫn dùng cache vì requirements không đổi; layer copy source và các layer sau nó chạy lại. Nếu đặt `COPY . .` trước `RUN pip install`, mọi thay đổi source làm mất cache của layer cài dependency và build chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng có lỗ hổng cho phép chạy lệnh trong container, process chạy root có quyền đọc/ghi mọi thứ mà container cho phép và có thể khai thác cấu hình sai hoặc volume để tăng mức ảnh hưởng sang host. `USER appuser` làm process ứng dụng bắt đầu với quyền hạn thấp, nên lỗ hổng không mặc nhiên có quyền root trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi 10 request lúc 10:00:59 và 10 request lúc 10:01:01, tức tối đa 20 request trong khoảng hai giây. Bộ đếm theo phút đồng hồ reset ở giây 00, còn sliding window luôn nhìn lại 60 giây gần nhất nên lần thứ 11 trong ví dụ bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tần suất request, còn cost guard giới hạn tổng chi phí của từng user theo tháng. Ví dụ user gửi request dưới 10 lần/phút nhưng mỗi request rất lớn: rate limit cho qua nhưng cost guard phải chặn khi hết ngân sách. Ngược lại, user có thể chưa tiêu gần hết ngân sách nhưng gửi quá 10 request/phút; rate limit chặn trước, còn cost guard vẫn còn quota.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu endpoint chung kiểm tra Redis và Redis mất kết nối trong 30 giây, probe của cả ba container đều trả lỗi. Orchestrator có thể coi từng container không còn sống và restart chúng, trong khi lỗi thực tế chỉ là dependency tạm thời. Tách `/health` giúp process vẫn sống; `/ready` trả 503 để load balancer ngừng gửi traffic mới cho các instance chưa sẵn sàng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Lịch sử được lưu theo `history:<user_id>` trong Redis nên mọi instance dùng chung state; `history_length` sẽ tiếp tục tăng theo các request của cùng user, dù request được đưa tới instance khác. Nếu dùng dict Python, mỗi instance có dict riêng và số này sẽ nhảy về 0 hoặc giá trị thấp hơn khi request tới instance chưa từng xử lý user đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi kiểm tra Railway, `/health`, `/ready` và `/ask` ban đầu trả 404. Nguyên nhân là Public URL có dấu `/` ở cuối trong khi lệnh curl lại thêm `/health`, tạo thành đường dẫn `//health`. Service vẫn Online nên tôi kiểm tra lại URL và request path thay vì giả định build/deploy hỏng. Sau khi bỏ dấu `/` cuối URL, `/health` và `/ready` trả 200, còn `/ask` không có key trả 401.
