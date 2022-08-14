# Hidonergy-Arduino
Aquí estarán el código utilizado para el proyecto "Hidronergy", hablando principalmente del codigo realizado en Arduino.

#include <Servo.h>

int sigRelay = 3; //Cable que trae señal del módulo WIFI WEMOS D1 MINI D3 //
int sigServo = 7; //Cable que trae señal del módulo WIFI WEMOS D1 MINI
int relay = 4; //Pin digital   0/1
int servo = 6; //se conecta a la señal del servo cable naranja  //PIN PWM valores de 0-255
Servo servoElectrodo;

void setup() {
  pinMode(sigRelay,INPUT);
  pinMode(relay,OUTPUT);
  pinMode(sigServo,INPUT);
  servoElectrodo.attach(servo);
}

void loop() {
  int encender = digitalRead(sigRelay);
  servoElectrodo.write(0);
  
  if (encender == 1){
    digitalWrite(relay,HIGH);
  }else{
    digitalWrite(relay,LOW);
  }

  int limpiar = digitalRead(sigServo);
  
  if (limpiar == 1){
    servoElectrodo.write(90);              // mueve el servo a 90 grados
    delay(1000);  
    servoElectrodo.write(0);              // mueve el servo a 0 grados
    delay(1000);  
  }

}
