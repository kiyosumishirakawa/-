// ----------------------------------------------------
// タイマーガジェット Step2版
// - 開始ボタンで 9 から 0 までカウントダウン
// - カウント中に短押しで一時停止
// - 一時停止中に短押しで再開
// - 長押しで 9 にリセット
// - delay() を使わず millis() で処理
// ----------------------------------------------------

// --- ピン定義 ---
const int PIN_DATA  = 8;   // 74HC595 DS
const int PIN_LATCH = 12;  // 74HC595 STCP
const int PIN_CLOCK = 13;  // 74HC595 SHCP
const int PIN_BTN   = 2;   // タクトスイッチ
const int PIN_BUZZ  = 9;   // アクティブブザー

// --- 7セグメントLED 点灯パターン ---
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

// --- 状態定義（ステートマシン） ---
enum State {
  IDLE,      // 待機中
  RUNNING,   // カウントダウン中
  PAUSED,    // 一時停止中
  FINISHED   // 終了
};

State currentState = IDLE;

// --- グローバル変数 ---
int currentNumber = 9;
unsigned long previousMillis = 0;
const long interval = 1000;

// 一時停止時に、1秒の途中経過を保存する
unsigned long pausedElapsed = 0;

// --- ボタン処理用変数 ---
int lastButtonState = HIGH;
int buttonState = HIGH;
unsigned long lastDebounceTime = 0;
const long debounceDelay = 50;

// 長押し判定用
unsigned long pressStartTime = 0;
const long longPressTime = 1000;  // 1000ms以上で長押し
bool longPressHandled = false;

void setup() {
  pinMode(PIN_DATA, OUTPUT);
  pinMode(PIN_LATCH, OUTPUT);
  pinMode(PIN_CLOCK, OUTPUT);
  pinMode(PIN_BUZZ, OUTPUT);

  pinMode(PIN_BTN, INPUT_PULLUP);

  Serial.begin(9600);
  Serial.println("Timer Gadget Step2 Started.");

  displayNumber(currentNumber);
}

void loop() {
  handleButton();

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

      // 1秒ごとに数字を減らす
      if (currentMillis - previousMillis >= interval) {
        previousMillis = currentMillis;
        currentNumber--;

        if (currentNumber <= 0) {
          currentNumber = 0;
          currentState = FINISHED;
          Serial.println("Finished!");
        }

        displayNumber(currentNumber);
        Serial.print("Countdown: ");
        Serial.println(currentNumber);
      }
      break;

    case PAUSED:
      // 一時停止中はカウントもブザーも止める
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

// --- ボタン処理 ---
void handleButton() {
  int reading = digitalRead(PIN_BTN);

  // 読み取り状態が変わったらチャタリング対策タイマーをリセット
  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }

  // 状態が一定時間安定したら、正式なボタン状態として扱う
  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != buttonState) {
      buttonState = reading;

      // ボタンが押された瞬間
      if (buttonState == LOW) {
        pressStartTime = millis();
        longPressHandled = false;
      }

      // ボタンが離された瞬間
      if (buttonState == HIGH) {
        if (!longPressHandled) {
          handleShortPress();
        }
      }
    }
  }

  // 押され続けている間に長押し判定
  if (buttonState == LOW && !longPressHandled) {
    if (millis() - pressStartTime >= longPressTime) {
      handleLongPress();
      longPressHandled = true;
    }
  }

  lastButtonState = reading;
}

// --- 短押し処理 ---
void handleShortPress() {
  Serial.println("Short Press!");

  if (currentState == IDLE) {
    currentState = RUNNING;
    currentNumber = 9;
    previousMillis = millis();
    displayNumber(currentNumber);
    Serial.println("Started!");
  }
  else if (currentState == RUNNING) {
    currentState = PAUSED;

    // 1秒カウントの途中経過を保存
    pausedElapsed = millis() - previousMillis;
    pausedElapsed = pausedElapsed % interval;

    digitalWrite(PIN_BUZZ, LOW);
    Serial.println("Paused!");
  }
  else if (currentState == PAUSED) {
    currentState = RUNNING;

    // 一時停止前の途中経過を引き継いで再開
    previousMillis = millis() - pausedElapsed;

    Serial.println("Resumed!");
  }
  else if (currentState == FINISHED) {
    resetTimer();
    Serial.println("Reset from FINISHED!");
  }
}

// --- 長押し処理 ---
void handleLongPress() {
  Serial.println("Long Press!");
  resetTimer();
}

// --- リセット処理 ---
void resetTimer() {
  currentState = IDLE;
  currentNumber = 9;
  previousMillis = 0;
  pausedElapsed = 0;
  digitalWrite(PIN_BUZZ, LOW);
  displayNumber(currentNumber);
  Serial.println("Reset to IDLE!");
}

// --- 74HC595経由で7セグに数字を表示する関数 ---
void displayNumber(int num) {
  if (num < 0 || num > 9) return;

  digitalWrite(PIN_LATCH, LOW);
  shiftOut(PIN_DATA, PIN_CLOCK, MSBFIRST, NUM_PATTERNS[num]);
  digitalWrite(PIN_LATCH, HIGH);
}
