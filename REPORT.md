# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Pham Huu Hai<br>
**MSSV:** 2A202602098<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

Quy ước trong báo cáo: `drive_008 #12` là dòng thứ 12 trong `labels/.../drive_008.txt` của gói YOLO đã nộp.

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_022`, `drive_033`, `drive_038` (huấn luyện), `drive_008` (thẩm định)
- Số vật thể thực tế: 75 (drive_008: 27, drive_022: 5, drive_033: 20, drive_038: 23). Con số này vượt mục tiêu 40–60; tôi giữ số thật, không xóa bớt hộp để vừa khoảng.
- Mã SHA-256 của gói YOLO của bạn: `cb46f358efa4328f75d354fd40b6417614a1a3beb33d7cb6f1493154f060e103`
- Mã SHA-256 của gói CVAT gốc của bạn: `e5a06e5be9f3e9a5479dd23a8b648233a276eaa429eae4f337c1a7e4ba714bea`
- Mã SHA-256 của bài độc lập đã khóa trước khi đối chiếu: `c7781bcd82a1e35f0e0b8b1eed319ad16c0ec4fab36a54cc48427babac9f4c47`
- Nguồn đối chiếu: bộ tham chiếu do người hướng dẫn thực hành cấp
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: `day2-reference-4img-v1`, nhận lúc 17:04 ngày 14/09/2026 (giờ máy, UTC+7)

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Tôi gán nhãn, tự kiểm tra và xuất bài lúc 16:20, rồi ghi mã SHA-256 của bài đã khóa ở trên trước khi nhận bộ tham chiếu lúc 17:04. Sau khi đối chiếu, tôi chỉ sửa các điểm có căn cứ ở mục 3: không thêm hay xóa hộp nào, và không đổi nhãn nào chỉ vì bộ tham chiếu ghi khác. Gói YOLO và gói CVAT gốc ở trên được xuất lại sau các sửa đổi đó; các file trong `day2_lab_outputs/` được tạo từ hai gói này.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| drive_008 #12 | bus | thân dài nối hai toa, nhiều cửa sổ và nhiều cửa lên xuống | `bus`: thân xe khách dài, nhiều cửa sổ hoặc hàng ghế |
| drive_008 #13 | truck | thùng ben chở đất tách riêng khỏi cabin, nhiều trục bánh | `truck`: có thùng, ben hoặc sàn chở hàng rõ ràng |
| drive_008 #16 | van | thân hộp một khối, đầu xe rất ngắn, cửa sau cao hết thân, không có thùng tách rời | `van`: thân hộp nhỏ, kín; `car` loại xe van thân hộp |
| drive_022 #2 | truck | thùng hàng hình hộp tách riêng phía sau cabin | `truck`: có thùng hàng rõ; không phải van kín một khối |
| drive_038 #19 | truck | xe cứu hộ có cần cẩu và cơ cấu kéo phía sau cabin | `truck`: thiết bị công vụ rõ ràng |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

`drive_038 #7` là xe van màu đỏ: lớp `van` được quyết định bởi hình dáng thân hộp một khối, còn `visibility=occluded` chỉ ghi nhận rằng một phần bên trái của xe bị xe tải xanh che. Nếu xe tải kia chạy đi, thuộc tính đổi thành `clear` nhưng lớp vẫn là `van`. Định dạng YOLO chỉ giữ lớp và hộp, nên thuộc tính phải lấy từ gói CVAT gốc.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Taxi ở `drive_033` (xyxy ≈ 564, 267, 612, 326) gán `bus` | lớp | bước đối chiếu cho thấy phía tham chiếu gán `car`; tôi cắt và phóng to hộp này: xe con cỡ nhỏ màu cam kiểu taxi, không có thân dài nhiều cửa sổ | `drive_033 #20` là `car`, theo quy tắc "taxi thuộc `car`" |
| 3 hộp `car` ở mép phải `drive_008` (xyxy ≈ 619–640 × 223–277, 626–640 × 261–347, 617–640 × 566–640) có `visibility=unclear` nhưng `review_state=confident` | thuộc tính | rà giá trị thuộc tính trong gói CVAT gốc | `drive_008 #17–#19` là `needs_review`: chỉ thấy một dải hẹp ở mép ảnh, chưa đủ bằng chứng để tự tin về lớp |
| 2 hộp `car` ở mép phải `drive_038` dừng ở x ≈ 630.6, trong khi xe bị mép ảnh cắt | hình học | rà lại hình học các hộp `truncated` | kéo hộp ra sát mép ảnh (x = 640), theo quy tắc "vẽ sát phần vật thể nhìn thấy"; phần xe nhìn thấy kéo dài tới mép |
| Xe ở mép trái `drive_038` (xyxy ≈ 0, 184, 54, 259) gán `van` | lớp | rà lại lớp khi tự kiểm tra | `drive_038 #21` là `car`, theo bảng phân biệt `car`/`van` |

- Số hộp `needs_review` trước và sau khi kiểm: 0 trước khi kiểm; 3 sau khi kiểm (`drive_008 #17–#19`).
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: 3 hộp ở mép phải `drive_008` nói trên. Tôi giữ chúng ở trạng thái `needs_review` và sẽ hỏi Lab Coach nên giữ hộp khi chỉ nhìn thấy một dải hẹp của xe ở mép ảnh, hay bỏ qua và ghi vào nhật ký quyết định.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `0 0.520281 0.525102 0.123594 0.073859` (`drive_022`, dòng 1)
- Tên lớp và tọa độ điểm ảnh `xyxy`: `car`, `[293.4, 312.4, 372.5, 359.7]` trên ảnh 640 × 640
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Kiểm tra định dạng chỉ xác nhận dòng có đủ năm số, mã lớp nằm trong khoảng 0–3 và hộp không vượt biên ảnh. Nó không biết hộp đó bao vật thể gì. Ví dụ trước khi sửa, chiếc taxi ở `drive_033` là một dòng hợp lệ nhưng sai lớp (`bus`). Chính gói tham chiếu cũng đúng định dạng nhưng mã lớp của nó bị lệch so với tên lớp (xem mục 6). Định dạng cũng không kiểm hộp có sát xe không, có bỏ sót xe không, hay có gán nhầm vật ngoài phạm vi không.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Cấu hình: YOLO11n, Ultralytics 8.4.145, cấu hình 8 vòng lặp, hạt giống 42, `freeze=10`, GPU, 27,33 giây (`training_run.json`). Nhật ký cho thấy huấn luyện dừng sớm ở vòng 4 (không cải thiện trong 3 vòng, `patience=3`), mô hình tốt nhất ở vòng 1 với mAP50 ≈ 0.007 trên ảnh thẩm định.
- Mô tả một dự đoán trong `detect_result.jpg`: ở ngưỡng tin cậy 0.25, mô hình không đưa ra hộp dự đoán nào trên `drive_008`, dù ảnh có 27 xe theo nhãn của tôi, gồm cả xe buýt nối toa, xe ben và nhiều xe con nhìn rõ.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Các bước kiểm định dạng và đồng nhất hai gói đều đạt, nên tôi chưa thấy dấu hiệu lỗi quy tắc. Điều cần kiểm lại là khối lượng và độ cân bằng dữ liệu: chỉ 3 ảnh huấn luyện, và các lớp hiếm có rất ít mẫu (car 57 hộp, bus 8, van 5, truck 5 trên cả bốn ảnh).
- Minh chứng nào có thể bác bỏ nhận định của bạn? Nếu hạ ngưỡng tin cậy (ví dụ 0.01) mà vẫn không có hộp nào, hoặc nhật ký huấn luyện cho thấy không đọc được nhãn, thì nguyên nhân là lỗi đường ống chứ không phải thiếu dữ liệu. Ngược lại, nếu thêm ảnh hoặc vòng lặp mà hộp xuất hiện đúng lớp thì nhận định được củng cố.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

Mô hình chỉ học từ 3 ảnh, dừng sau 4 vòng lặp, 10 lớp đầu của mạng bị đóng băng (`freeze=10`), và chỉ được thử trên đúng 1 ảnh từ cùng nhóm camera giao thông ban ngày. Không có tập kiểm tra độc lập, không có điều kiện đêm, mưa hay góc camera khác, nên một ảnh dự đoán tốt hoặc xấu đều không nói lên khả năng dùng thực tế. Kết quả này chỉ dùng để kiểm tra đường ống dữ liệu và gợi ý câu hỏi về nhãn, không dùng để chấm người gán nhãn.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 48
- IoU trung bình và trung vị: 0.8281 và 0.8500
- Mức đồng thuận lớp: 0.7292, tức 35/48
- Số hộp phía bạn không ghép được: 27 (tôi gán 75 hộp, phía tham chiếu có 50)
- Số hộp phía đối chiếu không ghép được: 2
- Một điểm khác biệt cụ thể: cả 13 cặp lệch lớp đều lệch theo đúng một vòng cố định: `bus` của tôi thành `van` ở phía tham chiếu, `truck` thành `bus`, `van` thành `truck`. Ví dụ `drive_008 #12` là xe buýt nối toa nhưng phía tham chiếu ghi `van`; `drive_008 #13` là xe ben chở đất nhưng phía tham chiếu ghi `bus`; `drive_008 #16` là xe van nhỏ nhưng phía tham chiếu ghi `truck`. Khi tôi thử đọc mã lớp của gói tham chiếu theo thứ tự `car, van, truck, bus`, mức đồng thuận lớp là 1.000 (48/48).
- Quy tắc hoặc hành động sửa phát sinh: (1) báo Lab Coach rằng gói tham chiếu có dấu hiệu lệch thứ tự lớp so với `data.yaml`; không sửa nhãn của tôi cho khớp với nó; (2) sửa chiếc taxi thành `car` theo quy tắc taxi (mục 3); (3) bổ sung một câu quy tắc: "Trước khi đối chiếu, kiểm thứ tự lớp của nguồn đối chiếu bằng vài vật thể dễ nhận như xe buýt nối toa"; (4) rà lại 27 hộp không ghép được theo quy tắc phạm vi về xe quá nhỏ hoặc chỉ thấy một phần.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?

Đồng thuận chỉ cho biết hai nguồn giống nhau, không cho biết nguồn nào đúng. Ở bài này, gói tham chiếu đúng định dạng nhưng lệch mã lớp; nếu tôi coi nó là đáp án thì sẽ sửa sai những nhãn đang đúng. Ngay cả khi đọc lại mã lớp và đạt 48/48, hai bên vẫn có thể cùng sai ở một xe. IoU chỉ đo độ chồng khít của các cặp đã ghép, không nói gì về 27 hộp không có cặp hay về lớp.

## 7. Kiểm tra kho GitHub cá nhân

- [ ] Có phiếu quy tắc với ba tình huống mơ hồ.
- [ ] Có kết quả kiểm hai gói xuất.
- [ ] Có thông tin lần huấn luyện và ảnh dự đoán.
- [ ] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [ ] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [ ] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là mẫu lệch lớp theo vòng cố định trong `comparison_iou.csv`, kiểm được bằng ảnh phủ `comparison_overlay.png` và bằng phép đọc lại mã lớp (0.729 lên 1.000). Câu hỏi cho Lab Coach: gói `day2-reference-4img-v1` có được tạo theo thứ tự lớp `car, van, truck, bus` không, và với xe chỉ thấy một dải hẹp ở mép ảnh thì nên gán hộp với `needs_review` hay bỏ qua?
