# Linux-tablet-mode-fix

Fix for disabling keyboard and touchpad in tablet mode

## Explanation

For some 2-in-1 laptops (e.g., Lenovo Yoga 7i), GNOME handles tablet mode fairly well (flip screen and enable virtual keyboard). However, keyboard and touchpad keep active even in tablet mode (not the desirable behaviour if you want to use the stylus or fully open the screen).

The instructions above help creating two ACPI event callbacks for when we enter tablet mode, disabling and enabling mouse and keyboard automatically.

This solution was tested in a [Lenovo Yoga 7i](https://www.lenovo.com/ca/en/p/laptops/yoga/yoga-2-in-1-series/yoga-7-14itl5/88ygc701456) running [Pop!_OS 22.04](https://pop.system76.com/). It should also work for Ubuntu 22.04 and variants (althout it was not tested in anything other than Pop!_OS).

## Instructions

### Create the ACPI event files

```bash
cat <<- "EOF" |  sudo tee /etc/acpi/events/lenovo-enable-tablet-mode
event=video/tabletmode TBLT 0000008A 00000001
action=/etc/acpi/disable-keyboard-touchpad.sh
EOF
```

```bash
cat <<- "EOF" |  sudo tee /etc/acpi/events/lenovo-disable-tablet-mode
event=video/tabletmode TBLT 0000008A 00000000
action=/etc/acpi/enable-keyboard-touchpad.sh
EOF
```

### Create the scripts to run xinput according to ACPI events

```bash
cat <<- "EOF" |  sudo tee /etc/acpi/disable-keyboard-touchpad.sh
#!/bin/bash

# Warning: xinput needs and XAUTHORITY environment variables
# Here we are setting them manually. Change those if necessary
export DISPLAY=":1"
export XAUTHORITY="/run/user/1000/gdm/Xauthority"

# Disable keyboard and touchpad in tablet mode
# Warning: You may need to replace the grep argument by your own kerboard string
xinput | grep 'Touchpad' | cut -d '=' -f 2 | cut -f 1 | xargs xinput disable
xinput | grep 'AT Translated Set 2 keyboard' | cut -d '=' -f 2 | cut -f 1 | xargs xinput disable
EOF
```

```bash
cat <<- "EOF" |  sudo tee /etc/acpi/enable-keyboard-touchpad.sh
#!/bin/bash

# Warning: xinput needs and XAUTHORITY environment variables
# Here we are setting them manually. Change those if necessary
export DISPLAY=":1"
export XAUTHORITY="/run/user/1000/gdm/Xauthority"

# Enable keyboard and touchpad when leaving tablet mode
# Warning: You may need to replace the grep argument by your own kerboard string
xinput | grep 'Touchpad' | cut -d '=' -f 2 | cut -f 1 | xargs xinput enable
xinput | grep 'AT Translated Set 2 keyboard' | cut -d '=' -f 2 | cut -f 1 | xargs xinput enable
EOF
```

### Set scripts to executable

```bash
sudo chmod +x /etc/acpi/enable-keyboard-touchpad.sh /etc/acpi/disable-keyboard-touchpad.sh
```

### Restart ACPI service

```bash
sudo systemctl restart acpid.service
```
