# input-remapper-xautopresets
## Automatic input-remapper preset manager for systems with access to xdotools
Automatically changes the active input-remapper preset for each connected device based on active window's class, with optional support for further differentiating between different windows of the same class based on the window's title. Devices can be configured individually, otherwise they will default to the global configuration file. `input-remapper-xautopresets` makes use of `input-remapper-control` to track devices, so it's aware when devices are connected/disconnected. Configuration files also support live-editing, so no need to restart after making changes; just save the file and xautopresets will know about the changes.

## Installation
  
- Use your preferred method to download either the release or project folder
- Unzip the package as necessary
- Run `/path/to/download/input-remapper-xautopresets/install`
  
The installer will copy all necessary files to their correct locations, and enable/start the required service  

It is safe to run the installer over a previously installed version
  
Use `systemctl --user [stop|disable|enable|start|status] input-remapper-xautopresets.service` to change service state

Alternatively, `input-remapper-xautopresets` can be used directly as an interface with systemctl:
```
input-remapper-xautopresets v0.9.6
Automatic input-remapper preset manager for systems with access to xdotools

Valid options:
-h        Display this table
-v        Display version number

Valid arguments:
enable|disable|restart|start|status|stop
runs systemctl --user {command} input-remapper-xautopresets.service

Please use 'systemctl --user start input-remapper-xautopresets.service' or provide an appropriate argument to run this program
```

## Uninstallation
`/path/to/download/input-remapper-xautopresets/uninstall`
```
input-remapper-xautopresets uninstaller

Default behavior is to leave configuration files in place

-a|A|c|C    Remove all cofiguration files (including EVERYTHING in all 'classes/' directories) in addition to program files.
-h          Display this menu
```

## Configuration
Once installed, input-remapper-xautopresets uses the following file tree:
```
~/
|-> .config/
|   |-> input-remapper/
|   |   |-> classes/
|   |   |   |-> [Class Name].conf  (Optional global class-specific configuration files. Medium priority)
|   |   |
|   |   |-> presets/[Device Name]/
|   |   |   |-> classes/
|   |   |   |   |-> [Class Name].conf (Optional device-and-class-specific configuration files. Highest priority)
|   |   |   |
|   |   |   |-> xautopresets.conf (Optional device-specific configuration file. High priority)
|   |   |   |-> xautopresets.log  (Device-specific log file)
|   |   |
|   |   |-> active             (Record of currently active profiles on all devices)
|   |   |-> xautopresets.conf  (Global configuration file. Lowest priority)    
|   |   |-> xautopresets.log   (Global log file)
|   | 
|   |-> systemd/user/
|       |-> input-remapper-xautopresets.service (systemd user service file)
|     
|-> .local/bin/
    |-> input-remapper-xautopresets (#!/bin/bash core script)
```
### Configuration File Priority
`input-remapper-xautopresets` will read through each applicable configuration file in order of increasing priority as follows:
- Global `xautopresets.conf`
- Global `[Class Name].conf`
- Device `xautopresets.conf`
- Device `[Class Name].conf`

Later files will override settings in earlier ones. This will allow users to avoid having to repeat settings from lower priority files in higher priority ones.

### xautopresets.conf
This file contains a list of window classes paired with a list of input-remapper preset names. Preset names should not include the .json extension.
```
exampleWindowclass1=Example Preset Name 1
exampleWindowclass2=Example Preset Name 1
*xampleWindowclass3=Example Pr?set Name 2
```
`*`/`?` wildcard characters and spaces are valid. Quotation marks, as well as leading and trailing spaces should only be included if actually part of the class/file name. Entries lower in the list will override earlier ones, so you can use wildcards early for more general matches, and get more specific the further into the configuration file you go for special cases as follows:
```
example*=preset1
*Window*=preset2
exampleWindow*=preset3
*WindowClass4=preset4
exampleWindowClass?=preset5
exampleWindowClass8=preset6
```
 A default global file is generated during installation or if it can't be found. It contains the following:  
```
_default=_Bypass
Input-remapper-gtk=_Bypass
```
`_default` is a window class used internally by the script. The preset assigned to this class will be loaded any time no other preset is configured, or if the configured preset cannot be located. You can change the preset assigned here to whatever you want.  
  
`_Bypass` is likewise a preset name used internally by the script. This is the preset that will disable input-remapper injection, and will be used in place of the `_default` preset if it can't be found or isn't configured  
  
`Input-remapper-gtk` is the window class name for `input-remapper`'s gui configuration tool. Injection needs to be stopped to make changes, so this seemed appropriate. Feel free to change.  
  
Use `xdotool selectwindow getwindowclassname` to get window class names for your applications

### [Class Name].conf
This file contains a list of window titles paried with a list of input-remapper preset names. It is intended to give the user the ability to have more than one configured preset for a given application, and change between them based on the window's name as shown in the title bar. Same basic syntax as xautoprofile, except:
```
example Window Title=Example Preset Name
```
Use `xdotool selectwindow getwindowname` (or look at the titlebar) to get window title. Wildcards and spaces supported just like with xautopresets.conf

#### Example firedragon.conf
```
_default=FireDragon
*- YouTube -*=Media
```
In this example, when using the FireDragon web browser (Class Name `firedragon`), any time a browser tab whose title includes the string `- YouTube -` is active, `Media.json` will be enabled on this device. For all other tabs, `FireDragon.json` will be enabled

### input-remapper-xautopresets.service
You can modify `Environment=DISPLAY=:0` to limit which displays the service monitors. Avoid other modifications unless you know what you're doing.  
  
Use `systemctl --user daemon-reload` every time you make changes to `input-remapper-xautopresets.service` 
