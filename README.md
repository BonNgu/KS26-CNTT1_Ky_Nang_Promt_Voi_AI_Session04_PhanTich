**So sánh Trade-off các giải pháp ra lệnh cho Kỹ thuật Prompting Nâng cao (Logic & Chuỗi)**

**1. Mục tiêu bài tập** 

Rèn luyện tư duy phản biện. Sinh viên đề xuất 2 giải pháp viết prompt khác nhau cho chủ đề Kỹ thuật Prompting Nâng cao (Logic & Chuỗi) và so sánh xem phương án nào tối ưu nhất.

**2. Bối cảnh & yêu cầu bài toán**

Bài toán yêu cầu thực hiện tối ưu hóa quy trình Chain-of-Thought, Prompt Chaining, xử lý đứt gãy.

**3. Quy tắc nghiệp vụ**

Đưa ra Giải pháp 1 (đơn giản, ngắn gọn) và Giải pháp 2 (chi tiết, có nhiều ràng buộc).

**4. Ràng buộc & bẫy dữ liệu**

Lập bảng so sánh tiêu chí để đánh giá ưu nhược điểm rõ ràng.

**5. Yêu cầu bài làm chi tiết**

**Bước 1: Đề xuất 2 prompt mẫu**

Giải pháp 1: Prompt đơn giản (một lệnh, suy luận ngầm)
markdown
Bạn là nhân viên bán hàng của Mây Store. Đọc tin nhắn khách bên dưới, kiểm tra tồn kho,
tính tổng tiền (có áp mã giảm giá và phí ship nếu hợp lệ) rồi trả lời khách bằng tiếng Việt thân thiện.

Danh mục: AT01 Áo thun trắng 150k (S:5, M:0, L:10); QJ02 Quần jeans 350k (28:3, 29:2, 30:0); VD03 Váy hoa nhí 280k (S:4, M:6).
SALE10: giảm 10% cho đơn từ 500k. Ship: nội thành HCM 25k, tỉnh khác 35k, miễn phí từ 700k.

Tin nhắn khách: {{customer_message}}
Giải pháp 2: Prompt chi tiết (Chain-of-Thought + Prompt Chaining + xử lý lỗi)

Gồm chuỗi 3 prompt, mỗi prompt một nhiệm vụ, đầu ra là JSON để kiểm tra giữa các bước.

P1: Trích xuất

markdown
# VAI TRÒ
Nhân viên nhập liệu cẩn thận của Mây Store.

# NHIỆM VỤ
Trích xuất thông tin đặt hàng từ tin nhắn thành JSON.

# QUY TẮC
1. Chỉ lấy thông tin CÓ TRONG tin nhắn. Không có → null. KHÔNG suy đoán.
2. Không phải yêu cầu đặt hàng → trả {"is_order": false} và dừng.
3. Số lượng mơ hồ ("vài", "mấy") → qty = null.
4. Liệt kê trường thiếu (customer_name, phone, address, size, qty) vào "missing_fields".

# ĐẦU RA
{"is_order":bool,"customer_name":str|null,"phone":str|null,"address":str|null,
 "items":[{"raw_name":str,"size":str|null,"qty":int|null}],
 "coupon":str|null,"missing_fields":[str]}

# TIN NHẮN
"""{{customer_message}}"""

P2: Kiểm tra và tính toán 

# VAI TRÒ
Kế toán kiêm thủ kho, chính xác từng con số.

# DỮ LIỆU
(Danh mục và chính sách như trên)

# ĐẦU VÀO
{{json_from_p1}}

# SUY LUẬN TỪNG BƯỚC
B1: Kiểm tra tên, SĐT (đủ 10 số, bắt đầu bằng 0?), địa chỉ.
B2: Từng sản phẩm: khớp SKU → kiểm tra size → so tồn kho.
B3: Chỉ tính tiền các món CÒN ĐỦ HÀNG.
B4: Kiểm tra điều kiện mã giảm giá, tính ship.
B5: Chốt status.

# XỬ LÝ LỖI
- Thiếu/sai liên hệ → MISSING_NAME / INVALID_PHONE / MISSING_ADDRESS
- Sản phẩm không khớp chắc chắn → UNKNOWN_PRODUCT (không tự chọn)
- Tồn < số lượng → OUT_OF_STOCK + gợi ý size còn hàng
- qty = null → AMBIGUOUS_QTY
- Mã không đủ điều kiện → discount = 0, ghi lý do
- status: OK | NEED_INFO | OUT_OF_STOCK | PARTIAL; nếu ≠ OK thì "provisional": true

# ĐẦU RA
{"reasoning":[str],"status":str,"item_checks":[...],"errors":[{"code":str,"detail":str}],
 "pricing":{"subtotal":int,"discount":int,"shipping":int,"total":int,"provisional":bool}}

P3: Soạn phản hồi

Bạn là CSKH Mây Store, xưng "shop" – "bạn", tối đa 1 emoji, tối đa 120 từ.
Đầu vào: {{json_from_p2}}
- status = OK → xác nhận đơn, nêu tổng tiền, hỏi khách xác nhận.
- status ≠ OK → KHÔNG chốt đơn; nêu từng vấn đề, đề xuất thay thế, hỏi đúng thông tin còn thiếu.
- Chỉ dùng số liệu có trong đầu vào; không dùng thuật ngữ kỹ thuật.

**Bước 2: Lập bảng so sánh Trade-off**

Tiêu chí	Giải pháp 1 (Đơn giản)	Giải pháp 2 (Chi tiết, chuỗi)
Độ dễ hiểu / dễ viết	Cao: một đoạn ngắn, viết trong vài phút	Trung bình: 3 prompt, cần hiểu luồng dữ liệu giữa các bước
Tốn token	Ít: 1 lần gọi, prompt ngắn	Nhiều hơn: 3 lần gọi, prompt dài, thêm JSON trung gian
Độ trễ (latency)	Thấp: 1 lượt phản hồi	Cao hơn: 3 lượt nối tiếp
Chặn lỗi biên (thiếu/sai dữ liệu)	Trung bình: dễ tự “đoán” khi thiếu tên, SĐT, số lượng	Cực tốt: có điều kiện lỗi rõ cho từng bẫy
Độ chính xác tính toán	Dễ sai khi nhiều bước (tồn kho, mã giảm, ship gộp một lượt)	Cao: tách bước, tính trên món còn hàng
Khả năng kiểm tra, debug	Thấp: chỉ thấy kết quả cuối, không biết sai ở đâu	Cao: mỗi bước có JSON để soi
Khả năng tự động hóa (nối code)	Khó: đầu ra là văn bản tự do, khó parse	Dễ: JSON, chèn được validate hoặc tra kho thật
Xử lý đứt gãy	Không có: lỗi nửa chừng vẫn trả lời như bình thường	Có: is_order=false hoặc status ≠ OK thì dừng, không chốt đơn
Bảo trì, mở rộng	Phải sửa cả khối prompt khi đổi chính sách	Sửa đúng module (ví dụ chỉ đổi P2)
Phù hợp khi	Thử nghiệm nhanh, đơn giản, ít rủi ro	Hệ thống thật, đơn có tiền, cần độ tin cậy


**Bước 3: Lựa chọn và chạy thử**

3.1. Phương án được chọn: Giải pháp 2

Lý do:

Sai sót có hậu quả tiền bạc. Chốt nhầm đơn hết hàng hoặc áp sai mã giảm giá gây thiệt hại và mất uy tín, nên độ chính xác quan trọng hơn vài trăm token tiết kiệm được.
Bài toán có nhiều bẫy dữ liệu (thiếu tên, SĐT sai, hết hàng, mã không hợp lệ). Chỉ Giải pháp 2 có điều kiện lỗi tường minh cho từng bẫy.
Cần tự động hóa thật. JSON trung gian cho phép nối với kho hàng, kiểm tra schema, và dừng sớm khi dữ liệu hỏng.
Chi phí tăng chấp nhận được. Có thể giảm bằng cách dùng mô hình nhỏ cho P1 và P3, chỉ dùng mô hình mạnh cho P2.
3.2. Chạy thử cả hai trên cùng một tin nhắn

Tin nhắn khách (có nhiều bẫy):

“Shop ơi mình lấy 2 áo thun trắng size M với 1 quần jean ống suông size 30. Giao 12 Nguyễn Huệ Q1. SĐT 0912 345 67. Có mã SALE10 á”

Bẫy: áo size M hết hàng, quần size 30 hết hàng, SĐT thiếu 1 số, không có tên.

Kết quả Giải pháp 1 (một prompt):

Chào bạn! Shop xác nhận đơn: 2 áo thun trắng size M (300.000đ) + 1 quần jeans size 30 (350.000đ) = 650.000đ. Áp mã SALE10 giảm 10% còn 585.000đ, cộng ship nội thành 25.000đ, tổng 610.000đ. Shop sẽ giao về 12 Nguyễn Huệ, Q1 nhé!

Đánh giá: Sai nghiêm trọng. Bỏ qua việc cả hai món đều hết hàng, vẫn tính tiền và chốt đơn, không phát hiện SĐT thiếu số, không hỏi tên người nhận. Đây chính là loại lỗi “AI dễ bỏ sót” mà đề bài cảnh báo.

Kết quả Giải pháp 2 (chuỗi 3 prompt):

P1:

{
  "is_order": true, "customer_name": null, "phone": "0912 345 67",
  "address": "12 Nguyễn Huệ Q1",
  "items": [
    {"raw_name": "áo thun trắng", "size": "M", "qty": 2},
    {"raw_name": "quần jean ống suông", "size": "30", "qty": 1}
  ],
  "coupon": "SALE10", "missing_fields": ["customer_name"]
}

P2:

{
  "reasoning": [
    "B1: Thiếu tên. SĐT sau khi bỏ khoảng trắng chỉ có 9 số, không hợp lệ. Địa chỉ Q1 là nội thành.",
    "B2: AT01 size M tồn 0 < 2: hết hàng. QJ02 size 30 tồn 0 < 1: hết hàng.",
    "B3: Không món nào đủ hàng, subtotal = 0.",
    "B4: subtotal 0 < 500.000đ nên SALE10 không áp dụng; chưa có hàng nên shipping = 0.",
    "B5: Có cả lỗi liên hệ và hết hàng: status = PARTIAL."
  ],
  "status": "PARTIAL",
  "errors": [
    {"code": "MISSING_NAME", "detail": "Chưa có tên người nhận"},
    {"code": "INVALID_PHONE", "detail": "SĐT chỉ có 9 chữ số"},
    {"code": "OUT_OF_STOCK", "detail": "Áo thun trắng M hết; còn S, L"},
    {"code": "OUT_OF_STOCK", "detail": "Quần jeans size 30 hết; còn 28, 29"},
    {"code": "COUPON_NOT_APPLIED", "detail": "Chưa đủ 500.000đ"}
  ],
  "pricing": {"subtotal": 0, "discount": 0, "shipping": 0, "total": 0, "provisional": true}
}

P3 :

Chào bạn, shop đã nhận yêu cầu của bạn ạ 😊 Hiện áo thun trắng size M và quần jeans size 30 đã hết hàng. Shop còn áo size S hoặc L, quần size 28 hoặc 29, bạn muốn đổi sang size nào ạ? Bạn cho shop xin họ tên người nhận và kiểm tra lại số điện thoại (đang thiếu một số) để shop chốt đơn nhé. Mã SALE10 áp dụng cho đơn từ 500.000đ.

3.3. Tổng kết kết quả chạy thử
Bẫy dữ liệu	Giải pháp 1	Giải pháp 2
Hết hàng (size M, size 30)	Bỏ sót	Phát hiện, gợi ý size thay thế
SĐT thiếu số	Bỏ sót	Phát hiện
Thiếu tên người nhận	Bỏ sót	Phát hiện, hỏi lại
Mã giảm giá không đủ điều kiện	Áp sai	Không áp, giải thích rõ
Chốt đơn khi dữ liệu lỗi	Có (sai)	Không (dừng đúng chỗ)

Kết luận: Với bài toán có tiền bạc và nhiều bẫy dữ liệu, Giải pháp 2 là tối ưu dù tốn token và thời gian hơn, vì nó biến “AI tự đoán” thành “AI suy luận có kiểm soát”. Giải pháp 1 chỉ nên dùng cho prototype nhanh hoặc tác vụ ít rủi ro.
