# input-remapper-xautopresets
## Automatic input-remapper preset manager for systems with access to xdotool and xprop
Automatically changes the active input-remapper preset for each connected device based on active window's class, with optional support for further differentiating between different windows of the same class based on the window's title. Devices can be configured individually, otherwise they will default to the global configuration. `input-remapper-xautopresets` makes use of `input-remapper-control` to track devices, so it's aware when devices are connected/disconnected. The configuration file also support live-editing, so no need to restart after making changes; just save the file and xautopresets will know about the changes.

## Installation
  
- Use your preferred method to download either the release or project folder
- Unzip the package as necessary
- Run `/path/to/download/input-remapper-xautopresets/install`
  
The installer will copy all necessary files to their correct locations, and enable/start the required service  

It is safe to run `install` over any previously installed/currently running version. If an update is incompatible with previous configuration files, `install` will automatically import and convert them without modifying the original files.

## Usage
### input-remapper-xautopresets

Use `systemctl --user [enable|disable|restart|start|status|stop] input-remapper-xautopresets.service` to change service state

Alternatively, `input-remapper-xautopresets` can be used directly as an interface for `systemctl`:
```
input-remapper-xautopresets v1.1.2
Automatic input-remapper preset manager for systems with access to xdotool

Valid options:
-h        Display this table
-v        Display version number

Valid arguments:
[enable|disable|restart|start|status|stop]:
    runs systemctl --user [command] input-remapper-xautopresets.service

getwindowinfo:
    Select a window with the mouse and print the window's class and title

Example useage:
    input-remapper-xautopresets start
    input-remapper-xautopresets getwindowinfo

Either provide a vaild argument or envoke 'systemctl' directly.
```
`input-remapper-xautopresets` will not allow you to run it directly. It must be controlled through `systemctl`, either directly, or through the interface provided by `input-remapper-xautopresets`

### xautopresets-config
Currently only capable of creating a new xautopresets.ini file, generating it from imported .conf data from previous versions if it exists, or else generating a default one if not.
```
xautopresets-config v0.1.5
Configuration file manipulator for xautopresets

Valid options:
-h        Display this table
-n        Generate new default xautopresets.ini file,
          importing data from previous versions if available
```

## Uninstallation
`/path/to/download/input-remapper-xautopresets/uninstall`
```
input-remapper-xautopresets uninstaller

Uninstaller MUST be passed one or more options; it will only remove what you tell it

-c          Remove configuration file, 'xautopresets.ini'
-h          Display this menu
-l          Remove all log files
-s          Remove script and service files
-o          Remove all configuration files from old (pre-1.0) versions

```
`uninstall` requires additional confirmation for each category in case you made a mistake. `uninstall -clso` will remove everything

## Configuration
Once installed, input-remapper-xautopresets uses the following file tree:
```
~/
|-> .config/
|   |-> input-remapper/
|   |   |-> logs/
|   |   |   |-> .active             (Temporary record of currently active profiles on all devices)
|   |   |   |-> [Device Name].log   (Device-specific log file)
|   |   |   |-> xautopresets.log    (Global log file)
|   |   |
|   |   |-> xautopresets.ini    (Configuration file)
|   | 
|   |-> systemd/user/
|       |-> input-remapper-xautopresets.service (systemd user service file)
|     
|-> .local/bin/
    |-> input-remapper-xautopresets (#!/bin/bash core script)
    |-> xautopresets-config         (#!/bin/bash config-file manipulation script)
```
### Reserved Values
`_DEFAULT` is a window class used internally by the script. The preset assigned to this class will be loaded any time no other preset is configured, or if the configured preset cannot be located.

`_STOP` is a preset name used internally by the script. This is the preset that will disable `input-remapper` injection.

`_IGNORE` is a preset name used internally by the script. This preset tells `xautopresets` to ignore the window change and do nothing.

### xautopresets.ini
Typical .ini format with section headers in square brackets ('[]'), and semicolon comments (';')


##### A Note on how the script works
For every window change, there are a total of four (4) configuration sections that may be relevant to each device should they exist: `[GLOBAL]`, `[GLOBAL:<CLASSNAME>]`, `[<DEVICENAME>]`, & `[<DEVICENAME>:<CLASSNAME>]`. For each device it is managing, `xautopresets` will:
1. Grab these four sections
2. Order them in a list of increasing priority
    - `[GLOBAL]` is the lowest priority and read first.
    - `[<DEVICENAME>:<CLASSNAME>]` is the highest and read last.
    - The priority of the other two sections is controlled with the `_SECTIONPRIORITY` setting in the `[PROGRAM]` section.
3. Trim the list to include only `_DEFAULT`, `<CLASSNAME>` and `<WINDOWWTITLE>` matches.
4. Sort it based on the `_DEFAULTPRIORITY` setting.
5. Reverse the list.
6. Check it for the first preset that exists on disk (including the internal presets, `_STOP` and `_IGNORE`).
7. Invoke `input-remapper-control` to set the device to it (or ignore the change if appropriate).

#### [PROGRAM]
```
; These settings control the behavior of xautopresets.
; Valid settings shown in parentheses; first shown is default.

_EXECUTION=_PARALLEL
; ( _PARALLEL | _SERIAL )
; Controls whether device configuration occurs in parallel or in series.
; Parallel processing has increased overhead, so depending on the number
; of devices you're controlling, it may be quicker to disable it.

_NOTSET=_STOP
; ( _STOP | _IGNORE )
; Final fallback behaivor if appropriate preset can't be found

_SECTIONPRIORITY=_DEVICE
; ( _DEVICE | _CLASS )
; Controls whether '[<DEVICENAME>]' or '[GLOBAL:<CLASSNAME>]'
; section has higher priority

_DEFAULTPRIORITY=_SECTION
; ( _SECTION | _CLASS )
; Controls whether xautopresets applies a default preset from a higher
; priority section before a class-name match from a lower one.
; Setting this to _CLASS means a <CLASSNAME> match in [GLOBAL] will be
; preferred over a _DEFAULT match in [<DEVICENAME>:<CLASSNAME>]
```

#### [IGNORE]
Uncommented devices listed in this section will not be managed by xautopresets

#### [GLOBAL] & [DEVICENAME]
These sections contain a list of window classes paired with a list of input-remapper preset names. Preset names should not include the `.json` extension.
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
 A default global file is generated during installation or if it can't be found. It contains the following, unless data from previous version .conf files was imported:
```
_DEFAULT=_STOP
Input-remapper-gtk=_STOP
```
  
`Input-remapper-gtk` is the window class name for `input-remapper`'s gui configuration tool. Injection needs to be stopped to make changes, so this seemed appropriate. Feel free to change.  
  
Use `input-remapper-xautopresets getwindowinfo` to get window class names for your applications. It is highly recommended that you copy and paste from here into your .ini file.

#### [GLOBAL:CLASSNAME] & [DEVICENAME:CLASSNAME]
These sections contain a list of window titles paried with a list of input-remapper preset names. It is intended to give the user the ability to have more than one configured preset for a given application, and change between them based on the window's name as shown in the title bar. Same basic syntax as `[GLOBAL]` and `[<DEVICENAME>]`, except:
```
example Window Title=Example Preset Name
```
Use `input-remapper-xautopresets getwindowinfo` to get window titles, and once again, copy/pasting from here is advised. Wildcards and spaces supported just like with `xautopresets.conf`.

##### Example [GLOBAL:firedragon]
```
_default=FireDragon
*- YouTube -*=Media
```
In this example, when using the FireDragon web browser (Class Name `firedragon`), any time a browser tab whose title includes the string `- YouTube -` is active, `Media.json` will be enabled on this device. For all other tabs, `FireDragon.json` will be enabled

### input-remapper-xautopresets.service
You can modify `Environment=DISPLAY=:0` to limit which displays the service monitors. Avoid other modifications unless you know what you're doing.  
  
Use `systemctl --user daemon-reload` every time you make changes to `input-remapper-xautopresets.service` 
