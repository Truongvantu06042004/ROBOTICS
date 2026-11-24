# Định vị và lập bản đồ đồng thời cho robot di động

Đề tài khảo sát hệ thống **SLAM sử dụng LiDAR 2D** cho robot TurtleBot3 Waffle Pi trong môi trường mô phỏng. Robot tự tìm các vùng biên để chọn mục tiêu, di chuyển bằng Nav2 và xây dựng bản đồ trong khi hoạt động.

Các thử nghiệm được thực hiện trên Ubuntu 22.04, ROS 2 Humble, Gazebo Classic, SLAM Toolbox, Nav2 và RViz. Robot được chạy trong hai môi trường mô phỏng `turtlebot3_world` và `turtlebot3_house`.

## Phương pháp

- **SLAM Toolbox** sử dụng dữ liệu LiDAR và odometry để xây dựng bản đồ và ước lượng vị trí robot.
- **Frontier** là biên giữa vùng trống đã biết và vùng chưa biết. Gói `frontier_exploration_ros2` phát hiện các cụm biên theo WFD và xếp thứ tự mục tiêu dựa trên MRTSP, sử dụng quy hoạch động giới hạn.
- **Nav2** lập kế hoạch và điều khiển robot đến mục tiêu. Trong cấu hình House, NavFn được dùng cho đường đi toàn cục và DWB cho điều khiển cục bộ.
- **RViz** hiển thị bản đồ, dữ liệu cảm biến và quá trình robot khám phá.

Đề tài thử hai ngưỡng kích thước cụm frontier: **tối thiểu 5 ô** và **tối thiểu 6 ô**. Đây là số ô tối thiểu của một cụm biên, không phải diện tích môi trường robot chắc chắn sẽ quan sát được.

## Những phần đã thực hiện

- Tích hợp mô phỏng robot, SLAM, khám phá frontier và điều hướng trên ROS 2 Humble.
- Thiết lập hai môi trường World và House, sau đó chạy thử với các ngưỡng frontier 5 ô và 6 ô.
- Tạo giao diện demo để chọn môi trường, ngưỡng frontier và thời gian chạy; có thể mở RViz, xem log và kết quả đã lưu.
- Ghi bản đồ, quỹ đạo, cấu hình và log; phân tích sai số định vị và đối chiếu bản đồ với dữ liệu tham chiếu phù hợp cho từng môi trường.
- Khảo sát khu vực khoảng hở của House và hiện tượng robot lặp lại mục tiêu frontier.

## Kết quả mô phỏng

Bảng dưới đây là kết quả của **bốn lượt chính có mở RViz**. Thời gian được tính theo đồng hồ thực, kể từ khi bộ ghi bắt đầu.

| Môi trường | Frontier tối thiểu | Thời gian ghi | Diện tích ô đã biết | RMSE vị trí | Tín hiệu hết frontier |
|---|---:|---:|---:|---:|---|
| World | 5 ô | 129,89 s | 21,07 m² | 0,020 m | 109,89 s |
| World | 6 ô | 155,12 s | 21,35 m² | 0,016 m | 135,10 s |
| House | 5 ô | 1.200,01 s | 86,29 m² | 0,047 m | Chưa ghi nhận |
| House | 6 ô | 1.200,01 s | 99,21 m² | 0,046 m | 1.195,01 s |

**Diện tích ô đã biết** bao gồm cả ô trống và ô chiếm dụng; đây không phải phần trăm diện tích sàn được bao phủ. **RMSE vị trí** được tính bằng cách so sánh quỹ đạo SLAM với tư thế Gazebo sau khi căn chỉnh tại mẫu đầu. Tư thế Gazebo và TF được ghép theo thời điểm nhận nên vẫn có sai số đồng bộ.

Với World, tỷ lệ ô trống trong bản đồ tham chiếu đã được SLAM ghi nhận là đã biết lần lượt là **98,79%** và **99,81%** cho ngưỡng 5 và 6 ô. Giá trị này phụ thuộc bản đồ tham chiếu và phép căn chỉnh; nó không phải độ chính xác tuyệt đối.

Với House, tại dung sai 0,15 m, tỷ lệ mẫu tim tường SDF có ô chiếm dụng gần tương ứng là **79,07%** và **89,25%**. Chỉ tiêu này đánh giá các mẫu tường đã chọn, không phải tỷ lệ bao phủ toàn bộ ngôi nhà.

## Hạn chế

- Lượt House 5 ô trong bảng dừng theo giới hạn 1.200 giây và chưa ghi nhận tín hiệu hết frontier. Lượt House 6 ô ghi nhận tín hiệu sau khoảng 1.195 giây; tín hiệu đó không chứng minh mọi khu vực đã được khảo sát.
- Một lượt House 5 ô không giới hạn được chạy riêng trong khoảng **24.956 giây**. RViz đóng và lượt ghi bị gián đoạn khi chưa có tín hiệu hết frontier; log cho thấy có mục tiêu được chọn lặp lại.
- Vùng xám gần khoảng hở House chưa được xác định là đã khám phá. Bản đồ House chưa có tham chiếu đầy đủ đã xác minh để tính tỷ lệ bao phủ sàn.
- Mỗi cấu hình trong bảng có một lượt chính. Kết quả chịu ảnh hưởng của tuyến đi và thời điểm dừng; cần chạy lặp thêm để đánh giá độ ổn định. Các số liệu trên chỉ phản ánh mô phỏng, chưa phải kết quả trên robot thật.

## Nội dung của kho GitHub

Kho này hiện chứa bản sao mã nguồn **`frontier_exploration_ros2`**. Thư mục `do_an_slam` gồm cấu hình thử nghiệm, giao diện demo, công cụ ghi và phân tích dữ liệu, báo cáo cùng kết quả mô phỏng đang được lưu riêng trên máy; những phần đó **chưa được đưa lên kho này**.

Để lấy mã frontier trong kho:

```bash
git clone https://github.com/Truongvantu06042004/ROBOTICS.git
```

Lệnh trên chỉ tải nội dung hiện có trên GitHub. Hướng dẫn chạy giao diện demo và các tệp kết quả của đề tài nằm trong thư mục `do_an_slam` trên máy.

## Nguồn tham khảo và giấy phép

Mã khám phá frontier được kế thừa từ [mertgulerx/frontier_exploration_ros2](https://github.com/mertgulerx/frontier_exploration_ros2), tại commit [`11864e4ac6789dc81fd71bad51e43339c01deca2`](https://github.com/mertgulerx/frontier_exploration_ros2/tree/11864e4ac6789dc81fd71bad51e43339c01deca2). Thuật toán frontier và SLAM không được xây dựng từ đầu trong đề tài.

Giấy phép của mã frontier gốc là [Apache License 2.0](LICENSE). Thông tin tác giả và trích dẫn của dự án gốc được giữ trong [CITATION.cff](CITATION.cff).
