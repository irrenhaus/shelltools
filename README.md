shelltools
==========

A collection of different shell tools


autorandr
---------

AutoRandR is a tool to automatically apply matching XRandR configurations.
Add the autorandr Bash script to your autostart folder.

It reads a directory with configurations automatically and applies the first one that matches the current setup.
If you set the AUTORANDR_DIR environment variable it uses the directory specified in that environment variable.
If you do not set the variable it looks for a "autorandr.d" subdirectory in the directory where the shell script lives.
It then loads all *.rc files in that directory.

Every *.rc configuration file has to implement two Bash functions named
* config_applies_for
* apply_config
and set an environment variable named AUTORANDR_CONFIG_NAME.

The function config_applies_for has one parameter: A list of available (connected) monitors.
If all required monitors for that configuration are present the function has to return 0.
If not it has to return anything other than 0.

If the configuration matches the AUTORANDR_CONFIG_NAME variable is used to print the text
Configuration $AUTORANDR_CONFIG_NAME can be used. Applying the configuration now...

After that the apply_config function is called.
This function should set all XRandR options needed.

For this there are some helpers:
*   contains_element: Checks if the first given argument (string) exists in the list of all other arguments.
*   all_channels_off: Sets all XRandR channels to "off".
*   all_channels_off_except: Sets all XRandR channels to "off" except the ones given as arguments.
*   configure_channel_off: Configures a single channel (given as the only argument) to "off".
*   configure_channel_on: Configures a single channel to "on". This needs four parameters:
        CHANNEL - The name of the channel to configure
        MODE - The new mode of the channel (for 1080p this would be the string "1920x1080")
        POS - The position of the channel (for the second monitor in a double-1080p-configuration this would be "1920x0")
        ROTATE - The rotation of the channel. Normally this is the string "normal".

An example configuration for a dual-screen 1080p setup:

```bash
AUTORANDR_CONFIG_NAME="Home Extended Desktop (DVI & VGA)"

config_applies_for() {
    contains_element "VGA-0" $@
    local VGA="$?"
    contains_element "DP-0" $@
    local DP="$?"

    if [ "$VGA" -eq "0" ] && [ "$DP" -eq "0" ]; then
        return 0
    fi

    return 1
}

apply_config() {
    all_channels_off
    configure_channel_on "DP-0" "1920x1080" "0x0" "normal"
    configure_channel_on "VGA-0" "1920x1080" "1920x0" "normal"
}
```
