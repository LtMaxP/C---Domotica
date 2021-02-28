# C---Domotica
Código proyecto de casa inteligente, con sensores de proximidad, accionares y música adaptada

#include <SD.h>  
#include <SPI.h>
#include <TMRpcm.h>  
#include <Servo.h>

#define trigPin 7
#define echoPin 6
#define pinSD 10     //define el pin para seleccionar la tarjeta SD
TMRpcm tmrpcm;   //Se crea un objeto de la librería TMRpcm
Servo motorServo;

int cancion = 1;
int actual = 0;
/*
int boton1 = 0; //encendido o apagado
int estadoBoton1 = 0;
*/
int servo = 3;
int rotacion = 0;
long duracion, distancia;
int Limite = 20 ;  //Limite decidido para la llegada del dueño
int estadoCasa = 0;

void setup(){
  tmrpcm.speakerPin = 9; //Pin conectado al parlante
  Serial.begin(9600);    //Se inicia la comunicación serial
  tmrpcm.quality(1); //mejora la calidad de sonido
  tmrpcm.setVolume(5); // volumen +5 satura
  
  if (!SD.begin(pinSD)) {  // see if the card is present and can be initialized:
    Serial.println("Fallo en la tarjeta SD");  //Aviso de que algo no anda bien 
    return;   //No hacer nada si no se pudo leer la tarjeta
  }
  else
  {Serial.println("Legible"); }

  pinMode(4, OUTPUT); //luces
  pinMode(8, OUTPUT); //Ventilador
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT); 
  motorServo.attach(servo); /*3*/ 
  motorServo.write(200);  //Puerta cerrada inicialmente

}

void loop()
{  
      
      /*
       * Posible modificación de canción  ++Siguiente canción en la lista de la memoria SD
      boton1 = digitalRead(2);
      Serial.println("Boton 1= " + boton1);  //Imprime algo :)
      
      if(boton1 == 0)
      {
        if(cancion <= 2)
        {
          cancion++;
        }
        else
        {
          cancion=1;
        }
      }
      */
      
      distancia = carEcho();
      
      if ( distancia < Limite  && distancia != 0)
      {
      
        Serial.println("Estado de la Casa: " + estadoCasa);
        if(estadoCasa == 0)
        {
          estadoCasa = 1;
          motorServo.write(100); //Se Abre la puerta
          delay(1000);
          /*Luces y ventilador encendidp*/
          digitalWrite(4,HIGH);
          digitalWrite(8,HIGH);
           Serial.println("Canon");  /*Imprime algo */
           tmrpcm.play("Canon.wav", 12);  //Se inicia la música deseada 3 temas insertados --- Canon - Prelude - paw
              
          delay(4000); /*Esperara 6seg a q pase*/
        }
         
           else
           {
              estadoCasa = 0;
              delay(500);
              motorServo.write(200);  //Cerrar puerta
              delay(500);
              tmrpcm.pause();   //Pausar tema
              digitalWrite(4, LOW);  //Apagado de luces
              digitalWrite(8, LOW); //Apagado parlante
              delay(500);
           }
       
       }
       
}

 int carEcho()
 {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);   
  delayMicroseconds(10); 
  digitalWrite(trigPin, LOW);      
  duracion = pulseIn(echoPin, HIGH) ;
  distancia = duracion / 2 / 29.1  ;
  Serial.println(String(distancia) + " cm.") ;
  return distancia;
 }
