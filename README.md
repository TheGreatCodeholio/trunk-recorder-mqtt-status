# Trunk Recorder MQTT Status (and Units!) Plugin <!-- omit from toc -->

This is a plugin for Trunk Recorder that publish the current status over MQTT. External programs can use the MQTT messages to collect and display information on monitored systems.

- [Install](#install)
- [Configure](#configure)
- [MQTT Messages](#mqtt-messages)
- [Mosquitto MQTT Broker](#mosquitto-mqtt-broker)
- [Docker](#docker)

## Install

1. **Build and install the current version of Trunk Recorder** following these [instructions](https://github.com/robotastic/trunk-recorder/blob/master/docs/INSTALL-LINUX.md). Make sure you do a `sudo make install` at the end to install the Trunk Recorder binary and libaries systemwide. The plugin will be built against these libraries.

2. **Install the Paho MQTT C & C++ Libraries**.

&emsp; _Install Paho MQTT C_

```bash
git clone https://github.com/eclipse/paho.mqtt.c.git
cd paho.mqtt.c

cmake -Bbuild -H. -DPAHO_ENABLE_TESTING=OFF -DPAHO_BUILD_STATIC=ON  -DPAHO_WITH_SSL=ON -DPAHO_HIGH_PERFORMANCE=ON
sudo cmake --build build/ --target install
sudo ldconfig
```

&emsp; _Install Paho MQTT C++_

```bash
git clone https://github.com/eclipse/paho.mqtt.cpp
cd paho.mqtt.cpp

cmake -Bbuild -H. -DPAHO_BUILD_STATIC=ON  -DPAHO_BUILD_DOCUMENTATION=TRUE -DPAHO_BUILD_SAMPLES=TRUE
sudo cmake --build build/ --target install
sudo ldconfig
```

&emsp; Alternatively, if your package manager provides recent Paho MQTT libraries:

```bash
sudo apt install libpaho-mqtt-dev libpaho-mqttpp-dev
```

3. **Build and install the plugin:**

&emsp; Plugin repos should be cloned in a location other than the trunk-recorder source dir.

```bash
mkdir build
cd build
cmake ..
sudo make install
```

&emsp; **IMPORTANT NOTE:** To avoid SEGFAULTs or other errors, plugins should be rebuilt after every new release of trunk-recorder.

## Configure

**Plugin options:**

| Key           | Required | Default Value        | Type   | Description                                                                                                                                                                              |
| ------------- | :------: | -------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| client_id     |          | tr-status            | string | This is the optional client id to send to MQTT.                                                                                                                                          |
| broker        |    ✓     | tcp://localhost:1883 | string | The URL for the MQTT Message Broker. It should include the protocol used: **tcp**, **ssl**, **ws**, **wss** and the port, which is generally 1883 for tcp, 8883 for ssl, and 443 for ws. |
| topic         |    ✓     |                      | string | This is the base topic to use. The plugin will create subtopics for the different types of status messages.                                                                              |
| unit_topic    |          |                      | string | Optional field to enable reporting of unit stats over MQTT.                                                                                                                              |
| message_topic |          |                      | string | Optional field to enable reporting of trunking messages over MQTT.                                                                                                                       |
| username      |          |                      | string | If a username is required for the broker, add it here.                                                                                                                                   |
| password      |          |                      | string | If a password is required for the broker, add it here.                                                                                                                                   |
| qos           |          | 0                    | int    | Set the MQTT message [QOS level](https://www.eclipse.org/paho/files/mqttdoc/MQTTClient/html/qos.html)                                                                                    |

**Trunk-Recorder options:**

| Key                          | Required | Default Value               | Type   | Description                                                                                |
| ---------------------------- | :------: | --------------------------- | ------ | ------------------------------------------------------------------------------------------ |
| [instance_id](./config.json) |          | <nobr>trunk-recorder</nobr> | string | Append an `instance_id` key to identify the trunk-recorder instance sending MQTT messages. |

**Plugin Usage:**

See the included [config.json](./config.json) for an example how to load this plugin.

```json
    "plugins": [
    {
        "name": "MQTT Status",
        "client_id": "tr_status",
        "library": "libmqtt_status_plugin.so",
        "broker": "tcp://io.adafruit.com:1883",
        "topic": "robotastic/feeds",
        "unit_topic": "robotastic/units",
        "username": "robotastic",
        "password": "",
        "qos": 0
    }]
```

If the plugin cannot be found, or it is being run from a different location, it may be necesarry to supply the full path:

```json
        "library": "/usr/local/lib/trunk-recorder/libmqtt_status_plugin.so",
```

## MQTT Messages

The plugin will provide the following messges to the MQTT broker depending on configured topics.

| Topic                   | Sub-Topic                                          | Retained | Description\*                                                      |
| ----------------------- | -------------------------------------------------- | :------: | ------------------------------------------------------------------ |
| topic                   | [rates](./example_messages.md#rates)               |          | Control channel decode rates                                       |
| topic                   | [config](./example_messages.md#config)             |    ✓     | Trunk-recorder config information                                  |
| topic                   | [systems](./example_messages.md#systems)           |    ✓     | List of configured systems                                         |
| topic                   | [system](./example_messages.md#system)             |          | System configuration/startup                                       |
| topic                   | [calls_active](./example_messages.md#calls_active) |          | List of active calls, updated every second                         |
| topic                   | [recorders](./example_messages.md#recorders)       |          | List of all recorders, updated every 3 seconds                     |
| topic                   | [recorder](./example_messages.md#recorder)         |          | Recorder status changes                                            |
| topic                   | [call_start](./example_messages.md#call_start)     |          | New call                                                           |
| topic                   | [call_end](./example_messages.md#call_end)         |          | Completed call                                                     |
| topic/trunk_recorder    | [`client_id`](./example_messages.md#plugin_status) |    ✓     | Plugin status, sent on startup or when the broker loses connection |
| unit_topic/shortname    | [call](./example_messages.md#call)                 |          | Channel grants                                                     |
| unit_topic/shortname    | [end](./example_messages.md#end)                   |          | Call end unit information\*\*                                      |
| unit_topic/shortname    | [on](./example_messages.md#on)                     |          | Unit registration (radio on)                                       |
| unit_topic/shortname    | [off](./example_messages.md#off)                   |          | Unit degregistration (radio off)                                   |
| unit_topic/shortname    | [ackresp](./example_messages.md#ackresp)           |          | Unit acknowledge response                                          |
| unit_topic/shortname    | [join](./example_messages.md#join)                 |          | Unit group affiliation                                             |
| unit_topic/shortname    | [data](./example_messages.md#data)                 |          | Unit data grant                                                    |
| unit_topic/shortname    | [ans_req](./example_messages.md#ans_req)           |          | Unit answer request                                                |
| unit_topic/shortname    | [location](./example_messages.md#location)         |          | Unit location update                                               |
| message_topic/shortname | [messages](./example_messages.md#messages)         |          | Trunking messages                                                  |

\* Some messages have been changed for consistency. Please see links for examples and notes.  
\*\* `end` is not a trunking message, but sent after trunk-recorder ends the call. This can be used to track conventional non-trunked calls.

## Mosquitto MQTT Broker

The [Mosquitto](https://mosquitto.org) MQTT is an easy way to have a local MQTT broker. It can be installed from a lot of package managers.

Starting it on a Mac:

```bash
/opt/homebrew/sbin/mosquitto -c /opt/homebrew/etc/mosquitto/mosquitto.conf
```

## Docker

The included Dockerfile will allow buliding a trunk-recorder docker image with this plugin included.

`docker-compose` can be used to automate the build and deployment of this image. In the Docker compose file replace the image line with a build line pointing to the location where this repo has been cloned to.

Docker compose file:

```yaml
version: "3"
services:
  recorder:
    build: ./trunk-recorder-mqtt-status
    container_name: trunk-recorder
    restart: always
    privileged: true
    volumes:
      - /dev/bus/usb:/dev/bus/usb
      - /var/run/dbus:/var/run/dbus
      - /var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket
      - ./:/app
```
