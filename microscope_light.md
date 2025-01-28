# Microscope Light
![20220803_161819](https://github.com/user-attachments/assets/ba228f6b-4bde-4e06-a87d-0e783abf7bd9)

[![20220803_162149](https://github.com/user-attachments/assets/d0747c07-ddf7-4177-864c-5c5df055b39b)

![20221208_223124](https://github.com/user-attachments/assets/f0565149-12c1-42cb-af8d-9c2845052325)


![20250127_235409](https://github.com/user-attachments/assets/df8dfcc8-98b0-4e52-9894-ab4feb85e309)


![20250127_235421](https://github.com/user-attachments/assets/5c297428-be35-4283-8f59-9cdb56025829)

![20250127_235358](https://github.com/user-attachments/assets/c48b0a9b-c735-4e3b-93eb-32520c38be12)


![20250127_234451](https://github.com/user-attachments/assets/dce24d6e-e014-4762-8f87-170e6f4e8468)



Code if you want to try making one:

```cpp
#include <Wire.h>
#include "Adafruit_VL6180X.h"
#include "Adafruit_seesaw.h"

#define SEESAW_ADDR          0x36

Adafruit_VL6180X vl = Adafruit_VL6180X();
Adafruit_seesaw ss;

int idle_brightness = 6;
int microscope_brightness = 8;
int current_brightness = 0;
int setpoint = 0;
bool activated = false;
uint8_t counter = 0;

uint8_t range = 0;
uint8_t status = 0;

void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, HIGH); 

  Serial.begin(115200);

  Serial.println("Adafruit VL6180x test!");
  if (! vl.begin(&Wire)) {
    Serial.println("Failed to find sensor");
    while (1);
  }
  Serial.println("Sensor found!");

  if (! ss.begin(SEESAW_ADDR)) { // || ! sspixel.begin(SEESAW_ADDR)) {
    Serial.println("Couldn't find seesaw on default address");
    while(1);
  }
  Serial.println("seesaw started");

  Serial.println("Turning on interrupts");
  ss.enableEncoderInterrupt();

  pinMode(10, OUTPUT);
  analogWrite(10, idle_brightness);

  pinMode(11, OUTPUT);
  analogWrite(11, 0);

  float lux = vl.readLux(VL6180X_ALS_GAIN_5); //potential fix for occasional freeze
  Serial.println("Init done");
}

void loop() {
  if ((status == VL6180X_ERROR_NONE && range<140) || counter%10 == 0) {
    range = vl.readRange();
    status = vl.readRangeStatus();
  }
  counter ++;
  int32_t delta = ss.getEncoderDelta();

  //Serial.print("Range: "); Serial.println(range);

  if (activated) {
    if (!(status == VL6180X_ERROR_NONE && range<90)) {
      activated = false;
    } 

    if(delta != 0) {
      microscope_brightness -= delta;
      if (microscope_brightness > 16) {
        microscope_brightness = 16;
      }
      if (microscope_brightness < 2) {
        microscope_brightness = 2;
      }
    }

    setpoint = microscope_brightness;
    analogWrite(10, current_brightness*current_brightness-1);
    analogWrite(11, current_brightness*current_brightness-1);
    //Serial.println(current_brightness);
    digitalWrite(LED_BUILTIN, HIGH); 
  }
  if (!activated) {
    if (status == VL6180X_ERROR_NONE && range<86) {
      activated = true;
    }

    if(delta != 0) {
      idle_brightness -= delta;
      if (idle_brightness > 16) {
        idle_brightness = 16;
      }
      if (idle_brightness < 2) {
        idle_brightness = 2;
      }
    }

    setpoint = idle_brightness;
    analogWrite(10, idle_brightness*idle_brightness-1);
    analogWrite(11, 0);
    //Serial.println(idle_brightness);
    digitalWrite(LED_BUILTIN, LOW);
  }

  if (current_brightness < setpoint) { //smooth brightness ramping
    current_brightness+=1;
    if (current_brightness > setpoint) {
      current_brightness = setpoint;
    }
    if (current_brightness > 16) {
      current_brightness = 16;
    }
  }
  else if (current_brightness > setpoint) {
    current_brightness-=1;
    if (current_brightness < setpoint) {
      current_brightness = setpoint;
    }
    if (current_brightness < 2) {
      current_brightness = 1;
    }
  }

  if (current_brightness != setpoint) { //dynamic refresh rate
    delay(30);
  }
  else if (activated) {
    delay(20);
  }
  else {
    delay(100);
  }
}
```

[return to main page](index.md)
