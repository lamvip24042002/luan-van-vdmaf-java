# Biên bản kiểm tra gói mã nguồn và dữ liệu

Ngày kiểm tra: **22/10/2026**, múi giờ Asia/Bangkok. Kiểm tra thực hiện trên bản sao `github_submission`; mã nguồn, dữ liệu và kết quả nghiên cứu gốc được giữ nguyên.

## Kết quả

| Hạng mục | Kết quả thực tế |
|---|---|
| Kiểm thử tự động | **26/26 PASS**, mã thoát 0 |
| Tính lại chỉ số từ dự đoán đã lưu | **45/45 PASS**, kiểm tra SHA-256 của 232 tệp được sử dụng, mã thoát 0 |
| Suy luận mới từ 45 checkpoint trên CUDA | **45/45 PASS**, kiểm tra SHA-256 của 235 tệp được sử dụng, mã thoát 0 |
| Suy luận mới từ `train_full_s42` trên CPU | Chạy hết 6.293 mẫu; chỉ số nằm trong ngưỡng sai số, nhưng đối chiếu xác suất nghiêm ngặt báo **`different`**, mã thoát 1 |
| Giải nén ở thư mục khác | Profile `test` và `full` hoàn tất; xác minh các gói ZIP và SHA-256 của dữ liệu giải nén |
| Lập kế hoạch huấn luyện ở thư mục khác | `--plan-only --seeds 42` xác minh đầu vào và lập đủ 9 cấu hình; **không huấn luyện lại** |

Kiểm tra dữ liệu lưu sẵn và suy luận mới là hai bước riêng. `audit_results.py` tính chỉ số từ CSV dự đoán đã có, không chạy mô hình. `evaluate.py` nạp checkpoint, chạy lại mô hình trên toàn bộ tập test rồi so sánh kết quả với CSV và chỉ số gốc. Cả hai giữ ngưỡng phân loại đã chọn trên validation, không tìm ngưỡng trên test.

## Môi trường đã chạy

| Thành phần | Phiên bản/thiết bị |
|---|---|
| Hệ điều hành | Windows 11, build 10.0.26200 |
| Python | 3.12.14 |
| PyTorch | 2.5.1+cu121 |
| PyTorch Geometric | 2.6.1 |
| NumPy | 1.26.4 |
| scikit-learn | 1.5.2 |
| GPU | NVIDIA GeForce RTX 3060 |
| Luồng CPU khi suy luận | 4 |
| Batch suy luận | 64, giữ cấu hình đã lưu |

Kiểm tra dùng môi trường Python hiện có của nghiên cứu. Chưa kiểm tra việc cài lại thư viện từ đầu trên máy sạch hoặc chạy thực tế trên Linux/macOS. Việc di chuyển sang thư mục khác đã được kiểm tra trên Windows.

## Các lệnh kiểm tra

Trong các lệnh bên dưới, `python` là trình thông dịch của môi trường có các phiên bản nêu trên. Chạy từ thư mục gốc gói nộp; `--out` phải là đường dẫn mới để giữ nguyên các lần kiểm tra trước.

```powershell
python -m unittest discover -s scripts -p "test*.py" -v
python scripts/audit_results.py --out evaluation_outputs/recorded_metrics.json
python scripts/evaluate.py --run train_full_s42 --device cpu --out evaluation_outputs/cpu_train_full_s42
python scripts/evaluate.py --all --device cuda --out evaluation_outputs/cuda_all45
```

Trình thông dịch thực tế dùng cho các lệnh đánh giá là `..\.venv-seminar\Scripts\python.exe`. Lệnh unittest được chạy từ thư mục cha với `-s github_submission/scripts`; `TEMP` và `TMP` trỏ tới thư mục tạm trong workspace. Các kiểm thử bao gồm:

- 2 kiểm thử dựng dữ liệu: phạm vi từ vựng, giới hạn đồ thị, thứ tự mã hóa và nhận diện trùng nguyên văn.
- 9 kiểm thử trainer: kiến trúc attention, ablation cạnh, đồ thị không có cạnh, bảo toàn tập mẫu, manifest đầu vào, token đặc biệt và ngưỡng validation.
- 5 kiểm thử kiểm toán CSV: tệp bị sửa cùng kích thước, đường dẫn vượt thư mục gói, trùng ID, so sánh float32 tại ngưỡng và cách chọn ngưỡng khi đồng hạng.
- 5 kiểm thử evaluator: chuyển thư mục, checkpoint bị sửa, tệp không có trong manifest, đường dẫn không hợp lệ và mục manifest trùng nhau.
- 5 kiểm thử giải nén: tệp ZIP bị sửa, zip-slip, hash nội dung giải nén sai, từ chối ghi đè dữ liệu khác và bỏ qua tệp đã có đúng nội dung.

Ở bản giải nén riêng trong thư mục tạm, đã chạy thêm:

```powershell
python scripts/prepare_data.py --assets-dir <thu-muc-github_release_assets> --profile test
python scripts/prepare_data.py --assets-dir <thu-muc-github_release_assets> --profile full
python scripts/train_all.py --plan-only --seeds 42
```

Profile `test` cài 12 tệp đầu vào và 5 checkpoint. Profile `full` xác minh đủ 11 gói tài sản, bổ sung 40 checkpoint còn lại cùng 24 tệp dữ liệu nguồn; 17 tệp đã đúng được bỏ qua. Kiểm tra giải nén sử dụng tài sản ở máy, không phải tải lại từ Internet. Bước đóng gói cũng kiểm tra danh sách thành viên, kích thước và CRC của 11 gói ZIP dữ liệu/mô hình.

## Độ khớp của suy luận mới

Ngưỡng đối chiếu được giữ nguyên: sai số tuyệt đối của chỉ số `1e-4`; xác suất dùng `atol=1e-5`, `rtol=1e-4`. Không nới ngưỡng sau khi thấy kết quả.

Trên CUDA, cả 45 checkpoint đạt cả điều kiện chỉ số và xác suất. Sai lệch xác suất tuyệt đối lớn nhất là **4,470348358 × 10⁻⁷**; sai lệch lớn nhất trên mọi chỉ số là **8,920582116 × 10⁻⁶**; sai lệch AP lớn nhất là **4,800584908 × 10⁻⁶**. Tổng thời gian phần suy luận đo được là 58,47 giây, chưa tính đọc tệp, xác minh hash và dựng mô hình.

Trên CPU, `train_full_s42` cho AP **0,49971464586234005**, so với AP gốc **0,49972405480476584**, chênh **9,408942426 × 10⁻⁶**. F1 tại ngưỡng validation giữ nguyên **0,506426735218509**; precision, recall và MCC tại ngưỡng này cũng giữ nguyên. Toàn bộ chỉ số đạt sai số tuyệt đối `1e-4`, nhưng chênh lệch xác suất lớn nhất là **0,0008165836334228516**, vượt điều kiện xác suất mặc định. Vì vậy evaluator giữ trạng thái `different` và mã thoát 1. CPU/CUDA có khác biệt tính toán số; kết quả này không được ghi thành PASS và không được dùng để thay thế chỉ số luận văn.

Tập test kiểm tra có **6.293 mẫu, 369 mẫu dương**. Đây là đánh giá lại trên phép chia đã lưu; không chứng minh khả năng tổng quát hóa trên dự án hoặc thời điểm mới.

## Tệp bằng chứng

- [Kết quả kiểm toán dự đoán đã lưu](recorded_metrics.json): 45 lượt chạy, chỉ số tính lại và hash tệp đã sử dụng.
- [Suy luận mới trên CUDA](cuda_all45.json): trạng thái từng checkpoint, môi trường, sai số và hash dữ liệu/mô hình.
- [Suy luận mới trên CPU](cpu_train_full_s42.json): đầy đủ chênh lệch và trạng thái `different`.

Phạm vi xác minh số liệu ở đây là đợt thí nghiệm seminar **9 cấu hình × 5 seed, 42–46**. Các mã nguồn và báo cáo thí nghiệm lịch sử khác được cung cấp để đối chiếu, nhưng không được mô tả là đã huấn luyện lại hoặc đánh giá lại trong lần đóng gói này.

