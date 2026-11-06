# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác

- **MSSV:** 2A202604093
- **Họ và tên:** NgoHongHoang
- **SEED:** 7613

## 1. Bằng chứng

| Tệp | Trạng thái |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) | Đã chạy và lưu output của các phần có thể chạy |
| [`lab1_report.json`](lab1_report.json) | Đã tạo; 15/15 kiểm tra đạt, có 10 dự đoán |
| [`run_proof.png`](run_proof.png) | Ảnh chụp ô báo cáo cuối, xác nhận 15/15 kiểm tra |
| [`hand_calc.jpg`](hand_calc.jpg) | Đã cập nhật bằng ảnh tính tay mới với năm phép tính |

Phần gọi LLM không có kết quả đủ mẫu: `gpt-6-luna` chạm giới hạn 50 yêu cầu/ngày trong thí nghiệm 4.A; `gpt-6.1-terra` báo tài khoản hết credits. Notebook ghi rõ giới hạn và tiếp tục chạy các phần không dùng API. Không suy diễn F1 hoặc chi phí LLM từ kết quả dở dang.

## 2. Kết quả chính

Các hàng dưới dùng cùng tập test; F1 SUB đo trên 40 tin cân bằng. Độ trễ là p95 khi gọi từng tin từ văn bản thô trên máy chạy bài.

| Cách | F1 test | F1 SUB 40 | p95 ms/tin | USD/1.000 tin |
| --- | ---: | ---: | ---: | ---: |
| BoW + LogReg | 0,896 | 0,857 | 0,061 | 0 |
| BoW + MLP | 0,956 | 0,947 | 0,126 | 0 |
| MiniLM + kNN | 0,936 | 0,923 | 33,442 | 0 |
| MiniLM + LogReg | 0,952 | 0,919 | 65,559 | 0 |
| MiniLM + MLP head | 0,971 | 0,947 | 16,690 | 0 |

Không có hàng LLM vì chưa phân loại đủ 40 tin. Các chi phí 0 ở bảng là **chi phí API**, không bao gồm máy chủ, điện và vận hành.

## 3. Dự đoán và thực tế

1. **`emb_tricky`:** Dự đoán Encoder + LogReg có thể đúng nhiều hơn BoW trên sáu câu ngụy trang. Kết quả hai cách cùng đúng **4/6**. Biểu diễn nghĩa giúp nhận ra “no cost” là lời mời quảng cáo, nhưng encoder lại bỏ sót câu “complimentary holiday” mà BoW bắt được; sáu câu quá ít để khẳng định mô hình nào tốt hơn.
2. **`dl_curve`:** Dự đoán MLP chỉ có thể nhỉnh hơn LogReg một chút vì cùng dùng BoW. Kết quả F1 test tăng từ **0,896 lên 0,956** (chênh 0,060), lớn hơn dự kiến. Validation loss thấp nhất ở epoch thứ 8 rồi tăng trong khi train loss giảm; nên dừng sớm. Hai cách còn khác thuật toán tối ưu, vì vậy không quy toàn bộ chênh lệch cho lớp ẩn.
3. **`ml_f1`:** Dự đoán F1 test khoảng **0,85** và precision cao hơn recall. Thực tế F1 **0,896**, precision **1,000**, recall **0,811**. Hướng so sánh đúng nhưng F1 cao hơn dự đoán; các từ khóa trong tập test cùng phân phối với train giúp mô hình bắt spam tốt hơn, trong khi vẫn bỏ sót 20/106 spam.

## 4. Phân tích 50 case khó

BoW + LogReg tụt mạnh nhất: F1 trên SUB thường **0,857** xuống **0,286** trên 50 case khó. Mô hình đúng **0/10** spam giọng tự nhiên và **1/6** tin né từ khóa vì chỉ thấy các token riêng lẻ, không thấy dụng ý của cả câu. Tập test thường cùng phân phối với train và chỉ loại trùng tuyệt đối, nên điểm test chưa phản ánh đầy đủ các kiểu lừa đảo mới.

- **Case #12:** Tin spam mở đầu thân thiện như người quen, sau đó dẫn tới lời mời dịch vụ. Các từ đầu câu giống hội thoại bình thường nên BoW và các mô hình được kiểm đều đoán sai.
- **Case #41:** Tin OTP hợp lệ bị MiniLM + MLP head đoán là spam. Cấu trúc tự động và lời cảnh báo “do not share” giống một số tin lừa đảo; đây là lỗi chặn nhầm cần ưu tiên giảm trong sản phẩm thật.

Năm case tự tạo ở ô bonus làm MiniLM + MLP head sai **4/5**: #100, #102 là quảng cáo viết như tin người quen bị bỏ sót; #103, #104 là tin ham chứa từ “FREE/WIN/CASH” bị chặn nhầm. Bộ case viết tay giúp tìm kiểu lỗi, không dùng để ước lượng tỷ lệ lỗi ngoài đời.

## 5. Thử thách cá nhân

**Đề #6 theo SEED 7613:** Yêu cầu LLM trả JSON `{"label": ..., "reason": ...}`, phân tích nhãn từ JSON và so với câu trả lời một từ trên 10 tin SUB. Notebook đã có `build_json_prompt`, `parse_json_label` và các kiểm tra cho JSON hợp lệ, nhãn không hợp lệ, chuỗi không phải JSON. Ô thử thách sẽ gọi cả hai cách trên cùng 10 tin, in F1, số token ước tính và USD khi API đủ quota.

**Kết quả hiện có:** parser qua các kiểm tra cục bộ. Chưa có kết quả 10 tin vì tài khoản chạm giới hạn yêu cầu và hết credits. JSON dự kiến tốn thêm output token; phần `reason` chỉ đáng giá khi cần xem hoặc giải trình quyết định. Không điền số đo giả định.

## 6. Phương án cho sản phẩm thật

Yêu cầu là 10 triệu SMS/ngày (trung bình khoảng 116 tin/giây) và p95 dưới 50 ms/tin. Bốn trong năm mô hình không dùng API đạt ngưỡng p95 trên máy thử; MiniLM + LogReg vượt SLA với p95 65,559 ms ở lần đo này. **BoW + MLP** là tuyến đầu phù hợp ở lần đo này: F1 test **0,956**, precision **1,000**, p95 **0,126 ms/tin**. MiniLM + MLP head tăng F1 test lên **0,971** nhưng chậm hơn và có hai tin ham bị chặn nhầm. Các số đo cần xác nhận lại trên máy chủ thật, ở tải cao và dữ liệu đại diện.

Có thể dùng BoW + MLP để xử lý tin chắc chắn, chuyển tin gần ngưỡng sang encoder hoặc kiểm tra thủ công; OTP và tin giao dịch cần ngưỡng chặn thận trọng hơn. Nếu dùng LLM cho mọi tin, theo giả định trong bài (100 input + 1 output token, giá `gpt-6-luna` đã cho) chi phí lý thuyết là **105 USD/ngày**, khoảng **3.150 USD/30 ngày**. Prompt thực tế dài hơn, và chưa có số độ trễ/F1 LLM đủ mẫu để chọn nó làm tuyến chính.

## 7. AI log và điều đã học

| Phần | Đã hỏi AI | AI đã hỗ trợ | Đã kiểm chứng / giới hạn |
| --- | --- | --- | --- |
| Notebook Phần 1–3 | Điền mã, dự đoán và giải thích | Hoàn thiện các ô còn thiếu và diễn giải kết quả | Chạy bộ kiểm tra, đo F1/độ trễ; cần tự hiểu và trình bày được từng hàm khi vấn đáp |
| Phần 4 và thử thách #6 | Viết prompt, parser JSON, so sánh chi phí | Viết mã và các trường hợp kiểm tra parser | Kiểm tra giả lập đạt; API thật bị giới hạn nên chưa có F1/chi phí thực nghiệm |
| Tổng hợp và báo cáo | So kết quả thường với 50 case khó, chọn phương án triển khai | Tổng hợp bảng, nêu lỗi và đánh đổi | Đối chiếu với `lab1_report.json`, output notebook và các case sai |

Điều nổi bật là accuracy cao trên tập test thường không bảo đảm mô hình xử lý được spam viết tự nhiên hoặc tin nhắn né từ khóa. F1 cũng phụ thuộc tập đánh giá: chỉ 40 tin trong SUB khiến vài dự đoán sai có thể đổi thứ hạng mô hình. Khi lọc tin thật, chặn nhầm OTP đáng quan tâm hơn một vài điểm F1, nên cần xem lỗi theo nhóm và đặt ngưỡng phù hợp.
