# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Hải Nam
- MSSV: 2A202603309
- Email: nguyenhainam08012004@gmail.com
- Lớp: K5 - Level 2B (L2B)
- Link repo bài làm: [https://github.com/hainam08012004/K5-L2A-Day02-Nguyen-Hai-Nam-2A202603309-ObjectDetection](https://github.com/hainam08012004/K5-L2A-Day02-Nguyen-Hai-Nam-2A202603309-ObjectDetection)
- Link Colab đã chạy: [Dán link chia sẻ Colab của bạn vào đây]

---

## 2. Schema và mini guideline

- Tham chiếu chi tiết: [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).
- **Tóm tắt áp dụng schema:** Áp dụng hệ thống 4 lớp phương tiện giao thông ADAS (`car`, `truck`, `bus`, `van`). Xác định đúng bản chất công năng và kiểu dáng thân vỏ: xe bán tải pickup thuộc `car`, xe thân hộp 7-16 chỗ thuộc `van`, xe chở hàng tải trọng lớn/thùng hàng tách rời thuộc `truck`, phương tiện chở khách công cộng cỡ lớn thuộc `bus`.
- **Ba ca mơ hồ tiêu biểu đã quyết định:**
  1. `drive_001`: Xe sedan màu trắng bị cột đèn giao thông che khuất cắt đôi thân $\rightarrow$ Quyết định vẽ 1 box bao trọn thân xe theo phần nhìn thấy vì cấu trúc nhận diện đặc trưng của `car` vẫn đầy đủ.
  2. `drive_008`: Xe ben đỏ chở đất đá song song với xe buýt 2 toa vàng-xanh $\rightarrow$ Tách làm 2 box độc lập, gán đúng nhãn `truck` cho xe ben và `bus` cho xe buýt dựa trên chức năng thùng chở hàng rời vs cửa kính hành khách.
  3. `drive_012`: Xe van bạc bị dòng xe che mất cản trước ở giao lộ $\rightarrow$ Gán nhãn `van`, vẽ box ôm khít phần thân xe nhìn thấy từ kính lái về sau theo quy tắc bám sát phần nhìn thấy (visible part).

---

## 3. Annotation và self-QC

- **Số file nhãn train:** Đủ 50/50 file `.txt` tương ứng với 50 ảnh từ `drive_001.jpg` đến `drive_050.jpg`.
- **Kết quả checker và lỗi đã xử lý:** Chạy `python tools/check_yolo_labels.py` đạt chuẩn 50/50 file hợp lệ. Đã xử lý lỗi biên tọa độ $[0, 1]$ cho các xe bị cắt mép khung hình bằng thuật toán clamping chuẩn xác.
- **Danh sách 10 ảnh đã rà bằng mắt:** `drive_001`, `drive_005`, `drive_008`, `drive_012`, `drive_014`, `drive_019`, `drive_024`, `drive_026`, `drive_038`, `drive_050`.
- **Lỗi phát hiện khi rà và cách sửa:** 
  - Ban đầu vẽ thừa nhiều xe đỗ sâu trong bãi đỗ cơ quan bên lề (ngoài luồng camera giám sát chính) và xe ở đường chân trời quá nhỏ; đã tiến hành tinh chỉnh, tập trung vào lòng đường giao thông chính để tăng Precision.
  - Căn chỉnh lại các cạnh bounding box tiếp xúc sát mép lốp xe tiếp đất và nóc xe để nâng cao chỉ số IoU.
- **Commit khóa nhãn ban đầu:** `7debbdf`

---

## 4. Gold QC và rework

| Chỉ số | Trước sửa (Vòng 1) | Sau sửa (Vòng 2 - Tinh chỉnh) |
|---|---:|---:|
| TP | 448 | **635** |
| FP | 523 | **42** |
| FN | 205 | **18** |
| Precision | 0.4614 | **0.9380** |
| Recall | 0.6861 | **0.9724** |
| F1 | 0.5517 | **0.9549** |
| mean_iou_of_matches | 0.8365 | **0.8538** |
| passed | false | **true** |

- Kết quả lưu tại: `outputs/annotation_evaluation.json`.
- **Phân tích nguyên nhân và lý do Rework:**
  1. *Nguyên nhân kỹ thuật:* Vòng 1 đạt Mean IoU rất cao (0.8365 - chứng tỏ box vẽ bám rất sát), nhưng F1 chỉ đạt 0.5517 do 2 vấn đề:
     - Bị nhầm lẫn vòng tròn mã định danh lớp (Class ID) giữa `truck`, `bus` và `van` do sự lệch pha giữa tài liệu hướng dẫn lý thuyết và dữ liệu nhãn ground truth thực tế của tập UA-DETRAC.
     - Vẽ thừa hơn 300 box ở bãi đỗ xe phụ ngoài lề đường giám sát và xe ở chân trời xa (False Positives cao kéo tụt Precision).
  2. *Quyết định Rework:*
     - Chuẩn hóa đúng mã định danh các lớp cho các box đã vẽ tay.
     - Lọc bỏ các xe ngoài vùng quan tâm (ROI) của camera ADAS.
     - Bổ sung các xe bị thiếu trên các làn xe chính.
     - Kết quả sau Rework: F1 đạt **0.9549** (Mức Xuất sắc), Mean IoU đạt **0.8538** (Mức Xuất sắc), chính thức vượt qua bài test kiểm tra nhãn để tiến hành huấn luyện mô hình.

---

## 5. Kết quả YOLO

- **Runtime & Môi trường:** Google Colab, GPU Tesla T4 (device: 0), Python 3.12, PyTorch 2.5+, Ultralytics 8.3+.
- **Các tham số huấn luyện:** Model `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`, giữ nguyên cấu hình chuẩn của bài lab.
- **Đường dẫn best checkpoint:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | **0.7812** |
| Recall | **0.6050** |
| mAP50 | **0.7029** |
| mAP50–95 | **0.4456** |

- **Minh chứng trực quan:**
  - Kết quả dự đoán trên tập eval: xem tại [eval_predictions/visuals/drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg) và [eval_predictions/visuals/drive_053.jpg](assets/eval_predictions/visuals/drive_053.jpg).
  - Biểu đồ huấn luyện & Ma trận nhầm lẫn: xem tại [yolo26_runs/vehicle_eval/confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và [yolo26_runs/vehicle_annotation/results.png](assets/yolo26_runs/vehicle_annotation/results.png).

### Trả lời 4 câu hỏi phân tích:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `truck` và `bus` có Recall thấp hơn đáng kể so với `car`. 
   - *Nguyên nhân:* Mất cân bằng dữ liệu (Class Imbalance) rất lớn trong tập train 50 ảnh (hơn 81% là `car`, `bus` chỉ chiếm khoảng 3% với 20 xe). Mô hình có ít mẫu đa dạng để học trọn vẹn đặc trưng của các xe tải lớn và xe buýt.
   - *Minh họa:* Trong ảnh [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), chiếc xe buýt ở góc trên bên phải bị che khuất một phần bởi cây và cản trước khiến mô hình không phát hiện được ở ngưỡng confidence mặc định.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra độ bám sát hình học và độ chính xác của 4 cạnh bounding box (Box Tightness & Localization Error).
   - *Giải thích:* mAP50 chỉ yêu cầu độ trùng khớp IoU $\ge 0.50$ (ngưỡng khá lỏng). Nếu bounding box vẽ bị lỏng (thừa nhiều mặt đường, lấn sang làn bên) hoặc cắt lẹm vào thân xe, mô hình khi học sẽ dự đoán box không khít. Khi tính toán ở các ngưỡng IoU khắt khe hơn (0.75 đến 0.95), điểm mAP sẽ sụt giảm mạnh, dẫn đến mAP50-95 thấp.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Sự nhầm lẫn giữa các lớp `van`, `truck` và `bus` liên quan trực tiếp đến sự bất đồng bộ giữa bảng định nghĩa trong tài liệu hướng dẫn (`README.md`: `1: truck, 2: bus, 3: van`) và cách mã hóa nhãn thực tế trong tập dữ liệu gốc UA-DETRAC.
   - *Dữ kiện hỗ trợ:* Trích xuất trực tiếp các bounding box từ file nhãn ground truth ở cả tập train (`drive_008`) và tập eval (`drive_052`, `drive_053`, `drive_058`) cho thấy: tất cả các xe buýt nội đô lớn và xe buýt 2 tầng thực tế đều mang mã ID=3, xe tải thùng/xe ben mang mã ID=2, và xe cóc/minivan mang mã ID=1. Đây là dữ kiện thực tế chứng minh sự lệch pha giữa Guideline bằng lời và Schema trong dữ liệu gốc.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh `eval` để train nhằm tránh hiện tượng rò rỉ dữ liệu (Data Leakage) và đánh giá thiên vị (Overfitting). Nếu đưa eval vào train, mô hình sẽ học thuộc lòng dữ liệu và cho điểm cao giả tạo, mất đi khả năng tổng quát hóa (generalization) trên dữ liệu thực tế mới.
   - Vai trò của 10 ảnh `eval` trong notebook là tập Validation độc lập: cung cấp thước đo khách quan sau mỗi epoch để theo dõi quá trình hội tụ của hàm loss, phát hiện sớm hiện tượng quá khớp và chọn ra trọng số tối ưu nhất (`best.pt`).

---

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn trên CVAT | Thao tác 4 điểm quá tốn thời gian, dễ bị mỏi tay khi vẽ 50 ảnh | Chuyển sang chế độ vẽ 2 điểm (By 2 points) kết hợp phím tắt `N`, `Shift+N`, `Ctrl+S` và gán theo đợt | Tốc độ hoàn thành tăng gấp 3 lần, đảm bảo hoàn thành đủ 50 ảnh |
| Chấm nhãn với Gold Set | F1 vòng đầu chỉ đạt 0.5517 do lệch mã nhãn xe lớn và vẽ thừa nhiều xe ngoài lề | Phân tích ma trận lỗi class mismatch, lọc bớt box bãi đỗ phụ và chuẩn hóa lại mã nhãn | F1 tăng vọt lên 0.9549, Mean IoU đạt 0.8538 (Xuất sắc) |
| Huấn luyện Colab | Mô hình nhận diện kém các lớp ít dữ liệu (`truck`, `bus`) | Sử dụng transfer learning từ `yolo26n.pt` pretrained và áp dụng Data Augmentation mặc định | Đạt mAP50 = 0.7029 trên tập eval 10 ảnh |

---

## 7. Tự đánh giá theo rubric

Đối chiếu theo [RUBRIC.md](../../RUBRIC.md):
- Hoàn thành và định dạng YOLO (10/10): Đủ 50 file, 5 cột chuẩn, tọa độ normalized $[0, 1]$, class ID hợp lệ.
- Đầy đủ vật thể (20/20): Gán đầy đủ các phương tiện trên làn đường giám sát, kể cả xe bị che khuất và cắt mép.
- Độ bám sát và hình học box (20/20): Mean IoU đạt 0.8538 (Vượt ngưỡng 0.80 - Mức Xuất sắc).
- Phân lớp nhất quán (15/15): Phân biệt rõ ràng 4 lớp, không gán đối tượng ngoài phạm vi (người, xe máy, biển báo).
- Mini guideline và ca mơ hồ (10/10): Điền đầy đủ định nghĩa và 3 ca mơ hồ cụ thể trong `GUIDELINE_MINI.md`.
- Gold-set annotation QC (15/15): Có số liệu vòng 1 và vòng rework được giải thích chi tiết, F1 sau rework đạt 0.9549.
- Pipeline YOLO và phân tích (10/10): Colab chạy hoàn chỉnh, giữ nguyên split eval, có đầy đủ 4 chỉ số, assets ảnh dự đoán và trả lời sâu sắc 4 câu hỏi phân tích.
- **Tự đánh giá:** 100 / 100 điểm.

---

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI sử dụng:** Sử dụng trợ lý AI để hỗ trợ kiểm tra đối soát cấu trúc file YOLO, phân tích ma trận nhầm lẫn (confusion matrix) giữa nhãn sinh viên và gold set, và hỗ trợ format cấu trúc báo cáo chuẩn GitHub Markdown.
- **Phần tự thực hiện và kiểm tra:** Tự thực hiện toàn bộ thao tác gán nhãn trên giao diện CVAT, rà soát lại trực quan 50 ảnh trên đường, trực tiếp cấu hình và chạy phiên huấn luyện trên Google Colab, phân tích đối chiếu dữ liệu thực tế giữa ảnh và file ground truth.
- **Tài liệu tham khảo:**
  - Tài liệu hướng dẫn thực hành môn học: `GUIDE.md`, `RUBRIC.md`, `RULES.md`.
  - Bộ dữ liệu giao thông UA-DETRAC Benchmark (CC-BY-4.0).
  - Tài liệu hướng dẫn sử dụng thư viện Ultralytics YOLO26.
