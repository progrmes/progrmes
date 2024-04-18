#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x3F,16,2); 
const int redl=4;
const int greenl=5;
const int fires=6;
const int smoks=7;
const int trigPin = 8;
const int echoPin = 9;
const int refire=10;
const int refull=11;
const int reswim=12;
int travelTime;
float distance;
char state='f';
unsigned long fm=0;
unsigned long sm=0;
unsigned long lm=0;
unsigned long hm=0;
void times(unsigned long mill, String message, byte din, byte ln) {
  int longtime = (millis() - mill) / 500;
  if (longtime % 2 == 0) {
    lcd.setCursor(din, ln);
    lcd.print(message);
    }else{
    lcd.setCursor(din, ln);
    lcd.print("     ");
    }
  } 
void sendSoundWave() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
}
void setup() {
  lcd.init();
  lcd.backlight();
  pinMode(redl, OUTPUT);
  pinMode(greenl, OUTPUT);
  pinMode(fires, INPUT);
  pinMode(smoks, INPUT);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(refire, OUTPUT);
  pinMode(refull, OUTPUT);
  pinMode(reswim, OUTPUT);
}
void loop() {
  sendSoundWave();               
  travelTime = pulseIn(echoPin, HIGH);   
  distance = travelTime * 0.034 / 2;     
  bool fire=digitalRead(fires);
  bool smok=digitalRead(smoks);
  if (state=="f" && fire==0 && smok==1){
   fm=0;
   sm=0;
   hm=0;
   lm=0;
   lcd.setCursor(6, 0);
   lcd.print("save");
   lcd.setCursor(0, 1);    
   lcd.print(distance);
   digitalWrite(greenl,HIGH);
   digitalWrite(redl,LOW);
   digitalWrite(refire,HIGH);
   digitalWrite(refull,HIGH);
   digitalWrite(reswim,HIGH);
   if (distance<3){
     state='h';
    }else if (distance>10){
     state='l';
     }
  }else{
    digitalWrite(redl,HIGH);
    digitalWrite(greenl,LOW);
    if (fire==1){
     if (fm==0){
       fm=millis();
       sm=0;
      }
     digitalWrite(refire,LOW);
     times(fm,"Fire",6,0); 
    }else if (smok==0){
      if (sm==0){
       sm=millis();
       fm=0;} 
     digitalWrite(refire,HIGH);
     times(sm,"Smok",6,0);
     }else{
     digitalWrite(refire,HIGH);
     sm=0;
     fm=0; 
     }if (state=='f'){
     hm=0;
     lm=0;
     lcd.setCursor(0, 6);
     lcd.print(distance);
     digitalWrite(refull,HIGH);
     digitalWrite(reswim,HIGH);
     if (distance<3){
       state='h';
      }else if (distance>10){
       state='l';
      }
     }else if (state=='h'){    
     if(hm==0){
       hm=millis();
       lm=0;}
     times(hm,"High water",2,1);
     if (fire==0){
       digitalWrite(reswim,LOW);}
     if (distance>3){
       state='f';
       } 
      }else if (state=='l'){
     if(lm==0){
       lm=millis();
       hm=0;
      }
     times(lm,"Low water",2,1);
     digitalWrite(refull,LOW);
     if (distance<=4){
       state='f';
      }
    }
  }
  delay(100);
}
