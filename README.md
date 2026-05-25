【ピン定義】（basic_design.md 3-1 から転記）
PIN_BUZZER = 3 // パッシブブザー
PIN_TRIG = 5 // 超音波センサー Trig
PIN_ECHO = 6 // 超音波センサー Echo

LCD_RS = 7 // LCD1602 RS
LCD_E = 8 // LCD1602 E
LCD_D4 = 9 // LCD1602 D4
LCD_D5 = 10 // LCD1602 D5
LCD_D6 = 11 // LCD1602 D6
LCD_D7 = 12 // LCD1602 D7

【状態管理】（basic_design.md 1-2 の状態名から転記）
currentState : int = 0
// 0: 初期化
// 1: 距離測定中
// 2: 安全状態
// 3: 注意状態
// 4: 危険状態
// 5: エラー状態

【タイマー（millis()用）】（basic_design.md 2-3 から転記）
lastMillisSensor : unsigned long = 0
lastMillisLCD : unsigned long = 0
lastMillisBuzzer : unsigned long = 0

SENSOR_INTERVAL : const unsigned long = 300
LCD_INTERVAL : const unsigned long = 300
CAUTION_INTERVAL: const unsigned long = 500
DANGER_INTERVAL : const unsigned long = 200

【センサー・入力値】（basic_design.md 2-1 から転記）
duration : long = 0 // Echo信号の長さ
distanceCm : float = 0.0 // 障害物までの距離（cm）

【距離判定用の定数】
CAUTION_DISTANCE : const int = 50 // 50cm未満で注意状態
DANGER_DISTANCE : const int = 20 // 20cm未満で危険状態
MAX_DISTANCE : const int = 400 // 400cm超は測定範囲外

【ブザー制御用】
buzzerOn : bool = false // ブザーのON/OFF状態を管理する

【その他のフラグ・カウンター】
isDistanceValid : bool = false // 距離が正常に測定できたかを表す
