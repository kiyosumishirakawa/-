void updateTimeByPot() {
  // analogRead のブレを減らすため、複数回読んで平均を取る
  long sum = 0;
  for (int i = 0; i < 8; i++) {
    sum += analogRead(PIN_POT);
  }

  int value = sum / 8;  // 0〜1023

  // 30秒単位の段数を計算
  int maxStep = (MAX_SETTING_SECONDS - MIN_SETTING_SECONDS + STEP_SECONDS - 1) / STEP_SECONDS;

  // 0〜1023 を 0〜maxStep に変換
  // maxStep + 1 にしてから constrain することで、最大付近を少し広くする
  int step = map(value, 0, 1023, 0, maxStep + 1);
  step = constrain(step, 0, maxStep);

  int newSettingSeconds = MIN_SETTING_SECONDS + step * STEP_SECONDS;

  // 最大値を超えた場合は 99.59 に補正
  if (newSettingSeconds > MAX_SETTING_SECONDS) {
    newSettingSeconds = MAX_SETTING_SECONDS;
  }

  if (newSettingSeconds != settingSeconds) {
    settingSeconds = newSettingSeconds;
    remainingSeconds = settingSeconds;
    updateDisplayBuffer();

    Serial.print("Analog: ");
    Serial.print(value);
    Serial.print(" / Set time: ");
    Serial.print(settingSeconds / 60);
    Serial.print(".");
    if (settingSeconds % 60 < 10) {
      Serial.print("0");
    }
    Serial.println(settingSeconds % 60);
  }
}
