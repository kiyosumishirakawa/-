const int notesHarryPotter[] = {
  REST, NOTE_D4,
  NOTE_G4, NOTE_AS4, NOTE_A4,
  NOTE_G4, NOTE_D5,
  NOTE_C5,
  NOTE_A4,

  NOTE_G4, NOTE_AS4, NOTE_A4,
  NOTE_F4, NOTE_GS4,
  NOTE_D4,
  NOTE_D4,

  NOTE_G4, NOTE_AS4, NOTE_A4,
  NOTE_G4, NOTE_D5,
  NOTE_F5, NOTE_E5,
  NOTE_DS5, NOTE_B4,

  NOTE_DS5, NOTE_D5, NOTE_CS5,
  NOTE_CS4, NOTE_B4,
  NOTE_G4,
  NOTE_AS4,

  NOTE_D5, NOTE_AS4,
  NOTE_D5, NOTE_AS4,
  NOTE_DS5, NOTE_D5,
  NOTE_CS5, NOTE_A4,

  NOTE_AS4, NOTE_D5, NOTE_CS5,
  NOTE_CS4, NOTE_D4,
  NOTE_D5,
  REST, NOTE_AS4,

  NOTE_D5, NOTE_AS4,
  NOTE_D5, NOTE_AS4,
  NOTE_F5, NOTE_E5,
  NOTE_DS5, NOTE_B4,

  NOTE_DS5, NOTE_D5, NOTE_CS5,
  NOTE_CS4, NOTE_AS4,
  NOTE_G4
};

const int durationsHarryPotter[] = {
  2, 4,
  -4, 8, 4,
  2, 4,
  -2,
  -2,

  -4, 8, 4,
  2, 4,
  -1,
  4,

  -4, 8, 4,
  2, 4,
  2, 4,
  2, 4,

  -4, 8, 4,
  2, 4,
  -1,
  4,

  2, 4,
  2, 4,
  2, 4,
  2, 4,

  -4, 8, 4,
  2, 4,
  -1,
  4, 4,

  2, 4,
  2, 4,
  2, 4,
  2, 4,

  -4, 8, 4,
  2, 4,
  -1
};

{
  notesHarryPotter,
  durationsHarryPotter,
  sizeof(notesHarryPotter) / sizeof(notesHarryPotter[0]),
  144,
  "Harry Potter"
}
