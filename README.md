3. Happy Birthday 拆分版

把原来的 melodyHappyBirthday[] 替换成这两组：

const int notesHappyBirthday[] = {
  NOTE_C4, NOTE_C4,
  NOTE_D4, NOTE_C4, NOTE_F4,
  NOTE_E4,

  NOTE_C4, NOTE_C4,
  NOTE_D4, NOTE_C4, NOTE_G4,
  NOTE_F4,

  NOTE_C4, NOTE_C4,
  NOTE_C5, NOTE_A4, NOTE_F4,
  NOTE_E4, NOTE_D4,

  NOTE_AS4, NOTE_AS4,
  NOTE_A4, NOTE_F4, NOTE_G4,
  NOTE_F4
};

const int durationsHappyBirthday[] = {
  4, 8,
  -4, -4, -4,
  -2,

  4, 8,
  -4, -4, -4,
  -2,

  4, 8,
  -4, -4, -4,
  -4, -4,

  4, 8,
  -4, -4, -4,
  -2
};
4. tracks[] 也要改

把原来的：

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

改成：

Track tracks[] = {
  {
    notesHarryPotter,
    durationsHarryPotter,
    sizeof(notesHarryPotter) / sizeof(notesHarryPotter[0]),
    144,
    "Harry Potter"
  },
  {
    notesHappyBirthday,
    durationsHappyBirthday,
    sizeof(notesHappyBirthday) / sizeof(notesHappyBirthday[0]),
    140,
    "Happy Birthday"
  }
};
5. 播放位置变量建议改名

原来是：

uint16_t notePairIndex = 0;

因为现在不是一对一对跳了，所以建议改成：

uint16_t playIndex = 0;

然后代码里所有 notePairIndex 都要跟着改。

6. updatePlayback() 里也要改

原来是这种：

int pitch = tr.melody[notePairIndex];
int divider = tr.melody[notePairIndex + 1];

现在要改成：

int pitch = tr.notes[playIndex];
int divider = tr.durations[playIndex];

然后播放完一个音后，原来是：

notePairIndex += 2;

现在改成：

playIndex++;

完整的 updatePlayback() 可以写成这样：

void updatePlayback() {
  if (state != PLAYING) {
    return;
  }

  Track& tr = tracks[currentTrack];

  if (playIndex >= tr.length) {
    resetPlayer();
    printStatus("END");
    return;
  }

  if (!noteActive) {
    int pitch = tr.notes[playIndex];
    int divider = tr.durations[playIndex];

    noteDurationMs = calcNoteDurationMs(divider, tr.tempo);
    noteStartMs = millis();
    noteActive = true;

    if (pitch == REST) {
      noTone(BUZZER_PIN);
    } else {
      tone(BUZZER_PIN, pitch, noteDurationMs);
    }

    return;
  }

  if (millis() - noteStartMs >= noteDurationMs) {
    noTone(BUZZER_PIN);
    playIndex++;
    noteActive = false;
  }
}
7. reset / select / start 里的位置也要改

凡是原来有：

notePairIndex = 0;

都改成：

playIndex = 0;

凡是原来有：

if (notePairIndex >= tracks[currentTrack].length)

都改成：

if (playIndex >= tracks[currentTrack].length)
8. 如果你已经做了 ±10秒快进快退，也要改

拆分以后，计算时间的函数要这样写：

unsigned long getCurrentPositionMs() {
  Track& tr = tracks[currentTrack];
  unsigned long positionMs = 0;

  for (uint16_t i = 0; i < playIndex; i++) {
    int divider = tr.durations[i];
    positionMs += calcNoteDurationMs(divider, tr.tempo);
  }

  return positionMs;
}
void setPositionByMs(unsigned long targetMs) {
  Track& tr = tracks[currentTrack];
  unsigned long totalMs = 0;

  for (uint16_t i = 0; i < tr.length; i++) {
    int divider = tr.durations[i];
    unsigned long durationMs = calcNoteDurationMs(divider, tr.tempo);

    if (totalMs + durationMs >= targetMs) {
      playIndex = i;
      noteActive = false;
      noTone(BUZZER_PIN);
      return;
    }

    totalMs += durationMs;
  }

  playIndex = tr.length - 1;
  noteActive = false;
  noTone(BUZZER_PIN);
}
