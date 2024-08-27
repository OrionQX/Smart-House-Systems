# Smart-House-Systems
Smart House Systems
#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <ThreeWire.h>
#include <RtcDS1302.h>

#define MINUTE 60000
#define SECOND 1000

LiquidCrystal_I2C lcd(0x27, 16, 2);

ThreeWire myWire(12, 11, 13);        // DAT, CLK, RST
RtcDS1302<ThreeWire> Rtc(myWire);    // RTC Object

Servo servo1; 

int sensorPin = A1;                
int esikDegeri = 100;              
int buzzerPin = 10;                
int veri;                          

int LDR = A0;

int trigger = 7;
int echo = 6;

int pirpin = 5;
int pirVal;
int lastPirVal = LOW;
unsigned long myTime;
char printBuffer[128];

int led1 = 3; 
int led2 = 4;
int led3 = 2;
int sound = 250;

void setup()
{
  lcd.init();
  lcd.backlight();
  lcd.clear();

  Rtc.Begin();
  
  pinMode(buzzerPin, OUTPUT);  
  pinMode(echo, INPUT );
  pinMode(trigger, OUTPUT); 
  pinMode(led1, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
  pinMode(pirpin, INPUT);
  servo1.attach(9);
  
  Serial.begin(9600);
}

void loop() {
  //----------------------------------- time and date
  RtcDateTime now = Rtc.GetDateTime();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Date: ");
  lcd.print(now.Day());
  lcd.print("/");
  lcd.print(now.Month());
  lcd.print("/");
  lcd.print(now.Year());

  lcd.setCursor(0, 1);
  lcd.print("Time: ");
  lcd.print(now.Hour());
  lcd.print(":");
  lcd.print(now.Minute());
  lcd.print(":");
  lcd.print(now.Second());

  delay(1000); // Update every second
  
  //----------------------------------- Rain Sensor
  veri = analogRead(sensorPin);    
  if(veri > esikDegeri){           
    digitalWrite(buzzerPin, HIGH); 
    delay(10);
    digitalWrite(buzzerPin, LOW);
    delay(10);
  }
  else{                            
    digitalWrite(buzzerPin, LOW);}
  
  //----------------------------------- LCD Display (Additional Information)
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Weather: 32C ");
  lcd.setCursor(0, 1);
  lcd.print("Air pressure:998 ");
  delay(1000);
  lcd.clear();
  
  
  //----------------------------------- Ultrasonic with Servo 
  long duration, distance;
  digitalWrite(trigger, LOW);
  delayMicroseconds(2);
  digitalWrite(trigger, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigger, LOW);
  duration = pulseIn(echo, HIGH);
  distance = duration * 0.034 / 2;

  if (distance > 10){
    servo1.write(90);
  }
  else if(distance <= 10){
    servo1.write(180);
    delay(3300);
    servo1.write(90);
    delay(1000);
    servo1.write(0);  // Changed to 0 degrees to avoid invalid position
    delay(1000);
  }
  
  //----------------------------------- PIR with LED 
  pirVal = digitalRead(pirpin);  // read current input value

  if (pirVal == HIGH) { // movement detected  
    digitalWrite(led2, HIGH);  // turn LED on

    if (lastPirVal == LOW) { // if there was NO movement before
      myTime = millis();
      sprintf(printBuffer, "%lu min %lu sec: Motion detected!", myTime/MINUTE, myTime%MINUTE/SECOND); 
      Serial.println(printBuffer);
      lastPirVal = HIGH;
    }
  } else { // no movement detected
    digitalWrite(led2, LOW); // turn LED off

  if (lastPirVal == HIGH){ // if there was a movement before
      myTime = millis();
      sprintf(printBuffer, "%lu min %lu sec: Motion ended!", myTime/MINUTE, myTime%MINUTE/SECOND); 
      Serial.println(printBuffer);
      lastPirVal = LOW;
    }
  }
  
  //----------------------------------- LDR with LED 
  int isik = analogRead(A0);
  Serial.println(isik);
  delay(50);

  if(isik > 2){
    digitalWrite(led1, LOW);
  }
  else {
    digitalWrite(led1, HIGH);
  }
}
