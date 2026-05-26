## 3. 重要ロジックの詳細設計

### 3-1. チャタリング防止（デバウンス処理）


【考え方】
今回の装置ではボタンを使用しないため、チャタリング防止処理は実装しない。

【処理の流れ】
なし

【必要な変数（Section 1 に追加済みか確認）】
なし

接下来是重点的 3-2 millis() を使ったタイマー管理。你前面已经决定基本不用 delay()，而是用 millis() 来避免蜂鸣器响的时候距离测量停止，所以这里要写清楚。

### 3-2. millis() を使ったタイマー管理


【考え方】
距離測定、LCD表示更新、ブザー制御をそれぞれ一定周期で実行する。
delay() を多用すると、その間は距離測定や表示更新が止まってしまうため、
millis() を使って処理を止めずに制御する。

【処理の流れ】

loop() の最初で現在時刻 now = millis() を取得する。
now - lastMillisSensor >= SENSOR_INTERVAL の場合、
measureDistance() を実行し、lastMillisSensor = now に更新する。
距離測定後、judgeState() で現在の状態を判定する。
now - lastMillisLCD >= LCD_INTERVAL の場合、
updateLCD() を実行し、lastMillisLCD = now に更新する。
updateBuzzer() の中で現在状態を確認する。
Caution または Danger の場合、
now - lastMillisBuzzer が設定した間隔以上か確認する。
条件を満たした場合、ブザーのON/OFFを切り替え、
lastMillisBuzzer = now に更新する。

【自分のシステムで millis() を使う処理】

距離測定：SENSOR_INTERVAL = 300ms ごと
LCD表示更新：LCD_INTERVAL = 300ms ごと
Caution時のブザー制御：CAUTION_INTERVAL = 500ms ごと
Danger時のブザー制御：DANGER_INTERVAL = 200ms ごと

最后是 3-3 その他の重要ロジック。你这个作品里最重要的逻辑是“距离 → 状态 → LCD/ブザー”的关系，所以这里写距离判定逻辑最合适。

### 3-3. その他の重要ロジック（距離判定と出力制御）


【処理の流れ】

measureDistance() で障害物までの距離を取得する。
距離が -1、0cm以下、または400cmを超える場合はエラー状態とする。
距離が20cm未満の場合は危険状態とする。
距離が20cm以上50cm未満の場合は注意状態とする。
距離が50cm以上の場合は安全状態とする。
判定した状態に応じて、LCD表示とブザー動作を切り替える。

【入力値と出力値の関係】

入力値（距離）	状態	LCD表示	ブザー
-1、0cm以下、400cm超	エラー状態	Out of range / No object	鳴らさない
50cm以上	安全状態	Dist: XXcm / Safe	鳴らさない
20cm以上50cm未満	注意状態	Dist: XXcm / Caution	ゆっくり鳴らす
20cm未満	危険状態	Dist: XXcm / Danger	短い間隔で鳴らす
