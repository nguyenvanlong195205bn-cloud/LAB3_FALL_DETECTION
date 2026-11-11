# LAB3_FALL_DETECTIO# LAB #3: XÂY DỰNG HỆ THỐNG PHÁT HIỆN TÉ NGÃ SỬ DỤNG ESP32, FREERTOS VÀ MQTT

**Họ tên:** Nguyễn Văn Long  
**Lớp:** Kỹ sư Kỹ thuật Viễn thông  

## Mô tả
ESP32 kết hợp cảm biến MPU6050 để thu thập dữ liệu gia tốc và góc nghiêng, phát hiện té ngã. Khi phát hiện té ngã, hệ thống tự động bật LED và buzzer cảnh báo. Dữ liệu được gửi lên MQTT Broker và hiển thị trên Website, hỗ trợ giám sát và điều khiển từ xa.

## Kiến trúc FreeRTOS

| Task | Core | Nhiệm vụ |
| :--- | :---: | :--- |
| `TaskTransmitData` | 0 | Kết nối WiFi/MQTT, gửi dữ liệu và nhận lệnh từ Web |N
