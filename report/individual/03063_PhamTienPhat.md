# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Tiến Phát
- MSSV: 2A202603063
- Email: s7phatnm@gmail.com
- Lớp: K5 Level 2A
- Link repo bài làm: https://github.com/Fat137/K5-L2-DAY02-PhamTienPhat-03063-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1e4XCtkPeCNezRg1XdhKxa5-RjqsM7TXx?usp=sharing 

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema giữ nguyên `0 car, 1 truck, 2 bus, 3 van`. Ba ca mơ hồ đã quyết định:

| Ảnh | Tình huống | Quyết định | Cách áp dụng schema |
| --- | --- | --- | --- |
| `drive_001` | Xe mini thân hình hộp, dễ nhầm `van` | `car` | Ngắn hơn sedan bên cạnh → xe con cỡ mini |
| `drive_019` | Xe trắng hình hộp cỡ nhỏ | `van` | Dài gần bằng sedan nhưng mui cao, phẳng tới đuôi, capo ngắn → minivan |
| `drive_028` | Xe trắng thân dài bị SUV và ô che phần đầu | `van` | Không thấy khe tách cabin – thùng nên không đủ căn cứ gán `truck`; ngắn hơn 2 lần xe con nên không phải `bus` |

Quy ước rút ra: phân biệt xe hình hộp theo **kích thước so với sedan** và **có/không khe cabin – thùng**; MPV (minivan 7 chỗ, mui dài phẳng) gán `van`, SUV gán `car`. Box gồm cả bánh xe, cạnh dưới chạm điểm lốp tiếp đất, không lấy bóng đổ.

Cần giảng viên làm rõ: (1) xe nhỏ ở xa / xe đỗ trong bãi có phải gán không (xem mục 4); (2) thứ tự ID lớp trong gold (xem mục 4).

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 (1095 box: car 921, truck 26, bus 94, van 54).
- Kết quả checker: `python tools/check_yolo_labels.py` → `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`.
- Cách làm: ảnh `drive_001`–`010` tự gán trên CVAT; ảnh `drive_011`–`050` gán trước bằng YOLO11x pretrained COCO (box car/truck/bus), sau đó rà và sửa trên CVAT (chủ yếu đổi lớp `van`, thêm xe sót, xóa box thừa) — xem mục 8.
- Danh sách 10 ảnh rà bằng mắt (chọn ngẫu nhiên, `random.seed(20261109)`): `drive_001, 004, 009, 012, 019, 024, 028, 032, 040, 050` 
- Lỗi phát hiện khi rà và cách sửa:

| Lỗi | Ảnh | Cách sửa |
| --- | --- | --- |
| Box bị xoay trong CVAT → khi export YOLO bị mất 2 xe | `drive_001` | Xóa, vẽ lại box thẳng |
| Bỏ sót xe tải ben ở giữa ảnh | `drive_008` | Thêm box `truck` |
| Cạnh dưới box cắt mất bánh xe | nhiều ảnh | Kéo cạnh dưới chạm đáy lốp |
| Xe buýt bị gán `car` | ảnh có xe buýt trắng + taxi | Đổi `bus` |
| Van bị gán `truck`/`bus`, xe con bị gán `truck`, xe ben bị gán `bus` (do model auto-label) | `drive_016, 021, 030, 031, 036, 039, 043, 044` | Đổi lớp |

- Commit khóa nhãn ban đầu: `f561f3e` — "CP4: khoa bo nhan ban dau 50 anh train + mini guideline".

## 4. Gold QC và rework

Chạy `python tools/evaluate_annotations.py --student-labels dataset/labels/train --gold-labels gold_train_labels` sau khi khóa nhãn. Kết quả: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 465 | — |
| FP | 630 | — |
| FN | 188 | — |
| Precision | 0.4247 | — |
| Recall | 0.7121 | — |
| F1 | 0.5320 | — |
| mean_iou_of_matches | 0.8494 | — |
| passed | Không (ngưỡng RUBRIC F1 ≥ 0.80); Đạt theo ngưỡng F1 ≥ 0.50 do lab coach bổ sung trên lớp| — |

Không rework sau khi nhận gold.

Phân tích (đã mở lại dữ liệu, không chỉ dựa vào FP/FN):

1. **Box sát tốt:** mean IoU 0.849 (mức "Xuất sắc" ≥ 0.80) — box nào khớp thì khớp rất sát.
2. **Gán nhiều hơn gold (nguyên nhân chính của FP):** tôi có 1095 box, gold 653. 485/630 FP là box không chồng với bất kỳ box gold nào, phần lớn cỡ 20–80 px (xe ở xa, xe đỗ ven đường). Lệch nặng nhất ở 10 ảnh tự gán, ví dụ `drive_003` (gold 18 / tôi 62), `drive_002` (13 / 50). Đây là lệch quy ước: guideline của tôi gán mọi xe nhận ra được, còn gold bỏ qua xe nhỏ ở xa. Chỉ 29 xe gold bị bỏ sót hoàn toàn.
3. **Lệch ID lớp có hệ thống giữa gold và schema:** ghép box theo vị trí (IoU ≥ 0.5, bỏ qua lớp):

| gold \ tôi | car | truck | bus | van |
|---|---:|---:|---:|---:|
| car | 465 | 0 | 0 | 6 |
| truck | 16 | 0 | 1 | 28 |
| bus | 0 | 19 | 0 | 0 |
| van | 0 | 0 | 53 | 0 |

   Cả 53 xe tôi gán `bus` đều được gold ghi `van`, cả 19 xe tôi gán `truck` đều được gold ghi `bus`. Nhầm lẫn 100% theo một quy luật cho thấy file gold nhiều khả năng dùng thứ tự ID khác (≈ 1 = van, 2 = truck, 3 = bus) so với schema README/`data.yaml`. Chỉ để phân tích, chấm lại với một bản sao gold đã đổi ID (không sửa gold gốc, không commit) cho F1 = 0.647, recall = 0.865.

## 5. Kết quả YOLO

- Runtime: Google Colab, GPU (`device: 0`); Ultralytics bản mới nhất cài bằng `%pip install -U ultralytics` (cell đầu không in số phiên bản). Notebook clone repo tại commit `f561f3e` (bản nhãn đã khóa).
- Tham số: giữ nguyên notebook — `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`. Không đổi split, không dùng eval để train.
- Best checkpoint: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không commit `.pt`).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4059 |
| Recall | 0.2333 |
| mAP50 | 0.2744 |
| mAP50–95 | 0.2025 |

Kết quả: [ảnh dự đoán eval](assets/eval_predictions/visuals/), [log train results.csv](assets/yolo26_runs/vehicle_annotation/results.csv), [đường cong train](assets/yolo26_runs/vehicle_annotation/results.png), [confusion matrix eval](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png).

Nhận xét:

- **`best.pt` là epoch 1.** mAP50 trên eval cao nhất ở epoch 1 (0.274) rồi dao động/giảm (0.106 ở epoch 10, 0.235 ở epoch 50) trong khi `train/cls_loss` giảm đều 4.59 → 0.77. Bốn chỉ số trên vì vậy là của model gần như chưa học xong.
- Ví dụ cụ thể: [drive_051](assets/eval_predictions/visuals/drive_051.jpg) — sedan đỏ gần camera bị dự đoán `van` 0.99; các xe con ở xa phía trên được nhận `car` 0.26–0.85.
- Confusion matrix eval: `car` khớp 83/94; 11/12 box gold `van` bị dự đoán `bus` — đúng quy luật lệch ID đã thấy ở gold train.

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp recall thấp nhất:** `van` (gần 0; 11/12 box gold `van` bị dự đoán `bus`), sau đó `truck` (1/5). Minh họa: confusion matrix eval và `drive_051` (sedan bị gán `van`).
2. **mAP50 cao nhưng mAP50–95 thấp:** cần kiểm tra độ sát của cạnh box ở IoU cao — cạnh dưới (bánh xe, bóng đổ), xe bị che, xe cắt khung. Ở đây khoảng cách 0.274 → 0.203 không lớn, nên box khớp vẫn tương đối sát; vấn đề chính là phân lớp và FP.
3. **Lỗi class liên quan guideline:** (a) thứ tự ID lớp giữa gold và schema không khớp (bảng mục 4, confusion matrix eval) nên `truck/bus/van` bị tính sai dù xe được nhận đúng loại; (b) guideline của tôi gán cả xe nhỏ ở xa còn gold thì không → 485 FP khi chấm annotation và nhiều FP nền khi đánh giá model.
4. **Không dùng 10 ảnh eval để train** để giữ một tập model chưa thấy, đánh giá khách quan. Trong notebook, eval còn là validation để chọn `best.pt`, nên không phải test set độc lập; lần chạy này cho thấy rõ: nhãn eval lệch ID khiến checkpoint được chọn là epoch 1.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP2 – export CVAT | Box xoay bị mất khi export YOLO 1.1 | Xóa, vẽ lại box thẳng; không kéo núm xoay | Đã sửa |
| CP2 – cài thư viện local | Tải torch rất chậm, bị ngắt | Chuyển sang chạy trên Colab | Đã xử lý |
| CP5 – chấm gold | F1 0.532 < 0.80 | Phân tích FP/FN, phát hiện gán thừa + lệch ID lớp gold | Báo coach; đạt theo ngưỡng 0.50 coach bổ sung |
| CP6 – train | `best.pt` rơi vào epoch 1 | Phân tích results.csv + confusion matrix | Giải thích ở mục 5 |

## 7. Tự đánh giá theo rubric

| Tiêu chí | Tình trạng |
|---|---|
| Định dạng YOLO (10) | 50/50 file, checker đạt |
| Đầy đủ vật thể (20) | Recall 0.712 với gold (0.865 nếu bỏ lệch ID); chỉ 29 xe bị sót hoàn toàn |
| Độ bám sát box (20) | mean IoU 0.849 |
| Phân lớp nhất quán (15) | Nhất quán theo schema README; lệch với gold do thứ tự ID (mục 4) |
| Mini guideline (10) | Có định nghĩa lớp, quy tắc box, 3 ca mơ hồ có lý do |
| Gold-set QC (15) | F1 0.532 — chưa đạt 0.80 theo RUBRIC, đạt 0.50 theo coach; có phân tích nguyên nhân |
| Pipeline YOLO (10) | Colab chạy xong, giữ split, có metrics, ảnh eval, trả lời 4 câu hỏi |

## 8. Công cụ AI và tài liệu tham khảo

- **Claude (Claude Code):** giải thích yêu cầu đề; góp ý guideline và cách phân lớp các ca khó.
- **Phần tự kiểm tra:** tự gán ảnh 1–50 trên CVAT; chạy checker, evaluator và notebook Colab.
- Không dùng gold train trước khi khóa nhãn; không sửa gold; không commit gold, token hay `.pt`.
- Tài liệu: tài liệu repo bài lab.
