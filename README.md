## 2. 各関数の詳細設計

---

### `setup()` — 初期化処理


【処理の流れ】

LCD1602を初期化する
lcd.begin(16, 2) を実行する
起動メッセージを表示する
ピンモードを設定する
PIN_TRIG → OUTPUT
PIN_ECHO → INPUT
PIN_BUZZER → OUTPUT
ブザーを停止状態にする
noTone(PIN_BUZZER) を実行する
デバッグ用にSerial通信を開始する
Serial.begin(9600) を実行する
初期状態を設定する
currentState = 1（距離測定中）にする

---

### `loop()` — メインループ


【処理の流れ】

＜毎ループ実行すること＞

現在時刻を取得する
now = millis()
一定周期ごとに距離を測定する
measureDistance() を呼び出す
距離が正常に取得できたか確認する
距離に応じて状態を判定する
judgeState(distanceCm) を呼び出す
currentState を更新する
LCD表示を更新する
updateLCD(distanceCm, currentState) を呼び出す
ブザーの状態を更新する
updateBuzzer(currentState) を呼び出す

＜currentState が 1（距離測定中）のとき＞

超音波センサーで距離を測定する
測定結果をもとに次の状態へ遷移する

＜currentState が 2（安全状態）のとき＞

LCDに距離とSafeを表示する
ブザーは鳴らさない

＜currentState が 3（注意状態）のとき＞

LCDに距離とCautionを表示する
ブザーをゆっくり鳴らす

＜currentState が 4（危険状態）のとき＞

LCDに距離とDangerを表示する
ブザーを短い間隔で鳴らす

＜currentState が 5（エラー状態）のとき＞

LCDにOut of rangeなどのエラー表示を出す
ブザーは鳴らさない

---

### `measureDistance()` — 超音波センサーで距離を測定する

**basic_design.md 2-2 との対応：**  
超音波センサーで障害物までの距離を測定する。

**引数：** なし

**戻り値：** `float`  
障害物までの距離をcm単位で返す。測定できない場合は `-1` を返す。


【処理の流れ】

TrigピンをLOWにして、センサーを安定させる
Trigピンを10マイクロ秒だけHIGHにする
TrigピンをLOWに戻す
EchoピンがHIGHになっている時間をpulseIn()で取得する
Echo信号が取得できない場合は -1 を返す
取得した時間から距離を計算する
距離 = duration × 0.0343 ÷ 2
計算した距離を返す

【エラー・異常ケース】

Echo信号が返ってこない場合：
-1 を返す
距離が0cm以下、または400cmを超える場合：
測定範囲外として扱う

---

### `judgeState()` — 距離に応じて状態を判定する

**basic_design.md 2-2 との対応：**  
距離に応じてSafe、Caution、Danger、Errorを判定する。

**引数：** `distanceCm`（float）: 障害物までの距離

**戻り値：** `int`  
現在の状態を表す数値を返す。


【処理の流れ】

distanceCm が0以下、または400cmを超えているか確認する
異常値の場合は 5（エラー状態）を返す
distanceCm が20cm未満の場合は 4（危険状態）を返す
distanceCm が20cm以上50cm未満の場合は 3（注意状態）を返す
distanceCm が50cm以上の場合は 2（安全状態）を返す

【エラー・異常ケース】

distanceCm が -1 の場合：
センサーが測定できなかったと判断し、5（エラー状態）を返す

---

### `updateLCD()` — LCD1602に距離と状態を表示する

**basic_design.md 2-2 との対応：**  
距離と状態をLCD1602に表示する。

**引数：**  
`distanceCm`（float）: 障害物までの距離  
`state`（int）: 現在の状態

**戻り値：** なし


【処理の流れ】

LCDの表示を更新するタイミングか確認する
LCD画面をクリアする
state がエラー状態の場合
1行目に "Out of range" を表示する
2行目に "No object" を表示する
state が正常状態の場合
1行目に "Dist: XXcm" を表示する
2行目に状態名を表示する
安全状態：Safe
注意状態：Caution
危険状態：Danger

【エラー・異常ケース】

距離が測定できない場合：
距離の数値は表示せず、エラーメッセージを表示する
前回の文字がLCDに残る場合：
lcd.clear() または空白文字で表示を消す

---

### `updateBuzzer()` — 状態に応じてブザーを制御する

**basic_design.md 2-2 との対応：**  
状態に応じてブザーを鳴らす、または停止する。

**引数：** `state`（int）: 現在の状態

**戻り値：** なし


【処理の流れ】

state を確認する
安全状態の場合
noTone(PIN_BUZZER) を実行し、ブザーを停止する
注意状態の場合
cautionBeep() を呼び出す
危険状態の場合
dangerBeep() を呼び出す
エラー状態の場合
noTone(PIN_BUZZER) を実行し、ブザーを停止する

【エラー・異常ケース】

センサー異常時：
誤警告を防ぐため、ブザーを鳴らさない

---

### `cautionBeep()` — 注意状態の警告音を鳴らす

**basic_design.md 2-2 との対応：**  
Caution時に低めの音をゆっくり鳴らす。

**引数：** なし

**戻り値：** なし


【処理の流れ】

millis()で現在時刻を取得する
前回ブザー状態を切り替えた時刻から一定時間が経過したか確認する
一定時間が経過していれば、ブザーのON/OFFを切り替える
ブザーONの場合は tone(PIN_BUZZER, 800) を実行する
ブザーOFFの場合は noTone(PIN_BUZZER) を実行する
lastMillisBuzzer を更新する

【エラー・異常ケース】

状態がCaution以外に変わった場合：
noTone(PIN_BUZZER) で停止する

---

### `dangerBeep()` — 危険状態の警告音を鳴らす

**basic_design.md 2-2 との対応：**  
Danger時に高めの音を短い間隔で鳴らす。

**引数：** なし

**戻り値：** なし


【処理の流れ】

millis()で現在時刻を取得する
前回ブザー状態を切り替えた時刻から一定時間が経過したか確認する
一定時間が経過していれば、ブザーのON/OFFを切り替える
ブザーONの場合は tone(PIN_BUZZER, 1200) を実行する
ブザーOFFの場合は noTone(PIN_BUZZER) を実行する
lastMillisBuzzer を更新する

【エラー・異常ケース】

状態がDanger以外に変わった場合：
noTone(PIN_BUZZER) で停止する
