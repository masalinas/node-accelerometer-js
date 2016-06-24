# node-red-accelerometer

A proof of concept to demostrate any measure captured from a accelerometer attached to a raspberry pi 3 and transport this information throw mqtt protocol to a hub server thats centralize this information using a mqtt message broker like mosca and message router like node-red to parse and manage. In this case we will use a node-red flow on the raspberry 3 and not python service to capture the accelerometer measures.

The Node-RED flow Designer in raspberry pi:

 ![picapp - apple macbook air](https://cloud.githubusercontent.com/assets/1216181/16336824/a50a6798-3a10-11e6-9176-ac30a31f5fe5.png)

The Node-RED flow Designer in server side:

![picapp - apple macbook air 2](https://cloud.githubusercontent.com/assets/1216181/16336999/69195666-3a12-11e6-8112-7ea83a57d1e0.png)

The Raspberry pi 3 and accelerometer connected

![node-red-accelerometer](https://cloud.githubusercontent.com/assets/1216181/16336750/fa8eed48-3a0f-11e6-8810-018dc226854c.JPG)

Architecture

# Hardware:

- [Raspberry pi 3](https://www.raspberrypi.org/): The Raspberry Pi is a series of credit card-sized single-board computers developed in the United Kingdom by the Raspberry Pi Foundation.
- [MPU6050](https://www.invensense.com/products/motion-tracking/6-axis/mpu-6050/): The MPU-6050™ parts are the world’s first MotionTracking devices designed for the low power, low cost, and high-performance requirements of smartphones, tablets and wearable sensors. 

# Infraestructure Techonologies on server

- [Raspbian](https://www.raspberrypi.org/downloads/raspbian/): Last raspbian for raspberry pi. Raspbian is the Foundation’s official supported operating system.
- [NodeJS](https://nodejs.org/): Event-driven I/O server-side JavaScript environment based on V8. Includes API documentation, change-log, examples and announcements.
- [mosca](https://github.com/mcollina/mosca): MQTT broker as a module http://mosca.io.
- [Node-Red](http://nodered.org/): A visual tool for wiring the Internet of Things.

# Infraestructure Techonologies on device

- [node-red-contrib-gpio](https://github.com/monteslu/node-red-contrib-gpio): A set of node-red nodes for connecting to johnny-five IO Plugins.
- [raspi-io](https://github.com/nebrius/raspi-io): An IO plugin for Johnny-Five that provides support for the Raspberry Pi.

# Frontend Techonologies on server

- [node-red-contrib-graphs](https://www.npmjs.com/package/node-red-contrib-graphs): A Node-RED graphing package.

# Installation:

Install GPIO Node-red packages on raspberry-pi 3 device:

- Stop node-red if is still running
  ```
  node-red-stop
  ```
- Start a root user from pi user:
  ```
  sudo su -
  ```
  
- Modifiy the start user of the node-red service and set root
  ```
  sudo nano /lib/systemd/system/nodered.service 
  ```  

- Start node-red, this will create a .node-red appplication directory to configure the new node red instance from root account
  ```
  node-red-start
  node-red-stop
  ```  

- Install in this last directory the new node packages to access the raspberry pi GPIO from nodejs
  ```
  cd .node-red
  npm install node-red-contrib-gpio
  npm install raspi-io
  ```  
  
- Exit the root account and restart node-red. Now the node-red instance start from root account, so the application directory of the node-red will be /roo/.node-red where we had installed all GPIO packages. This new nodes will used in our future flows to manage this pins of the raspberry pi.

  ```
  exit
  node-red-start
  ``` 
- Access Node-Red raspberry pi Web designer
```
   http://192.168.1.45:1880
```

- Copy and import this flow from node-red import clipboard to crete the raspberry pi flow
```
[{"id":"e0dd9e02.d117f","type":"mqtt-broker","z":"5e77b74b.936dc8","broker":"192.168.1.29","port":"1883","clientid":"","usetls":false,"verifyservercert":true,"compatmode":true,"keepalive":"60","cleansession":true,"willTopic":"","willQos":"0","willRetain":null,"willPayload":"","birthTopic":"","birthQos":"0","birthRetain":null,"birthPayload":""},{"id":"19d415f0.73ba1a","type":"nodebot","name":"","username":"","password":"","boardType":"raspi-io","serialportName":"","connectionType":"local","mqttServer":"","pubTopic":"","subTopic":"","tcpHost":"","tcpPort":"","sparkId":"","sparkToken":"","beanId":"","impId":""},{"id":"635ce998.acf998","type":"gpio out","z":"5e77b74b.936dc8","name":"","state":"I2C_READ_REQUEST","pin":"","i2cDelay":"0","i2cAddress":"104","i2cRegister":"59","outputs":1,"board":"19d415f0.73ba1a","x":209,"y":187,"wires":[["186bfa84.8252f5"]]},{"id":"3562881e.bf6c38","type":"debug","z":"5e77b74b.936dc8","name":"","active":true,"console":"false","complete":"false","x":534,"y":38,"wires":[]},{"id":"186bfa84.8252f5","type":"function","z":"5e77b74b.936dc8","name":"process i2c","func":"var data = msg.payload;\nvar int16 = function(high, low) {\n  var result = (high << 8) | low;\n  // if highest bit is on, it is negative\n  return result >> 15 ? ((result ^ 0xFFFF) + 1) * -1 : result;\n};\n\n\nmsg.payload = {\n    accelerometer : {\n        x: int16(data[0], data[1]),\n        y: int16(data[2], data[3]),\n        z: int16(data[4], data[5])\n    },\n    temperature : int16(data[6], data[7]),\n    gyro : {\n        x: int16(data[8], data[9]),\n        y: int16(data[10], data[11]),\n        z: int16(data[12], data[13])\n    }\n};\n\nreturn msg;","outputs":1,"x":334,"y":99,"wires":[["3562881e.bf6c38","f5563d59.de0aa"]]},{"id":"3086d0ad.426e8","type":"inject","z":"5e77b74b.936dc8","name":"","topic":"","payload":"14","payloadType":"string","repeat":"","crontab":"","once":false,"x":107,"y":97,"wires":[["635ce998.acf998"]]},{"id":"24705baf.328ae4","type":"gpio out","z":"5e77b74b.936dc8","name":"","state":"I2C_WRITE_REQUEST","pin":"","i2cDelay":"0","i2cAddress":"104","i2cRegister":"","outputs":0,"board":"19d415f0.73ba1a","x":493,"y":354,"wires":[]},{"id":"6997c4ec.8c13bc","type":"inject","z":"5e77b74b.936dc8","name":"","topic":"","payload":"","payloadType":"none","repeat":"","crontab":"","once":false,"x":143,"y":354,"wires":[["4c294d92.b11134"]]},{"id":"4c294d92.b11134","type":"function","z":"5e77b74b.936dc8","name":"i2c bytes","func":"msg.payload = [0x6B, 0x00];\nreturn msg;","outputs":1,"x":321,"y":354,"wires":[["24705baf.328ae4"]]},{"id":"f5563d59.de0aa","type":"mqtt out","z":"5e77b74b.936dc8","name":"","topic":"mpu6050","qos":"","retain":"","broker":"e0dd9e02.d117f","x":526,"y":129,"wires":[]}]
```

Install mosca mqtt message broker on the server side:
```
  npm install mosca bunyan -g
  mosca -v | bunyan
```

Execute npm to install node-red message router and the UI modules on the server, the package.json has all dependencies:
```
  npm install
```

Start node-red
```
  node node_modules/node-red/red.js
```

Access Node-Red server Web designer
```
  http://localhost:1880/
```

- Copy and import this flow from node-red import clipboard to crete the raspberry pi flow
```
[{"id":"d4a328fc.110fd8","type":"mqtt in","z":"19f847a6.9b1408","name":"","topic":"mpu6050","qos":"2","broker":"e8b9681b.ed6e18","x":123,"y":102,"wires":[["9bfd210a.90696"]]},{"id":"9bfd210a.90696","type":"debug","z":"19f847a6.9b1408","name":"","active":true,"console":"false","complete":"false","x":367,"y":102,"wires":[]},{"id":"e8b9681b.ed6e18","type":"mqtt-broker","z":"19f847a6.9b1408","broker":"localhost","port":"1883","tls":null,"clientid":"","usetls":false,"compatmode":true,"keepalive":"60","cleansession":true,"willTopic":"","willQos":"0","willRetain":null,"willPayload":"","birthTopic":"","birthQos":"0","birthRetain":null,"birthPayload":""}]
```

# Licenses
The source code is released under Apache 2.0.
