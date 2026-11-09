# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Minh Hiếu
- MSSV: 2A202603091
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/Hieudevnekkk/K5-L2A-Day02-NguyenMinhHieu-2A202603091-ObjectDetection
- Link Colab đã chạy: **Không có.** Colab báo lỗi "Colab is currently experiencing issues provisioning runtimes" nên không cấp được runtime (xem mục 6). Phần train/eval được chạy bằng chính notebook trên PC cá nhân (Windows, GPU).

## 2. Schema và mini guideline

Xem [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt các ca đã quyết định:

- `drive_039`: xe buýt ghép nối (hai khoang nối bằng khớp xoay) vẽ **một** box `bus` cho cả xe, vì mỗi phương tiện là một box.
- `drive_005`: xe SUV đen thân cao gán `car`, không gán `van`.
- `drive_005`: xe trắng có cabin và thùng riêng gán `truck`; nếu thân liền khối thì chuyển `van`.

Phần cần giảng viên làm rõ: ranh giới `van` và `bus` (xem mục 5, câu 3, và mục 4).

## 3. Annotation và self-QC

- Số file nhãn train: 50
- Kết quả checker: `python tools/check_yolo_labels.py` đạt 50/50 file hợp lệ.
- Cách gán nhãn: dùng CVAT. Frame 0–2 (72 box) gán tay; frame 3–49 (574 box) được gán trước bằng một model YOLO26 (COCO) chạy qua CVAT rồi sửa lại tất cả các frame mà model gán sai hoặc thiếu bằng tay, sau đó export YOLO Detection 1.0.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_003`, `drive_005`, `drive_009`, `drive_012`, `drive_018`, `drive_024`, `drive_029`, `drive_033`, `drive_039`, `drive_046`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Box nhìn chung bám sát thân xe (khớp với mean IoU 0.849 ở mục 4); xe buýt ghép nối (`drive_029`, `drive_039`) đúng một box, xe bị cắt rìa ảnh (`drive_029`, `drive_033`) box chạm rìa, xe ba bánh (`drive_024`) và xe máy (`drive_046`) không gán, đúng theo guideline.
  - Lỗi class: ở `drive_005`, xe trắng giữa ảnh (khoảng x 403–447, y 132–205) đang gán `van` trong khi guideline quyết định `truck`; ở `drive_012`, xe wagon trắng bên trái (khoảng x 22–108, y 255–315) đang gán `van`, theo guideline đáng lẽ là `car`; ở `drive_012` xe khách nhỏ màu be (khoảng x 415–475, y 125–200) đang gán `bus` nhưng có thể là `van`.
  - Lỗi bỏ sót: một số xe nhỏ ở xa không có box (ví dụ góc trên bên trái `drive_009`).
  - Nguồn gốc lỗi: nhãn được model gán trước rồi sửa tay (dòng "Cách gán nhãn" ở trên), nhưng phần phân biệt `van`/`bus`/`car` vẫn còn nhầm sau khi sửa; đây cũng là chỗ gold QC cho thấy lệch nhiều nhất (mục 4).
  - Cách sửa: **không sửa**, vì bộ nhãn đã đạt gold QC theo ngưỡng mới (không rework, mục 4). Các lỗi trên được ghi lại để phân tích ở mục 5.
- Commit khóa nhãn ban đầu: `56669de` ("Day02: khoa nhan ban dau (truoc rework)").

## 4. Gold QC và rework

Ngưỡng đạt đã được thông báo đổi thành F1 >= 0.5 (mean IoU >= 0.70 giữ nguyên); chạy evaluator với `--min-f1 0.5`.

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 434 | không rework |
| FP | 446 | không rework |
| FN | 219 | không rework |
| Precision | 0.4932 | không rework |
| Recall | 0.6646 | không rework |
| F1 | 0.5662 | không rework |
| mean_iou_of_matches | 0.8488 | không rework |
| passed | true (ngưỡng F1 0.5) | không rework |

Kết quả: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json). **Không rework**, vì bộ nhãn đã đạt ngưỡng mới.

Phân tích lỗi (chỉ mô tả dữ kiện, chưa kết luận nguyên nhân):

- Theo lớp, chỉ `car` có box khớp gold (434 TP). `truck`, `bus`, `van` đều 0 TP.
- Nhầm lớp ở IoU >= 0.5: 52 box mình ghi `bus` mà gold ghi `van`; 29 box `van` mà gold ghi `truck`; 16 box `truck` mà gold ghi `bus`; 14 box `car` mà gold ghi `truck`.
- Số box: mình có 696 `car` / 95 `bus` / 68 `van` / 21 `truck`; gold có 533 / 20 / 54 / 46. Mình thừa nhiều box `car` nhỏ ở xa (262 FP `car`).
- Mean IoU 0.849 cho thấy box khớp thì khá sát; vấn đề chính nằm ở phân lớp và độ phủ, không phải hình học box.

## 5. Kết quả YOLO

- Runtime: PC cá nhân Windows. Checkpoint chạy tại `C:\Users\Admin\day2-run\Day2-Lab\outputs\yolo26_runs\vehicle_annotation\weights\best.pt`. GPU NVIDIA GeForce RTX 3060, Ultralytics 8.4.208, Torch 2.11.0+cu128.
- Tham số: giữ nguyên notebook (`yolo26n.pt`, 50 epoch, imgsz 640, batch -1, seed 20260909, device 0, amp bật, `data.yaml` của repo); xem `assets/yolo26_runs/vehicle_annotation/args.yaml`. Không đổi tham số.
- Chỉ khác notebook ở chỗ chạy trên PC thay vì Colab.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4045 |
| Recall | 0.2357 |
| mAP50 | 0.2741 |
| mAP50–95 | 0.2025 |

Nguồn: [model_eval_metrics.json](assets/model_eval_metrics.json), [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv).

Ảnh và chỉ số: [ảnh dự đoán 10 ảnh eval](assets/eval_predictions/visuals/), [confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png), [đường cong train](assets/yolo26_runs/vehicle_annotation/results.png).

Quan sát về quá trình train: epoch tốt nhất (nên `best.pt`) là **epoch 1** (mAP50–95 0.202). Sau đó giảm: epoch 10 chỉ 0.039, epoch 50 đạt 0.170. Tập train chỉ 50 ảnh và eval chỉ 10 ảnh nên số liệu dao động mạnh.

Trả lời bốn câu hỏi:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?** `van`: 0/12 box `van` trong eval được dự đoán đúng (11 bị đoán là `bus`, 1 bị bỏ sót), recall = 0. Các lớp khác (đọc từ [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)): `car` 83/94 ≈ 0.88, `bus` 1/2, `truck` 1/5. Ảnh minh họa: [`drive_058`](assets/eval_predictions/visuals/drive_058.jpg), nhãn eval có 6 `van` và 1 `bus`, nhưng model chỉ dự đoán các box `bus` ở bên trái (bus 0.80, bus 0.48, bus 0.26) và không có box `van` nào.
2. **mAP50 cao nhưng mAP50–95 thấp thì kiểm gì trên box?** Phải kiểm độ sát của box (cạnh lệch, box lỏng), vì mAP50 chỉ cần IoU 0.5 còn mAP50–95 đòi IoU cao hơn. Ở bài này mAP50 chỉ 0.274 và mAP50–95 là 0.202 (khoảng 74%), nên khoảng cách không lớn; hạn chế chính nằm ở phân lớp và độ phủ hơn là hình học box. Mean IoU của các box khớp gold ở mục 4 là 0.849, cũng cho thấy box khá sát.
3. **Lỗi class nào liên quan guideline? Dữ kiện nào?** Lỗi `van` bị đoán thành `bus` (11/12 box `van` ở eval). Dữ kiện hỗ trợ: ở gold QC (mục 4) có 52 box mình ghi `bus` mà gold ghi `van`, và mình có 95 box `bus` so với 20 của gold, nên model học từ nhãn train thiếu nhất quán giữa `van` và `bus`. Guideline mới chỉ định nghĩa `van` là xe thân liền khối cỡ nhỏ/vừa còn `bus` là xe cỡ lớn, chưa mô tả đủ ca ở giữa. Đây là liên hệ có dữ kiện chứ chưa chứng minh nguyên nhân, vì tập eval nhỏ và `van` cũng bị lệch số lượng.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation?** Nếu train trên eval thì model đã thấy đáp án, chỉ số cao giả và không đo được khả năng khái quát. Trong notebook, 10 ảnh này vẫn được dùng làm validation để chọn `best.pt` qua các epoch, nên đó không phải test set độc lập; chỉ số chỉ là ước lượng tham khảo.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Chạy notebook trên Colab, 10/11/2026 (giờ: 23h40) | "Colab is currently experiencing issues provisioning runtimes. We are aware of the issue and are working to resolve it." | Đợi và thử lại; sau đó chạy cùng notebook trên PC cá nhân | Không có link Colab; kết quả lấy từ PC. Ảnh chụp màn hình lỗi: [assets/colab_runtime_error.png](assets/colab_runtime_error.png) |
| Tải kết quả | File `day2_colab_outputs.zip` đầu tiên chỉ 480 byte, thiếu ảnh và log | Chạy lại và xuất đầy đủ thư mục outputs | Dùng bản đầy đủ đã kiểm tra |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):

- Hoàn thành và định dạng YOLO: có (50 file, checker đạt).
- Đầy đủ vật thể / bám sát box: chỉ có số liệu gián tiếp từ gold QC (recall 0.665, mean IoU 0.849); đã rà bằng mắt 10 ảnh ở mục 3, còn sót một số xe nhỏ ở xa.
- Phân lớp nhất quán: **chưa tốt** (`truck`/`bus`/`van` đều 0 TP ở gold QC), đã nêu ở mục 4 và 5.
- Mini guideline: có, 3 ca mơ hồ.
- Gold-set QC: đạt theo ngưỡng F1 >= 0.5 và mean IoU >= 0.70, không rework.
- Pipeline YOLO: đã chạy, có metrics và ảnh eval, không dùng eval để train; **không chạy trên Colab** (thiếu link Colab).

## 8. Công cụ AI và tài liệu tham khảo

- **Claude Code (Anthropic):**
  - Dựng một function Nuclio chạy model YOLO26 (COCO) trong CVAT và dùng nó gán nhãn tự động cho frame 3–49 của task `Day2` (574 box); sau đó mình sửa tay trong CVAT rồi export (880 box).
  - Phần tự kiểm tra: tự chạy train/eval trên PC (RTX 3060).
- CVAT, Ultralytics YOLO26, Google Colab (không chạy được).
- Không dùng gold để gán nhãn hoặc sửa nhãn; chỉ dùng để chấm sau khi khóa nhãn.
