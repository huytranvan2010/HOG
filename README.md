<script type="text/javascript" async
src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js? 
config=TeX-MML-AM_CHTML"
</script>

# HOG
HOG - Histogram of Oriented Gradients là một feature descriptor thường được sử dụng để trích xuất các đặc trưng của bức ảnh. Nó được dùng khá nhiều trong computer vision với những ứng dụng chính như sau:
- Nhận diện người (human detection). HOG lần đầu tiên được giới thiệu bởi Dalal và Triggs [Histograms of Oriented Gradients for Human Detection](https://lear.inrialpes.fr/people/triggs/pubs/Dalal-cvpr05.pdf)
- Nhận diện khuôn mặt (face detection)
- Nhận diện các vật thể khác
- Tạo feature cho bài toán phân loại ảnh. Bản thân HOG là feature descriptor do đó nó trích xuất ra các feature đặc trưng, từ đó có thể sử dụng để áp dụng vào các mô hình khác để phân loại ảnh.

Một số điểm làm HOG khác biệt so với các feature desciptor khác:
- HOG tạo trung vào cấu trúc, hình dạng của object. Điều này có thể được mô tả thông qua edge direction (hướng của biên), edge direction lại được xác định bởi độ lớn gradient (gradient magnitude) và hướng của gradient(gradient orientation)
- Hướng và độ lớn gradient được xác định cục bộ (bức ảnh được chia thành nhiều phần và mỗi phần sẽ xác định hướng và độ lớn gradient)
- Cuối cùng HOG sẽ tạo ra histogram riêng cho mỗi phần này

Có 5 bước để xây dựng HOG cho bức ảnh:
1. Chuẩn hóa ảnh
2. Tính gradient theo hai hướng x, y
3. Tính toán histogram cho từng vùng của bức ảnh
4. Chuẩn hóa các khối (blocks)
5. Tập hợp các Histograms of Oriented gradients để tạo feature vector cuối

## Bước 1: Chuẩn hóa ảnh
Cũng giống các bài toán khác đầu tiên chúng ta cần resize kích thước tất cả các hình ảnh trong tập dữ liệu về một kích thước chung, hay dùng 64 x 128.
![Preprocessing](/images/preprocessing.jpg)
Việc chuẩn hóa ảnh là tùy chọn, trong một số trường hợp nó có thể cải thiện performance của HOG descriptor. Nếu dùng ảnh màu thay cho ảnh xám hiệu suất cũng cải thiện đôi chút.
Dưới đây là một số phương pháp chuẩn hóa chính:
- Chuẩn hóa gamma/power : lấy $ \log(p) $ của mỗi pixel p trong ảnh đầu vào. Tuy nhiên Dalal và Triggs chứng minh phương pháp này có thể ảnh hưởng đến performance.
- Chuẩn hoá căn bậc hai (square root normalization): lấy $ \sqrt(p) $ của mỗi pixel p trong hình ảnh đầu vào. Chuẩn hóa căn bậc hai thường làm tăng độ chính xác hơn là giảm.
- Chuẩn hóa phương sai (variance normalization): Ở đây, chúng ta tính cần giá trị cường độ điểm ảnh trung bình $\mu $ và độ lệch tiêu chuẩn $\sigma $ của hình ảnh đầu vào. Với mỗi điểm ảnh ta trừ đi giá trị trung bình của cường độ điểm ảnh và sau đó được chuẩn hóa bằng cách chia cho độ lệch chuẩn: $ p' = (p - \mu) / \sigma $

## Bước 2: Tính gradient theo hai hướng x, y


