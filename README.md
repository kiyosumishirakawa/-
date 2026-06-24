/*
  Simple Music Player
  Songs adapted from robsoncouto/arduino-songs:
  - Harry Potter / Hedwig's Theme
  - Happy Birthday

  Function:
  CENTER button: play / pause / resume
  LEFT button:
    - not playing: previous track
    - playing: seek backward
  RIGHT button:
    - not playing: next track
    - playing: seek forward
  RESET button: reset player
*/

// ===== Notes =====
#define NOTE_B0 31
#define NOTE_C1 33
#define NOTE_CS1 35
#define NOTE_D1 37
#define NOTE_DS1 39
#define NOTE_E1 41
#define NOTE_F1 44
#define NOTE_FS1 46
#define NOTE_G1 49
#define NOTE_GS1 52
#define NOTE_A1 55
#define NOTE_AS1 58
#define NOTE_B1 62
#define NOTE_C2 65
#define NOTE_CS2 69
#define NOTE_D2 73
#define NOTE_DS2 78
#define NOTE_E2 82
#define NOTE_F2 87
#define NOTE_FS2 93
#define NOTE_G2 98
#define NOTE_GS2 104
#define NOTE_A2 110
#define NOTE_AS2 117
#define NOTE_B2 123
#define NOTE_C3 131
#define NOTE_CS3 139
#define NOTE_D3 147
#define NOTE_DS3 156
#define NOTE_E3 165
#define NOTE_F3 175
#define NOTE_FS3 185
#define NOTE_G3 196
#define NOTE_GS3 208
#define NOTE_A3 220
#define NOTE_AS3 233
#define NOTE_B3 247
#define NOTE_C4 262
#define NOTE_CS4 277
#define NOTE_D4 294
#define NOTE_DS4 311
#define NOTE_E4 330
#define NOTE_F4 349
#define NOTE_FS4 370
#define NOTE_G4 392
#define NOTE_GS4 415
#define NOTE_A4 440
#define NOTE_AS4 466
#define NOTE_B4 494
#define NOTE_C5 523
#define NOTE_CS5 554
#define NOTE_D5 587
#define NOTE_DS5 622
#define NOTE_E5 659
#define NOTE_F5 698
#define NOTE_FS5 740
#define NOTE_G5 784
#define NOTE_GS5 831
#define NOTE_A5 880
#define NOTE_AS5 932
#define NOTE_B5 988
#define NOTE_C6 1047
#define NOTE_CS6 1109
#define NOTE_D6 1175
#define NOTE_DS6 1245
#define NOTE_E6 1319
#define NOTE_F6 1397
#define NOTE_FS6 1480
#define NOTE_G6 1568
#define NOTE_GS6 1661
#define NOTE_A6 1760
#define NOTE_AS6 1865
#define NOTE_B6 1976
#define NOTE_C7 2093
#define NOTE_CS7 2217
#define NOTE_D7 2349
#define NOTE_DS7 2489
#define NOTE_E7 2637
#define NOTE_F7 2794
#define NOTE_FS7 2960
#define NOTE_G7 3136
#define NOTE_GS7 3322
#define NOTE_A7 3520
#define NOTE_AS7 3729
#define NOTE_B7 3951
#define NOTE_C8 4186
#define NOTE_CS8 4435
#define NOTE_D8 4699
#define NOTE_DS8 4978
#define REST 0

// ===== Pin settings =====
// docs/detailed_design.md に合わせる
#define BUZZER_PIN     8
#define RIGHT_SW_PIN   2
#define CENTER_SW_PIN  3
#define LEFT_SW_PIN    6
#define RESET_SW_PIN   7

#define DEBOUNCE_MS    50
#define SEEK_STEP      4

enum PlayerState {
  IDLE,
  PLAYING,
  PAUSED
};

struct ButtonState {
  bool lastReading;
  bool stableState;
  unsigned long lastChangedMs;
};

struct Track {
  const int* melody;
  uint16_t length;
  uint16_t tempo;
  const char* name;
};

// ===== Songs =====
// melody format:
// { NOTE, duration, NOTE, duration, ... }
// duration: 4 = quarter note, 8 = eighth note
// negative duration means dotted note, e.g. -4 = dotted quarter note

const int melodyHarryPotter[] = {
  REST, 2, NOTE_D4, 4,
  NOTE_G4, -4, NOTE_AS4, 8, NOTE_A4, 4,
  NOTE_G4, 2, NOTE_D5, 4,
  NOTE_C5, -2,
  NOTE_A4, -2,

  NOTE_G4, -4, NOTE_AS4, 8, NOTE_A4, 4,
  NOTE_F4, 2, NOTE_GS4, 4,
  NOTE_D4, -1,
  NOTE_D4, 4,

  NOTE_G4, -4, NOTE_AS4, 8, NOTE_A4, 4,
  NOTE_G4, 2, NOTE_D5, 4,
  NOTE_F5, 2, NOTE_E5, 4,
  NOTE_DS5, 2, NOTE_B4, 4,

  NOTE_DS5, -4, NOTE_D5, 8, NOTE_CS5, 4,
  NOTE_CS4, 2, NOTE_B4, 4,
  NOTE_G4, -1
};

const int melodyHappyBirthday[] = {
  NOTE_C4, 4, NOTE_C4, 8,
  NOTE_D4, -4, NOTE_C4, -4, NOTE_F4, -4,
  NOTE_E4, -2,

  NOTE_C4, 4, NOTE_C4, 8,
  NOTE_D4, -4, NOTE_C4, -4, NOTE_G4, -4,
  NOTE_F4, -2,

  NOTE_C4, 4, NOTE_C4, 8,
  NOTE_C5, -4, NOTE_A4, -4, NOTE_F4, -4,
  NOTE_E4, -4, NOTE_D4, -4,

  NOTE_AS4, 4, NOTE_AS4, 8,
  NOTE_A4, -4, NOTE_F4, -4, NOTE_G4, -4,
  NOTE_F4, -2
};

Track tracks[] = {
  {
    melodyHarryPotter,
    sizeof(melodyHarryPotter) / sizeof(melodyHarryPotter[0]),
    144,
    "Harry Potter"
  },
  {
    melodyHappyBirthday,
    sizeof(melodyHappyBirthday) / sizeof(melodyHappyBirthday[0]),
    140,
    "Happy Birthday"
  }
};

const uint8_t TRACK_COUNT = sizeof(tracks) / sizeof(tracks[0]);

// ===== Player variables =====
PlayerState state = IDLE;
uint8_t currentTrack = 0;

// notePairIndex points to NOTE position.
// Example: melody[0] = NOTE, melody[1] = duration
uint16_t notePairIndex = 0;

unsigned long noteStartMs = 0;
unsigned long noteDurationMs = 0;
bool noteActive = false;

ButtonState btnCenter = { HIGH, HIGH, 0 };
ButtonState btnLeft   = { HIGH, HIGH, 0 };
ButtonState btnRight  = { HIGH, HIGH, 0 };
ButtonState btnReset  = { HIGH, HIGH, 0 };

// ===== Utility =====
const char* stateName(PlayerState s) {
  switch (s) {
    case IDLE: return "IDLE";
    case PLAYING: return "PLAYING";
    case PAUSED: return "PAUSED";
  }
  return "UNKNOWN";
}

bool isShortPress(uint8_t pin, ButtonState& btn) {
  bool reading = digitalRead(pin);
  unsigned long now = millis();

  if (reading != btn.lastReading) {
    btn.lastReading = reading;
    btn.lastChangedMs = now;
  }

  if ((now - btn.lastChangedMs) > DEBOUNCE_MS && reading != btn.stableState) {
    btn.stableState = reading;

    if (btn.stableState == LOW) {
      return true;
    }
  }

  return false;
}

unsigned long calcNoteDurationMs(int divider, uint16_t tempo) {
  unsigned long wholeNote = (60000UL * 4) / tempo;

  if (divider > 0) {
    return wholeNote / divider;
  }

  if (divider < 0) {
    unsigned long duration = wholeNote / abs(divider);
    return duration + duration / 2;
  }

  return 0;
}

void printStatus(const char* eventName) {
  Serial.print(eventName);
  Serial.print(", track=");
  Serial.print(currentTrack);
  Serial.print(", name=");
  Serial.print(tracks[currentTrack].name);
  Serial.print(", index=");
  Serial.print(notePairIndex / 2);
  Serial.print(", state=");
  Serial.println(stateName(state));
}

// ===== Player controls =====
void startPlayback() {
  if (notePairIndex >= tracks[currentTrack].length) {
    notePairIndex = 0;
  }

  state = PLAYING;
  noteActive = false;
  printStatus("PLAY");
}

void pausePlayback() {
  noTone(BUZZER_PIN);
  noteActive = false;
  state = PAUSED;
  printStatus("PAUSE");
}

void resumePlayback() {
  state = PLAYING;
  noteActive = false;
  printStatus("RESUME");
}

void resetPlayer() {
  noTone(BUZZER_PIN);
  state = IDLE;
  notePairIndex = 0;
  noteActive = false;
  printStatus("RESET");
}

void selectTrack(int8_t delta) {
  int8_t next = (int8_t)currentTrack + delta;

  if (next < 0) {
    next = TRACK_COUNT - 1;
  }

  if (next >= TRACK_COUNT) {
    next = 0;
  }

  currentTrack = (uint8_t)next;
  notePairIndex = 0;
  noteActive = false;
  noTone(BUZZER_PIN);
  state = IDLE;

  printStatus("SELECT_TRACK");
}

void seekPlayback(int8_t stepNotes) {
  int16_t next = (int16_t)(notePairIndex / 2) + stepNotes;

  if (next < 0) {
    next = 0;
  }

  uint16_t maxNoteIndex = (tracks[currentTrack].length / 2) - 1;
  if (next > maxNoteIndex) {
    next = maxNoteIndex;
  }

  notePairIndex = (uint16_t)next * 2;
  noteActive = false;
  noTone(BUZZER_PIN);

  if (stepNotes > 0) {
    printStatus("SEEK_FORWARD");
  } else {
    printStatus("SEEK_BACK");
  }
}

// ===== Playback update =====
void updatePlayback() {
  if (state != PLAYING) {
    return;
  }

  Track& tr = tracks[currentTrack];

  if (notePairIndex >= tr.length) {
    resetPlayer();
    printStatus("END");
    return;
  }

  if (!noteActive) {
    int pitch = tr.melody[notePairIndex];
    int divider = tr.melody[notePairIndex + 1];

    noteDurationMs = calcNoteDurationMs(divider, tr.tempo);
    noteStartMs = millis();
    noteActive = true;

    if (pitch == REST) {
      noTone(BUZZER_PIN);
    } else {
      tone(BUZZER_PIN, pitch, noteDurationMs * 0.9);
    }

    return;
  }

  if (millis() - noteStartMs >= noteDurationMs) {
    noTone(BUZZER_PIN);
    notePairIndex += 2;
    noteActive = false;
  }
}

// ===== Arduino setup / loop =====
void setup() {
  Serial.begin(9600);

  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(RIGHT_SW_PIN, INPUT_PULLUP);
  pinMode(CENTER_SW_PIN, INPUT_PULLUP);
  pinMode(LEFT_SW_PIN, INPUT_PULLUP);
  pinMode(RESET_SW_PIN, INPUT_PULLUP);

  Serial.println("Simple Music Player Ready");
  printStatus("INIT");
}

void loop() {
  bool centerPressed = isShortPress(CENTER_SW_PIN, btnCenter);
  bool leftPressed = isShortPress(LEFT_SW_PIN, btnLeft);
  bool rightPressed = isShortPress(RIGHT_SW_PIN, btnRight);
  bool resetPressed = isShortPress(RESET_SW_PIN, btnReset);

  if (resetPressed) {
    resetPlayer();
  }

  if (centerPressed) {
    if (state == IDLE) {
      startPlayback();
    } else if (state == PLAYING) {
      pausePlayback();
    } else if (state == PAUSED) {
      resumePlayback();
    }
  }

  if (leftPressed) {
    if (state == PLAYING || state == PAUSED) {
      seekPlayback(-SEEK_STEP);
    } else {
      selectTrack(-1);
    }
  }

  if (rightPressed) {
    if (state == PLAYING || state == PAUSED) {
      seekPlayback(SEEK_STEP);
    } else {
      selectTrack(1);
    }
  }

  updatePlayback();
}
