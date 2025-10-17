#include <Wire.h>
#include "MAX30105.h"
#include "heartRate.h"

MAX30105 particleSensor;

// Biến toàn cục
long irValue;
float beatsPerMinute;
int beatAvg;

void setup()
{
  Serial.begin(115200);
  Serial.println("Khởi động hệ thống đo nhịp tim & SpO2...");

  // Khởi tạo I2C và cảm biến
  if (!particleSensor.begin(Wire, I2C_SPEED_STANDARD))
  {
    Serial.println("Không tìm thấy cảm biến MAX30102!");
    while (1);
  }

  // Cấu hình cảm biến
  particleSensor.setup(); // thiết lập mặc định: 69 mẫu/s, LED 50mA
  particleSensor.setPulseAmplitudeRed(0x0A); // LED đỏ
  particleSensor.setPulseAmplitudeIR(0x0A);  // LED hồng ngoại
  particleSensor.setPulseAmplitudeGreen(0);  // Tắt LED xanh
}

void loop()
{
  // Đọc giá trị IR
  irValue = particleSensor.getIR();

  if (checkForBeat(irValue) == true)
  {
    // Tính thời gian giữa 2 nhịp
    static long lastBeat = 0;
    long delta = millis() - lastBeat;
    lastBeat = millis();

    beatsPerMinute = 60 / (delta / 1000.0);

    // Giới hạn hợp lý (40-180 BPM)
    if (beatsPerMinute < 180 && beatsPerMinute > 40)
    {
      beatAvg = (beatAvg * 3 + beatsPerMinute) / 4;
    }
  }

  Serial.print("IR=");
  Serial.print(irValue);
  Serial.print(" BPM=");
  Serial.print(beatsPerMinute);
  Serial.print(" Avg BPM=");
  Serial.println(beatAvg);

  delay(100);
}
