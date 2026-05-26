#include <LiquidCrystal.h>

// LCD1602: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

void setup() {
  // LCDを16文字×2行で初期化する
  lcd.begin(16, 2);

  // 表示をクリアする
  lcd.clear();

  // 1行目に表示
  lcd.setCursor(0, 0);
  lcd.print("LCD Test OK");

  // 2行目に表示
  lcd.setCursor(0, 1);
  lcd.print("Back Sonar");
}

void loop() {
  // 今回はLCD表示テストのみなので、繰り返し処理はなし
}
