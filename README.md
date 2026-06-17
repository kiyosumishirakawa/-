1. 先追加メロディ用变量

放在这段后面比较合适：

bool longPressHandled = false;

下面追加：

// --- メロディ用 ---
// ド・ミ・ソ・高いド・ソ・高いド
const int MELODY_NOTES[] = {
  523, 659, 784, 1047, 784, 1047
};

const int MELODY_DURATIONS[] = {
  160, 160, 160, 350, 160, 600
};

const int MELODY_LENGTH = sizeof(MELODY_NOTES) / sizeof(MELODY_NOTES[0]);

int melodyIndex = 0;
unsigned long melodyPreviousMillis = 0;
bool melodyPlaying = false;
bool melodyGap = false;
const int MELODY_GAP = 40;  // 音と音の間の短い休み

音高大概是：

523  = ド
659  = ミ
784  = ソ
1047 = 高いド
2. 倒计时到 0 的地方改一下

你现在这里是：

if (remainingSeconds <= 0) {
  remainingSeconds = 0;
  currentState = FINISHED;
  Serial.println("Finished!");
}

改成：

if (remainingSeconds <= 0) {
  remainingSeconds = 0;
  currentState = FINISHED;
  startMelody();
  Serial.println("Finished!");
}

也就是进入 FINISHED 的瞬间启动メロディ。

3. 把 FINISHED 里的原蜂鸣器逻辑换掉

你现在的 case FINISHED: 是这一大段：

case FINISHED:
  // 終了時のブザー
  {
      int cycle = currentMillis % 1000;

      if (cycle < 400) {
      if ((cycle / 50) % 2 == 0) {
          digitalWrite(PIN_BUZZ, HIGH);
      } else {
          digitalWrite(PIN_BUZZ, LOW);
      }
      } else {
          digitalWrite(PIN_BUZZ, LOW);
      }
  }
  break;

整段替换成：

case FINISHED:
  updateMelody();
  break;

这样结束后就会持续播放メロディ。

4. resetTimer() 里停止メロディ

你现在 resetTimer() 里有：

digitalWrite(PIN_BUZZ, LOW);

建议改成：

stopMelody();

完整这一段变成：

void resetTimer() {
    currentState = IDLE;
    remainingSeconds = settingSeconds;
    previousMillis = 0;
    pausedElapsed = 0;

    stopMelody();
    updateDisplayBuffer();

    Serial.println("Reset to IDLE!");
}

这样长按重置或结束后短按重置时，音乐会停止。

5. IDLE / PAUSED 里的停止声音也建议改成 noTone()

你现在有：

digitalWrite(PIN_BUZZ, LOW);

对于 passive buzzer，更推荐改成：

noTone(PIN_BUZZ);

比如：

case IDLE:
    noTone(PIN_BUZZ);
    updateTimeByPot();
    break;

以及：

case PAUSED:
    noTone(PIN_BUZZ);
    break;
6. RUNNING 中的短い「ピッ」音也建议改成 tone()

你现在是：

if (currentMillis - previousMillis < 50) {
    digitalWrite(PIN_BUZZ, HIGH);
} else {
    digitalWrite(PIN_BUZZ, LOW);
}

passive buzzer 用这个可能声音不明显。建议改成：

if (currentMillis - previousMillis < 50) {
    tone(PIN_BUZZ, 2000);
} else {
    noTone(PIN_BUZZ);
}

这样倒计时中每秒会有一个短音。

7. 最后追加メロディ函数

把下面这几个函数放到代码最后，allDigitsOff() 后面也可以：

// ----------------------------------------------------
// メロディ開始
// ----------------------------------------------------
void startMelody() {
    melodyIndex = 0;
    melodyPreviousMillis = millis();
    melodyPlaying = true;
    melodyGap = false;

    tone(PIN_BUZZ, MELODY_NOTES[melodyIndex]);
}

// ----------------------------------------------------
// メロディ更新（delayなし）
// ----------------------------------------------------
void updateMelody() {
    if (!melodyPlaying) {
        startMelody();
        return;
    }

    unsigned long currentMillis = millis();

    if (!melodyGap) {
        // 音を鳴らしている時間が終わったら一度止める
        if (currentMillis - melodyPreviousMillis >= MELODY_DURATIONS[melodyIndex]) {
            noTone(PIN_BUZZ);
            melodyGap = true;
            melodyPreviousMillis = currentMillis;
        }
    } else {
        // 短い休みが終わったら次の音へ
        if (currentMillis - melodyPreviousMillis >= MELODY_GAP) {
            melodyIndex++;

            // 最後まで行ったら最初から繰り返す
            if (melodyIndex >= MELODY_LENGTH) {
                melodyIndex = 0;
            }

            melodyGap = false;
            melodyPreviousMillis = currentMillis;
            tone(PIN_BUZZ, MELODY_NOTES[melodyIndex]);
        }
    }
}

// ----------------------------------------------------
// メロディ停止
// ----------------------------------------------------
void stopMelody() {
    melodyPlaying = false;
    melodyGap = false;
    melodyIndex = 0;
    noTone(PIN_BUZZ);
}
