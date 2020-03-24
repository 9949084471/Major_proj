//SecurityAlarm(online controllable)
AIM: To create our own IOT use case and design an architecture to solve our usecase(here online controllable smart bulb) and also controlling through the thinkspeak website

Introduction: many times robbery happen during night when lights are off.this project has LDR with which we can code microcontroller to ON buzzer(security alarm) only when lights are off and also only when a person(theif) is detected near the sensitive zone of house with data sensed by IR sensors.ThingSpeak is a cloud based data platform which is used to send and receive the data in real time using HTTP protocol. It is a platform used to monitor and control devices virtually through internet.this project can help us by alerting us when some robbery gonna happen.



COMPONENTS REQUIRED: 1.ESP8266 (NodeMCU) 2.Micro USB cable(generally used for mobile charging) 3.Jumper wires 4.IR sensors(2) 5.LDR sensor(1) ,6.BUZZER(2),7.resistor 300ohms(1).

WORKING:

Here the LDR sensor is used to detect whether it's dark or not. Since LDR sensor generates variable resistance based on the amount of light falling on it, it has to be connected like a potentiometer. One end of the LDR sensor is connected to 5V and other end is connected to fixed resistance(~300 ohms) which is further connected to ground. NodeMCU has one ADC pin (A0) which is connected to point between fixed resistance and one end of the LDR sensor. Since the LDR sensor gives variable resistance therefore variable voltage will be generated at A0 according to the intensity of light falling on LDR.

IR sensors are used to detect if someone is crossing the setup(smart bulb) or not. It detects the obstacle or motion in the surrounding. The transmitter(looks to be black in color) will transmit IR rays which will be reflected back if it falls on some object. The reflected ray will be received by receiver diode(transparent one) and hence will confirm the presence of object and the corresponding buzzer will be beeped.

IR sensor has 3 pins, two of which are VCC and ground and one is output pin. The output of IR sensor gets high if detects presence of some object. This pin is connected to GPIO pin of NodeMCU so whenever the IR sensor detects someone passing through the street it triggers the buzzer. In our case one BUZZER will be turned on. Now we have to control the buzzers over the internet using ThingSpeak. Click on Sharing and select the “Share channel view with everyone” option.Now,go to API keys and copy the URL given in “Update a Channel Feed”. We have to edit this URL to change the status of buzzer.Here we are setting field 5, field 6  as 1 to turn on the buzzers. Copy this URL and paste it in a new tab. It will turn on the buzzers with some delay time. You can observe the change in field charts.

Buzzer1's positive terminal is connected to D3 and negative terminal to ground,Buzzer2's +ve terminal to D4,-ve terminal to ground 

CODE :

//First include all the required libraries.
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ThingSpeak.h>

//Replace SSID and password given in code with you Wi-Fi SSID and password.

const char* ssid = "Security Alarm"; 
const char* password = "OY BULLABBAY";

//initializing the Client.
WiFiClient client;

//Copy channel number, read and write API keys from ThingSpeak as shown above

unsigned long myChannelNumber =1025299;
const char * myWriteAPIKey = "YTNEG3CX5MLLYCD0"; 
const char * myReadAPIKey = "VNLU7IV3BHK7IUJ9";

//Define variable for GPIO pins of buzzers and IR sensors, ADC channel
int buzzer_2; int buzzer_1;
int ir1 = D0;int buzzer1=D3;
int ir2 = D1; int buzzer2=D4;
int ldr = A0;
int val =0;

void setup() { 
Serial.begin(9600);

//Set the pinMode for pins of led and IR sensor on the NodeMCU.
pinMode(ir1,INPUT); pinMode(buzzer1,OUTPUT);

pinMode(ir2,INPUT); pinMode(buzzer2,OUTPUT);

//Initialization of Wi-Fi and ThingSpeak 

WiFi.begin(ssid, password);
ThingSpeak.begin(client); }

void loop() {

//Now we take digital value of the IR sensors and analog value of LDR sensor and store them in variables. 
int s1 = digitalRead(ir1); 
int s2 = digitalRead(ir2); 

val = analogRead(ldr);

Serial.println("IR 1 value:");
Serial.print(s1);//prints ir sensor value
Serial.println("IR 2 value:");
Serial.print(s2); 
Serial.println("LDR value:"); 
Serial.print(val);//prints ldr value
Serial.println("");
Serial.println("");
//Now check the value of LDR sensor for low light.(test case) Here I have set value as 200 means if the analog value of LDR is lower than 200 then it will be dark and hence it will turn on the buzzer iff IR sensors detect some obstacle or motion. If the analog value of the LDR sensor is more than 200 then it will be considered as room is bright and buzzer will not beep.

/*Note: consider that people living in that house doesn't have habit of sleep walk and they manually ON other light if required.also consider that thief don't know where the switchboard is. */

if(val<200) 
{ 
if(s1==0) 
noTone(buzzer1);
else 
tone(buzzer1,15000);//15KHz frequency sound will be produced

if(s2==0)  
noTone(buzzer2);
else 
tone(buzzer2,15000);//15KHz frequency sound will be produced
} 
else 
{ noTone(buzzer1);noTone(buzzer2);}

//Finally upload the data on the ThingSpeak cloud by using function ThingSpeak.writeField(). It take channel number, field number, data (you want to upload in respective field) and write API key. Here we are uploading LDR sensor data, IR sensors data and LEDs data to the ThingSpeak cloud. 

ThingSpeak.writeField(myChannelNumber, 7,val, myWriteAPIKey);
ThingSpeak.writeField(myChannelNumber, 3,s1, myWriteAPIKey);
ThingSpeak.writeField(myChannelNumber, 4,s2, myWriteAPIKey); 
ThingSpeak.writeField(myChannelNumber, 5,buzzer2, myWriteAPIKey);
ThingSpeak.writeField(myChannelNumber, 6,buzzer1, myWriteAPIKey);

//Here is the code for changing the state of buzzers using ThingSpeak. We have already shown above the procedure to change the state of the buzzers. buzzer_1, buzzer_2 stores the last state of buzzers from the ThingSpeak using the function ThingSpeak.readIntField which takes channel number, respective field number and read API key. If the state of some buzzer is “15000” then we turn on the respective buzzer and else we turn off the respective buzzer.

buzzer_1 = ThingSpeak.readIntField(myChannelNumber, 5, myReadAPIKey); 
buzzer_2= ThingSpeak.readIntField(myChannelNumber, 6, myReadAPIKey);

if(buzzer_1==15000) { tone(buzzer1,15000); }
else { noTone(buzzer1); }

if(buzzer_2==15000) {tone(buzzer2,15000); }
else { noTone(buzzer2); }

}


This is how a SecurityAlarm works, it only beeps if it is dark and someone is detected by ir sensor. And it can be also controlled manually from anywhere in the world using ThingSpeak IoT cloud.
