// ----------------------------------------------------
// タイマーガジェット 4Digit MM.SS版
// - 4桁7セグで MM.SS 表示
// - 初期値 00.09
// - 短押し：開始 / 一時停止 / 再開
// - 長押し：リセット
// - 0になったらブザー鳴動
// - delay()なし、millis()使用
// ----------------------------------------------------

// --- 74HC595 ピン定義 ---
const int PIN_DATA  = 8;   // DS
const int PIN_LATCH = 12;  // STCP
const int PIN_CLOCK = 13;  // SHCP

// --- ボタン・ブザー ---
const int PIN_BTN  = 2;
const int PIN_BUZZ = 9;

// --- 4Digit 桁選択ピン ---
const int DIGIT_PINS[4] = {
  3, 4, 5, 6
};

// true  = カソードコモン / 共阴极
// false = アノードコモン / 共阳极
// さっき1234が正常表示できた設定に合わせる
const bool COMMON_CATHODE = true;

// --- 7セグメントLED 点灯パターン ---
// bit0=A, bit1=B, bit2=C, bit3=D, bit4=E, bit5=F, bit6=G, bit7=DP
const byte NUM_PATTERNS[10] = {
  B00111111, // 0
  B00000110, // 1
  B01011011, // 2
  B01001111, // 3
  B01100110, // 4
  B01101101, // 5
  B01111101, // 6
  B00000111, // 7
  B01111111, // 8
  B01101111  // 9
};

// --- 状態定義 ---
enum State {
  IDLE,
  RUNNING,
  PAUSED,
  FINISHED
};

State currentState = IDLE;

// --- タイマー設定 ---
const int INITIAL_SECONDS = 9;       // まずは 00.09 からテスト
int remainingSeconds = INITIAL_SECONDS;

unsigned long previousMillis = 0;
const long interval = 1000;
unsigned long pausedElapsed = 0;

// --- 4Digit 表示用 ---
int displayDigits[4] = {0, 0, 0, 9}; // MMSS
int currentDigit = 0;
unsigned long lastRefreshTime = 0;
const unsigned long refreshInterval = 2; // 1〜3ms程度

// --- ボタン処理用 ---
int lastButtonState = HIGH;
int buttonState = HIGH;
unsigned long lastDebounceTime = 0;
const long debounceDelay = 50;

// --- 長押し判定 ---
unsigned long pressStartTime = 0;
const long longPressTime = 1000;
bool longPressHandled = false;

void setup() {
  pinMode(PIN_DATA, OUTPUT);
  pinMode(PIN_LATCH, OUTPUT);
  pinMode(PIN_CLOCK, OUTPUT);
  pinMode(PIN_BUZZ, OUTPUT);

  pinMode(PIN_BTN, INPUT_PULLUP);

  for (int i = 0; i < 4; i++) {
    pinMode(DIGIT_PINS[i], OUTPUT);
  }

  allDigitsOff();
  sendSegments(0);

  Serial.begin(9600);
  Serial.println("Timer Gadget 4Digit MM.SS Started.");

  updateDisplayBuffer();
}

void loop() {
  // 4桁表示は常に高速更新する
  refreshDisplay();

  // ボタン入力処理
  handleButton();

  // 状態ごとの処理
  unsigned long currentMillis = millis();

  switch (currentState) {
    case IDLE:
      digitalWrite(PIN_BUZZ, LOW);
      break;

    case RUNNING:
      // カウントダウン中の短い「ピッ」音
      if (currentMillis - previousMillis < 50) {
        digitalWrite(PIN_BUZZ, HIGH);
      } else {
        digitalWrite(PIN_BUZZ, LOW);
      }

      // 1秒ごとに残り秒数を減らす
      if (currentMillis - previousMillis >= interval) {
        previousMillis = currentMillis;
        remainingSeconds--;

        if (remainingSeconds <= 0) {
          remainingSeconds = 0;
          currentState = FINISHED;
          Serial.println("Finished!");
        }

        updateDisplayBuffer();

        Serial.print("Remaining: ");
        Serial.println(remainingSeconds);
      }
      break;

    case PAUSED:
      digitalWrite(PIN_BUZZ, LOW);
      break;

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
  }
}

// ----------------------------------------------------
// ボタン処理
// ----------------------------------------------------
void handleButton() {
  int reading = digitalRead(PIN_BTN);

  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != buttonState) {
      buttonState = reading;

      // 押された瞬間
      if (buttonState == LOW) {
        pressStartTime = millis();
        longPressHandled = false;
      }

      // 離された瞬間
      if (buttonState == HIGH) {
        if (!longPressHandled) {
          handleShortPress();
        }
      }
    }
  }

  // 押され続けている場合、長押し判定
  if (buttonState == LOW && !longPressHandled) {
    if (millis() - pressStartTime >= longPressTime) {
      handleLongPress();
      longPressHandled = true;
    }
  }

  lastButtonState = reading;
}

// ----------------------------------------------------
// 短押し処理
// ----------------------------------------------------
void handleShortPress() {
  Serial.println("Short Press!");

  if (currentState == IDLE) {
    currentState = RUNNING;
    previousMillis = millis();
    Serial.println("Started!");
  }
  else if (currentState == RUNNING) {
    currentState = PAUSED;

    pausedElapsed = millis() - previousMillis;
    pausedElapsed = pausedElapsed % interval;

    digitalWrite(PIN_BUZZ, LOW);
    Serial.println("Paused!");
  }
  else if (currentState == PAUSED) {
    currentState = RUNNING;

    previousMillis = millis() - pausedElapsed;

    Serial.println("Resumed!");
  }
  else if (currentState == FINISHED) {
    resetTimer();
    Serial.println("Reset from FINISHED!");
  }
}

// ----------------------------------------------------
// 長押し処理
// ----------------------------------------------------
void handleLongPress() {
  Serial.println("Long Press!");
  resetTimer();
}

// ----------------------------------------------------
// リセット処理
// ----------------------------------------------------
void resetTimer() {
  currentState = IDLE;
  remainingSeconds = INITIAL_SECONDS;
  previousMillis = 0;
  pausedElapsed = 0;

  digitalWrite(PIN_BUZZ, LOW);
  updateDisplayBuffer();

  Serial.println("Reset to IDLE!");
}

// ----------------------------------------------------
// remainingSeconds を MM.SS 表示用の4桁に変換
// ----------------------------------------------------
void updateDisplayBuffer() {
  int minutes = remainingSeconds / 60;
  int seconds = remainingSeconds % 60;

  displayDigits[0] = minutes / 10;   // 分の10の位
  displayDigits[1] = minutes % 10;   // 分の1の位
  displayDigits[2] = seconds / 10;   // 秒の10の位
  displayDigits[3] = seconds % 10;   // 秒の1の位
}

// ----------------------------------------------------
// 4桁ダイナミック点灯
// ----------------------------------------------------
void refreshDisplay() {
  unsigned long currentMillis = millis();

  if (currentMillis - lastRefreshTime >= refreshInterval) {
    lastRefreshTime = currentMillis;

    allDigitsOff();

    int num = displayDigits[currentDigit];
    byte pattern = NUM_PATTERNS[num];

    // MM.SS の「.」を2桁目の後ろに表示
    // currentDigit == 1 が2桁目
    if (currentDigit == 1) {
      pattern = pattern | B10000000; // DP ON
    }

    sendSegments(pattern);
    digitOn(currentDigit);

    currentDigit++;
    if (currentDigit >= 4) {
      currentDigit = 0;
    }
  }
}

// ----------------------------------------------------
// 74HC595へ段データ送信
// ----------------------------------------------------
void sendSegments(byte pattern) {
  byte data = pattern;

  if (!COMMON_CATHODE) {
    data = ~pattern;
  }

  digitalWrite(PIN_LATCH, LOW);
  shiftOut(PIN_DATA, PIN_CLOCK, MSBFIRST, data);
  digitalWrite(PIN_LATCH, HIGH);
}

// ----------------------------------------------------
// 指定桁をON
// ----------------------------------------------------
void digitOn(int digitIndex) {
  if (COMMON_CATHODE) {
    digitalWrite(DIGIT_PINS[digitIndex], LOW);
  } else {
    digitalWrite(DIGIT_PINS[digitIndex], HIGH);
  }
}

// ----------------------------------------------------
// 全桁OFF
// ----------------------------------------------------
void allDigitsOff() {
  for (int i = 0; i < 4; i++) {
    if (COMMON_CATHODE) {
      digitalWrite(DIGIT_PINS[i], HIGH);
    } else {
      digitalWrite(DIGIT_PINS[i], LOW);
    }
  }
}
