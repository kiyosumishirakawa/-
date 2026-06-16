void updateTimeByPot() {
  int value = analogRead(PIN_POT);  // 0〜1023

  int minutes = map(value, 0, 1023, MIN_MINUTES, MAX_MINUTES);
  minutes = constrain(minutes, MIN_MINUTES, MAX_MINUTES);

  int newSettingSeconds = minutes * 60;

  if (newSettingSeconds != settingSeconds) {
    settingSeconds = newSettingSeconds;
    remainingSeconds = settingSeconds;
    updateDisplayBuffer();

    Serial.print("Set minutes: ");
    Serial.println(minutes);
  }
}
