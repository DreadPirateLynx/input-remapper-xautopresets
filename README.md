# input-remapper-xautopresets
## Automatic input-remapper preset manager for systems with access to xdotools
Automatically changes the active input-remapper preset for each connected device based on active window's class, with optional support for further differentiating between different windows of the same class based on the window's title. Devices can be configured individually, otherwise they will default to the global configuration file. `input-remapper-xautopresets` makes use of `input-remapper-control` to track devices, so it's aware when devices are connected/disconnected. Configuration files also support live-editing, so no need to restart after making changes; just save the file and xautopresets will know about the changes.

## Installation
  
- Use your preferred method to download either the release or project folder
- Unzip the package as necessary
- Run `/path/to/download/input-remapper-xautopresets/install`
  
The installer will copy all necessary files to their correct locations, and enable/start the required service  
  
Use `systemctl --user [stop|disable|enable|start|status] input-remapper-xautopresets.service` to change service state

## Configuration
Once installed, input-remapper-xautopresets uses the following file tree:
```
~/
|-> .config/
|   |-> input-remapper/
|   |   |-> presets/
|   |   |   |-> [Device Name]/
|   |   |       |-> [Class Name].conf (Optional device-and-class-specific configuration files. Highest priority)
|   |   |       |-> xautopresets.conf (Optional device-specific configuration file. Medium priority)
|   |   |       |-> xautopresets.log  (Device-specific log file)
|   |   |
|   |   |-> xautopresets.conf  (Global configuration file. Lowest priority)    
|   |   |-> xautopresets.log   (Global log file)
|   | 
|   |-> systemd/user/
|       |-> input-remapper-xautopresets.service (systemd user service file)
|     
|-> .local/bin/
    |-> input-remapper-xautopresets (#!/bin/bash core script)
```
### xautopresets.conf
This file contains a list of window classes paired with a list of input-remapper preset names. Preset names should not include the .json extension.  
```
exampleWindowclass1=Example Preset Name 1
exampleWindowclass2=Example Preset Name 1
*xampleWindowclass3=Example Pr?set Name 2
```
`*`/`?` wildcard characters and spaces are valid. Quotation marks, as well asleading and trailing spaces should only be included if actually part of the class/file name.
A default global file is generated during installation or if it can't be found. It contains the following:  
```
_default=_Bypass
Input-remapper-gtk=_Bypass
```  
`_default` is a window class used internally by the script. This is the preset that will be loaded any time no other preset is configured, or if the configured preset cannot be located.  
`_Bypass` is likewise a preset name used internally by the script. This is the preset that will disable input-remapper injection, and will be used in place of the `_default` preset if it can't be found or isn't configured  
`Input-remapper-gtk` is the window class name for `input-remapper`'s gui configuration tool. Injection needs to be stopped to make changes, so this seemed appropriate. Feel free to change.  
The device-specific file is higher priority than the global one. No settings in global file will ever apply to any device with its own file.  
Use `xdotool selectwindow getwindowclassname` to get window class names for your applications

### [Class Name].conf
This file contains a list of window titles paried with a list of input-remapper preset names. It is intended to give the user the ability to have more than one configured preset for a given application, and change between them based on the window's name as shown in the title bar. It is the highest priority configuration file, and if a device has this file in its folder, it will override any settings for [Class Name] found in both the global and device xautopresets.conf files for that device. Same basic syntax as xautoprofile, except:
```
example Window Title=Example Preset Name
```
Use `xdotool selectwindow getwindowname` (or look at the titlebar) to get window title. Wildcards and spaces supported just like with xautopresets.conf

#### Example firedragon.conf
```
_default=FireDragon
*- YouTube -*=Media
```
In this example, when using the FireDragon web browser (Class Name firedragon), any time a browser tab which includes the string "- YouTube -" is active, the "Media" preset will be enabled on this device. For all other tabs, the "FireDragon" preset will be enabled

### input-remapper-xautopresets.service
You can modify `Environment=DISPLAY=:0` to limit which displays the service monitors. Avoid other modifications unless you know what you're doing.
Use `systemctl --user daemon-reload` every time you make changes to `input-remapper-xautopresets.service` 
