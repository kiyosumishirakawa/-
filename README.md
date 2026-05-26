// パッシブブザーの単体テスト
const int PIN_BUZZER = 3;

void setup() {
  pinMode(PIN_BUZZER, OUTPUT);
}

void loop() {
  // 800Hzの音を鳴らす
  tone(PIN_BUZZER, 800);
  delay(500);

  // 音を止める
  noTone(PIN_BUZZER);
  delay(500);
}
