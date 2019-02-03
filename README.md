# WiPy 2.0 Temperature Controlled Fan
PWM fans with temperature based speed control which can be placed on a central heating convector, with a JavaScript based web interface.

### Summary
Central heating systems with condensing boilers operate more efficiently if the return water temperature is below 55°C ([Wikipedia](https://en.wikipedia.org/wiki/Condensing_boiler)). One way to extract more heat from a radiator is having fans which blow cold air past it. Not only does this lower the temperature of the return water, it also warms up rooms faster. This project presents a solution where a WiPy is used to control these fans, where the fan speed depends on the temperature of the incoming water. The parameters which control the fan are editable via a browser based user interface.

I have installed this on top of the main convector (which is lowered in a pit) in my living room. Seven PWM controlled fans - normally used for cooling PC's - pull cold air through the convector.

### Operating Principle
The fans should only operate if the central heating is working. Therefor a temperature sensor is attached to the incoming water pipe of the convector. If the temperature here is above the threshold temperature the fans start rotating at the lowest speed. The warmer the incoming water becomes the faster the fans spin. A small hysteresis is used when determining if the fans must be stopped.

![graph.png](https://github.com/erikdelange/WiPy-2.0-Temperature-Controlled-Fan/blob/master/graph.png)

### HTML and JavaScript
The user interface can be found in *index.html*. It uses the W3.CSS framework for formatting the web page. The status panel shows the actual temperature of the in- and outgoing water in the convector and the current duty cycle of the fans. The settings panel allows you to change the parameters of the controlling algorithm.

Interaction with the server is done using JavaScript's fetch API. After loading the webpage all fields in the UI are initially filled after a call to /api/init (see *onLoadEvent()*). Every 30 seconds the status fields are refreshed (*setInterval()*). After changing one of the settings fields the new value is sent to the server via /api/set. Every interaction with the server expects a JSON response, except when calling /api/reset.

![ui.png](https://github.com/erikdelange/WiPy-2.0-Temperature-Controlled-Fan/blob/master/ui.png)

Use F12 on Chrome and have a look the messages printed in the console.

The reset button on the UI is useful when a new version of the software has been transferred the WiPy using FTP. It allows you to restart the WiPy - and thus activate the new software - after it has been uploaded.

### Python Code
The fans are controlled by a daemon process (*daemon()*) which measures the temperature of the incoming water regularly and then adjust the fan speed. The control parameters for the speed control algorithm are stored in NVRAM so they are retained after a reset. A socket server is used to communicate with the UI. Command are received from the client via HTTP's query string, and responses from server to client use JSON.

### Hardware
The heart of the controller is a WiPy 2.0. Attached are two (or at least one) DS18x20 temperature sensors. One measures the temperature of the incoming water, the other one of the outgoing water. Pycom's library *onewire.py* is used to retrieve the readings from these sensors. The WiPy's PWM API is used to control the speed of PWM controlled computer case fans (I've used Arctic PWM PST fans which have an additional connector to allow daisy chaining of up to 4 fans). A daemon process checks the temperature every 30 seconds and adjusts the fan speed. As the WiPy cannot drive a PWM fan directly (3V3 vs 5V) BS170 FETs are used. Although a BS170 can officially not be opened completely at a 3V3 gate voltage it did work OK for me. Because the Arctic documentation specifies only 4 fans can be daisy chained two PWM outputs are used.

The PCB for this project looks like:

![pcb.png](https://github.com/erikdelange/WiPy-2.0-Temperature-Controlled-Fan/blob/master/pcb.png)

Note that the 7805 voltage regulator is attached to an aluminum profile for cooling. Bringing down 12V to 5V at 100mA does generate some heat.

The stripboard design is:

![stripboard.png](https://github.com/erikdelange/WiPy-2.0-Temperature-Controlled-Fan/blob/master/stripboard.png)

Not the most efficient design from space perspective but big enough to be handled comfortable, and requires only a limited number of connecting wires.

### Overkill
Yes, this is absolutely not the lowest cost solution for this problem. A temperature controlled switch like the W1209 is much cheaper and does not require any programming at all. However my solution was - at least to me - much more fun as I like programming and had the WiPy in stock anyhow. Remember the law of the instrument: "for a man with a hamer every problem looks like a nail" :)

### Using
* WiPy 2.0
* Pycom MicroPython 1.20.0.rc1 [v1.9.4-bc4d7d0] on 2018-12-12; WiPy with ESP32
* DS18x20 temperature sensors
* Arctic P14 PWM PST CO fans
