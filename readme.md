# Home Assistant Package for **Südwind Ambientika smart**

This package provides multiple config files that will create REST sensors, (REST) scripts, automations, and other config entities for using the [Südwind Ambientika smart](https://www.ambientika.eu/en/ambientika-smart/sw10035) within [Home Assistant](https://www.home-assistant.io/).
Since I haven't found any smart *single-room heat recovery ventilation system* within the EU that fits all my requirements, I decided to take matters into my own hands. I purchased the Südwind Ambientika smart and started to reverse-engineer the REST API used to remotely control the unit using the [Android App](https://play.google.com/store/apps/details?id=sw.ambientika.app). For those who are interested in my **requirements**: 
- thin wall thickness (max. 25 cm)
- high heat recovery rate (min. 88%)
- smart/remote-controlled (ideally: Home Assistant Integration)

If you are interested in the **reverse-engineering process** see the corresponding [documentation](reverse_engineering.md) for reference.



## Installation and Setup
> [!NOTE]  
> _This package is delivered "as-is" → do not expect to get support in this repository!_  
> _However, if you find technical problems with this package (e.g., server changes, compatibility issues, YAML syntax changes, etc.), feel free to open an issue. Or even better: fix the issue and submit a pull request. The same applies to feature requests and enhancements._  

> [!IMPORTANT]  
> This setup assumes that you have an account registered in the official app and that **your Ambientika Device is already working** via the official App.

### 1. Download and extract the package to the config folder
Download the repository from GitHub as ZIP-file, extract it and place it in the correct folder:
`/config/packages/ambientika_smart`
<details>
  <summary>Screenshot</summary>  
  <img src="images/folder-structure.png" alt="example/intended folder structure" width=368px/>
</details>
<br>

> [!NOTE]  
> If you want to use my example approaches for the Lovelace UI, the images must be copied to `/config/www/ambientika_smart/`


### 2. Setting up `configuration.yaml`: 
Add the `homeassistant:`, `package:` and `ambientika_smart:` lines to your configuration.yaml as shown in my example here:
``````
homeassistant:
  packages:
    ambientika_smart_0: !include packages/ambientika_smart/0_general.yaml
    ambientika_smart_1: !include packages/ambientika_smart/1_master.yaml
    ambientika_smart_2: !include packages/ambientika_smart/2_slave.yaml
    ...
``````
> [!WARNING]  
> The `ambientika_smart_0:` or `0_general.yaml` provides some general mandatory stuff for this package. Never delete this, otherwise the package will stop working!

> [!NOTE]  
> For each device you want to integrate you need a line with `ambientika_smart_x` _(where **x** represents any number)_. 
> Make sure to match your device role (Master/Slave) with the correct master/slave yaml.


### 3. Setting up `secrets.yaml`:
First, navigate to the `packages/ambientika_smart/` folder and
1. Rename the `secrets_example.yaml` → `secrets.yaml`.
2. Then edit the lines shown here. **The device serial is equivalent to the MAC address of your device**. Typically, you can see and copy the MAC-address from your router.
``````
ambientika_username: "example@yahoo.com"
ambientika_password: "my-secret-password"

ambientika_device_serial_1: 1234567890ABC
ambientika_device_serial_1: 4567890ABCDEF

ambientika_device_status_1: https://app.ambientika.eu:4521/device/device-status?deviceSerialNumber=1234567890ABC
ambientika_device_status_1: https://app.ambientika.eu:4521/device/device-status?deviceSerialNumber=4567890ABCDEF
``````
> [!NOTE]  
> Same here: For each device you need a line with `ambientika_device_serial_x` and a line with `ambientika_device_status_x` and fill them with the corresponding serial-number.


### 4. Setting up `*_master.yaml` and `*_slave.yaml`:
<details>
<summary>For details about <b>Master-Slave role and operation</b> open here.</summary> 

> Basically there is a single master device, which you can control. For that device you can define the operation mode, fan-speed, humidity target-level and light-level. This device then connects to its assigned slaves and controls their behavior. This means the devices are joining forces in **Push-Pull operations**, where:
> - one or more device blows out used air (exhaust air phase) 
> - one or more opposite device draws in fresh air (supply air phase)
> 
> After 60-90 seconds, the devices are changing directions. The master coordinates this change to ensure balanced air circulation. The Ambientika ventilation system also has **ceramic heat exchangers for heat recovery**. During the exhaust air phase, the heat exchanger stores the heat energy, which is then transferred to the incoming fresh air in the subsequent supply air phase.  
> 
> If you use an operation-mode where sensor values are taken into account (Smart, Auto, AwayHome and Surveillance) the values from the master are used to do decisions. So it might be useful to set the device in the most challenging environment to be the master.
> 
> <img src="images/Master-Slave_explained.svg" alt="Master-Slave map"/>
</details> 

Now you need to set up a configuration files for each device. They are separated based on the device role:
- **`0_general.yaml`** provides some general mandatory stuff for this package, like the REST-commands, authentication and the filter notification and reset automation. **WARNING**: Never delete this file, it is mandatory!
- **`1_master.yaml`** provides everything a master device needs like sensors, input entities, switch for turn on/turn off and the change-mode automation.
- **`2_slave.yaml`** provides only sensors, since slaves can not be controlled as a user.

**You can use these files out-of-the-box if they fulfill your needs**.
If not and you have multiple devices that are not covered with the existing yaml files, do this **for each device you want to integrate in your Home Assistant**:
1. **Duplicate** the master-yaml or slave-yaml and give it a suitable name.
2. **Replace** all occurrences of `master_1` or `slave_2` in the copied file by pressing `CTRL + H`, choose a suitable replacement (Tip: reuse name+number from the filename) and hit **replace all**.
3. **Register the new file in your `configuration.yaml`** (/config) like described in Step 2. Otherwise it will not be loaded on Home Assistant start.
4. **Maintain the secrets of the device** to your `secrets.yaml` (/config/packages/ambientika_smart/) like described in step 3. Otherwies

> [!WARNING]  
> Do not rename the `friendly_name`/`name`/`alias` now, as the entity_id is derived from the display names. Changing the display name now will break some dependencies, so it is important to restart Home Assistant with the package, before changing the display names as described in [7. Adapt names, if necessary](#7-adapt-names-if-necessary)


### 5. Restart Home Assistant
1. Go to "**Developer tools**", Tab "**YAML**" first and click on "**CHECK CONFIGURATION**" before restarting!
   - If you have any configuration issues, Home Assistant might restart into "**Recovery Mode**"! You don't want that.
   - If an error is displayed, read the message carefully and follow the instructions!
2. Then: Click on "**RESTART**", "**Restart Home Assistant**" (Do not use _Quick Reload_) and proceed with "**OK**".


### 6. Check if the package is working
Once Home Assistant is restarted, the `automation.ambientika_authenticate` automation will be triggered and obtain the access token automatically. The token will be stored in the `input_text.ambientika_access_token` entity (type: `password`), which by default is excluded from the recorder to protect writing sensitive information into the persistent database. In addition, the token is automatically renewed as soon as the previous token is valid for less than 5 days.

You can check the incoming values in the "**Developer tools**" > "**STATES**" and filter for "**Ambientika**".
<details>
  <summary>See more Details</summary>  
<img src="images/Developer-tools.png" alt="Developer tools / STATES" width=675px/>

> The sensors "**Ambientika 1 Humidity (filtered)**" and "**Ambientika 1 Temperature (filtered)**" are producing a `unknown` value, for the first 5-15 minutes. Afterwards they should return smooth values ([see the Documentation for Details](reverse_engineering.md#filter-values)). If not, you might check the `entity_id` for both sensors to ensure that they are connected to the correct raw valued sensor.
</details>  


### 7. Adapt names, if necessary
The idea is: I would use numbering and device role for entity names and IDs to reduce the amount of editing. But in the user interface, I would use descriptive terms such as "Ambientika Bedroom" for better understanding and for Voice Assistants. Therefore, only the `friendly_name`/`name`/`alias` section can be adapted with another mass-replacement:
- **Replace** all occurrences of `Master-1` or `Slave-2` in the yaml-files by pressing `CTRL + H`, choose a suitable replacement (e.g. Bedroom, Childroom, Bathroom, Kitchen, etc.) and hit **replace all**.
- Repeat his step for all master and slave yaml-files, but **don't edit the `0_general.yaml`** in this step.


### 9. Filter Notification and Reset
The only maintenance the device needs is to wash the filter from time to time. It seems like there is a internal clock counting the working hours and reporting the filter status as "Good", "Medium" or "Bad". In `0_general.yaml` there is a **[Filter Notification and Reset Automation](0_general.yaml#L82)** that will be triggered, when `sensor.ambientika_master_1_filter_status` or `sensor.ambientika_slave_2_filter_status` are changing their state to "Bad". The automation will then send a **sticky notification** to all mobile devices in Home Assistant:

<img src="images/filter_notification.jpg" width=440/>

When you washed the filter, you _should_ be able to reset the filter working hours by pressing the **Reset Filter** button on your mobile (active connection to your Home Assistant instance mandatory!). the filter-reset will be sent to the control-server and the filter state _should_ switch back to "Good".  
> But why do I write "_should_ reset" instead of "will reset"? ► Check the [Open Topics](#open-topics); that's why.

#### ToDo:
If you want to enable the filter notification and filter reset also for other devices rather then master_1 or slave_2, you can do **one of the following**:
- In the [0_general.yaml on Line 88](0_general.yaml#L88) set up the list of filter status sensors. Then go to "**Developer tools**", Tab "**YAML**" and click on "**CHECK CONFIGURATION**" first. If the check is ok, click on "**AUTOMATIONS**" in the list to reload the automations-config. Double-check the automation for issues.
- Alternative: Go to the automation editor of "Ambientika Filter Notification" and click on "**MIGRATE**". This will duplicate the automation and you can use the UI editor now to add the filter status sensors to the state trigger. Afterwards you must turn off the yaml-controlled automation from the package!

<br>

-----

## User Interface
I have also created some UI examples, which are provided in the table below.  
Please note that the SVG images included in this package are originally from the Android app. In case new images are added in the future, feel free to use the [description of picture extraction](reverse_engineering.md#getting-the-images-for-the-ui) in my reverse-engineering description to update the package accordingly.

> [!NOTE]  
> If you want to use my example approaches for the Lovelace UI, the images must be copied to `/config/www/ambientika_smart/` first!


### Examples  
The examples from the table below are working for the device ``master_1``. For all other devices you must replace:
- All ``master_1`` ► ``slave_2`` or whatever renaming you have done in [4. Setting up master and slave](#4-setting-up-masteryaml-and-slaveyaml).  

In the Home Assistant editor the replacing function can be started be pressing `CTRL + F` on your keyboard: <br>
<img src="images/find_replace-all.png" width=430/>

| Image | YAML-code |
| ----- | --------- |
| ![](images/lovelace_example-0.png) | [lovelace.yaml_example-0](lovelace.yaml_example-0) |
| ![](images/lovelace_example-1.png) | [lovelace.yaml_example-1](lovelace.yaml_example-1) <sup>(1)</sup> |
| ![](images/lovelace_example-2.png) | [lovelace.yaml_example-2](lovelace.yaml_example-2) <sup>(1)</sup> |
| ![](images/lovelace_example-3.png) | [lovelace.yaml_example-3](lovelace.yaml_example-3) <sup>(1,2)</sup> |

<sup>(1)</sup> To use this card for a slave-device, remove the second part of the vertical stack. It would show failures anyway as there are no control entities for a slave-device.  
<sup>(2)</sup> This approach takes advantage of the [Mushroom Template Card](https://github.com/piitaya/lovelace-mushroom/blob/main/docs/cards/template.md). In order to use it you must install the Mushroom cards first. I would recommend to use [HACS](https://hacs.xyz/) to install them.

<br>


## Open Topics
- Unfortunately the [Ambientika API](https://app.ambientika.eu:4521/swagger/index.html) is acting strange on some things.
   1. For slave devices: A not actual operation-mode is received from the API. The received value equals to the last set operation-mode when this device was a master (from the device onboarding, since devices will always be onboarded as master).
   2. The filter reset does not work with other filter conditions rather than "Bad". This disables the possibility to reset the operating hours if you clean it more often. Big dislike, since I used to clean them all in one go and reset the operating hours fol all if possible.
   3. The filter rest does not work on slave devices. The filter condition on the slave is calculated independent from the master (as all sensor values). And even tho the condition is shown as "Bad", the API call is sent and the answer is 200 (success), the filter remains in "Bad" state.
   4. When I look at No 3. and 4. it may also be, that the filter reset is broken in general... (Date: 2025-03-16)
- I am not able to provide a custom integration. However, others managed to do so. You can find the official Integration on Github: **[ambientika](https://github.com/ambientika) / [HomeAssistant-integration-for-Ambientika](https://github.com/ambientika/HomeAssistant-integration-for-Ambientika)** and on [HACS](https://www.hacs.xyz/).
- This package and the official integration is depending on the cloud with all advantages and disadvantages. If you like to get a local control of your device, you might check this repository: [sragas / ambientika-local-control](https://github.com/sragas/ambientika-local-control). But the documentation is not on the desired level which makes this a more advanced and sophisticated approach.
- A [Template-Fan](https://www.home-assistant.io/integrations/fan.template/) may be a thing. But I'm not sure, if this makes any sense with this device.

<br>

## Like my work?  
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/I3I4160K4Y)
