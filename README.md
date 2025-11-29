# ESP32_4x-BTS7960

# H-Bridge Motor Controller with ESP32

## Tổng quan
Mạch điều khiển động cơ DC công suất cao sử dụng 4 module BTS7960 (43A H-Bridge) được điều khiển bởi ESP32, với buffer 74HC244D để tăng cường tín hiệu điều khiển. 

## Sơ đồ khối hệ thống

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  ESP32   │────▶│ 74HC244D │────▶│ BTS7960  │────▶│  Motor   │
│          │     │  Buffer  │     │ H-Bridge │     │  (x4)    │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                                                    │
     │           ┌──────────┐                            │
     └──────────▶│ Encoder/ │◀───────────────────────────┘
                 │   Hall   │
                 └──────────┘
```

## Các khối chức năng chính

### 1.  POWER (Mạch nguồn)
- **Nguồn logic**: 5V cho BTS7960 và 74HC244D
- **Nguồn ESP32**: 3.3V 
- **Nguồn động cơ**: 6-27VDC (tùy BTS7960)
- Bao gồm mạch lọc nguồn và bảo vệ

### 2. H-BRIDGE (4x BTS7960)
**Thông số BTS7960:**
- Điện áp hoạt động: 5. 5V - 27V
- Dòng điện liên tục: 43A mỗi kênh
- PWM frequency: lên đến 25kHz
- Bảo vệ: Quá nhiệt, quá dòng, ngắn mạch
- Điều khiển chiều quay: L_EN, R_EN
- Điều khiển tốc độ: L_PWM, R_PWM

**Mỗi module BTS7960 có 8 chân:**
```
BTS7960 Module:
├─ RPWM  : PWM chiều thuận (Right PWM)
├─ LPWM  : PWM chiều nghịch (Left PWM)  
├─ R_EN  : Enable chiều thuận
├─ L_EN  : Enable chiều nghịch
├─ R_IS  : Current sense chiều thuận
├─ L_IS  : Current sense chiều nghịch
├─ VCC   : 5V logic
└─ GND   : Ground
```

**Kết nối động cơ:**
- M+ và M-: Kết nối 2 cực động cơ DC
- B+ và B-: Nguồn động cơ (6-27V)

### 3. LOGIC (74HC244D Buffer)
**Chức năng:**
- Buffer tín hiệu từ ESP32 (3.3V) lên 5V cho BTS7960
- Tăng khả năng đẩy dòng cho tín hiệu điều khiển
- Cách ly bảo vệ ESP32

**Thông số 74HC244D:**
- 8 kênh buffer 3-state
- Input: 3.3V từ ESP32
- Output: 5V ra BTS7960
- Dòng output: 35mA mỗi kênh
- Tốc độ cao, phù hợp PWM

**Sơ đồ kết nối buffer:**
```
ESP32 (3.3V)          74HC244D          BTS7960 (5V)
GPIO_PWM1    ────▶    1A ──▶ 1Y   ────▶  RPWM_1
GPIO_DIR1    ────▶    2A ──▶ 2Y   ────▶  LPWM_1
GPIO_PWM2    ────▶    3A ──▶ 3Y   ────▶  RPWM_2
GPIO_DIR2    ────▶    4A ──▶ 4Y   ────▶  LPWM_2
... 
```  

### 4. ESP32 (ESP32-WROOM-32)
**Chân GPIO sử dụng:**

```cpp
// Motor Control (thông qua 74HC244D)
#define MOTOR1_RPWM  GPIO_32
#define MOTOR1_LPWM  GPIO_23
#define MOTOR2_RPWM  GPIO_13
#define MOTOR2_LPWM  GPIO_05
#define MOTOR3_RPWM  GPIO_12
#define MOTOR3_LPWM  GPIO_26
#define MOTOR4_RPWM  GPIO_15
#define MOTOR4_LPWM  GPIO_04

// ANALOG Sensor
#define A0           GPIO_14
#define A1           GPIO_34
#define A2           GPIO_25
#define A3           GPIO_33
#define A4           GPIO_27
#define A5           GPIO_35
#define A6           GPIO_36
#define A7           GPIO_39
// Enable pins (có thể dùng chung hoặc riêng)
đã kéo lên mức HIGH (luôn enable)

// Communication
#define SERIAL_TX    GPIO_17
#define SERIAL_RX    GPIO_16
#define I2C_SDA      GPIO_21 ( CHỈ DÙNG NẾU KHÔNG DÙNG SERVO 3 VÀ 4)
#define I2C_SCL      GPIO_22 ( CHỈ DÙNG NẾU KHÔNG DÙNG SERVO 3 VÀ 4)
#dedine Servo_1      GPIO_18
#dedine Servo_2      GPIO_19
#dedine Servo_3      GPIO_21
#dedine Servo_4      GPIO_22

```

### 5. EX PIN (Đầu nối mở rộng)

**SERIAL Connector:**
- TX, RX: UART communication
- GND, 3.3V
- Dùng cho debug hoặc giao tiếp với thiết bị ngoài

**I2C Connector:**
- SDA, SCL: I2C bus
- GND, 3. 3V
- Kết nối cảm biến, màn hình, v.v.

**ENCODER Connector:**
- Encoder A, B phase cho mỗi động cơ
- Đếm xung để đo tốc độ/vị trí
- Pull-up resistor tích hợp

**HALL Connector:**
- Hall sensor signals
- GND, 3.3V/5V
- Dùng cho BLDC hoặc cảm biến vị trí

### 6. LED (Chỉ thị trạng thái)
- **Power LED**: Nguồn hoạt động
- **Status LED**: Trạng thái ESP32
- **Motor LED**: Hoạt động của từng kênh động cơ

## Thông số kỹ thuật

### Nguồn điện
- **Nguồn logic**: 5V/2A (cho BTS7960, 74HC244D)
- **Nguồn ESP32**: 3.3V/500mA
- **Nguồn động cơ**: 6-27V/43A mỗi kênh (tối đa 172A cho 4 kênh)
- **Bảo vệ**: Diode, capacitor lọc, fuse

### Điều khiển động cơ
- **Số kênh**: 4 kênh độc lập
- **Dòng max**: 43A liên tục/kênh
- **PWM frequency**: 1-25kHz (khuyến nghị 10-20kHz)
- **Điều khiển**: 
  - Tốc độ: PWM duty cycle 0-100%
  - Chiều: RPWM/LPWM
  - Phanh: Cả 2 PWM = HIGH
  - Dừng: Cả 2 PWM = LOW

### Giao tiếp
- **UART**: 115200 baud (configurable)
- **I2C**: 100kHz/400kHz
- **PWM input**: Encoder, Hall sensor

## Code mẫu Arduino

```cpp
// Cấu hình PWM cho ESP32
const int pwmFreq = 15000;  // 15kHz
const int pwmResolution = 8; // 8-bit (0-255)

// Motor 1
const int M1_RPWM_PIN = 32;
const int M1_LPWM_PIN = 13;
void setup() {
  // Cấu hình PWM channels
  ledcSetup(0, pwmFreq, pwmResolution); // Channel 0 for M1_RPWM
  ledcSetup(1, pwmFreq, pwmResolution); // Channel 1 for M1_LPWM
  
  ledcAttachPin(M1_RPWM_PIN, 0);
  ledcAttachPin(M1_LPWM_PIN, 1);
}

void motorControl(int speed) {
  // speed: -255 đến +255
  if (speed > 0) {
    // Quay thuận
    ledcWrite(0, speed);      // RPWM
    ledcWrite(1, 0);          // LPWM
  } else if (speed < 0) {
    // Quay nghịch
    ledcWrite(0, 0);          // RPWM
    ledcWrite(1, -speed);     // LPWM
  } else {
    // Dừng
    ledcWrite(0, 0);
    ledcWrite(1, 0);
  }
}

void motorBrake() {
  // Phanh nhanh
  ledcWrite(0, 255);
  ledcWrite(1, 255);
}

void loop() {
  // Quay thuận tốc độ 50%
  motorControl(128);
  delay(2000);
  
  // Phanh
  motorBrake();
  delay(500);
  
  // Quay nghịch tốc độ 75%
  motorControl(-192);
  delay(2000);
  
  // Dừng
  motorControl(0);
  delay(1000);
}
```

## Sơ đồ kết nối chi tiết

### BTS7960 Module (mỗi kênh)
```
ESP32          74HC244        BTS7960        Motor
             ┌─────────┐    ┌─────────┐
GPIO_PWM ───▶│ Buffer  │───▶│  RPWM   │
GPIO_DIR ───▶│         │───▶│  LPWM   │    ┌───────┐
             └─────────┘    │  R_EN   │───▶│   M+  │
3.3V ──────────────────────▶│  L_EN   │    │       │
                            │  VCC    │◀───│  DC   │
GND  ──────────────────────▶│  GND    │    │ Motor │
                            │  M+     │───▶│   M-  │
Power+ ────────────────────▶│  B+     │    └───────┘
Power- ────────────────────▶│  B-     │
                            └─────────┘
```

### Tính toán nguồn
```
Mỗi BTS7960:
- Logic: ~100mA @ 5V
- Motor: 0-43A @ 6-27V

Tổng hệ thống (4 motors):
- Logic: ~500mA @ 5V
- ESP32: ~500mA @ 3.3V
- Motors: 0-172A @ 6-27V (tùy tải)

Khuyến nghị:
- Nguồn logic: 5V/2A
- Nguồn motor: 12-24V/60A+ (với fuse bảo vệ)
```

## Ứng dụng
- **Robot di động**: 4 bánh mecanum/omni wheel
- **AGV**: Automated Guided Vehicle
- **RC Car**: Xe điều khiển từ xa công suất cao
- **CNC**: Máy CNC mini với động cơ DC

## Lưu ý an toàn

### ⚠️ Quan trọng
1. **Nguồn động cơ**: 
   - Sử dụng nguồn riêng cho động cơ (không dùng chung với logic)
   - Nối GND chung giữa nguồn logic và nguồn động cơ
   - Dùng capacitor 1000-4700µF gần BTS7960

2. **Tản nhiệt**:
   - BTS7960 cần tản nhiệt khi dòng > 20A
   - Dùng quạt hoặc heatsink phù hợp
   - Nhiệt độ max: 150°C (có bảo vệ)

3. **PWM**:
   - Tần số PWM: 10-20kHz (giảm tiếng kêu)
   - Không đổi chiều đột ngột khi tốc độ cao
   - Dùng ramping để tăng/giảm tốc độ mượt

4. **Encoder**:
   - Pull-up 10kΩ cho encoder pins
   - Dùng interrupt để đếm xung chính xác
   - Debouncing nếu cần

5. **Bảo vệ**:
   - Fuse trên nguồn động cơ
   - Diode flyback cho động cơ có cảm (BTS7960 đã tích hợp)
   - Tránh ngắn mạch M+/M-

## Troubleshooting

| Vấn đề | Nguyên nhân | Giải pháp |
|--------|-------------|-----------|
| Motor không quay | Enable chưa được kích hoạt | Kiểm tra R_EN, L_EN = HIGH |
| Motor quay yếu | PWM quá thấp hoặc nguồn yếu | Tăng duty cycle, kiểm tra nguồn |
| Motor nóng | Dòng quá cao, tần số PWM thấp | Giảm tải, tăng tần số PWM |
| BTS7960 nóng | Dòng cao, tản nhiệt kém | Thêm heatsink/quạt |
| Motor giật | PWM frequency thấp | Tăng lên 15-20kHz |
| ESP32 reset | Nhiễu từ motor | Tách nguồn, thêm capacitor lọc |

## Tài liệu tham khảo
- [BTS7960 Datasheet](https://www.infineon.com/dgdl/Infineon-BTS7960-DS-v01_00-EN.pdf?fileId=db3a30433fa9412f013fbe32289b7c17)
- [74HC244 Datasheet](https://www. ti.com/lit/ds/symlink/sn74hc244.pdf)
- [ESP32 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)

## Tác giả
- Designer: Huybuideeptry
- Tool: EasyEDA
- Version: 1.0

---
**Made with ❤️ and lack of ☕ for high-power motor control**
