# lenovo-15ACH6H
Installing Linux on Lenovo Legion 5 15ACH6H


# Secure Boot
If you have secure boot enabled in UEFI, it is required to install and deploy the machine owner keys properly. Otherwise, NVIDIA drivers (including HDMI output) will not work. To do that, Linux installer on 1st reboot launches the MOK installer from UEFI to import the keys. The problem is, that the keyboard in MOK util program does not work, it does not react to any keys (including external USB keyboard) and you have to hard reset the computer and proceed without secure boot.

Fix:
1. If you are installing new Ubuntu or Mint installation, proceed with it. During installation, mark "secure boot" and choose a password for the private key, then proceed with the rest of installation. Before first reboot, go to point 3

2. If you already rebooted your system and now you have a broken installation:
Boot your system, in terminal run:
```
cd /var/lib/shim-signed/mok
sudo mokutil --import MOK.der
```
Choose and enter new password for it, then reboot and proceed with next point

3. During rebooting, repeatedly hit F2 to enter UEFI setup

4. Do not change anything in setup, immediately exit

5. Now, the keyboard in MOK util is alive and you can confirm key import. Enter the previously chosen password. Done.

# HDMI not working
Use Ubuntu drivers program to install NVIDIA drivers - I used nvidia-550.
If, after reboot, they don't work and syslog shows error, that it cannot communicate with the kernel - read the chapter above about secure boot.

# Cannot enable Bluetooth
If the GUI control panel cannot enable bluetooth (nothing happens), try from command line: `rfkill unblock bluetooth`

# Enable battery cap
The laptop can limit the battery charging to 60%, when used mostly on AC, to conserve battery cycles. In Windows, it is controlled via Lenovo Vantage utility.
In Linux, to enable battery cap:

`sudo echo 1 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode`

To disable:
`sudo echo 0 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode`

# In dual boot, time is off by several hours and fixes itself after about 1min after startup
- Set Windows installation to treat BIOS clock (RTC) as UTC. Import the following registry key:
```
﻿Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation]
     "RealTimeIsUniversal"=hex(b):01,00,00,00,00,00,00,00
```

- reboot the Windows. After reboot, clock will be invalid. Correct it e.g. by updating it from internet time
- reboot to Linux and type `sudo timedatectl set-local-rtc 0`

# Touching USB mouse wakes up from standby
If this is unintended behavior, follow the advice from [Mint Forum](https://forums.linuxmint.com/viewtopic.php?t=413168), to create service that commands USB to ignore wakeup:
```
lspci | grep USB | cut -d " " -f 1 > ~/t7s1 \
&& cat /proc/acpi/wakeup > ~/t7s2 && grep -f ~/t7s1 ~/t7s2 > ~/t7s3 \
&& cat ~/t7s3 | cut -c 1-4 > ~/t7s1 \
&& sed -i -e 's|^|echo "|' -e 's|$|" > /proc/acpi/wakeup|' ~/t7s1 \
&& tr -d '\t' < ~/t7s1 > ~/t7s2 && sed -i '1s|^|#!/bin/sh\n|' ~/t7s2 \
&& sudo cp -f ~/t7s2 /usr/local/bin/nowusb.sh \
&& sudo chmod +x /usr/local/bin/nowusb.sh && rm -f ~/t7s1 ~/t7s2 ~/t7s3 \
&& printf '%s\n' '[Unit]' 'Description=no-wakeup-usb' '[Service]' \
'ExecStart=/usr/local/bin/nowusb.sh' '[Install]' \
'WantedBy=multi-user.target' \
| sudo tee /etc/systemd/system/no_wakeup_usb.service \
&& sudo systemctl enable no_wakeup_usb.service
```
