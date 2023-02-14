---
layout: post
title: Amiibo Emulation with JoyControl using a Linux VM.
author: Maxi
tags: [2023, guide, gaming]

---

These instructions will help you set up [joycontrol](https://github.com/Poohl/joycontrol.git) on any PC* using a Virtual Machine Software like VirtualBox and a Bluetooth Dongle.

- *This guide is compatible with PCs and Laptops (both for ARM64 and x86_64).
- *Compatible with MacBooks (using Parallels and a Linux distribution).

## Setting up the Virtual Machine
In this tutorial, Personally I will use Ubuntu [20.04.2](http://old-releases.ubuntu.com/releases/20.04.2/ubuntu-20.04.2.0-desktop-amd64.iso) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 
- You can also try on [VMWare Player](https://www.vmware.com/products/workstation-player.html) and [Parallels for macOS](https://www.parallels.com/products/desktop/) but this section won't cover the installation of a Virtual Machine on those softwares for now.

Here's also an official tutorial by Canonical on how to make a VM using VirtualBox: [here](https://ubuntu.com/tutorials/how-to-run-ubuntu-desktop-on-a-virtual-machine-using-virtualbox#2-create-a-new-virtual-machine)
- In VirtualBox, select your ISO.![Selecting Ubuntu ISO](https://cdn.discordapp.com/attachments/1055972618286669925/1074034541452333146/Screenshot_2023-02-11_at_15.26.57.png "Selecting Ubuntu ISO")

- Set the VM's hardware (we don't need more than 4 GBs of RAM and 4 CPUs of processor, since we will be using it for Amiibos).
![Setting VM hardware](https://cdn.discordapp.com/attachments/1055972618286669925/1074034541196484760/Screenshot_2023-02-11_at_15.27.53.png "Setting VM hardware")

- Set at least 20-25 GBs of VHD, or it may not be enough for Ubuntu.
![Setting VM hardware](https://cdn.discordapp.com/attachments/1055972618286669925/1074034540949033030/Screenshot_2023-02-11_at_15.28.00.png "Setting VM hardware")

- To summarize:
![Summary](https://media.discordapp.net/attachments/1055972618286669925/1074034540554752000/Screenshot_2023-02-11_at_15.28.03.png "Summary")

- Then press start (the green arrow).
![Summary](https://cdn.discordapp.com/attachments/1055972618286669925/1074034540181475328/Screenshot_2023-02-11_at_15.28.10.png "Summary")

After booting to Ubuntu, you will be greeted to a 'Welcome' screen and two options, 'Try Ubuntu' and 'Install Ubuntu', you will click on the second one.

> After finishing the install wizard (the minimal version of your distribution is enough) you should have a functional Linux VM. (if you encounter problems until here, you can simply google them).

## Using [bdaddr](https://github.com/thxomas/bdaddr.git)

1. Install some dependencies for the whole guide, used by [bdaddr](https://github.com/thxomas/bdaddr.git) itself and [joycontrol](https://github.com/Poohl/joycontrol.git)

```bash
sudo apt update
sudo apt install python3-dbus libhidapi-hidraw0 libbluetooth-dev bluez git python3-pip
sudo apt install libbluetooth-dev
sudo pip3 install aioconsole hid crc8
```

- For Ubuntu ≤20.04.2 users, they should be using `hid==1.0.4` instead of 1.0.5

```bash
sudo pip3 install hid==1.0.4
```

2. Clone the [bdaddr](https://github.com/thxomas/bdaddr.git) repository, since we will be compiling it.

```bash
git clone https://github.com/thxomas/bdaddr.git
cd bdaddr
make
```

- 'make' should output this as a proof of a successful compile.

```bash
gcc -c bdaddr.c
gcc -c oui.c
gcc -o bdaddr bdaddr.o oui.o -lbluetooth
```

After cloning and compiling, there's our file! You should end up with a file called 'bdaddr', that you can execute by running `./bdaddr` :)

- Optionally we can install it onto our /usr/local/bin folder.

```bash
sudo install bdaddr /usr/local/bin
```

When its run, it should output something like:

```bash
Manufacturer: Foo Corporation (x)
Device address: XX:XX:XX:XX:XX:XX
```

- If 'bdaddr' outputs `Unsupported manufacturer` then, your dongle may not be compatible with 'bdaddr' and you should follow this guide: [Poohl/joycontrol#4](https://github.com/Poohl/joycontrol/issues/4)

## After the installation of [bdaddr](https://github.com/thxomas/bdaddr.git)

- Disclaimer: This only affects the VM itself, no changes are being made to your device (dongle).

1. Change your Bluetooth Address to "94:58:CB:" since this type of MAC "is in Nintendo's range" and may help to establish a more stable connection.
   - It appears that this fixes most Switch V12+ connection issues.

```bash
sudo bdaddr -i hci0 "94:58:CB:44:55:66"
```

2. Edit the bluetooth.service to change the ExecStart variable.

```bash
sudo nano /lib/systemd/system/bluetooth.service
```

```bash
ExecStart=/usr/lib/bluetooth/bluetoothd -C -P sap,input,avrcp
```

- This will break compatibility with bluetooth controllers, headphones etc. and will stop working due to lack of drivers.
- Make sure to save your changes! (⌃O ⌃X)

Then restart your Bluetooth Interface.

```bash
sudo hciconfig hci0 reset
sudo systemctl daemon-reload
sudo systemctl restart bluetooth.service
```

3. The 'bdaddr' command will now reflect your new and shiny Bluetooth MAC Address.

```bash
Manufacturer: Foo Corporation (x)
Device address: 94:58:CB:44:55:66
```

## Using [joycontrol](https://github.com/Poohl/joycontrol.git)

Now to the thing itself:

1. Clone the [joycontrol](https://github.com/Poohl/joycontrol.git) repository.

```bash
git clone https://github.com/Poohl/joycontrol.git
cd joycontrol
```

2. Now it should be ready to run!

```bash
sudo python3 run_controller_cli.py PRO_CONTROLLER
```

- Go to the 'Change Grip Order' menu, wait for the application to recognize the 'PRO_CONTROLLER'.
- It may take a while for the Switch to catch the Bluetooth connection from [joycontrol](https://github.com/Poohl/joycontrol.git).

Use `-r auto` to reconnect to the Switch automatically.

```bash
sudo python3 run_controller_cli.py PRO_CONTROLLER -r auto
```

## Usage

After 'joycontrol' pairs to the Nintendo Switch, press ENTER.
Something like `cmd >>` should appear in the screen.
From that interface, you can press buttons, import NFC backups (read and write), and much more.

- A lot of warnings may appear during connection of controller to the Switch, do not be worried about them, most of them are debug logs.

```
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.016394599278767904 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.03600834210713705 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.03544424374898275 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.0028030713399251304 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.03562281926472982 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.003548367818196615 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.035586579640706384 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.003255112965901693 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.03640435536702474 s too slow!
[16:15:12] joycontrol.protocol _writer::181 WARNING - Code is running 0.03431223233540853 s too slow!
```

To import 'Amiibos' or 'NFC backups':

```
cmd >> nfc /path/to/directory/file.bin
```

- This should output `added nfc content`.

To navigate the Switch using commands, you may use `a, b, x, y, up, down, left, right, capture, home, minus, plus, l, r, zl, zr, l_stick, r_stick`. 
- You may find more options in the `help` command in the `cmd >>` prompt.

## Demo

![A demonstration of joycontrol running on a Linux VM on a MacBook Air M1](https://media.discordapp.net/attachments/1020042420882247773/1073683102670520361/image.png "A demonstration of joycontrol running on a Linux VM on a MacBook Air M1")

# FAQ
#### About Parallels
> If you don't know how to install a Linux VM in Parallels, please take a look at this official guide: https://kb.parallels.com/128445

#### My setup:
My laptop is a MacBook Air M1, I used Parallels 17 with Ubuntu 20.04.2 in which I connected a 3.0 Bluetooth Dongle with a Broadcom Chip (BCM2070), then I followed this guide. As you can see in the demo it worked perfectly. :)

# Acknowledgements

Thank you to the Joy-Con Droid Discord and the community that made this possible and made the collection of all the relevant information for this guide much easier! :)

- Check Joy-Con Droid's [Discord](https://discord.gg/5SFhf5C) for more help and support.
- Check the [#common-bugs](https://discord.com/channels/622545628131491860/1020042420882247773) channel for troubleshooting: 
- Check this guide [Poohl/joycontrol#4](https://github.com/Poohl/joycontrol/issues/4) for more information about bluetooth compatibility: 
- I also took a bunch of information from this gist: [Colemickens' Guide](https://gist.github.com/colemickens/b08d1a339f4483c6b3c08e739d6cf5d0)

This guide is algo on GitHub Gists!
[fixmyblock/Amiibo-Emulation-with-JoyControl-using-a-Linux VM.md](https://gist.github.com/fixmyblock/5e72775f2fef9247461e9f1e4ec790a8)

:D
\- Maxi