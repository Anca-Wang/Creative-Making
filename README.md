# Creative-Making  
Final code in arduino:  

#include <Wire.h>  
#include "Adafruit_MPR121.h"  
#include <Servo.h>  
#ifndef _BV  
#define _BV(bit) (1 << (bit))   
#endif  

Servo myservo; 
Adafruit_MPR121 cap = Adafruit_MPR121();  

int pos = 0;  
int brightness = 0;    
int fadeAmount = 5;  
uint16_t lasttouched = 0;  
uint16_t currtouched = 0;  


void setup() {  
  Serial.begin(9600);  
  pinMode(13, OUTPUT);  
  pinMode(5, OUTPUT);  
  myservo.attach(9);  
  
  while (!Serial) {  
    delay(10);  
  }  
  
  Serial.println("Adafruit MPR121 Capacitive Touch sensor test");   
  
  if (!cap.begin(0x5A)) {  
    Serial.println("MPR121 not found, check wiring?");  
    while (1);  
  }  
  Serial.println("MPR121 found!");  
}  

void loop() {  
  currtouched = cap.touched();  
  analogWrite(13, brightness);  
  brightness = brightness + fadeAmount;  
     
  for (uint8_t i=0; i<12; i++) {  
    // it if *is* touched and *wasnt* touched before, alert!  
    if ((currtouched & _BV(i)) && !(lasttouched & _BV(i)) ) {  
      Serial.print(i); Serial.println(" touched");  
      for (pos = 0; pos <= 90; pos += 0.5) {   
        myservo.write(pos);              
        delay(100);                    
      }  
      for (pos = 90; pos >= 0; pos -= 0.5) {   
        myservo.write(pos);              
        delay(100);           
      }  
    }  
    // if it *was* touched and now *isnt*, alert!  
    if (!(currtouched & _BV(i)) && (lasttouched & _BV(i)) ) {  
      Serial.print(i); Serial.println(" released");  
    }  
  }  

  
  if (analogRead(A0) < 1020){  
    analogWrite(5, brightness);  
    brightness = brightness + fadeAmount;  
    if (brightness <= 0 || brightness >= 255) {  
      fadeAmount = -fadeAmount;  
    }  
    delay(100);  
    }  

    if (analogRead (A0) > 1020){  
      digitalWrite(11, LOW);  
    }  


  lasttouched = currtouched;  
  return;  
  Serial.print("\t\t\t\t\t\t\t\t\t\t\t\t\t 0x"); Serial.println(cap.touched(), HEX);  
  Serial.print("Filt: ");  
  
  for (uint8_t i=0; i<12; i++) {  
    Serial.print(cap.filteredData(i)); Serial.print("\t");  
  }  
  
  Serial.println();  
  Serial.print("Base: ");  
  
  for (uint8_t i=0; i<12; i++) {  
    Serial.print(cap.baselineData(i)); Serial.print("\t");  
  }  
  
  Serial.println();  
  
  delay(100);  
}  
