# REPORT — Lab Day 12

| Thông tin | Giá trị |
|---|---|
| Họ và tên | DinhBaoHung |
| MSSV (đúng bằng `--student-id` đã chạy) | 2A202602524 |
| GitHub | hungdinh2611 |
| Seed chia dữ liệu / seed mô hình | 9542 / 6022, 6023, 6024, 6025, 6026 |
| Số seed huấn luyện (`--seeds`) | 5 |

Chạy đầy đủ bước 0–7 bằng `python run_all.py --student-id 2A202602524`, không dùng `--quick`.
Bằng chứng gốc nằm trong [evidence/](evidence/); không thay đổi ngưỡng cổng hoặc bộ kiểm thử.
Môi trường: Python 3.10.4, NumPy 1.23.1, Pandas 2.3.3, Matplotlib 3.5.3.

## Kết quả cuối

L2 là trung bình sai số vị trí tại 1, 2 và 3 giây. Với mô hình học, bảng ghi min–max của 5 seed; mô hình cố định chỉ có một giá trị.
Tỷ lệ va chạm tính trên 2883 quỹ đạo kiểm thử, không phải tỷ lệ khung hình.

| Bộ điều khiển | L2 vòng hở (m), min–max | Va chạm vòng kín trên log (%), min–max | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1.2549 | 19.840 | BLOCK |
| IDM | 1.5287 | 0.069 | Baseline |
| EgoMLP | 0.7567–0.7638 | 14.083–24.558 | BLOCK |
| PercMLP | 0.6184–0.6237 | 0.763–1.075 | BLOCK |
| PercMLP+ | 0.8051–0.8465 | 0.312–0.382 | RELEASE (7/7) |

Nguồn: [openloop.json](evidence/openloop.json), [closedloop.json](evidence/closedloop.json), [gate.json](evidence/gate.json).
Chi tiết L2 từng seed của PercMLP+ được tính lại từ checkpoint và lưu tự động trong [report_metrics.json](evidence/report_metrics.json); trung bình đã được đối chiếu với `openloop.json`.

## Câu 1 · Dữ liệu

Dữ liệu có 992,679 dòng thuộc 9,993 quỹ đạo ban đầu; loại 383 quỹ đạo trùng còn 9,610 quỹ đạo.
Tương quan giữa `Acc_LV` và đạo hàm tốc độ xe dẫn chỉ 0.04535, trong khi cột xe theo sau đạt 0.92909; vì vậy dùng `Acc_LV` sẽ khai thác nhầm sự kiện phanh.
Thay thế bằng `Speed_LV` được làm mượt trung bình trượt 5 mẫu rồi lấy đạo hàm theo thời gian với dt = 0.1 s; phần đệm NaN không được coi là dữ liệu thật.
Kiểm tra hình học cho độ lệch giữa hiệu vị trí và khoảng cách bằng 4.5 m, đúng chiều dài xe; bỏ số hạng này sẽ hiểu sai khoảng cách cản trước–cản sau.
Loại trùng trước khi chia và chia theo toàn bộ quỹ đạo để các khung lân cận hoặc bản sao của cùng một chuyển động không lọt vào cả train và test, tránh điểm đánh giá lạc quan do rò rỉ.
Nguồn: [data_audit.json](evidence/data_audit.json).

## Câu 2 · Phát lại

Chạy lại cùng mã cho 0/600 đoạn khác bit; refactor float32 khác bit ở 600 đoạn nhưng 0 đoạn vượt dung sai, sai lệch tuyệt đối lớn nhất 0.00033752 m/s².
Dùng điều kiện `abs(new − golden) <= 0.001 + 0.001 * abs(golden)`, coi hai NaN đệm cùng vị trí là bằng nhau: thay đổi thứ tự phép tính không nên bị nhầm thành đổi hành vi.
AEB làm 8 đoạn vượt dung sai; tại đoạn 58, khung 98, gap = 4.3033 m và closing = 2.9189 m/s cho TTC = 1.4743 s < 1.5 s, lệnh đổi từ -5.4363 thành -6.0 m/s².
Sai khác xuất hiện khi TTC chạm ngưỡng và IDM chưa phanh mạnh bằng AEB; nếu IDM đã yêu cầu phanh mạnh hơn −6 thì phép `min` không đổi lệnh.
Phát nhanh nhất không có khung trễ/rớt; phát thời gian thật có 582 trễ và 579 rớt trên 59,641 khung, làm 370 đoạn vượt dung sai vì phải giữ lệnh cũ khi xử lý chưa kịp.
Nguồn: [replay.json](evidence/replay.json), [replay_aeb_frames.json](evidence/replay_aeb_frames.json).

## Câu 3 · Kịch bản

Lưới chạy cruise 12.0 m/s, gap 2.7601–59.3123 m, closing 0–12.0 m/s, tổng 2700 ô (60 × 45).
Biên vật lý là `gap < dv * t_react + dv² / (2 * b_max)` với `b_max = 9 m/s²`: quãng đường mất trong lúc chờ cộng khoảng cách cần để triệt tiêu tốc độ tiến gần; dv = 0 không thuộc vùng bất khả thi.
Vùng không ai tránh được tăng từ 2.222% lên 7.444% khi trễ 0 → 0.6 s; vùng va chạm IDM tăng đúng 5.222 điểm phần trăm, còn phần tránh được nhưng IDM vẫn đâm bằng 0%.
FullBrake khớp biên vật lý với 0 ô lệch ngoài vùng sát biên ở cả hai độ trễ; đây là kiểm chứng số học của mô phỏng, không phải bằng chứng an toàn ngoài đời.
Đã khai thác 614 ca phanh; ngưỡng phanh người lái ở phân vị 99.9% là 3.6248 m/s², trong khi 8.556% lưới đòi hỏi mạnh hơn ngưỡng đó ngay cả chưa tính trễ: log thiếu độ phủ vùng khó.
Nguồn: [scenarios.json](evidence/scenarios.json), [scenarios_idm.png](evidence/scenarios_idm.png). Phân vị 99.9% là ngưỡng thực nghiệm, không phải giá trị phanh cực đại tuyệt đối.

## Câu 4 · Vòng hở và vòng kín

| Bộ điều khiển | L2 trung bình qua seed (m) | Hạng L2 | Va chạm trung bình qua seed (%) | Hạng vòng kín |
|---|---|---|---|---|
| const-vel | 1.2549 | 3 | 19.840 | 4 |
| IDM | 1.5287 | 4 | 0.069 | 1 |
| EgoMLP | 0.7588 | 2 | 18.959 | 3 |
| PercMLP | 0.6214 | 1 | 0.902 | 2 |

Ba cặp đảo thứ hạng là constant-velocity/IDM, IDM/EgoMLP và IDM/PercMLP: IDM từ hạng 4 theo L2 lên hạng 1 theo va chạm.
IDM có L2 1.5287 m vì dự báo giả định xe dẫn giữ tốc độ và không bắt chước chính xác người lái; khi tự lái nó cập nhật khoảng cách và tốc độ tương đối liên tục nên vẫn tránh được phần lớn va chạm.
EgoMLP chỉ quan sát lịch sử tốc độ bản thân, không biết gap hoặc xe dẫn đang chậm lại; L2 0.7588 m thấp hơn constant-velocity nhưng sai số tích lũy sau khi tự lái tạo trạng thái ngoài phân phối huấn luyện.
Lát cắt xe dẫn sắp phanh chỉ chiếm 4.447% cửa sổ; PercMLP đạt L2 0.6214 m toàn bộ nhưng 0.9163 m trên lát cắt, nên trung bình che bớt lỗi của tình huống khó.
Vòng hở đo mức khớp log còn vòng kín đo hậu quả của quyết định liên tiếp; vì vậy phải đối chiếu cả hai, đồng thời tách ca bất khả thi khỏi lỗi điều khiển.
Nguồn: [openloop.json](evidence/openloop.json), [closedloop.json](evidence/closedloop.json), [openloop_vs_closedloop.png](evidence/openloop_vs_closedloop.png).

## Câu 5 · Một lần chạy chưa đủ

Trên cùng 2883 quỹ đạo, EgoMLP có seed tốt nhất 6025 đạt 14.083% và seed xấu nhất 6023 đạt 24.558%, chênh 10.475 điểm phần trăm.
Wilson 95% của seed tốt là [12.861; 15.400]%, còn seed xấu là [23.021; 26.162]%; hai khoảng không chồng nhau, cho thấy độ nhạy với seed không thể bỏ qua.
Wilson mô tả bất định của tỷ lệ trên tập quỹ đạo hữu hạn, không thay thế dải biến thiên do huấn luyện nhiều seed và không phải xác suất mô hình an toàn 95%.
Nếu chỉ chọn seed tốt, có thể báo EgoMLP tốt hơn constant-velocity (19.840%); chọn seed xấu lại cho kết luận ngược, nên chọn một lần chạy sẽ dễ gây thiên lệch.
Tôi báo cả min–max và khoảng tin cậy; cổng sử dụng seed xấu nhất và cận trên, đồng thời R5 chặn EgoMLP vì dải biến thiên vượt 1 điểm phần trăm. Các log cũng có thể phụ thuộc nhau, nên Wilson không bao quát mọi nguồn bất định.
Nguồn: [closedloop.json](evidence/closedloop.json), [gate.json](evidence/gate.json).

## Câu 6 · Tác nhân phản ứng

Bảng dưới dùng độ trễ ego 0.6 s; mỗi tỷ lệ là phần trăm của toàn bộ 2700 ô, lấy kết quả seed được pipeline chọn theo va chạm bất kỳ trong thế giới phản ứng.

| Bộ điều khiển | Bất kỳ va chạm, xe sau phát lại (%) | Bất kỳ va chạm, xe sau phản ứng (%) | Va chạm sau, phát lại (%) | Va chạm sau, phản ứng (%) |
|---|---|---|---|---|
| const-vel | 77.630 | 77.630 | 0.000 | 0.000 |
| IDM | 75.741 | 7.444 | 68.296 | 0.000 |
| EgoMLP | 79.037 | 79.037 | 0.000 | 0.000 |
| PercMLP | 75.148 | 28.667 | 46.519 | 0.000 |
| PercMLP+ | 75.185 | 7.963 | 67.222 | 0.000 |

Hiệu chỉnh cho `T = 1.4 s`, với W1 phân phối time headway = 0.3507, tốt hơn xe không phản ứng = 0.8539; người lái có time headway trung vị 1.8717 s.
Xe sau bắt đầu cách 18.8 m và phản ứng trễ 0.6 s; W1 giảm cho thấy một đặc điểm hành vi khớp log hơn, chưa chứng minh toàn bộ hành vi chân thực.
IDM bị tính va chạm sau ở 68.296% lưới khi xe sau cứ giữ tốc độ bất chấp ego phanh; cho xe sau phản ứng thì tỷ lệ này còn 0.000%.
Đây là va chạm giả do giả định tác nhân không thích ứng với quỹ đạo mới của ego; nó làm IDM có vẻ gần constant-velocity khi xét va chạm bất kỳ, trong khi thế giới phản ứng phân biệt rõ hai bộ điều khiển.
Tôi đối chiếu cả va chạm trước/sau và cả hai thế giới để tránh phạt nhầm bộ điều khiển biết phanh; vẫn cần kiểm thử thêm nhiều kiểu tài xế và độ trễ.
Nguồn: [agents.json](evidence/agents.json).

## Câu 7 · Cổng phát hành và sửa

| Ứng viên | Quyết định | Quy tắc không đạt |
|---|---|---|
| const-vel | BLOCK | R2, R3, R4, R6 |
| EgoMLP | BLOCK | R2, R3, R4, R5, R6 |
| PercMLP | BLOCK | R2, R3, R4, R6 |
| PercMLP+ | RELEASE | Không có (7/7 đạt) |

PercMLP bị chặn bởi R2, R3, R4, R6: cận trên Wilson toàn log 1.522% > ngưỡng 1.069%; cận trên lát cắt phanh 8.847% > 6.064%; vùng tránh được nhưng vẫn đâm chiếm 21.222% toàn lưới > 2%; thế giới phản ứng 28.667% > 9.444%.
Hai nguyên nhân là thiếu kịch bản cắt vào khó trong log (R4/R6) và sai số bắt chước tích lũy khi mô hình đi vào trạng thái do chính nó tạo ra (R2/R3).
Bản sửa bổ sung 20,000 kịch bản cắt vào có nhãn IDM rồi chạy 2 vòng DAgger mỗi seed: thu trạng thái learner thực sự đi qua và dùng dự báo IDM tại chính trạng thái đó để học khả năng phục hồi.
Sau sửa, cận trên Wilson toàn log còn 0.682%, vùng tránh được nhưng vẫn đâm còn 0.519%; PercMLP+ qua đủ 7/7 quy tắc với nguyên ngưỡng, còn L2 tăng từ 0.6214 lên 0.8219 m — điểm bắt chước đẹp không đồng nghĩa hành vi an toàn hơn.
Qua cổng chỉ chứng minh không kém baseline trong phạm vi kiểm thử và các biên cho phép, chưa chứng minh tốt hơn IDM; mô hình đang học chính IDM. Ngoài dải cruise 4–16 m/s của các kịch bản cắt vào bổ sung, ở 20 m/s bản sửa vẫn đâm trong vùng tránh được 0.074–1.889% so với IDM 0.000%.
Tôi chấp nhận RELEASE trong phạm vi cổng của bài lab nhưng chưa đề xuất thay IDM ngoài đời; cần giữ một tập đánh giá độc lập chưa dùng để sửa mô hình, mở rộng tốc độ/kịch bản và chạy shadow mode trước quyết định phát hành thực tế.
Nguồn: [gate.json](evidence/gate.json), [fix.json](evidence/fix.json), [closedloop.json](evidence/closedloop.json), [openloop.json](evidence/openloop.json).

## Câu 8 · Bài lab này không chứng minh được gì

Thứ nhất, mô phỏng chỉ có chuyển động dọc 1D: cắt vào là thay đổi trạng thái tức thời, không có đánh lái, người đi bộ hay giao lộ; qua 2700 ô không chứng minh an toàn trong tương tác giao thông nhiều chiều.
Thứ hai, log 10 Hz chỉ kéo dài khoảng 7–11 s và xe dẫn phát lại không phản ứng với ego; hiệu chỉnh một tham số `T = 1.4 s` không bao quát tài xế mất tập trung, phanh khác nhau hoặc phản ứng xã hội, nên cần dữ liệu dài hơn và nhiều mô hình tác nhân.
Thứ ba, động lực học cố định giả định phanh tối đa 9 m/s², không có mặt đường trơn, dốc, hỏng cơ cấu hay cảm biến thực; thử nhiễu gap 1 m và tốc độ 0.5 m/s mới chỉ là kiểm tra hẹp, cần mô phỏng cảm biến, độ trễ thay đổi và hardware-in-the-loop.
Thứ tư, dùng cùng tập đánh giá/cổng để phát hiện lỗi rồi đánh giá bản sửa có thể dẫn đến tối ưu theo bài kiểm tra; cần bộ holdout độc lập, nhiều seed và nhiều nguồn dữ liệu, thay vì xem 7/7 quy tắc là bảo đảm an toàn phổ quát.
Thứ năm, lưới là lấy mẫu đều các tham số chứ không phải phân phối giao thông thực; 7.444% bất khả thi ở trễ 0.6 s không phải xác suất tai nạn ngoài đời, và Wilson trên 2883 quỹ đạo không mô tả hết rủi ro hiếm hay phụ thuộc giữa các log.
Phép thử cắt vào ở 20 m/s, ngoài dải cruise 4–16 m/s của dữ liệu cắt vào bổ sung, trong `fix.json` cho thấy cần kiểm thử biên miền vận hành, độ bám, độ trễ, comfort/jerk và lưu thông; chỉ sau đánh giá độc lập, thử nghiệm trên sân kín và shadow mode mới đủ cơ sở cân nhắc triển khai có giới hạn.
Nguồn: [scenarios.json](evidence/scenarios.json), [closedloop.json](evidence/closedloop.json), [agents.json](evidence/agents.json), [fix.json](evidence/fix.json).
