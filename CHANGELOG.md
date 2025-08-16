
# Change Log
All notable changes to this project will be documented in this file.  
You can also take a look into the [Milestones](https://github.com/Flo-R1der/ambientika-smart_4_home-assistant/milestones?state=closed) as I'm doing the detailed documentation there.


## [v0.2.1](/../../compare/v0.2.0...v0.2.1) - 2025-08-16
### Added
- Script for filter reset added to master devices (2248a46)

### Improvements
- UI examples updated to show the filter reset button too (1d02162)

### Fix
- remove slave device from filter notification automation (see comment in #28) to prevent filter notifications, while the filter cant be reset (8e552b7)

<br>


## [v0.2.0](/../../compare/v0.1.2...v0.2.0) - 2025-04-09

- Reorganized the YAML structure: files are now split into "general" and "per-device" sections, making duplication for additional devices easier and well-documented.
- Added documentation (text and image) explaining how master and slave devices work together.
- Reduced the number of sensors for slave devices by removing control-related entities (e.g. fan speed, humidity target), as these are now managed by the master.
- Slaves can now be linked to a master via a custom attribute.
- Operation mode on slave devices now mirrors the master’s operation mode.
- All device-related entity names and IDs have been changed to improve "search and replace" operations.
- Device-related secrets have also been renamed to simplify "search and replace".
  

### Update from previous versions
> [!CAUTION]  
> **BREAKING CHANGE**:  
> Applying this version over a previous installation will break the package unless you follow the steps below carefully.

1. In your `configuration.yaml` remove the previous package registration under ``homeassistant:``, ``packages:``, ``ambientika_smart:``.
2. Navigate to your package folder and delete all YAML files, **except for `secrets.yaml`**.
3. Open **Developer tools** and click on **RESTART** → **Restart Home Assistant** (Do not use Quick Reload) and proceed with **OK**.
4. After the restart, go to **Setting** → **Devices & Services** and open the **Entities** tab.
5. Search for "Ambientika", select all entities marked as 'unavailable' (use multi-select) and delete them.
6. Set-Up the package by following the latest [Installation and Setup](readme.md#installation-and-setup) instructions.
7. Update your UI with the new [UI-Examples](readme.md#user-interface) and your personal or custom UI elements, displaying Ambientika Information.
8. Update your personal or custom Automation/Script/Template/... with the new entity ID structure:
   - package-related: **no changes**
   - device related:  
      - `ambientika_1_*` → `ambientika_master-1_*`
      - `ambientika_2_*` → `ambientika_slave-2_*`
      - _...and so on, for each device_

> [!IMPORTANT]  
> **The new entity IDs are already reflected in the updated YAML files. If you skip deleting the old entities, the new setup may not work correctly and could break the package!**

<br>


## [v0.1.2](/../../milestone/4?closed=1) - 2024-11-20
### Added
- **`lightSensorLevel` control** as `input_number` entity, to `change-mode` command payload and to automation "Ambientika 1 Change-Mode"
  

### Improvements
- easier setup for Automation "Ambientika 1 Change-Mode"
- restructure automation "Ambientika 1 Change-Mode" to use automation also from switch
- switch is now triggering the automation, without script duplicates

### Fix
- failed automation step when Operating-Mode == 'Off'
- typos
  
### Documentation updates 
- formatting with Note/Info/Warning blocks
- open topics
  
<br>


## [v0.1.1](https://github.com/Flo-R1der/ambientika-smart_4_home-assistant/milestone/1?closed=1) - 2024-11-20
### Added
- Auto-update on the 'input_select' and 'input_number' whenever the operating mode, seletcted fan-speed or humidity-level is changing; independent from the cause of change
- Notification for bad filters
- Script for filter-reset
- Entities `deviceRole` `lightSensorLevel` and `signalStrenght` added


<br>

## v0.1 - 2024-07-22
- initial release
