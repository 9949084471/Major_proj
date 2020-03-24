MAJOR_PROJECT-IOT_STREET_LIGHT
AIM: To create our own IOT use case and design an architecture to solve our usecase and also controlling through the thinkspeak in smartphone.

INTRODUCTION ABOUT PROJECT : Most of the places have automatic street light which can sense the daytime and nighttime, and automatically turns on and off according the night and day.so,in this project by adding one more constraint to turn on the light that is Street light will only glow if there is darkness and someone is passing through the street. ThingSpeak is a cloud based data platform which is used to send and receive the data in real time using HTTP protocol. It is used in Iot application to store and monitor the data from anywhere in the world over internet.

MAIN_THEME : The main objective of this project is to reduce the power consumption by glowing the Street light only when it is needed. In this project we are demonstrating the prototype of the Smart Street Light with 3 IR sensors, 1 LDR sensor and 3 LEDs - each representing one street light. We will also update the LDR sensor data to the ThingSpeak and control the LEDs (Street lights) over the internet from anywhere in the world

COMPONENTS REQUIRED: 1.ESP8266 NodeMCU 2.Micro USB cable 3.LEDs(3) 4.Jumper wires 5.IR sensors(3) 6.LDR sensor(1)

PROCEDURE: The circuit mainly consists ESP8266, LDR sensor, IR sensors and LEDs.

Here the LDR sensor is used to detect whether it is daytime or night time. Since LDR sensor generates variable resistance based on the amount of light falling on it, it has to be connected like a potentiometer. One end of the LDR sensor is connected to 5V and other end is connected to fixed resistance which is further connected to ground. NodeMCU has one ADC pin (A0) which is connected to point between fixed resistance and one end of the LDR sensor as shown in the circuit diagram. Since the LDR sensor gives variable resistance therefore variable voltage will be generated at A0 according to the amount of light falling on LDR.

IR sensors are used to detect if someone is crossing the street or not. It detects the obstacle or motion in the surrounding. The transmitter will transmit IR rays which will be reflected back if it falls on some object like person, animal, vehicles, etc. The reflected ray will be received by receiver diode and hence will confirm the presence of object and the corresponding LED will be glowed. This method will save significant amount of electricity as the street light will only turns on if there is someone present in the Street. IR sensor has 3 pins, two of which are VCC and ground and one is output pin. The output of IR sensor gets high if detects presence of some object. This pin is connected to GPIO pin of NodeMCU so whenever the IR sensor detects someone passing through the street it triggers the Street light. In our case one LED will be turned on. Now we have to control the LEDs over the internet using ThingSpeak. Click on Sharing and select the “Share channel view with everyone” radio button.Now,go to API keys and copy the URL given in “Update a Channel Feed”. We have to edit this URL to change the status of LED.Here we are setting field 5, field 6 and field 7 as 1 to turn on the LEDs. Copy this URL and paste it in a new tab. It will turn on the LEDs with some delay time. You can observe the change in field charts.

CODE EXPLANATION :

//First include all the required libraries. #include <ESP8266WiFi.h>; #include <WiFiClient.h>; #include <ThingSpeak.h>;

//Replace SSID and password given in code with you Wi-Fi SSID and password. const char* ssid = "Smart_street_light"; const char* password = "qwerty123@@";

//initializing the Client. WiFiClient client;

//Copy channel number, read and write API keys from ThingSpeak as shown above unsigned long myChannelNumber = 795820; const char * myWriteAPIKey = "FZTOUARV558GRZ8J"; const char * myReadAPIKey = "T52GT3QQOQBVPG4V";

//Define variable for GPIO pins of leds and IR sensors, ADC channel int led_1; int led_2; int led_3;

int ir1 = D0; int led1 = D5;

int ir2 = D1; int led2 = D6;

int ir3 = D2; int led3 = D7;

int ldr = A0; int val =0;

void setup() { Serial.begin(9600); delay(10);

//Set the pinMode for pins of led and IR sensor on the NodeMCU. pinMode(ir1,INPUT); pinMode(led1,OUTPUT);

pinMode(ir2,INPUT); pinMode(led2,OUTPUT);

pinMode(ir3,INPUT); pinMode(led3,OUTPUT); //Initialization of Wi-Fi and ThingSpeak WiFi.begin(ssid, password); ThingSpeak.begin(client); }

void loop() {

//Now we take digital value of the IR sensors and analog value of LDR sensor and store them in variables. int s1 = digitalRead(ir1); int s2 = digitalRead(ir2); int s3 = digitalRead(ir3); s3 = not(s3);

val = analogRead(ldr);

Serial.print(s1); Serial.print(":"); Serial.print(s2); Serial.print(":"); Serial.print(s3); Serial.print(" "); Serial.println(val);

//Now check the value of LDR sensor for low light. Here I have set value as 800 means if the analog value of LDR is lower than 700 then it will be night time or low light and hence it will turn on the led if IR sensors detect some obstacle or motion. If the analog value of the LDR sensor is more than 700 then it will be considered as daytime and LEDs will not glow even if IR sensor detects someone passing the street. if(val<800) { if(s1==0) { digitalWrite(led1,LOW); } else { digitalWrite(led1,HIGH); } if(s2==0) { digitalWrite(led2,LOW); } else { digitalWrite(led2,HIGH); } if(s3==0) { digitalWrite(led3,LOW); } else { digitalWrite(led3,HIGH); } } else { digitalWrite(led1,LOW); digitalWrite(led2,LOW); digitalWrite(led3,LOW); }

//Finally upload the data on the ThingSpeak cloud by using function ThingSpeak.writeField(). It take channel number, field number, data (you want to upload in respective field) and write API key. Here we are uploading LDR sensor data, IR sensors data and LEDs data to the ThingSpeak cloud. ThingSpeak.writeField(myChannelNumber, 1,val, myWriteAPIKey); ThingSpeak.writeField(myChannelNumber, 2,s1, myWriteAPIKey); ThingSpeak.writeField(myChannelNumber, 3,s2, myWriteAPIKey); ThingSpeak.writeField(myChannelNumber, 4,s3, myWriteAPIKey); ThingSpeak.writeField(myChannelNumber, 5,led1, myWriteAPIKey); ThingSpeak.writeField(myChannelNumber, 6,led2, myWriteAPIKey); ThingSpeak.writeField(myChannelNumber, 7,led3, myWriteAPIKey);

//Here is the code for changing the state of LEDs using ThingSpeak. We have already shown above the procedure to change the state of the LEDs. Led_1, led_2, led_3 stores the last state of led from the ThingSpeak using the function ThingSpeak.readIntField which takes channel number, respective field number and read API key. If the state of some led is “1” then we turn on the respective led and if the state of some led is “0” we turn off the respective led. led_1 = ThingSpeak.readIntField(myChannelNumber, 5, myReadAPIKey); led_2 = ThingSpeak.readIntField(myChannelNumber, 6, myReadAPIKey); led_3 = ThingSpeak.readIntField(myChannelNumber, 7, myReadAPIKey);

if(led_1==1) { digitalWrite(led1,HIGH); } else { digitalWrite(led1,LOW); }

if(led_2==1) { digitalWrite(led2,HIGH); } else { digitalWrite(led2,LOW); }

if(led_3==1) { digitalWrite(led3,HIGH); } else { digitalWrite(led3,LOW); } }

This is how a Smart Street Light works, it only glows if it is a night time and someone is passing through the street. And it can be also controlled manually from anywhere in the world using ThingSpeak IoT cloud.
