# input-remapper-manager
#!/bin/bash script to automatically change input-remapper profiles based on active window

copy input-remapper-manager into ~/.local/bin.
launch from cli or add to shell .rc file.
suggest launching as daemon (run 'input-remapper-manager&').
does not currently accept arguments.

After first run, .conf and .log files will be created in ~/.config/input-remapper.
.conf file syntax:
exampleWindowClass=exampleProfileName
no whitespace allowed.
supports live update of contents without restarting script.
no other configuration necessary.

script uses directory tree and input-remapper-control to track devices.
hotswapping of devices is supported.

Still Needed:
bypass support to deactivate injection on specific devices.
detailed documentation.
attempt to improve efficiancy of conditions logic.
parallel iteration through devices.
