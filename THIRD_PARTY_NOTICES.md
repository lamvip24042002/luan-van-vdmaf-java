# Nguồn kế thừa và thông tin giấy phép

Tài liệu này ghi nhận nguồn của mã và dữ liệu trong gói phục vụ đánh giá luận văn. Ngày kiểm tra: **22/10/2026**. Các thành phần của bên thứ ba giữ nguyên quyền của tác giả/chủ sở hữu tương ứng. Gói này không áp một giấy phép chung mới lên toàn bộ mã, dữ liệu, checkpoint hoặc tài liệu.

## VDMAF

- Nguồn: [yangli12138/VDMAF](https://github.com/yangli12138/VDMAF).
- Công trình: Y. Li, Q. Luo, P. Wu và H. Zheng, *VDMAF: Cross-language source code vulnerability detection using multi-head attention fusion*, Information and Software Technology, 183, 107739, 2025. [DOI](https://doi.org/10.1016/j.infsof.2025.107739).
- Tài liệu đối chiếu trong workspace gốc ghi rằng bản `VDMAF_main.zip` dùng làm nguồn được tải ngày 06/07/2026. Gói mã nộp có các phần kế thừa, điều chỉnh để chạy thí nghiệm và phần phát triển VDMAF-Java; không phải bản nguyên vẹn của kho gốc.
- Tệp [LICENSE của upstream](https://github.com/yangli12138/VDMAF/blob/main/LICENSE), đọc khi đóng gói, mang **MIT License**, `Copyright (c) 2026 yangli12138`. Toàn văn được giữ tại [VDMAF-MIT.txt](VDMAF-MIT.txt).
- README hiện tại của upstream đồng thời ghi CC BY 4.0 ở mục giấy phép. Đây là thông tin không đồng nhất giữa README và LICENSE của nguồn; ghi nhận cả hai thay vì tự quyết định lại giấy phép của upstream. Có thể xem nguyên bản tại [README upstream](https://github.com/yangli12138/VDMAF/blob/main/README.md).

## Bộ dữ liệu Java dẫn xuất từ MegaVul

- Nguồn sử dụng trực tiếp: [athrv/megavul-vulnerability-detection-java trên Hugging Face](https://huggingface.co/datasets/athrv/megavul-vulnerability-detection-java).
- Ba tệp được sử dụng: `train-00000-of-00001.parquet`, `validation-00000-of-00001.parquet`, `test-00000-of-00001.parquet`. Bản gốc trong workspace nằm tại `ts/VDMAF_main/newidea/Java/megavul/`.
- Số dòng nguồn tương ứng: **31.461 / 4.195 / 6.293**, tổng **41.949**. Nhãn `is_vulnerable` bằng 1 biểu thị mẫu có lỗ hổng, 0 biểu thị lớp còn lại theo dữ liệu nguồn. Không thực thi các hàm Java trong bộ dữ liệu khi đánh giá mô hình.
- Dữ liệu giữ các trường nguồn như tên repository, đường dẫn tệp và commit hash. Các đoạn mã đến từ nhiều dự án; sự hiện diện trong dataset không thay thế thông tin quyền của các dự án gốc.
- Khi kiểm tra, trang dataset và cây tệp được truy cập không hiển thị một giấy phép dataset rõ ràng. Không tìm thấy LICENSE đi kèm bản parquet cục bộ. **Không gán MIT, CC BY hoặc giấy phép tự suy đoán cho dataset.** Thông tin quyền của dữ liệu được giữ là chưa xác định trong gói này; đối chiếu với nguồn khi sử dụng lại.
- Đây là **bản dẫn xuất Java trên Hugging Face**. Không mô tả nó là bộ Java do bài báo MegaVul gốc công bố. Công trình MegaVul gốc mô tả dữ liệu C/C++: C. Ni và cộng sự, *MegaVul: A C/C++ Vulnerability Dataset with Comprehensive Code Representation*, [arXiv:2406.12415](https://arxiv.org/abs/2406.12415).

Các tệp `labels_all.tsv`, dump đồ thị Joern, tensor đồ thị/chuỗi, từ vựng và tập ID kiểm thử không trùng chính xác được tạo từ bộ dữ liệu trên bằng pipeline nghiên cứu. Tập kiểm thử dùng mô hình giữ đủ **6.293** mẫu, trong đó **369** mẫu dương tính. Các bộ train/validation sau loại đồ thị rỗng lần lượt có **31.448 / 4.193** mẫu. Mã SHA-256 trong các manifest của gói dùng để xác định đúng nội dung tệp; không được hiểu là thông tin giấy phép.

## PCFinder và công cụ phụ thuộc

- Phân tích về kiểm tra quyền kế thừa ý tưởng từ Y. Shi và cộng sự, *Facilitating Access Control Vulnerability Detection in Modern Java Web Applications With Accurate Permission Check Identification*, IEEE TIFS, 20, 10683–10698, 2025. [DOI](https://doi.org/10.1109/TIFS.2025.3614424). Các đặc trưng thử nghiệm được mô tả trong luận văn là đặc trưng tái dựng lấy cảm hứng từ PCFinder, không đồng nhất với chạy đầy đủ artifact PCFinder.
- Joern được dùng để sinh CPG; [tài liệu CPG chính thức](https://docs.joern.io/code-property-graph/). Bản dump đã lưu cho phép tái dựng tensor mà không phải chạy lại Joern.
- PyTorch, PyTorch Geometric, NumPy, pandas, PyArrow, SciPy, scikit-learn và các thư viện khác được cài theo tệp requirements/môi trường. Các bộ cài và môi trường ảo không được coi là mã do tác giả luận văn sở hữu. Giấy phép của từng thư viện được cung cấp bởi bản phân phối tương ứng.

Các kết quả lịch sử và đợt bổ sung 11/10/2026 có nguồn đầu vào, môi trường và protocol riêng. Ghi công nguồn không đồng nghĩa với xác nhận rằng mọi kết quả của upstream đã được tái lập lại trong lần đóng gói này.

