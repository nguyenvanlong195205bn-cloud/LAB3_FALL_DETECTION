# LAB 3: XÂY DỰNG HỆ THỐNG PHÁT HIỆN TÉ NGÃ (FALL DETECTION) SỬ DỤNG ESP32, FREERTOS VÀ MQTT

**Họ và tên:** Nguyễn Văn Long

**Lớp:** KSKTVT

**Đề tài:** Thiết kế Mạch phát hiện té ngã bằng phép đo gia tốc đột ngột và thay đổi góc nghiêng, tích hợp IoT Dashboard[cite: 1].

---

## 1. Giới thiệu bài toán
Hệ thống được thiết kế nhằm giám sát và cảnh báo sự cố té ngã cho người dùng (đặc biệt là người cao tuổi hoặc người bệnh)[cite: 1]. Sử dụng vi điều khiển **ESP32** kết hợp cảm biến gia tốc và con quay hồi chuyển **MPU6050** để thu thập dữ liệu chuyển động theo thời gian thực[cite: 1]. 
* Khi phát hiện có va chạm mạnh kết hợp thay đổi góc nghiêng bất thường, hệ thống sẽ kích hoạt còi báo động và đèn LED tại chỗ[cite: 1], đồng thời truyền dữ liệu qua giao thức **MQTT** lên Web Dashboard để hiển thị biểu đồ và hỗ trợ điều khiển từ xa[cite: 1].
* Ứng dụng hệ điều hành thời gian thực **FreeRTOS** để phân chia các tác vụ (Task) chạy đa nhiệm trên hai nhân (Core 0 & Core 1) của ESP32 nhằm đảm bảo độ trễ thấp và tính ổn định cao[cite: 1].

---

## 2. Công nghệ sử dụng
* **Phần cứng vi điều khiển:** ESP32 NodeMCU / DevKit V1.
* **Cảm biến:** MPU6050 (6 trục: Gia tốc kế và Con quay hồi chuyển)[cite: 1].
* **Cơ cấu chấp hành:** Đèn LED cảnh báo (qua điện trở 220Ω), Còi Buzzer[cite: 1].
* **Hệ điều hành nhúng:** FreeRTOS (Đa nhiệm, quản lý Queue, phân chia Core 0 và Core 1)[cite: 1].
* **Giao thức truyền thông & IoT:** 
  * Wi-Fi (Kết nối mạng không dây)[cite: 1].
  * MQTT Protocol (PubSubClient qua Broker `broker.emqx.io`)[cite: 1].
* **Giao diện Web Dashboard:** HTML5, CSS, JavaScript kết hợp thư viện **Chart.js** (vẽ biểu đồ thời gian thực) và **MQTT.js** (giao tiếp WebSocket)[cite: 1].
* **Môi trường mô phỏng:** Wokwi Simulator.

---

## 3. Sơ đồ kết nối phần cứng (Pinout)

| Linh kiện | Chân linh kiện | Chân kết nối ESP32 | Ghi chú |
| :--- | :--- | :--- | :--- |
| **MPU6050** | VCC | 3V3 | Nguồn cấp |
| | GND | GND | Mát chung |
| | SDA | GPIO 21 | Giao tiếp I2C Data[cite: 3] |
| | SCL | GPIO 22 | Giao tiếp I2C Clock[cite: 3] |
| **Đèn LED** | Anode (+) | GPIO 12 | Qua điện trở hạn dòng 220Ω[cite: 1, 3] |
| | Cathode (-) | GND | Mát[cite: 3] |
| **Còi Buzzer** | Chân dương (+) | GPIO 14 | Tín hiệu cảnh báo[cite: 1, 3] |
| | Chân âm (-) | GND | Mát[cite: 3] |

---

## 4. Chức năng hệ thống

* **Phát hiện té ngã tự động (Auto Mode):** 
  * Đọc liên tục giá trị gia tốc tổng ($a_{acc}$) và góc nghiêng (Roll, Pitch) từ cảm biến MPU6050[cite: 1].
  * Thuật toán nhận diện khi gia tốc vượt ngưỡng va chạm kết hợp góc nghiêng lệch quá giới hạn trong khoảng thời gian cho phép[cite: 1].
* **Cảnh báo tại chỗ:** 
  * Tự động bật sáng đèn LED và hú còi Buzzer khi xảy ra sự cố té ngã[cite: 1].
* **Truyền thông dữ liệu IoT (MQTT):** 
  * Gửi gói tin JSON chứa thông số gia tốc, góc nghiêng và trạng thái thiết bị lên MQTT Broker định kỳ[cite: 1].
  * Nhận các lệnh điều khiển từ xa qua các Topic MQTT[cite: 1].
* **Giao diện Web Dashboard:** 
  * Hiển thị các giá trị dạng số (Digital Display) và trực quan hóa bằng **2 biểu đồ thời gian thực** (Biểu đồ gia tốc và Biểu đồ góc nghiêng)[cite: 1].
  * Cung cấp các nút bấm (`button`) để chuyển đổi chế độ **AUTO / MANUAL**, bật/tắt thủ công LED/Buzzer và nút **RESET** trạng thái cảnh báo[cite: 1].

---

## 5. Kiến trúc FreeRTOS (Phân chia Task)
* **`TaskReadData` (Core 1 - Ưu tiên 2):** Đọc dữ liệu từ MPU6050, tính toán gia tốc tổng, góc Roll/Pitch, chạy thuật toán phát hiện té ngã và đưa dữ liệu vào hàng đợi (`Queue`)[cite: 1].
* **`TaskAlert` (Core 1 - Ưu tiên 2):** Xử lý sự kiện cảnh báo từ Queue để điều khiển trạng thái bật/tắt của LED (GPIO 12) và Buzzer (GPIO 14) theo chế độ tự động hoặc thủ công[cite: 1].
* **`TaskTransmitData` (Core 0 - Ưu tiên 1):** Quản lý kết nối Wi-Fi, duy trì vòng lặp kết nối MQTT Broker, publish dữ liệu telemetry và subscribe nhận lệnh điều khiển từ Web[cite: 1].
