1. 先追加缺少的音符

你原代码没有 NOTE_AS4，链接里的 Happy Birthday 用到了它。
在 #define NOTE_A4 440 附近追加：

#define NOTE_AS4 466
2. 修改 Track 结构体

你现在的 durations 是 uint8_t，不能保存 -4、-2 这种负数。
所以把原来的：

struct Track {
  const uint16_t* melody;
  const uint8_t* durations;
  uint8_t length;
  const char* name;
};

改成：

struct Track {
  const uint16_t* melody;
  const int8_t* durations;
  uint8_t length;
  const char* name;
  uint16_t tempo;
};
3. 把所有 dur 数组的类型改一下

比如原来是：

const uint8_t durTwinkle[] = {

改成：

const int8_t durTwinkle[] = {

其他几个也一样：

const int8_t durKimigayo[] = {
const int8_t durHappyBirthday[] = {
const int8_t durHarryPotter[] = {

原来那些正数 4, 8, 2 不受影响。

4. 替换 Happy Birthday 的 melody 和 dur

把你现在这两段：

const uint16_t melodyHappyBirthday[] = {
  ...
};

const uint8_t durHappyBirthday[] = {
  ...
};

替换成这个：

const uint16_t melodyHappyBirthday[] = {
  NOTE_C4, NOTE_C4,
  NOTE_D4, NOTE_C4, NOTE_F4,
  NOTE_E4, NOTE_C4, NOTE_C4,

  NOTE_D4, NOTE_C4, NOTE_G4,
  NOTE_F4, NOTE_C4, NOTE_C4,

  NOTE_C5, NOTE_A4, NOTE_F4,
  NOTE_E4, NOTE_D4, NOTE_AS4, NOTE_AS4,

  NOTE_A4, NOTE_F4, NOTE_G4,
  NOTE_F4
};

const int8_t durHappyBirthday[] = {
  4, 8,
  -4, -4, -4,
  -2, 4, 8,

  -4, -4, -4,
  -2, 4, 8,

  -4, -4, -4,
  -4, -4, 4, 8,

  -4, -4, -4,
  -2
};

这个就是把链接里的：

NOTE_C4,4, NOTE_C4,8, ...

拆成你项目需要的：

melodyHappyBirthday[] = 音符
durHappyBirthday[] = 时长
5. 修改 tracks[] 注册部分

你原来大概是这样：

Track tracks[] = {
  { melodyTwinkle, durTwinkle, ..., "きらきら星" },
  { melodyKimigayo, durKimigayo, ..., "君が代" },
  { melodyHappyBirthday, durHappyBirthday, ..., "happybirthday" },
  { melodyHarryPotter, durHarryPotter, ..., "harrypotter" }
};

改成这样：

Track tracks[] = {
  { melodyTwinkle, durTwinkle, (uint8_t)(sizeof(melodyTwinkle) / sizeof(melodyTwinkle[0])), "きらきら星", 240 },
  { melodyKimigayo, durKimigayo, (uint8_t)(sizeof(melodyKimigayo) / sizeof(melodyKimigayo[0])), "君が代", 240 },
  { melodyHappyBirthday, durHappyBirthday, (uint8_t)(sizeof(melodyHappyBirthday) / sizeof(melodyHappyBirthday[0])), "happybirthday", 140 },
  { melodyHarryPotter, durHarryPotter, (uint8_t)(sizeof(melodyHarryPotter) / sizeof(melodyHarryPotter[0])), "harrypotter", 240 }
};

这里 happybirthday 用 140，是因为链接里的原代码 tempo = 140。其他曲子先用 240，这样基本能保持你原来 1000 / duration 的速度感。

6. 加一个计算音长的函数

放在 updatePlayback() 前面即可：

unsigned long calcNoteDurationMs(int8_t divider, uint16_t tempo) {
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

这个函数就是为了处理 -4、-2 这种附点音符。

7. 修改 updatePlayback() 里的音长计算

找到这行：

noteDurationMs = 1000UL / tr.durations[playIndex];

改成：

noteDurationMs = calcNoteDurationMs(tr.durations[playIndex], tr.tempo);

然后这里也建议稍微改一下，支持以后加休止符：

if (currentVolumeRaw < 10 || tr.melody[playIndex] == 0) {
  noTone(BUZZER_PIN);
} else {
  tone(BUZZER_PIN, tr.melody[playIndex], noteDurationMs);
}

这样改完之后，你的 Happy Birthday 就会接近链接里的版本，而不是现在这种过度简化版。
