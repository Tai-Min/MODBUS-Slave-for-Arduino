# MODBUS-Slave-for-Arduino

This library turns Arduino board into MODBUS server device capable of responding to eight basic RTU frames:
+ Read coil status (0x01)
+ Read input status (0x02)
+ Read holding register(0x03)
+ Read input register (0x04)
+ Force single coil (0x05)
+ Preset single register (0x06)
+ Force multiple coils (0x0F)
+ Preset multiple registers (0x10)

The library expects full RTU frames consisting of:
+ device ID (1 byte)
+ function ID (1 byte)
+ data (n bytes)
+ crc (2 bytes)

Also, the library is able to detect invalid frame and respond to it with adequate exception frame.

## Usage
### MSlave variable initializer:
```cpp
template <uint16_t dil, uint16_t dol, uint16_t ail, uint16_t aol>
MSlave(uint8_t id, HardwareSerial *serial);
```
**dil** - length of digital inputs array<br />
**dol** - length of digital outputs array<br />
**ail** - length of analog inputs array<br />
**aol** - length of analog outputs array<br />
**id** - unique id of the server <br />
**serial** - address to Arduino's HardwareSerial object 
<br />

### Enable or disable CRC in request/response/exception frames:
```cpp
void disableCRC();
void enableCRC();
```
CRC is enabled by default.
<br />

### Check if there is request frame in serial buffer and parse it:
```cpp
bool available();
```
returns true only if one of eight rtu frames has been successfully processed <br />
This function should be used as often as possible to provide responsive and dependable server.
<br />

### Read from digital/analog input/output array:
```cpp
bool digitalRead(uint16_t address, bool mode);
uint16_t analogRead(uint16_t address, bool mode); 
```
**address** - position in specified array <br />
**mode:** 
+ INPUT - things from the outside
+ OUTPUT - things written by using analogWrite function

### Write to digital/analog output array:
```cpp
void digitalWrite(uint16_t address, bool value);
void analogWrite(uint16_t address, uint16_t value);
```
**address** - position in specified array <br />
**value** - value to be written
<br />

## Example
```cpp
#include "MSlave.h"

int ledPin = 13;
int potPin = A0;
int pwmLedPin = 11;
int buttonPin = 3;

int deviceID = 1;

//1 digital input (address 0)
//1 digital output (address 0)
//1 analog input (address 0)
//2 analog outputs (adresses 0,1)
//addresses works just like arrays
//so if you have 2 analog outputs, you can access adresses from 0 to 1
//if you have 40 analog inputs, you can access adresses from 0 to 39 etc
MSlave<1, 1, 1, 2> server(deviceID, &Serial);//initialize slave device

void setup()
{
  pinMode(ledPin, OUTPUT);
  pinMode(buttonPin, INPUT);
  pinMode(potPin, INPUT);
  pinMode(pwmLedPin, OUTPUT);
  server.disableCRC();//no need for crc check in this example
  Serial.begin(115200);
  Serial.setTimeout(15);
}

void loop()
{
  if (server.available())//check whether some data exchange happened with this device
  {
    digitalWrite(ledPin, server.digitalRead(0, INPUT));//digitalRead external boolean data received from client devices
    analogWrite(pwmLedPin, server.analogRead(0, INPUT));//analogRead external uint16_t data received from client devices
    server.digitalWrite(0, digitalRead(buttonPin));//digitalWrite button's state to server object so it will be available to be read from clients
    server.analogWrite(0, analogRead(potPin));//analogWrite potentiometer's state to server object so it will be available to be read from clients
    if(server.digitalRead(0, OUTPUT))//read buttons's state from server object
    {
      server.analogWrite(1, 512);//analogWrite some value server object so it will be available to be read from clients
    }
  }
}
```
### frame to turn on led on pin 13:
| 1 | 5 | 0 0 | 255 0 | <br />
### frame to change light intensity of pwm led on pin 11:
| 1 | 6 | 0 0 | 128 0 | <br />
### frame to read state of the button on pin 3:
| 1 | 2 | 0 0 | 0 1 | <br />
### frame to read state of the potentiometer on pin A0:
| 1 | 4 | 0 0 | 0 1 | <br />
### frame to read state of led on pin 13:
| 1 | 1 | 0 0 | 0 1 | <br />
### frame to read state of both potentiometer and address with index 1 (server.analogWrite(1, 512)):
| 1 | 4 | 0 0 | 0 2 | <br />
## About
The library was written using following documents and sites:
+ MODBUS over Serial Line Specification and Implementation Guide V1.02
+ Modbus Protocol Reference Guide
+ http://www.simplymodbus.ca/
