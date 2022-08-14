# Hidonergy - Módulo wifi
Aquí estarán el código utilizado para el proyecto "Hidronergy", en este caso es el código del Módulo wifi

#include <ESP8266WiFi.h>

int encendido = 0; //guarda el estado del botón de encendido
int limpiarElectrodo = 0; //guarda el estado del boton de limpieza de electrodos

//Pines para enviar señales al arduino
int sigRelay = 0; //GPIO0 D3 pin del WEMOS D1 MINI
int sigServo = 2; //GPIO2 D4 pin del WEMOS D1 MINI

float minSinc = 0; //Guarda el valor mínimo promedio de 100 lecturas del sensor de gas

//Datos del WIFI del telefono
const char* ssid = "hidronergy";//your-ssid
const char* password = "123456789";//your-password


//Crea un servidor WEB en el puerto 80 (http)
WiFiServer server(80);

void setup()
{
  Serial.begin(9600); //velocidad de datos para el Serial Monitor

  //Sincronizar Sensor
  //*********************************************************//
  //Realiza 100 lecturas del sensor de gas, una cada 200 milisegundos
  //suma las lecturas y saca el promedio, que guarda en minSinc
  //minSinc se tomará como la lectura mínima del sensor de gas
  int i = 0;
  float gasSensor = 0;
  float totalSinc = 0;
  Serial.println("***");
  Serial.println("***");
  Serial.println("Sincronizando sensor de gas");
  for(i = 0; i < 100; i++){
    Serial.print(".");
    gasSensor = analogRead(A0);
    totalSinc = totalSinc + gasSensor;
    delay(200);
  }
  minSinc = totalSinc / 100;
  Serial.println("");
  Serial.print("Sensor de gas sincronizado, lectura minima promedio: ");
  Serial.println(minSinc);
  //*********************************************************//

  //define el pin sigRelay y sigServo como salida de datos
  pinMode(sigRelay, OUTPUT);
  digitalWrite(sigRelay, LOW); //setea a sigRelay con 0
  pinMode(sigServo, OUTPUT);
  digitalWrite(sigServo, LOW); //setea a sigServo con 0

//trata de conectarse al wifi del telefono
  Serial.println("-------------------------------------------------------");
  Serial.print("Conectando al WIFI");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi Conectado");  
 //Inicializa el WebServer y muestra información de la IP en el Monitor Serial 
  server.begin();  
  Serial.println("Servidor inicializado");
  Serial.println("-------------------------------------------------------");
  Serial.print("Direccion IP del servidor: "); // Prints IP address on Serial Monitor
  Serial.println(WiFi.localIP());
  Serial.print("Para Acceder a la pagina web use: http://");
  Serial.print(WiFi.localIP());
  Serial.println("-------------------------------------------------------");
  Serial.println("/");


  
  
}

void loop()
{
  //Busca clientes web para desplegar la página web
  WiFiClient client = server.available();
  if (!client)
  {
    return;
  }
  Serial.println("Esperando conexión...");
  while(!client.available())
  {
    delay(1);
  }

  String request = client.readStringUntil('\r');
  Serial.println(request);
  client.flush();

//Verifica instrucciones enviadas desde la pagina web  

//Enciende proceso de electrólisis y monitorea el sensor de gas
  if(request.indexOf("/RELAYON") != -1)
  {
    digitalWrite(sigRelay, HIGH); // Enciende el RELAY
    digitalWrite(sigServo, LOW); // Detiene el servo
    encendido = 1;
    limpiarElectrodo = 0;
  }else if(request.indexOf("/RELAYOFF") != -1) //Detiene electrólisis y monitoreo
  {
    digitalWrite(sigRelay, LOW); // Cierra el relay
    digitalWrite(sigServo, LOW); // Detiene el servo
    encendido = 0;
    limpiarElectrodo = 0;
  }

//Apaga el proceso de electrólisis y mueve el servo para limpiar electrodos
  if(request.indexOf("/CLEAN") != -1)
  {
    digitalWrite(sigRelay, LOW); // Cierra el relay
    digitalWrite(sigServo, HIGH); // Enciende el servo
    encendido = 0;
    limpiarElectrodo = 1;
//    delay(500);
//    digitalWrite(sigRelay, LOW); // Cierra el relay
//    digitalWrite(sigServo, LOW); // Detiene el servo
//    encendido = 0;
//    limpiarElectrodo = 0;
  }
//hace lectura del sensor de gas
  float gasSensor = analogRead(A0);
  Serial.print(gasSensor);
  Serial.println(" - ");
  float lecturaSinc = gasSensor - minSinc;
  if (lecturaSinc < 0){
    lecturaSinc = 0;
  }
  int avance = 0;
  avance = lecturaSinc * 100 / (1024 - minSinc);    //toma minSinc como el 0%, por regla de 3 calcula el porcentaje de la lectura del sensor
                                                    //1024 es la lectura máxima del sensor, le resta el minimo minSinc y se optiene el valor del 100%
  
  
  client.println("HTTP/1.1 200 OK"); // standalone web server with an ESP8266
  client.println("Content-Type: text/html");
  client.println("");
  client.println("<!DOCTYPE HTML> <html> <head> <title>HIDRONERGY MONITOR</title> <style type='text/css'> body{ background-color:#293d69; color:#ffffff; font-family: Arial, Helvetica, sans-serif; font-size:30px; line-height:1.6em; margin:0; } .container{ width:80%; margin:auto; overflow:hidden; text-align:center; } #main-header { background-color:rgb(65, 138, 216); color:#fff; } .button1{ border:1px #fff solid;  background-color:red; color:#000; padding:60px 60px; margin-top:10px; border-radius:50%; font-size:50px; } .button2{ border:1px #fff solid; background-color:gray;  color:#000; padding:60px 50px; margin-top:10px; border-radius:50%; font-size:50px; } .button3{ border:1px #fff solid; background-color:#9be2d4;  color:#000; padding:60px 50px; margin-top:10px; border-radius:50%; font-size:50px; } .sigRelay{ border:1px #fff solid; background-color:#000; color:#fff; padding:80px 80px; margin-top:10px; border-radius:50%; } .meter { box-sizing: content-box; height: 100px; /* Can be anything */ position: relative; margin: 60px 0 20px 0; /* Just for demo spacing */ background: #555; border-radius: 25px; padding: 10px; box-shadow: inset 0 -1px 1px rgba(255, 255, 255, 0.3); } .meter > span { display: block; height: 100%; border-top-right-radius: 8px; border-bottom-right-radius: 8px; border-top-left-radius: 20px; border-bottom-left-radius: 20px; background-color: rgb(43, 194, 83); background-image: linear-gradient( center bottom, rgb(43, 194, 83) 37%, rgb(84, 240, 84) 69% ); box-shadow: inset 0 2px 9px rgba(255, 255, 255, 0.3), inset 0 -2px 6px rgba(0, 0, 0, 0.4); position: relative; overflow: hidden; } .meter > span:after, .animate > span > span { content: ""; position: absolute; top: 0; left: 0; bottom: 0; right: 0; background-image: linear-gradient( -45deg, rgba(255, 255, 255, 0.2) 25%, transparent 25%, transparent 50%, rgba(255, 255, 255, 0.2) 50%, rgba(255, 255, 255, 0.2) 75%, transparent 75%, transparent ); z-index: 1; background-size: 50px 50px; animation: move 2s linear infinite; border-top-right-radius: 8px; border-bottom-right-radius: 8px; border-top-left-radius: 20px; border-bottom-left-radius: 20px; overflow: hidden; }  .animate > span:after { display: none; }  @keyframes move { 0% { background-position: 0 0; } 100% { background-position: 50px 50px; } } .orange > span { background-image: linear-gradient(#f1a165, #f36d0a); }  .red > span { background-image: linear-gradient(#f0a3a3, #f42323); }  .nostripes > span > span, .nostripes > span::after { background-image: none; }  .dots-bars-7 { width: 40px; height: 80px; --c: linear-gradient(currentColor 0 0); background:  var(--c) 0  0, var(--c) 0  100%,  var(--c) 50%  0, var(--c) 50%  100%,  var(--c) 100% 0,  var(--c) 100% 100%; background-size: 8px 50%; background-repeat: no-repeat; animation: db7-0 1s infinite; position: relative; overflow: hidden; } .dots-bars-7:before { content: ""; position: absolute; width: 8px; height: 8px; border-radius: 50%; background: currentColor; top:calc(50% - 4px); left:-8px; animation:inherit; animation-name:db7-1; }  @keyframes db7-0 {  16.67% {background-size:8px 30%, 8px 30%, 8px 50%, 8px 50%, 8px 50%, 8px 50%}  33.33% {background-size:8px 30%, 8px 30%, 8px 30%, 8px 30%, 8px 50%, 8px 50%}  50%  {background-size:8px 30%, 8px 30%, 8px 30%, 8px 30%, 8px 30%, 8px 30%}  66.67% {background-size:8px 50%, 8px 50%, 8px 30%, 8px 30%, 8px 30%, 8px 30%}  83.33% {background-size:8px 50%, 8px 50%, 8px 50%, 8px 50%, 8px 30%, 8px 30%} }  @keyframes db7-1 {  20%  {left:0px}  40%  {left:calc(50%  - 4px)}  60%  {left:calc(100% - 8px)}  80%,  100% {left:100%} }  @media(max-width:600px){  #main-header{ width:100%; } .container{ width:auto; } .sigRelay{ width:auto; } } </style> <meta http-equiv='refresh' content='5'> </head> <body>");
  client.println("<header id='main-header'> <div class='container'> <h1>HIDRONERGY</h1> <h3>MONITOR</h3> </div> </header>");
  client.println("<div class='container'> <a href='/RELAYON'><button class='button1'>ON</button></a> <a href='/RELAYOFF'><button class='button2'>OFF</button></a> <a href='/CLEAN'><button class='button3'>CLEAN</button></a></div>");
  //verifica si se debe desplegar animacion del proceso de producción de hidrógeno.
  if (encendido == 1){;
    client.println("<div class='container'> <h4 style='color: rgb(105, 184, 105);'>GENERANDO HIDROGENO...</h4> <center><div class='dots-bars-7'></div></center> </div>");
  }
  if (limpiarElectrodo == 1){;
    client.println("<div class='container' style='border: 5px solid red;'> <h4 style='color: rgb(105, 184, 105);'>LIMPIEZA DE ELECTRODOS...</h4> <center><div class='dots-bars-7'></div></center> </div>");
  }
  client.print("<div class='container'> <h4>Total Almacenado:</h4> <div class='meter red'>   <span style='width:");
  //verifica si se debe desplegar barra de progreso del proceso de producción de hidrógeno.
  if (encendido == 1){
     client.print(avance);
  }else{
     client.print(0);
  }
  client.println("%'></span> </div> <h4>Total Almacenado:</h4> <div class='meter'>   <span style='width: 0%'></span> </div> <h4>Total Almacenado:</h4> <div class='meter'>   <span style='width: 0%'></span> </div> <h4>Total Almacenado:</h4> <div class='meter'>   <span style='width: 0%'></span> </div> <h4>Total Almacenado:</h4> <div class='meter'>   <span style='width: 0%'></span> </div> </div> </body> </html>");
  delay(1);
  Serial.println("Cliente desconectado...");
  Serial.println("");
}
