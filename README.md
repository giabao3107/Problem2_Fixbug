# Dự án: Xây dựng và Kiểm thử Chiến lược Giao dịch Chứng khoán Kết hợp Phân tích Kỹ thuật và Học máy

## Giới thiệu Tổng quan

Dự án này trình bày chi tiết quá trình nghiên cứu, xây dựng và kiểm thử một chiến lược giao dịch chứng khoán phức hợp, được áp dụng trên dữ liệu thị trường chứng khoán Việt Nam trong giai đoạn biến động từ đầu năm 2022 đến cuối năm 2024. Mục tiêu chính của dự án là phát triển một hệ thống giao dịch có khả năng thích ứng với sự thay đổi của thị trường bằng cách kết hợp các chỉ báo phân tích kỹ thuật kinh điển và các mô hình học máy tiên tiến.

Trọng tâm của nghiên cứu là sự kết hợp giữa ba chỉ báo kỹ thuật phổ biến—Relative Strength Index (RSI), Parabolic SAR (PSAR), và Mẫu hình nến Engulfing—với các thuật toán học máy như XGBoost và LightGBM. Qua đó, chiến lược không chỉ tìm kiếm các tín hiệu giao dịch tiềm năng dựa trên các quy tắc định sẵn mà còn tối ưu hóa việc lựa chọn cổ phiếu và quản lý rủi ro một cách hệ thống, với hy vọng vượt qua những hạn chế của các phương pháp truyền thống.

## Dữ liệu và Kỹ thuật Tạo biến

Cơ sở dữ liệu cho quá trình phân tích là tệp `vietnam_stocks_data.csv`, chứa dữ liệu giá lịch sử (Mở cửa, Cao nhất, Thấp nhất, Đóng cửa, Khối lượng) của các cổ phiếu trên thị trường Việt Nam. **Lưu ý:** Do kích thước lớn (>100MB), tệp dữ liệu này không được lưu trữ trực tiếp trên GitHub. Vui lòng tải xuống tệp dữ liệu từ [Google Drive](https://drive.google.com/file/d/your-file-id/view) hoặc liên hệ với tác giả để nhận tệp dữ liệu.

Để các mô hình học máy có thể nhận diện được các mẫu hình phức tạp của thị trường, một quá trình kỹ thuật tạo biến (feature engineering) đã được thực hiện một cách có hệ thống. Từ dữ liệu gốc, một tập hợp phong phú gồm 33 biến số mới đã được tạo ra. Các biến này được thiết kế để nắm bắt các khía cạnh khác nhau của hành vi giá và khối lượng, chẳng hạn như động lượng (RSI momentum), sự thay đổi xu hướng (PSAR trend change), sự đột biến về khối lượng (Volume Anomaly), và mức độ biến động (Volatility). Những biến số này cung cấp cho mô hình một cái nhìn đa chiều và sâu sắc hơn về trạng thái của từng cổ phiếu.

## Phương pháp luận và Kiến trúc Chiến lược

Chiến lược giao dịch được xây dựng theo một kiến trúc đa tầng, kết hợp giữa hệ thống lọc dựa trên quy tắc phân tích kỹ thuật và hệ thống lựa chọn được tăng cường bởi học máy.

### Tầng 1: Hệ thống Lọc và Chấm điểm Cổ điển

Lõi của chiến lược là một bộ lọc dựa trên các quy tắc logic từ ba chỉ báo chính. Hệ thống này phân loại trạng thái của cổ phiếu thành các tín hiệu như "strong_buy" (mua mạnh) hay "weak_buy" (mua yếu) dựa trên sự kết hợp của các điều kiện như RSI nằm trong vùng tăng giá (trên 50), giá vượt lên trên đường PSAR, và sự xuất hiện của mẫu hình nến Bullish Engulfing. Đồng thời, một hệ thống chấm điểm trên thang 100 được thiết kế để lượng hóa sức mạnh của từng tín hiệu, cung cấp một thước đo định lượng ban đầu.

### Tầng 2: Lựa chọn Cổ phiếu Tăng cường bằng Học máy

Đây là tầng cao cấp nhất của chiến lược, nơi sức mạnh của học máy được phát huy để tinh chỉnh quyết định giao dịch. Một điểm số cuối cùng (`final_score`) được tính toán cho mỗi cổ phiếu, dựa trên một công thức trọng số kết hợp giữa xác suất mua được dự đoán bởi mô hình học máy (Ensemble model), điểm số từ hệ thống cổ điển, và lợi nhuận kỳ vọng từ mô hình hồi quy.

Quy trình lựa chọn cổ phiếu hàng ngày sử dụng một ngưỡng điểm thích ứng. Thay vì một ngưỡng cứng, chiến lược chỉ xem xét các cổ phiếu có điểm số cuối cùng nằm trong nhóm cao nhất (ví dụ: top 12%) của tất cả các cổ phiếu trong ngày. Cách tiếp cận này giúp chiến lược tự động điều chỉnh độ "khắt khe", chỉ chọn những cơ hội tốt nhất trong những ngày thị trường tích cực và vẫn có thể tìm thấy cơ hội hiếm hoi trong những ngày thị trường ảm đạm.

### Tầng 3: Kiểm thử Ngược (Backtesting)

Hiệu quả của chiến lược được đánh giá thông qua một quá trình kiểm thử ngược chi tiết trên dữ liệu lịch sử. Bắt đầu với số vốn giả định là 1,000,000 VND, hệ thống mô phỏng việc mở và đóng vị thế, tính toán chi phí giao dịch, và quản lý danh mục. Các quy tắc quản lý rủi ro được áp dụng một cách nghiêm ngặt, bao gồm chốt lời ở mức 15%, cắt lỗ ở mức 8%, và các điều kiện thoát lệnh linh hoạt khác như Trailing Stop hoặc khi tín hiệu xu hướng yếu đi (RSI cắt xuống dưới 50).

## Kết quả Phân tích và Hiệu suất

Kết quả kiểm thử ngược cho thấy chiến lược có hiệu suất ấn tượng về mặt lợi nhuận. Từ số vốn ban đầu, chiến lược đã kết thúc giai đoạn kiểm thử với giá trị danh mục đạt 2,287,820 VND, tương đương tổng lợi nhuận 128.78% sau gần ba năm.

Phân tích sâu hơn cho thấy một bức tranh đa chiều. Tỷ lệ thắng (Win Rate) của chiến lược chỉ ở mức 38.26%. Thành công không đến từ việc "đoán đúng" thường xuyên, mà đến từ một tỷ lệ Lời/Lỗ (Win/Loss Ratio) rất cao. Lợi nhuận trung bình trên mỗi lệnh thắng lớn hơn rất nhiều so với mức lỗ trung bình, tạo ra một kỳ vọng toán học dương mạnh mẽ. Đây là đặc điểm của một chiến lược "Home Run", phụ thuộc vào một số ít giao dịch thắng lớn để bù đắp cho nhiều giao dịch thua lỗ nhỏ.

Tuy nhiên, rủi ro của chiến lược là rất đáng kể, với mức sụt giảm vốn tối đa (Maximum Drawdown) lên tới -63.21%. Một phát hiện quan trọng là chiến lược hoạt động hiệu quả vượt trội trong các giai đoạn thị trường có biến động mạnh, khẳng định bản chất là một chiến lược bắt sóng và đi theo xu hướng (momentum and trend-following).

## Hướng dẫn Cài đặt và Sử dụng

Để chạy lại các phân tích trong dự án này, môi trường của bạn cần đáp ứng các yêu cầu sau:

1.  **Cài đặt Python**: Đảm bảo bạn đã cài đặt Python phiên bản 3.8 trở lên.
2.  **Cài đặt Thư viện**: Các thư viện cần thiết cho dự án được liệt kê trong tệp `requirements.txt`. Bạn có thể cài đặt chúng bằng lệnh sau:
    ```sh
    pip install -r requirements.txt
    ```
3.  **Chạy Phân tích**: Mở và chạy toàn bộ các ô lệnh trong tệp Jupyter Notebook `problem2_Fixbug.ipynb` để tái tạo lại quá trình từ tiền xử lý dữ liệu, huấn luyện mô hình, đến kiểm thử ngược và phân tích kết quả.

## Hạn chế và Hướng phát triển Tương lai

Nghiên cứu này, dù chi tiết, vẫn có những hạn chế cố hữu như các giả định trong môi trường backtest (không có trượt giá, khớp lệnh tức thì) và khả năng mô hình bị "lỗi thời" (model drift) theo thời gian.

Để đưa chiến lược vào ứng dụng thực tế, các hướng phát triển trong tương lai là cần thiết, bao gồm:
*   **Quản lý Rủi ro Nâng cao**: Áp dụng các kỹ thuật cắt lỗ động dựa trên chỉ báo ATR và xác định quy mô vị thế dựa trên mức độ rủi ro.
*   **Tích hợp Dữ liệu Đa dạng**: Làm giàu mô hình bằng các nguồn dữ liệu thay thế như dữ liệu vĩ mô hoặc phân tích tin tức tài chính.
*   **Khám phá các Kiến trúc Mô hình Tiên tiến**: Áp dụng các mô hình học sâu như LSTM hoặc Transformer để nắm bắt các mối quan hệ phức tạp hơn trong dữ liệu chuỗi thời gian.
