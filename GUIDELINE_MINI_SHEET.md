# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Pham Huu Hai<br>
**MSSV:** 2A202602098<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

Ba tình huống dưới đây dùng quyết định của chính tôi, không lấy từ bộ tham chiếu. `#n` là dòng thứ n trong file nhãn YOLO đã nộp.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_038 #7`, xe màu đỏ, xyxy ≈ (494, 109, 582, 205)
- Dấu hiệu nhìn thấy: thân hộp cao, đầu xe ngắn liền một khối, một hàng cửa sổ ngắn bên hông; không có thân dài nhiều hàng ghế như các xe buýt trong cùng ảnh
- Quy tắc áp dụng: `van` là thân hộp nhỏ, kín; `bus` cần thân xe khách dài, nhiều cửa sổ hoặc hàng ghế
- Quyết định: `van`, với `visibility=occluded` (một phần bên trái bị xe tải xanh che), `boundary=inside`, `review_state=confident`
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng ảnh lên 100% để đếm cửa sổ và xem chiều dài thân; nếu vẫn không phân biệt được thì giữ lớp hợp lý nhất, đặt `needs_review`, ghi lý do và hỏi Lab Coach.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_022 #2`, xe màu trắng, xyxy ≈ (574, 256, 629, 330)
- Dấu hiệu nhìn thấy: cabin nhỏ phía trước và thùng hàng hình hộp tách riêng phía sau
- Quy tắc áp dụng: `truck` khi có thùng hoặc sàn chở hàng rõ ràng; `van` là thân kín một khối, không có khoang hàng tách biệt
- Quyết định: `truck`, với `visibility=clear`, `boundary=inside`, `review_state=confident`
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Tìm khe giữa cabin và thùng khi phóng 100%; nếu không thấy rõ thì đặt `needs_review`, ghi lý do và không đoán theo màu hay kích thước.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008`, hộp `car` ở mép phải, xyxy ≈ (626, 261, 640, 347)
- Dấu hiệu nhìn thấy khi phóng 100%: chỉ thấy một dải hẹp khoảng 14 điểm ảnh của thân xe ở mép phải; phần còn lại nằm ngoài khung
- Giá trị `visibility`: `unclear`
- Giá trị `boundary`: `truncated`
- Trạng thái `review_state`: `needs_review`
- Lý do: xe bị mép ảnh cắt gần hết nên `truncated` là đúng, và bằng chứng nhìn thấy quá ít nên `unclear` là đúng. Với lượng bằng chứng đó thì không thể tự tin về lớp, nên tôi đặt `needs_review` cho hộp này và hai hộp `car` khác ở cùng mép phải, và giữ lại chúng để hỏi Lab Coach thay vì tự xóa.

## 6. Xác nhận tự kiểm tra

- [ ] Đã rà đủ bốn ảnh.
- [ ] Đã kiểm vật thể thiếu và trùng.
- [ ] Đã kiểm lớp và hình học từng hộp.
- [ ] Mỗi hộp có đủ ba thuộc tính.
- [ ] Đã xử lý mọi hộp `needs_review`.
- [ ] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [ ] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [ ] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [ ] Số vật thể thực tế: 75 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
