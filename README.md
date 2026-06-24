2. 追加这两个函数

放在 calcNoteDurationMs() 后面即可。

unsigned long getCurrentPositionMs() {
  Track& tr = tracks[currentTrack];
  unsigned long positionMs = 0;

  for (uint16_t i = 0; i < notePairIndex; i += 2) {
    int divider = tr.melody[i + 1];
    positionMs += calcNoteDurationMs(divider, tr.tempo);
  }

  return positionMs;
}

void setPositionByMs(unsigned long targetMs) {
  Track& tr = tracks[currentTrack];
  unsigned long totalMs = 0;

  for (uint16_t i = 0; i < tr.length; i += 2) {
    int divider = tr.melody[i + 1];
    unsigned long durationMs = calcNoteDurationMs(divider, tr.tempo);

    if (totalMs + durationMs >= targetMs) {
      notePairIndex = i;
      noteActive = false;
      noTone(BUZZER_PIN);
      return;
    }

    totalMs += durationMs;
  }

  // 目标时间超过歌曲长度时，跳到最后一个音符
  notePairIndex = tr.length - 2;
  noteActive = false;
  noTone(BUZZER_PIN);
}
3. 替换 seekPlayback()

把原来的 seekPlayback(int8_t stepNotes) 整个替换成这个：

void seekPlayback(int direction) {
  unsigned long currentPosition = getCurrentPositionMs();
  unsigned long targetPosition = 0;

  if (direction > 0) {
    targetPosition = currentPosition + SEEK_STEP_MS;
    setPositionByMs(targetPosition);
    printStatus("SEEK_FORWARD_10S");
  } else {
    if (currentPosition > SEEK_STEP_MS) {
      targetPosition = currentPosition - SEEK_STEP_MS;
    } else {
      targetPosition = 0;
    }

    setPositionByMs(targetPosition);
    printStatus("SEEK_BACK_10S");
  }
}
4. 修改 loop() 里的调用

找到左按钮这里：

seekPlayback(-SEEK_STEP);

改成：

seekPlayback(-1);

找到右按钮这里：

seekPlayback(SEEK_STEP);

改成：

seekPlayback(1);

也就是最后变成：

if (leftPressed) {
  if (state == PLAYING || state == PAUSED) {
    seekPlayback(-1);
  } else {
    selectTrack(-1);
  }
}

if (rightPressed) {
  if (state == PLAYING || state == PAUSED) {
    seekPlayback(1);
  } else {
    selectTrack(1);
  }
}
