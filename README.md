# Arch Linux Dual Boot Windows Bluetooth Installation & Configuration

## Why pairing ?

> Basically, when you pair your device, your Bluetooth service generates a unique set of pairing keys. First, your computer stores the Bluetooth device's MAC address and pairing key. Second, your Bluetooth device stores your computer's MAC address and the matching key. This usually works fine, but the MAC address for your Bluetooth port will be the same on both Linux and Windows (it is set on the hardware level). Thus, when you re-pair the device in Windows or Linux and it generates a new key, that key overwrites the previously stored key on the Bluetooth device. Windows overwrites the Linux key and vice versa.
[Mario Olivio Flores](https://unix.stackexchange.com/questions/255509/bluetooth-pairing-on-dual-boot-of-windows-linux-mint-ubuntu-stop-having-to-p)


## Install packages

```shell
> $ pacman -S bluez bluez-utils pulseaudio-bluetooth chntpw
```

## Activate Bluetooth
```shell
> rfkill list
0: phy0: Wireless LAN
        Soft blocked: no
        Hard blocked: no
1: hci0: Bluetooth
        Soft blocked: no
        Hard blocked: no

# if blocked :
> rfkill unblock all

> $ systemctl enable bluetooth.service 
> $ systemctl start bluetooth.service
> pulseaudio --start
```

## Get the pairing key

1. Mount your Windows partition

```shell
> $ mkdir /mnt/win
> $ mount /dev/sdXY /mnt/win
```

2. Navigate to your System32 config folder and start `chntpw`

```shell
> cd Windows/System32/config

> chntpw -e SYSTEM

> cd CurrentControlSet\Services\BTHPORT\Parameters\Keys
# if there is no CurrentControlSet, then try ControlSet001

# Shows your Bluetooth port s MAC address
(...)\Services\BTHPORT\Parameters\Keys> ls
Node has 1 subkeys and 0 values
  key name
  <a1234567abcd>

> cd a1234567abcd

> ls
# lists the existing device s MAC addresses
Node has 0 subkeys and 1 values
    16  3 REG_BINARY         <123456abcdef>

> hex 123456abcdef
:00000  XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX ..ignore.chars..

# XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX is your pairing key
```

3. Update the pairing key
```shell
>$ su -

$> cd /var/lib/bluetooth/

$> ls
# Your PC s MAC address
AA:BB:CC:DD:11:22

$> cd AA:BB:CC:DD:11:22

$> ls
# Your device s MAC Adress
XX:YY:ZZ:77:88:99  ...  cache  settings

$> cd XX:YY:ZZ:77:88:99

$> vim info

...
[LinkKey]
Key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX # Here replace by the pairing key
...

# Restart bluetooth
> $ systemctl restart bluetooth
```

## Connect to your device using `bluetoothctl`

```shell
> bluetoothctl

[bluetooth] default-agent

[bluetooth] scan on
  Discovery started
  [CHG] Controller AA:BB:CC:DD:11:22 Discovering: yes
  [NEW] Device XX:YY:ZZ:77:88:99 DEVICE_NAME

[bluetooth] trust XX:YY:ZZ:77:88:99
  Changing XX:YY:ZZ:77:88:99 trust succeeded

[bluetooth] connect XX:YY:ZZ:77:88:99
  Attempting to connect to XX:YY:ZZ:77:88:99
  [CHG] Device XX:YY:ZZ:77:88:99 Connected: yes
```

## Links
[Bluetooth DualBoot Pairing](https://wiki.archlinux.org/title/bluetooth#For_Windows)

[Bluetooth Pairing on Dual Boot of Windows & Linux Mint/Ubuntu - Stop having to Pair Devices](https://unix.stackexchange.com/questions/255509/bluetooth-pairing-on-dual-boot-of-windows-linux-mint-ubuntu-stop-having-to-p)

[PulseAudio](https://wiki.archlinux.fr/PulseAudio)