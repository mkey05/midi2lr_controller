#include <Encoder.h>
#include <Wire.h>
#include <Bounce2.h>
#include <Adafruit_Trellis.h>
#define LED     13 // Pin for heartbeat LED (shows code is working)
#define CHANNEL 1  // MIDI channel number

Adafruit_Trellis trellis;


int encoderPins[4][2] = {
  {11, 12}, //encoder 1
  {4, 5}, //encoder 2
  {6, 10}, //encoder 3
  {8, 9}, //encoder 4
}; 

Bounce push_button_1 = Bounce();
Bounce push_button_2 = Bounce();

Bounce switch_button_1 = Bounce();
Bounce switch_button_2 = Bounce();
Bounce switch_button_3 = Bounce();
Bounce switch_button_4 = Bounce();

const int swPin_1 = 7;
const int swPin_2 = A4;
const int swPin_3 = A5;
const int swPin_4 = 13;

const int button_1 = A0;
const int button_2 = A1;
const int button_3 = A2;
const int button_4 = A3;

int debounce_time = 5;

uint8_t       heart        = 0;  // Heartbeat LED counter
unsigned long prevReadTime = 0L; // Keypad polling timer

uint8_t note[] = {
  60, 61, 62, 63,
  56, 57, 58, 59,
  52, 53, 54, 55,
  48, 49, 50, 51
};

Encoder *encoders[4];
boolean toReadEncoder[4];
long encPosition[12];
long tempEncPosition;

void setup() {
  pinMode(LED, OUTPUT);
  trellis.begin(0x70); // Pass I2C address
#ifdef __AVR__
  // Default Arduino I2C speed is 100 KHz, but the HT16K33 supports
  // 400 KHz.  We can force this for faster read & refresh, but may
  // break compatibility with other I2C devices...so be prepared to
  // comment this out, or save & restore value as needed.
  TWBR = 12;
#endif
  trellis.clear();
  trellis.writeDisplay();
  pinMode(swPin_1, INPUT_PULLUP);
  pinMode(swPin_2, INPUT_PULLUP);
  pinMode(swPin_3, INPUT_PULLUP);
  pinMode(swPin_4, INPUT_PULLUP);
  pinMode(button_1, INPUT_PULLUP);
  pinMode(button_2, INPUT_PULLUP);
  pinMode(button_3, INPUT_PULLUP);
  pinMode(button_4, INPUT_PULLUP);
  digitalWrite(swPin_1, HIGH);
  digitalWrite(swPin_2, HIGH);
  digitalWrite(swPin_3, HIGH);
  digitalWrite(swPin_4, HIGH);
  digitalWrite(button_1, LOW);
  digitalWrite(button_2, LOW);
  digitalWrite(button_3, LOW);
  digitalWrite(button_4, LOW);
  push_button_1.attach(button_1);
  push_button_1.interval(debounce_time);
  push_button_2.attach(button_2);
  push_button_2.interval(debounce_time);
  switch_button_1.attach(swPin_1);
  switch_button_1.interval(debounce_time);
  switch_button_2.attach(swPin_2);
  switch_button_2.interval(debounce_time);
  switch_button_3.attach(swPin_3);
  switch_button_3.interval(debounce_time);
  switch_button_4.attach(swPin_4);
  switch_button_4.interval(debounce_time);
  for (int e = 0; e < 4; e++) {
    if (encoderPins[e][0] != 99 && encoderPins[e][1] != 99) {
      encoders[e] = new Encoder(encoderPins[e][0], encoderPins[e][1]);
      toReadEncoder[e] = true;
    }
    else {
      toReadEncoder[e] = false;
    }
  }
}


void loop() {

  if (digitalRead(button_3) == HIGH && digitalRead(button_4) == LOW ) { //shift button is engaged
    for (int e = 0; e < 4; e++) { //loop through all encoders
      if (toReadEncoder[e] == true) { //check if we should read this encoder
        tempEncPosition = encoders[e]->read(); //get encoder position
        if (tempEncPosition > encPosition[e]) { //this position is greater than the last
          usbMIDI.sendControlChange(e + 4, 1, CHANNEL);
          encPosition[e] = tempEncPosition; //update position
        } else if (tempEncPosition < encPosition[e]) { //this position is less than the last
          usbMIDI.sendControlChange(e + 4, 127, CHANNEL);
          encPosition[e] = tempEncPosition; //update position
        } else {
          //do nothing
        }
      }
    }
  }
  else if (digitalRead(button_3) == LOW && digitalRead(button_4) == HIGH ) { //shift button is engaged
    for (int e = 0; e < 4; e++) { //loop through all encoders
      if (toReadEncoder[e] == true) { //check if we should read this encoder
        tempEncPosition = encoders[e]->read(); //get encoder position
        if (tempEncPosition > encPosition[e]) { //this position is greater than the last
          usbMIDI.sendControlChange(e + 8, 1, CHANNEL);
          encPosition[e] = tempEncPosition; //update position
        } else if (tempEncPosition < encPosition[e]) { //this position is less than the last
          usbMIDI.sendControlChange(e + 8, 127, CHANNEL);
          encPosition[e] = tempEncPosition; //update position
        } else {
          //do nothing
        }
      }
    }
  }
  else { //shift button is engaged
    for (int e = 0; e < 4; e++) { //loop through all encoders
      if (toReadEncoder[e] == true) { //check if we should read this encoder
        tempEncPosition = encoders[e]->read(); //get encoder position
        if (tempEncPosition > encPosition[e]) { //this position is greater than the last
          usbMIDI.sendControlChange(e, 1, CHANNEL);
          encPosition[e] = tempEncPosition; //update position
        } else if (tempEncPosition < encPosition[e]) { //this position is less than the last
          usbMIDI.sendControlChange(e, 127, CHANNEL);
          encPosition[e] = tempEncPosition; //update position
        } else {
          //do nothing
        }
      }
    }
  }

  unsigned long t = millis();

  if ((t - prevReadTime) >= 20L) { // 20ms = min Trellis poll time
    if (trellis.readSwitches()) { // Button state change?

      for (uint8_t i = 0; i < 16; i++) { // For each button...
        if (trellis.justPressed(i)) {
          usbMIDI.sendNoteOn(note[i], 127, CHANNEL);

          trellis.setLED(i);
        } else if (trellis.justReleased(i)) {
          usbMIDI.sendNoteOff(note[i], 0, CHANNEL);
          trellis.clrLED(i);
        }
      }
      trellis.writeDisplay();
    }

    switch_button_1.update();

    if (switch_button_1.fell() == true) {
      usbMIDI.sendNoteOn(70, 127, CHANNEL); // send note on
    }

    if (switch_button_1.rose() == true) {
      usbMIDI.sendNoteOn(70, 0, CHANNEL); // send note off
    }

    switch_button_2.update();

    if (switch_button_2.fell() == true) {
      usbMIDI.sendNoteOn(72, 127, CHANNEL); // send note on
    }

    if (switch_button_2.rose() == true) {
      usbMIDI.sendNoteOn(72, 0, CHANNEL); // send note off
    }

    switch_button_3.update();

    if (switch_button_3.fell() == true) {
      usbMIDI.sendNoteOn(71, 127, CHANNEL); // send note on
    }

    if (switch_button_3.rose() == true) {
      usbMIDI.sendNoteOn(71, 0, CHANNEL); // send note off
    }

    switch_button_4.update();

    if (switch_button_4.fell() == true) {
      usbMIDI.sendNoteOn(73, 127, CHANNEL); // send note on
    }

    if (switch_button_4.rose() == true) {
      usbMIDI.sendNoteOn(73, 0, CHANNEL); // send note off
    }

    push_button_1.update();

    if (push_button_1.fell() == true) {
      usbMIDI.sendNoteOn(74, 0, CHANNEL); // send note on
    }

    if (push_button_1.rose() == true) {
      usbMIDI.sendNoteOn(74, 127, CHANNEL); // send note off
    }

    push_button_2.update();

    if (push_button_2.fell() == true) {
      usbMIDI.sendNoteOn(75, 0, CHANNEL); // send note on
    }

    if (push_button_2.rose() == true) {
      usbMIDI.sendNoteOn(75, 127, CHANNEL); // send note off
    }

    prevReadTime = t;
    digitalWrite(LED, ++heart & 32); // Blink = alive

  }
  while (usbMIDI.read()); // Discard incoming MIDI messages
}

