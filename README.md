#include <LiquidCrystal.h>

// LCD1602: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

// 超音波センサー
const int PIN_TRIG = 5;
const int PIN_ECHO = 6;

long duration;
float distanceCm;

void setup() {
  // LCD初期化
  lcd.begin(16, 2);
  lcd.clear();

  // ピン設定
  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);

  // デバッグ用
  Serial.begin(9600);

  lcd.setCursor(0, 0);
  lcd.print("Distance Test");
  lcd.setCursor(0, 1);
  lcd.print("Starting...");
  delay(1000);
  lcd.clear();
}

void loop() {
  distanceCm = measureDistance();

  lcd.clear();

  if (distanceCm <= 0 || distanceCm > 400) {
    lcd.setCursor(0, 0);
    lcd.print("Out of range");
    lcd.setCursor(0, 1);
    lcd.print("No object");

    Serial.println("Out of range");
  } else {
    lcd.setCursor(0, 0);
    lcd.print("Dist:");
    lcd.print((int)distanceCm);
    lcd.print("cm");

    lcd.setCursor(0, 1);
    lcd.print("Sensor OK");

    Serial.print("Distance: ");
    Serial.print(distanceCm);
    Serial.println(" cm");
  }

  delay(300);
}

// 距離を測定する関数
float measureDistance() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  duration = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duration == 0) {
    return -1;
  }

  return duration * 0.0343 / 2;
}
