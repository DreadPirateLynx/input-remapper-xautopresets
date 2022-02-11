# input-remapper-manager
#!/bin/bash script to automatically change input-remapper profiles based on active window

Copy input-remapper-manager into ~/.local/bin.
Launch from cli or add to shell .rc file.
Suggest launching as daemon (run 'input-remapper-manager&').
Does not currently accept arguments.

After first run, .conf and .log files will be created in ~/.config/input-remapper.
.conf file syntax:
ExampleWindowClass=exampleProfileName
No whitespace allowed.
Supports live update of contents without restarting script.
No other configuration necessary.
Note that "default" is a windowclass used internally by the script. Do not change this classname in the .conf, but the profile name, "Default", can be changed as desired.

Script uses directory tree and input-remapper-control to track devices.
Hotswapping of devices is supported.

Still Needed:
Bypass support to deactivate injection on specific devices.
Detailed documentation.
Attempt to improve efficiancy of conditions logic.
Parallel iteration through devices.
