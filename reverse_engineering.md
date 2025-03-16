# Reverse-Engineering Process
This document provides an overview of the steps I took to gather the necessary information and make the package functional:
1. I downloaded and installed the [Ambientika Android App](https://play.google.com/store/apps/details?id=sw.ambientika.app)
2. I registered the device using the official app to get everything set up.
3. Now I installed the [PCAPdroid - network monitor](https://play.google.com/store/apps/details?id=com.emanuelef.remote_capture) to be able to see the requested URLs and Ports.
4. After a bit of configuration, I was able to track the network traffic of the App and see the servers and ports that it connected to:
   <br><img src="images/com.emanuelef.remote_capture-edit.jpg" alt="PCAPdroid Screenshot" width="50%"/>
5. Next, I searched for the server documentation and discovered it was available on [Swagger UI](https://app.ambientika.eu:4521/swagger/index.html)
6. After locating the documentation, the process involved:
   - reading the documentation
   - testing various REST calls using `curl`
   - iterating until successful
   - implementing the functional REST calls in Home Assistant configuration files
   - adjusting the configuration to ensure compatibility with Home Assistant

In addition to the aforementioned approach, I intercepted the network traffic using an old tablet running Android 4. Originally the plan was to do this on my actual phone or a virtual/simulated device. But according to the [description](https://emanuele-f.github.io/PCAPdroid/tls_decryption.html) of the [PCAPdroid mitm-Addon](https://github.com/emanuele-f/PCAPdroid-mitm/releases) such a [man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) does **not work** under the following conditions:
- Android > 7 
- app target SDK > 23  

So my ancient Android 4 Tablet was the perfect candidate to decrypt the messages and verify the REST Endpoints and payloads, as well as their frequency.  



## Setting Up the Home Assistant Package
To start with the new package, I first created the necessary folders and added entries to the `configuration.yaml` following the instructions in the [Installation and Setup](readme.md#installation-and-setup) section. Next, I created each YAML file one by one and filled them with the necessary code to ensure the required entities appeared.
> _**Note**: The **Ambientika Token** is excluded from the recorder because it contains sensitive information. Therefore, it should not be written to the history database!_



## Creating the REST Commands and REST Sensors
The package primarily relies on the following three [Home Assistant Integrations](https://www.home-assistant.io/integrations):
- [RESTful Command](https://www.home-assistant.io/integrations/rest_command/) (to expose REST commands as ``service`` for a script or automation)
- [RESTful Sensor](https://www.home-assistant.io/integrations/sensor.rest/) (for setting up a single ``sensor``)
- [RESTful](https://www.home-assistant.io/integrations/rest/) (for multiple ``sensor`` or ``binary_sensor`` from the same REST endpoint)

<br>
  
The package also includes several helpers and automations to support the REST commands.
- [Input Text](https://www.home-assistant.io/integrations/input_text/) to store the access token
- [Input Number](https://www.home-assistant.io/integrations/input_number/) for the target fan speed and target humidity level
- [Input Select](https://www.home-assistant.io/integrations/input_select/) to define the operation mode
- [Automation](https://www.home-assistant.io/docs/automation/) to automatically update the operation mode, fan speed, or humidity level, if it changes
- [Template Switch](https://www.home-assistant.io/integrations/switch.template/) to enable/disable the unit 

<br> 

--------
Essentially, the process involved testing and YAML work, using the [Swagger UI](https://app.ambientika.eu:4521/swagger/index.html) as a reference in the following order:

### 1. **Get the authentication token** into Home Assistant (since the tokens expire after 180 days)
   ````
   curl -X POST https://app.ambientika.eu:4521/users/authenticate \
     -H 'Content-Type: application/json' \
     -d '{"username":"example@yahoo.com","password":"my-secret-password"}'
   ````
   This command could be easily implemented in [rest_commands.yaml](rest_commands.yaml#L2) 
   - Refer to the documentation for the correct syntax
   - The command will then be exposed as a service in Home Assistant

### 2. **Check remaining days** for monitoring and renewing the token, before it expires
   ````
   curl -X GET https://app.ambientika.eu:4521/users/token-info \
     -H 'Content-Type: application/json' \
     -H 'authorization: Bearer my-super-secret-token'
   ````
   The feedback value is a `datetime`, which needs to be converted to days. So first I needed to convert it to a timestamp, subtract the current time, and divide by 60/60/24 to convert from seconds to minutes, hours, and finally, days. According to the settings in the [sensors.yaml](sensors.yaml#L54), the token info is polled every hour (3600 seconds).

### 3. **Automation: Refresh Token** once Home Assistant is started or if the remaining days fall below a certain value
   See the [automation.yaml](automation.yaml) for details. It essentially retrieves the token and sets the value of the `input_text.ambientika_access_token` (see [helpers.yaml](helpers.yaml) and [customize.yaml](customize.yaml#L9))

### 4. **Get the Device Status** as Sensors in Home Assistant
   ````
   curl -X GET   https://app.ambientika.eu:4521/device/device-status?deviceSerialNumber=123456789ABC \
     -H 'Content-Type: application/json' \
     -H 'authorization: Bearer my-super-secret-token'
   ````
   This request returns all relevant status information of the device. Therefore, I use the  [RESTful integration](https://www.home-assistant.io/integrations/rest/) to return the received data as Home Assistant entities. As can be seen from the [sensors.yaml](sensors.yaml#L7), this command provides:
   - sensor:
      - Operating Mode
      - last Operating Mode
      - Fan Speed
      - Filter Status
      - Temperature (raw)
      - Humidity (raw)
      - Air Quality
   - binary_sensor:
      - Humidity Alarm
      - Night Alarm
      
   > **Note**: Unfortunately `resource` or `resource_template` do not support combining a fixed string with a secret, such as the device ID. Therefore, this URL is listed separately as `device-status-1` in the `secrets.yaml`.

### 5. **Change Mode** command
   ````
   curl -X POST https://app.ambientika.eu:4521/device/change-mode \
     -H 'Content-Type: application/json' \
     -H 'authorization: Bearer my-super-secret-token' \
     -d '{"deviceSerialNumber":"123456789ABC","operatingMode":"Smart","fanSpeed":1,"humidityLevel":1}'
   ````
   This command is also implemented in [rest_commands.yaml](rest_commands.yaml#L10) and exposed as a service, allowing it to be used in automations and scripts. 

### 6. **Control Entities** and **Automation**  
   To control the `operatingMode` several Input entities where necessary. As can be seen from the [helpers.yaml](helpers.yaml#L9), there are three input entities:
   - Fan Speed (only necessary in `operatingMode` **ManualHeatRecovery**)
   - Humidity Level (only necessary in `operatingMode` **Auto** and **Surveillance**)
   - Mode Selection (always necessary, except in mode `Off`)
   
   To push this information to the Ambientika Control Server, the **Change Mode** command from the previous point is used, as can be seen in the [automation.yaml](automation.yaml#L31), including the values for Fan Speed and Humidity Level.
   
### 7. **Switch** for on/off control  
   In the end, I decided to implement a [Template Switch](https://www.home-assistant.io/integrations/switch/) in the [switch.yaml](switch.yaml) to be able to turn the device on and off the device with any hardware button you might have integrated into your Home Assistant instance. The actions of the switch are basically a copy of the actions from [Automation](automation.yaml#L31). This includes a [homeassistant.update_entity](https://www.home-assistant.io/integrations/homeassistant/#service-homeassistantupdate_entity) command after a short delay to get the updated `operatingMode` after turning the device on or off.
 
  
## Filter values

Unfortunately the numerical Values from the sensors had a lot of jitter, so I tried to find a suitable [filter](https://www.home-assistant.io/integrations/filter/) for these sensors. This involved a lot of trial and error, waiting for values to be generated, and comparing the different tests:  
<img src="images/sensor-filter_humidity__lowpass_vs_time-simple-moving-average_2.png" alt="Filter comparison: Lowpass vs. Time Simple Moving Average" width="600px"/>
<br>

As can be seen in the picture above: 
- The **Time Simple Moving Average**-Filter (orange and dark yellow) creates a nice graph, close to the original entity with it "raw value". But it still has some jitter that needs to be cleared out.
- The **Lowpass**-Filter (purple and cyan) on the other hand, make the graph smooth, but with unacceptable deviation from the original entity.

So, both were combined to create smooth values:  
<img src="images/sensor-filter_result.gif" alt="Filter result" width="434px"/>
<br>
> _**Note**: Don't get confused with these high values. I made the settings, filters and screenshots when the wall was freshly plastered. Therefore, the device was measuring a lot of moisture from the wall. The values reduced to a normal level after a couple of weeks._


## Getting the Images for the UI
To get the images for the UI, I wanted to reuse the original images from the Android app. Of course, I could have made a couple of screenshots, cropped them, and used those as PNG files, but I followed a more sophisticated workflow:
1. I extracted the APK using the [APK Extractor](https://play.google.com/store/apps/details?id=com.ytheekshana.apkextractor)  and transferred it to the PC.
2. Disassembled the APK using the [Apktool](https://apktool.org/): `apktool d '.\Ambientika APP.apk'`.
3. I searched for image files. In this case, I found that .svg files were used to render the user interface.
4. After examining the images, I tried to match them with the `operatingMode`.

| Image                                       | operatingMode                                                   |
| ------------------------------------------- | --------------------------------------------------------------- |
| ![](images/Casetta_completa.svg)            | Smart <br>Auto* <br>Night <br>AwayHome <br>ManualHeatRecovery** |
| ![](images/Casetta_espulsione.svg)          | Expulsion                                                       |
| ![](images/Casetta_flusso_master_slave.svg) | MasterSlaveFlow                                                 |
| ![](images/Casetta_flusso_slave_master.svg) | SlaveMasterFlow                                                 |
| ![](images/Casetta_immissione.svg)          | Intake                                                          |
| ![](images/Casetta_orologio.svg)            | TimeExpultion***                                                |
| ![](images/Casetta_vuota.svg)               | Surveillance* <br>Off                                           |

\*   The humidity slider is active, and the set humidity target is shown next to the house. Also, the humidity-value on the left is shown in red, if the `humidityAlarm` is active.  
\**  The temperature slider is active, and the current fan level is shown next to the house.  
\*** According to the [App-Manual](https://www.ambientika.eu/cms/wp-content/uploads/P06506000_manual_Smart-APP_EN-oktober-2023.pdf) (page 22/24) the TimeExpultion Mode is running for 20 minutes on max. fan speed. Afterwards it returns to the mode, stored in the `lastOperatingMode` attribute.

