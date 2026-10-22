# Gói đánh giá luận văn VDMAF-Java

Mã nguồn, dữ liệu kiểm thử và mô hình phục vụ đánh giá đợt thí nghiệm bổ sung **11/10/2026**: **9 cấu hình × 5 seed = 45 lượt chạy**. Gói được đóng ngày **22/10/2026**.

Kho công khai: [lamvip24042002/luan-van-vdmaf-java](https://github.com/lamvip24042002/luan-van-vdmaf-java). Các tệp dữ liệu và checkpoint nằm trong [Release v1.0-evaluation](https://github.com/lamvip24042002/luan-van-vdmaf-java/releases/tag/v1.0-evaluation). Mã nguồn đầy đủ và kết quả nhỏ được đóng trong **source_and_results.zip**; không cần tải môi trường Python hay toàn bộ thư mục làm việc của tác giả.

## 1. Bắt đầu

1. Tải [source_and_results.zip](https://github.com/lamvip24042002/luan-van-vdmaf-java/releases/download/v1.0-evaluation/source_and_results.zip) và `SHA256SUMS.txt` từ Release. Kiểm tra SHA-256 của ZIP bằng `Get-FileHash source_and_results.zip -Algorithm SHA256` (PowerShell) hoặc `sha256sum source_and_results.zip` (Linux), so với dòng tương ứng trong `SHA256SUMS.txt`, rồi giải nén vào một thư mục mới.
2. Mở terminal tại thư mục vừa giải nén (nơi có `README.md`, `scripts/`, `MANIFEST.json`).
3. Tạo môi trường **Python 3.12**; bản gốc dùng 3.12.14. Windows:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements-audit.txt
python scripts/audit_results.py --out evaluation_outputs/recorded_metrics.json
```

Linux/macOS dùng `source .venv/bin/activate`. Nếu Windows chặn Activate.ps1, gọi trực tiếp `.venv\Scripts\python.exe` thay cho `python`; không cần đổi chính sách hệ thống.

Lệnh audit kiểm tra đủ 45 lượt, các hash liên quan, nhãn/ID mẫu, ngưỡng chọn trên validation và tính lại các chỉ số từ dự đoán đã lưu. **Đây là kiểm tra kết quả lưu sẵn, chưa chạy suy luận mô hình.** Kết quả đầy đủ phải ghi `status: passed` và đủ 45 lượt. Xem `VERIFICATION.md` về kiểm tra thực tế khi đóng gói.

## 2. Chạy mô hình trên toàn bộ tập test

Cài PyTorch CPU và các phụ thuộc đúng phiên bản:

```powershell
python -m pip install torch==2.5.1 --index-url https://download.pytorch.org/whl/cpu
python -m pip install -r requirements.txt
python scripts/prepare_data.py --download --profile test
python scripts/evaluate.py --run train_full_s42 --device cpu --out evaluation_outputs/check_s42
```

`prepare_data.py` tải các asset từ release, kiểm tra hash trước khi giải nén và không ghi đè tệp có nội dung khác. Profile `test` tải tensor đầu vào cho cả hai chế độ vocabulary cùng checkpoint của nhóm `train_full` (5 seed). Gói dữ liệu giữ cả train/validation để có thể kiểm tra và huấn luyện lại; lệnh `evaluate.py` chỉ suy luận trên test.

Không có mạng: tải các ZIP dữ liệu/model từ Release vào thư mục `downloads/`, rồi chạy:

```powershell
python scripts/prepare_data.py --assets-dir downloads --profile test
```

Tập test: **6.293 mẫu, 369 dương tính**. Với `train_full_s42`, kết quả lưu gốc: **AP = 0.4997240548**, **ROC-AUC = 0.8691734875**, **F1 = 0.5064267352** với ngưỡng validation **0.95**. AP ở đây là `average_precision_score`, không phải diện tích PR hình thang. So sánh nhị phân dùng `probability > threshold` theo mã gốc.

Evaluator kiểm tra checksum trước khi nạp dữ liệu/checkpoint, giữ ngưỡng validation, xuất CSV dự đoán và JSON sai lệch. Sai số số học CPU/GPU có thể khác; công cụ báo sai lệch và trả mã thoát lỗi khi vượt dung sai đã ghi trong báo cáo. Không chọn lại ngưỡng bằng test.

**Kết quả CPU đã quan sát khi đóng gói:** `train_full_s42` có AP `0.4997146459`, F1 giữ nguyên; sai lệch xác suất lớn nhất khoảng `0.000817`. Tất cả chỉ số được so sánh nằm trong dung sai `1e-4`, nhưng xác suất từng mẫu vượt dung sai chặt `atol=1e-5, rtol=1e-4`; vì vậy công cụ trả `different` và mã thoát 1 trên cấu hình CPU này. Đây là kết quả đã ghi nhận, không được đổi thành `passed`. Dùng GPU và môi trường gốc để kiểm tra độ khớp chặt hơn; đọc sai lệch thực tế trong `summary.json` và `VERIFICATION.md`.

Để đánh giá lại **cả 45 checkpoint**:

```powershell
python scripts/prepare_data.py --download --profile full
python scripts/evaluate.py --all --device cpu --out evaluation_outputs/all_runs
```

GPU NVIDIA có thể dùng `--device cuda`. Môi trường gốc: `torch==2.5.1+cu121`, `torch-geometric==2.6.1`, RTX 3060. Để dùng CUDA 12.1, thay lệnh cài torch bằng `python -m pip install torch==2.5.1 --index-url https://download.pytorch.org/whl/cu121`. Bản đóng gói không chứa driver, CUDA toolkit hoặc môi trường ảo.

## 3. Dữ liệu và tổ chức tệp

| Tập | Số mẫu đưa vào mô hình | Mẫu dương |
|---|---:|---:|
| Train | 31.448 | 1.836 |
| Validation | 4.193 | 228 |
| Test | 6.293 | 369 |

Nguồn là bản Java dẫn xuất [athrv/megavul-vulnerability-detection-java](https://huggingface.co/datasets/athrv/megavul-vulnerability-detection-java). Parquet nguồn có 31.461/4.195/6.293 dòng; chênh lệch train/validation do loại đồ thị rỗng. Giữ nguyên phép chia, ID, nhãn và thứ tự mẫu đã dùng trong thí nghiệm.

```text
scripts/                              Mã huấn luyện, dựng dữ liệu, kiểm tra và đánh giá
output/seminar_20261011/protocol.json   Protocol gốc: 9 cấu hình, seed 42–46
output/seminar_20261011/inputs/         Tensor, vocabulary, thống kê và manifest
output/seminar_20261011/runs/           45 checkpoint, cấu hình, dự đoán và chỉ số
output/seminar_20261011/analysis/       Tổng hợp gốc, so sánh ghép cặp và độ nhạy
ts/VDMAF_main/newidea/Java/graph/       Nhãn, mã Joern và dump đầu vào
ts/VDMAF_main/newidea/Java/megavul/     Ba parquet nguồn
ts/                                   Mã và tài liệu lịch sử để đối chiếu
MANIFEST.json                         SHA-256 và kích thước từng tệp trong gói
ASSETS.json                           SHA-256/kích thước/các tệp của ZIP bổ sung
evaluation_outputs/                   Kết quả chạy mới, không sửa bằng chứng gốc
```

Các đường dẫn tuyệt đối trong JSON lịch sử được giữ nguyên như bằng chứng nguồn gốc; công cụ đánh giá mới xác định đường dẫn theo thư mục gói hiện tại. Không cần tạo tài khoản Windows hay đường dẫn giống máy tác giả. Trainer và dữ liệu lịch sử được giữ nguyên byte để đối chiếu hash.

## 4. Huấn luyện và dựng đầu vào lại

Sau khi tải profile `full`, huấn luyện từ đầu trong thư mục mới:

```powershell
python scripts/train_all.py --device cuda --out evaluation_outputs/retrained
```

Lệnh chạy 9 cấu hình × seed 42–46 với cùng siêu tham số; có thể dùng `--seeds 42` để kiểm tra trước. `--device cpu` được hỗ trợ nhưng sẽ chậm hơn. Mỗi lần chạy phải dùng thư mục đầu ra mới. Không ghi kết quả mới đè lên 45 lượt gốc. Các phép toán GPU có thể không tái lập từng bit.

Dựng lại tensor từ parquet và dump Joern đã lưu:

```powershell
python scripts/prepare_data.py --download --profile full
python scripts/build_seminar_inputs_20261011.py --root . --output evaluation_outputs/rebuilt_inputs
```

Builder giữ chính sách min-count 5, tối đa 2.000 node, tối đa 200 token cuối, ánh xạ AST/CFG/CDG/REACHING_DEF = 0/1/2/3. Hash tệp được tạo lại có thể khác do metadata/serialization; đối chiếu ID, nhãn, tensor, vocabulary và invariants. Bản dump gốc cho phép dựng tensor mà không phải cài Joern. Mã sinh đồ thị được kèm để kiểm tra; việc dựng lại CPG từ công cụ Joern mới không được cam kết cho kết quả giống từng byte.

Kiểm thử mã:

```powershell
python -m unittest discover -s scripts -p "test_*.py" -v
```

## 5. Phạm vi kết luận

Đợt 45 lượt là phần có đủ dữ liệu, checkpoint và đường đánh giá trực tiếp trong gói này. `all_full` là đối chứng vocabulary xây từ mọi split; `train_full` và các ablation dùng vocabulary chỉ từ train. Các kết quả khác nhau được giữ nguyên, không chỉ chọn cấu hình tốt nhất.

Các bảng lịch sử Reveal/Devign/Bagheri, pilot Java và đặc trưng lấy cảm hứng từ PCFinder trong luận văn có mã/tài liệu tham khảo đi kèm. **Gói này chưa chứa toàn bộ dữ liệu, checkpoint và môi trường để tái lập mọi bảng lịch sử đó**; không dùng chúng thay thế cho kết quả 45 lượt mới.

Mean/SD qua 5 seed mô tả biến thiên tối ưu hóa trên một phép chia cố định. Tập test loại trùng nguyên văn là phân tích độ nhạy sau huấn luyện; không phải phép chia theo dự án/thời gian. Concat/untyped thay đổi số tham số; chưa phải đối chứng cân bằng tham số. Train-only vocabulary không loại mọi dạng rò rỉ từ phép chia gốc.

Xem [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) về nguồn và quyền dữ liệu. Không tự gán giấy phép chung cho dữ liệu bên thứ ba. Gói không chứa bản thảo có thông tin cá nhân, môi trường ảo, cache, công cụ cài đặt hay tệp tạm của tác giả.
