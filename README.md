# Wemos IR Learning Remote
A learning IR remote using the Wemos D1 Mini 

This project is based on the excellent ![IRremoteESP8266](https://github.com/markszabo/IRremoteESP8266) project by markszabo.

## Requirements
##### Hardware - 
- Wemos D1 Mini
- TSOP 1838 IR Receiver
- IR LED
- Resistors - 100Ω & 15kΩ - 1 each
- Capacitor - 4.7μF - 1 pc.
##### Software -
- Node-RED
- MongoDB
- Arduino IDE
- This project - 'ino' file to be flashed onto Wemos using Arduino IDE, Node-RED flow to be imported into Node-RED.

## Introduction
Using the **Wemos IR Learning Remote**, you can -
1. generate IR signals of any consumer remote by sending a HTTP/MQTT message (this feature is from the original IRremoteESP8266 project)
2. generate IR signals of any consumer IR remote from the IR signals of other remotes using a user-specified mapping
3. create the above IR code to IR code as well as IR code to MQTT (see #4 below) message mappings using a sequence of sending MQTT "commands" and remote button presses. 
4. generate custom MQTT messages from IR signals using a user-specified mapping

The  mappings are stored in a MongoDB database which is exposed via HTTP API configured in the Node-RED flow of this project.

The project combines two examples in the IRremoteESP8266 project into a single codebase so that both "IR blasting" and "Learning" are implemented in the same MCU unit (Wemos D1 Mini). The two example files are:
https://github.com/markszabo/IRremoteESP8266/blob/master/examples/IRMQTTServer/IRMQTTServer.ino
and
https://github.com/markszabo/IRremoteESP8266/blob/master/examples/IRrecvDumpV2/IRrecvDumpV2.ino

## Details
### 1. Generating IR signals via network
IR codes can be sent to the Wemos via MQTT or HTTP to activate the IR LED. See the examples from the IRremoteESP8266 project for documentation on the HTTP API/MQTT message format for IR generation and receiving. This feature is useful when one wants to programmatically send an IR command to a device like TV, Cable TV STB, etc.,

### 2. Generating IR signals via other IR signals
This feature is useful when one wants to control several devices using the same remote. Shining a remote at the IR Receiver connected to the Wemos generates the previously mapped remote signal of another remote. The mapping is per-button (remote1.buttonA to remote2.buttonB) and any number of remotes can be used. There is no limit on the number of mappings that can be made. The same remote and button combination cannot have more than one mapping as of now. IF you try to do this, the second mapping attempt will simply overwrite the first mapping.

### 3. Mapping process
The mapping process essentially creates a record in the database with the following 4 fields:
remote, btnname, key, value. Each of these have to be "set" individually, and then the mapping record is to be saved.
The following describes each of these fields and how they are to be "set".

* **remote** - (Optional) name of the "source" remote (e.g., SamsungTV). Can be an arbitrary string. This is set by sending a MQTT message to the topic "ir_server/cmd" a message of the form "REMOTE <source_remote_name>"
* **btnname** - (Optional) name of the source remote button (e.g., Power). Can be an arbitrary string. This is set by sending a MQTT message to the topic "ir_server/cmd" a message of the form "BTNNAME <source_remote_button_name>"
* **key** - (Required) IR code of the source remote button being currently mapped, e.g., "7_E0E09966". This is set by first sending the MQTT message string "KEY", to the topic "ir_server/cmd" and then pressing the source remote button (to be mapped) while pointing the source remote at the IR receiver connected to the Wemos.
* **value** - (Optional) IR code of the destination remote button to which "key" is being mapped to, e.g., "2_C0000C". This is set by first sending the MQTT message string "VALUE", to the topic "ir_server/cmd" and then pressing the destination remote button while pointing the destination remote at the IR receiver connected to the Wemos.

After setting the 4 fields as explained above, the mapping record is to be saved by sending the MQTT message string "SAVE", to the topic "ir_server/cmd".

In all the above cases, the topic "ir_server/cmd" is used. This topic is subscribed to by the Node-RED flow of this project.

### 4. Generating network signals (MQTT) from IR signals
This feature can be used to trigger programmatic events/processes using an IR remote. To use this feature, one has to create a mapping as described above, with the value seting omitted. Now, whenever a mapped remote button is pressed (pointing the remote at the Wemos' IR REceiver), an MQTT message of the form "<remote:btnname>" is published on the topic "ir_server/btn". So any MQTT client subscribing to this topic will know the remote name and button that was pressed. This can be used from a script, for e.g., to trigger some process. 

#### About the Node-RED flow:
The Node-RED flow requires the ![node-red-contrib-mongodb2](https://github.com/ozomer/node-red-contrib-mongodb2) node module to be installed in the Node-RED instance. Once the flow is imported into Node-RED, the **mongo2** node needs to be configured with the details of your MongoDB instance. 
The Node-RED flow in this project creates several HTTP API that support the Wemos IR Learning Remote, in addition to subscribing to and processing a few MQTT topics necessary for the proper operation of the remote. These include setting the mapping record fields and saving/updating the mapping records in the DB.



![WiringDiagram](https://github.com/ajithvasudevan/WemosIRLearningRemote/raw/master/Wemos%20Learning%20Remote%20-%20Wiring%20Diagram.png)
