#include <LiquidCrystal.h>

// LCD1602: RS, E, D4, D5, D6, D7
// 这里要和你现在LCD接线一致
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

// Ultrasonic Sensor
const int trigPin = 5;
const int echoPin = 6;

// Passive Buzzer
const int buzzerPin = 3;

long duration;
float distanceCm;

void setup() {
  lcd.begin(16, 2);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(buzzerPin, OUTPUT);

  noTone(buzzerPin);

  lcd.setCursor(0, 0);
  lcd.print("Back Sonar");
  lcd.setCursor(0, 1);
  lcd.print("Ready");
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
    noTone(buzzerPin);
    delay(300);
    return;
  }

  lcd.setCursor(0, 0);
  lcd.print("Dist:");
  lcd.print((int)distanceCm);
  lcd.print("cm");

  lcd.setCursor(0, 1);

  if (distanceCm < 20) {
    lcd.print("State:Danger");
    dangerBeep();
  } 
  else if (distanceCm < 50) {
    lcd.print("State:Caution");
    cautionBeep();
  } 
  else {
    lcd.print("State:Safe");
    noTone(buzzerPin);
    delay(300);
  }
}

float measureDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH, 30000);

  if (duration == 0) {
    return -1;
  }

  return duration * 0.0343 / 2;
}

// 危険：短い間隔で高めの音
void dangerBeep() {
  tone(buzzerPin, 1200);
  delay(100);
  noTone(buzzerPin);
  delay(100);
}

// 注意：ゆっくり低めの音
void cautionBeep() {
  tone(buzzerPin, 800);
  delay(100);
  noTone(buzzerPin);
  delay(400);
}
