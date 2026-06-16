// ----------------------------------------------------
// 4Digit 7-Segment Display 最小测试版
// 目的：通过74HC595 + 动态点灯显示 1234
// ----------------------------------------------------

// --- 74HC595 ピン定義 ---
const int PIN_DATA  = 8;   // DS
const int PIN_LATCH = 12;  // STCP
const int PIN_CLOCK = 13;  // SHCP

// --- 桁選択ピン ---
const int DIGIT_PINS[4] = {
  3, 4, 5, 6
};

// true  = カソードコモン / 共阴极
// false = アノードコモン / 共阳极
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

// 显示内容：1234
int displayDigits[4] = {
  1, 2, 3, 4
};

int currentDigit = 0;
unsigned long lastRefreshTime = 0;

// 每一位点亮的时间，单位ms
// 1〜3ms 左右都可以
const unsigned long refreshInterval = 2;

void setup() {
  pinMode(PIN_DATA, OUTPUT);
  pinMode(PIN_LATCH, OUTPUT);
  pinMode(PIN_CLOCK, OUTPUT);

  for (int i = 0; i < 4; i++) {
    pinMode(DIGIT_PINS[i], OUTPUT);
  }

  allDigitsOff();
  sendSegments(0);

  Serial.begin(9600);
  Serial.println("4Digit Display Test Started.");
}

void loop() {
  refreshDisplay();
}

// --- 动态点灯刷新 ---
void refreshDisplay() {
  unsigned long currentMillis = millis();

  if (currentMillis - lastRefreshTime >= refreshInterval) {
    lastRefreshTime = currentMillis;

    // 先全部关掉，防止残影
    allDigitsOff();

    // 送出当前数字的段数据
    int num = displayDigits[currentDigit];
    sendSegments(NUM_PATTERNS[num]);

    // 打开当前这一位
    digitOn(currentDigit);

    // 下次显示下一位
    currentDigit++;
    if (currentDigit >= 4) {
      currentDigit = 0;
    }
  }
}

// --- 发送段数据到74HC595 ---
void sendSegments(byte pattern) {
  byte data = pattern;

  // 如果是共阳极，段信号需要反转
  if (!COMMON_CATHODE) {
    data = ~pattern;
  }

  digitalWrite(PIN_LATCH, LOW);
  shiftOut(PIN_DATA, PIN_CLOCK, MSBFIRST, data);
  digitalWrite(PIN_LATCH, HIGH);
}

// --- 打开指定桁 ---
void digitOn(int digitIndex) {
  if (COMMON_CATHODE) {
    // 共阴极：该位公共端 LOW 时亮
    digitalWrite(DIGIT_PINS[digitIndex], LOW);
  } else {
    // 共阳极：该位公共端 HIGH 时亮
    digitalWrite(DIGIT_PINS[digitIndex], HIGH);
  }
}

// --- 关闭所有桁 ---
void allDigitsOff() {
  for (int i = 0; i < 4; i++) {
    if (COMMON_CATHODE) {
      digitalWrite(DIGIT_PINS[i], HIGH);
    } else {
      digitalWrite(DIGIT_PINS[i], LOW);
    }
  }
}
