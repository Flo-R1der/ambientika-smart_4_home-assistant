
# Change Log
All notable changes to this project will be documented in this file.  
You can also take alook into the [Milestones](https://github.com/Flo-R1der/ambientika-smart_4_home-assistant/milestones?state=closed) as I'm doing the detailed documentation there.


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
