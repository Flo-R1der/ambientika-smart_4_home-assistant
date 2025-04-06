
# Change Log
All notable changes to this project will be documented in this file.  
You can also take a look into the [Milestones](https://github.com/Flo-R1der/ambientika-smart_4_home-assistant/milestones?state=closed) as I'm doing the detailed documentation there.


## [v0.2.0](/../../milestone/2?closed=1) - 2025-04-xx

- Replaced the old division of yaml by domain with a new one separating the files by "general" and "per device". So duplicating the yaml code for more devices is now easier and well documented.
- Documentation how master and slave devices are working together as text and picture.
- All device related entity names and IDs has been changed for better "search and replace".
- The device related secrets have been renamed for better "search and replace".
  

### Update from previous versions
> [!CAUTION]  
> **BREAKING**
> When applying these configuration files over a previous version, the package will break! Please follow the steps mentioned below to update to a most recent version.

1. In your `configuration.yaml` remove the previous package registration under ``homeassistant:``, ``package:``, ``ambientika_smart:``.
2. Navigate to your package folder and remove all yaml-files, **except the `secrets.yaml`**.
3. Go to "Developer tools" and click on "RESTART" > "Restart Home Assistant" (Do not use Quick Reload) and proceed with "OK".
4. After the restart go to "Setting", "Devices & Services" and click on the **Entities-Tab**.
5. Search for "Ambientika", select all 'unavailable' entities (use the selection mode) and delete them.
6. Set-Up the package as described in the most recent [Installation and Setup](readme.md#installation-and-setup) instructions.
7. Update your UI with the new [UI-Examples](readme.md#user-interface) and your personal or custom UI elements, displaying Ambientika Information.
8. Update your personal or custom Automation/Script/Template/... code with the new entity-IDs following this logic:
   - package-related: no changes
   - device related: `ambientika_1_*` → `ambientika_master-1_*`
   - device related: `ambientika_2_*` → `ambientika_slave-2_*`
   - device related: ... _(for each device)_

> [!IMPORTANT]  
> The change of IDs is already considered in the yaml code of this package. Updating your existing package without deleting the entities, might not work and break the package!

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
