# project-IOT

#include "UbidotsESPMQTT.h" //https://github.com/ubidots/ubidots-mqtt-esp

const int ACPin = A0;         //set arduino signal read pin
#define ACTectionRange 20;    //set Non-invasive AC Current Sensor tection range (5A,10A,20A)

#define VREF 3.27

#define TOKEN "BBFF-SYCgIVugFyS8602lzCs6t6T61O6Lgp" // Your Ubidots TOKEN
#define WIFINAME "Jallad" //Your SSID
#define WIFIPASS "talha123" // Your Wifi Pass

Ubidots client(TOKEN);

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}


float Rate_of_Current_Value_per_min = 0;
float Current_Value = 0;
float Power  = 0;
float amount_consumed = 0;



float readACCurrentValue()
{
  float ACCurrtntValue = 0;
  float peakVoltage = 0;
  float voltageVirtualValue = 0;  //Vrms
  for (int i = 0; i < 5; i++)
  {
    peakVoltage += analogRead(ACPin);   //read peak voltage
    delay(1);
  }
  peakVoltage = peakVoltage / 5;
  voltageVirtualValue = peakVoltage * 0.707;    //change the peak voltage to the Virtual Value of voltage

  /*The circuit is amplified by 2 times, so it is divided by 2.*/
  voltageVirtualValue = (voltageVirtualValue / 1024 * VREF ) / 2;

  ACCurrtntValue = voltageVirtualValue * ACTectionRange;

  return ACCurrtntValue;
}

void setup()
{
  // put your setup code here, to run once:
  client.ubidotsSetBroker("business.api.ubidots.com"); // Sets the broker properly for the business account
  client.setDebug(true); // Pass a true or false bool value to activate debug messages
  Serial.begin(115200);
  client.wifiConnection(WIFINAME, WIFIPASS);
  client.begin(callback);

}

void loop()
{

  for (int k = 0; k < 60 ; k++)
  {

    float ACCurrentValue = readACCurrentValue(); //read AC Current Value
    Serial.print(ACCurrentValue);
    Serial.println(" A");
    Current_Value += ACCurrentValue;
    delay(1000);
  }


  Rate_of_Current_Value_per_min = Current_Value / 60;

  Serial.print("Rate of Current in a min"); Serial.print(Rate_of_Current_Value_per_min); Serial.println(" Am");

  if (!client.connected())
  {
    client.reconnect();
  }

  client.add("current", Rate_of_Current_Value_per_min);//Insert your variable Labels and the value to be sent
  client.ubidotsPublish("nodemcu");
  client.loop();
  Rate_of_Current_Value_per_min = 0;
  Current_Value = 0;

}
