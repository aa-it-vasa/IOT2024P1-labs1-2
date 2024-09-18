# Lab 1 - Lab setup + MQTT with Arduino MKR WiFi1010

In this lab you will create a new IoT device using the Arduino MKR WiFi1010 board. The device will measure the room temperature and transmit to a remote server using the MQTT protocol. You will establish an insecure link and measure the transmission latency. You will also implement logic to perform actuation on the sensor based on user commands.

For the lab we will use Arduino IDE: https://www.arduino.cc/en/Main/Software to program the Arduino MKR WiFi1010 board.

## Preliminaries: Install the Arduino IDE

Download the latest version of the Arduino IDE to your laptop. Follow the instructions here: https://support.arduino.cc/hc/en-us/articles/360019833020-Download-and-install-Arduino-IDE. 

You will also need to install some libraries in the IDE through _Sketch > Include Libraries > Manage Libraries_. At least the following libraries are needed
- _ArduinoBearSSL_
- _ArduinoECCX08_ 
- _ArduinoMqttClient_ 

You will also need to install support for the board we are using, this is done under _Tools > Board > Board Manager_. The one needed is called _Arduino AVR Boards_. 

The source files must also be downloaded from Github to your local machine. Easiest is if you clone this GitHub repository, either using command line tools or GitHub Desktop (https://docs.github.com/en/desktop/overview/about-github-desktop).

## Assemble the Arduino board

Collect the board and the required components. You will need:

1. [Arduino MKR WiFi1010 board](https://store.arduino.cc/arduino-mkr-wifi-1010)
2. Breadboard
3. Thermistor [MCP9700](http://ww1.microchip.com/downloads/en/DeviceDoc/20001942G.pdf) if in Turku or [LMT87](http://ww1.microchip.com/downloads/en/DeviceDoc/20001942G.pdf) if in Vaasa.
4. Up to 5 breadboard connectors
5. A USB type A to USB micro cable
6. WiFi credentials for Arduino

Insert the Arduino on the breadboard. Make sure that Arduino pins are on either sides of the central ridge of the breadboard and that all the pins are inserted in the breadboard. (See the image below or ask if you're unsure).

Now we will verify that Arduino IDE has read and write access to the Arduino board. In the Arduino IDE, open the Sketch `TestLed` available from the folder Lab1 from the files you cloned to your local machine. Select the board _MKR WiFI 1010_  (Tools->Board->SAMD 32-bits ARM cortex M0+Boards) and be sure you use the correct serial port (below the selected board in the Tools menu). 

Verify/compile and upload the sketch to the Arduino MKR WiFi 1010 board (in the menu Sketch->Verifu/Compile and Upload)). If everything goes well, you will see a yellow LED blinking on the board. You are done for the Arduino setup as you can compile and upload an program on the board from the virtual environment.

## MQTT Client Setup

The overall architecture of the system you will implement is described in the figure bellow:
![](./figs/Lab1_Overview.png)

In order to subscribe to the messages that your Arduino board will send, we need a MQTT client. Fortunately this is available already in the Codespaces terminal available from the repository on Github. You can also use a local MQTT client, e.g., [MQTTX](https://mqttx.app/). 

## Arduino + Temperature Sensor

Now we are set to measure room temperature and transmit to a remote MQTT server.

### In Turku
First setup the hardware as shown in the circuit diagram below.

![](./figs/schematic_mkr_1000.png)

**IMPORTANT: Do not flip Vcc and GND connections. Also, do not short Vcc and GND. Power the board via the USB cable AFTER you verify that the circuit is correct**

The setup itself should similar to the figure below. The thermistor IC's pin numbers are determined by holding the flat end towards you, with pins facing downwards and counting from left. From the schematic, you connect first pin of the thermistor to third pin on Arduino marked as **Vcc**. Connect the second pin of the thermistor to second pin on the Arduino situated on the other side of the Vcc and is marked as **DAC0/A0**. Connect the third pin of the Arduino to the fourth pin of the Arduino board, marked as **GND**. GND pin is on the same side as Vcc. 

![](./figs/breadboard_mkr_1000.png)

Once the hardware is setup, open the Sketch `Turku/mqtt_unsecure` available from the Lab1 folder. 

### In Vaasa

The setup itself should similar to the figure above, however note that the thermistors pins are in different order, see 

![](./figs/lmt87.png)

Numbers are indicated in the figure. From the schematic, you connect the third pin of the thermistor to third pin on Arduino marked as **Vcc**. Connect the second pin 
of the Arduino to the fourth pin of the Arduino board, marked as **GND**. GND pin is on the same side as Vcc. Connect the first pin of the thermistor to second pin on the Arduino situated on the other side of the Vcc and is marked as **DAC0/A0**. 

Once the hardware is setup, open the Sketch `Vaasa/mqtt_unsecure` available from the Lab1 folder. 

### For both Turku and Vaasa

Have a look at the code and understand what it does before uploading the code. Change the content of the variable _group = "MyGroup"_ Use the provided WiFi username and password (check the defined variable from the .h header file). Change the topic to something unique. 

Compile the code and upload it to the board and verify the state of the Arduino board by connecting to the *Monitor* _(in Tools-> Serial monitor)_ in the Arduino IDE. 

The measurements will be sent to the public MQTT broker `broker.hivemq.com` (check this in the code). You can subscribe to your topic with the HiveMQ webclient https://www.hivemq.com/demos/websocket-client/. 

Also try (in the Codespaces terminal) to execute the mosquitto_sub command below. Substitute `responseTopic` within the quotes (keep the quotes) with the topic your device is sending the messages into. 

```bash
mosquitto_sub -h broker.hivemq.com -t "responseTopic"
```

### To do

1. Read and understand the code. Specifically understand the `getTemp` function. Explain its working in your report. Consult the datasheet:
   [Turku](http://ww1.microchip.com/downloads/en/DeviceDoc/20001942G.pdf) or [Vaasa](https://www.ti.com/lit/ds/symlink/lmt87.pdf) if necessary. Get a working understanding of the [WiFiNINA](https://www.arduino.cc/en/Reference/WiFiNINA) and [ArduinoMqttClient](https://github.com/arduino-libraries/ArduinoMqttClient)    library and shortly explain what functionality in these libraries are used in the code. (1 p)
2. What is the value returned by mqttClient.messageQoS() ? What does it mean? (1 p)
3. What is the value returned by mqttClient.messageRetain() ? What does it mean? (1 p)

## Command and Reponse

The code also has a subscriber component built in. Have a look at the subscriber function `onMqttMessage`. This is called when the device receives a message on a specific topic. To see it working, open another terminal in Codespaces and execute the command below:
```bash
mosquitto_pub -h broker.hivemq.com -t "commandTopic" -m "command"
```
Replace "commandTopic" with the appropriate topic name and "command" with appropriate message (read and understand the function `onMqttMessage`).

### To do

1.  Modify your subscriber to implement these two commands (2 p)
    1. The **ON** command will turn on the onboard LED. 
    2. Similarly, **OFF** command will turn off the onboard LED.
    3. Any other command will not generate a response. 
2. Modify the publisher to transmit messages only when an appropriate **TEMP** command is received. (2 p)
3. Test the implementation by sending the **ON**, **OFF** and **TEMP** commands on the relevant topic using either. (2 p)
4. In your report provide a block diagram of the implemented system and explain the setup shortly. (2 p)

Hint: Check [String](https://www.arduino.cc/reference/en/language/variables/data-types/stringobject/) to create strings. Read character by character and append to the string. Don't forget to terminate with a Null character (`'\0'`) in the end. 

## Wrap Up

Well, that's it for this first lab. You can move to the 2nd lab!