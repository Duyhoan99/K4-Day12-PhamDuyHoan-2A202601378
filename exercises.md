# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: điền câu trả lời bên dưới câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phạm Duy Hoàn  Mã học viên: 2A202601378

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Tình huống: Khi deploy lên Cloud nhưng quên cài biến môi trường API_TOKEN trong Dashboard. Nếu có mặc định "changeme", app vẫn khởi động bình thường nhưng cho phép bất kỳ ai gọi API miễn phí hoặc dùng token mặc định đó. Việc "chết sớm" báo lỗi ngay lúc khởi động giúp phát hiện và sửa cấu hình lập tức trước khi lộ API ra ngoài.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Log JSON: `{"event": "chat_completed", "severity": "INFO", "ts": "2026-08-10T16:50:00+00:00", "client_id": "sv01", "usd_cost": 0.0001}`
1. Lọc và nhóm log tự động trên hệ thống Cloud (ví dụ lọc theo severity: "ERROR" hoặc theo client_id).
2. Tính toán và đo lường chi phí phát sinh (usd_cost) cũng như vẽ biểu đồ giám sát theo thời gian thực.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.8 GB |
| Multi-stage | 320 MB |

Giải thích: Phần dung lượng chênh lệch (~1.5GB) bao gồm các công cụ biên dịch (gcc, build-essential), bộ nhớ đệm pip cache, và các file header không cần thiết cho môi trường runtime.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Layer cài đặt dependency `RUN pip install` được dùng lại từ cache. Chỉ các layer từ `COPY app ./app` trở đi mới phải chạy lại. Nếu đặt `COPY . .` lên trước `RUN pip install`, mỗi lần sửa 1 dòng code Docker sẽ làm mất cache và phải tải/cài lại toàn bộ thư viện python.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Kẻ tấn công lợi dụng lỗ hổng RCE trong app Python để thực thi lệnh hệ thống trong container. Vì container chạy root, họ thoát khỏi container (container escape) và có ngay quyền root trên máy host. Lệnh `USER appuser` chuyển process sang user thường (người dùng không có quyền quản trị), cắt đứt chuỗi tấn công ngay từ trong container.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

Header `WWW-Authenticate: Bearer` là bắt buộc theo chuẩn HTTP RFC 6750 để thông báo cho client biết phương thức xác thực yêu cầu. Trả về cùng một thông báo lỗi giúp bảo mật, tránh tiết lộ thông tin chi tiết cho kẻ tấn công biết họ đã dò đúng phần nào (scheme hay token).

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

Client gửi được tối đa 10 request trước khi bị 429. Nếu bỏ `min(capacity, ...)`, sau 10 phút im lặng xô sẽ tích lũy 100 token, cho phép client gửi liên tiếp 100 request một lúc làm quá tải hệ thống.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

Hạn mức $30/tháng: Thiệt hại tối đa $30 trong vài giờ và phải chờ sang tháng sau mới hồi phục.
Hạn mức $1/ngày: Thiệt hại tối đa $1 và service sẽ tự động khôi phục vào 00:00 UTC ngày tiếp theo.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

1. Redis mất kết nối 30s.
2. Cả 3 container đều báo unhealthy.
3. Orchestrator hiểu nhầm app bị sập và liên tục restart cả 3 container.
4. Khi Redis sống lại, ứng dụng vẫn sập do đang trong quá trình restart dây chuyền.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Lỗi gặp phải: Response 404/Timeout ở lượt gọi kiểm tra đầu tiên. 
Nguyên nhân: Môi trường Free Tier trên Render tự động "ngủ đông" (Spin down) khi không có request trong 15 phút.
Khắc phục: Gửi một request đánh thức app trước hoặc tăng thời gian timeout kiểm tra.
