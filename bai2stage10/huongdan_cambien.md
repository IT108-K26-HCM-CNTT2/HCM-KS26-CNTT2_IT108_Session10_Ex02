# Hướng dẫn cài đặt hệ thống cảm biến độ ẩm trang trại Lan

Chào thầy, đây là tài liệu em ghi chép lại cách setup bộ cảm biến độ ẩm để tự động bật máy bơm tưới lan nhé.

## 1. Chuẩn bị môi trường phần mềm
Đầu tiên, em mở terminal lên và cài đặt các thư viện cần thiết bằng lệnh sau:

\`\`\`bash
# Cập nhật hệ thống trước khi cài
sudo apt update && sudo apt upgrade -y
# Cài đặt thư viện điều khiển chân GPIO và đọc cảm biến
pip install RPi.GPIO Adafruit-DHT
\`\`\`

Trong quá trình viết code, lưu ý các biến cấu hình quan trọng như chân tín hiệu `pin_sensor = 4` và ngưỡng độ ẩm kích hoạt máy bơm `humidity_threshold = 60`. Tránh dùng nhầm ký tự \# trong chuỗi cấu hình nếu không muốn bị lỗi comment.

## 2. Mã nguồn điều khiển máy bơm (Python)
Dưới đây là đoạn code mẫu dùng để đọc giá trị từ cảm biến và điều khiển relay máy bơm:

\`\`\`python
import time
import Adafruit_DHT
import RPi.GPIO as GPIO

# Cấu hình phần cứng
SENSOR = Adafruit_DHT.DHT22
DHT_PIN = 4        # Chân dữ liệu kết nối cảm biến
PUMP_PIN = 17      # Chân kích hoạt Relay máy bơm nước

GPIO.setmode(GPIO.BCM)
GPIO.setup(PUMP_PIN, GPIO.OUT)

def kiem_tra_do_am():
    # Đọc dữ liệu độ ẩm và nhiệt độ từ cảm biến
    humidity, temperature = Adafruit_DHT.read_retry(SENSOR, DHT_PIN)
    
    if humidity is not None:
        print(f"Độ ẩm hiện tại: {humidity:.1f}% | Nhiệt độ: {temperature:.1f}°C")
        if humidity < 60.0:
            print("Độ ẩm thấp -> Bật máy bơm tưới lan!")
            GPIO.output(PUMP_PIN, GPIO.HIGH)
        else:
            print("Độ ẩm ổn định -> Tắt máy bơm.")
            GPIO.output(PUMP_PIN, GPIO.LOW)
    else:
        print("Lỗi: Không đọc được dữ liệu từ cảm biến!")

try:
    while True:
        kiem_tra_do_am()
        time.sleep(10)
except KeyboardInterrupt:
    GPIO.cleanup()
    print("Đã thoát chương trình và dọn dẹp GPIO.")
\`\`\`

## 3. Bảng so sánh loại cảm biến (Cũ vs Mới)

| Tiêu chí so sánh | Cảm biến cũ (DHT11) | Cảm biến mới (DHT22 / AM2302) |
| :--- | :--- | :--- |
| **Độ chính xác độ ẩm** | Thấp ($\pm$ 5%) | Cao ($\pm$ 2-5%) |
| **Độ chính xác nhiệt độ** | $\pm$ 2°C | $\pm$ 0.5°C |
| **Tần suất đọc dữ liệu** | 1 lần / giây | 0.5 lần / giây (2 giây/lần) |
| **Độ bền ngoài trời** | Kém, dễ hỏng khi ẩm cao | Tốt hơn, có lớp vỏ bảo vệ |

## 4. Checklist kiểm tra thiết bị thực tế (Task List)
- [x] Kiểm tra nguồn điện ổn định 5V cho Raspberry Pi và cảm biến.
- [x] Lắp đặt cảm biến ở vị trí thoáng gió, tránh ánh nắng trực tiếp chiếu vào.
- [x] Kiểm tra kết nối dây tín hiệu GPIO chân số `4` và chân relay `17`.
- [ ] Chạy thử chương trình Python và kiểm tra độ nhạy của máy bơm.
- [ ] Bàn giao hệ thống cho chủ trang trại theo dõi qua màn hình giám sát.
