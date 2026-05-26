## 4. デバッグ出力計画（任意）

| No | 確認したい内容 | 挿入する関数 | Serial.println の内容例 |
|:---|:---|:---|:---|
| 1 | 距離が正しく測定できているか | `measureDistance()` | `Serial.println(distanceCm);` |
| 2 | 状態判定が正しいか | `judgeState()` | `Serial.println(currentState);` |
| 3 | LCD表示が更新されているか | `updateLCD()` | `Serial.println("LCD update");` |
| 4 | ブザー制御が実行されているか | `updateBuzzer()` | `Serial.println("Buzzer update");` |
