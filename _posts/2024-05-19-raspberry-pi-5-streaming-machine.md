---
layout: post 
title: "🕹️ Raspberry Pi 5 into Streaming Gaming Console"
tags: ['gaming']
---

If you are as excited as I'm when you think about streaming games from your PC to your TV and getting a console-like experience at 4K HDR and 60FPS, then... you better get an Apple TV 4K or Nvidia Shield, and save your time. 😉

I started realising my idea from upgrading my network, by getting a better Wi-Fi PCIe card for my PC and router with WiFi 6. 
Previous setup wasn't able to keep stable 80MB/s transfers, mostly due to weak Wi-Fi signal at my PC. I also had to switch from Raspberry Pi 3b+ to Raspberry Pi 5, because RPI3 doesn't support 4K resolution.

To make sure everything will go smoothing from that point I've also took a case with active cooling, official microHDMI 2.1 cable, and official 27W power adapter.

...and that was just a start of my troubles. At time of writing this post (May 2024) [Raspberry Pi 5 doesn't work nice with moonlight-qt](https://github.com/moonlight-stream/moonlight-qt/issues/1133) and there is no official Steam Link app for ARM64, so I took me a few days of digging to get the setup working. I'm writing this post so you don't have to waste your time.

Here you can find a repository with all the scripts: [https://github.com/jpomykala/moonlight-pi](https://github.com/jpomykala/moonlight-pi)

## Before we start

I assume that you have already done the followings:

Gaming PC side:

- Installed Steam
- Installed Sunshine
- Configured a static address on your router
- Configured Wake-On-Lan (WOL)

Raspberry Pi 5 side:

- Installed Raspbian Bookworm Lite (x64)
- Installed moonlight-qt on Raspberry Pi

I won't cover those topics in this post, as they are straightforward to do. I also assume you have some
basic IT skills to find information on your own, like finding a MAC address, using a command line, bash etc.

## Moonlight and Raspberry Pi 5

Unfortunately, at time I'm writing the post the default configuration that comes with moonlight-qt doesn't play nicely
with Raspberry Pi 5, but way to make it work by creating a custom `egls.json` configuration file.

I've a 4K TV, so in my case the `egls.json` file looks like this:
```
{
  "device": "/dev/dri/card1",
  "outputs": [
    {
      "name": "HDMI1",
      "mode": "3840x2160"
    }
  ]
}
```
For different resolutions you should just change the `mode` value. 

**Important:** Raspberry Pi 5 has 2 microHDMI ports, and the above `egls.json` assumes that you connect your TV to the microHDMI port next to power input. 

Let's run it:

```
QT_QPA_EGLFS_KMS_CONFIG=eglfs.json moonlight-qt
```

Now, you should see a moonlight app on your TV screen, and if you PC is running it should be there. You can use a small cog icon in the top-right corner to adjust the streaming settings to your liking. In my case I've selected 4K resolution and 60FPS with ~80MBs of bitrate. I've set encoders and decoder settings to "Automatic" and I checked "HDR (Experimental)"

## Connecting Xbox Controller

For connecting an Xbox Controller I've used an official Xbox dongle and ['xone' driver](https://github.com/medusalix/xone)
that worked like a charm on a first shot. Please follow the instructions on the GitHub page to install the driver as they might change in the future.

Unfortunately, my previous attempt of connecting Xbox Controller via Bluetooth didn't work for some reason, 
so I just turned off built-in Bluetooth and Wi-Fi, just to save power and decrease the interference using `rfkill block wlan` and `rfkill block bluetooth`.

## Streaming Steam Big Picture

By default, moonlight opens a window where we can select our PC and change the settings. Once we choose a PC then we have to choose an app.
Fortunately, moonlight-qt developers covered a case if we would like to automate the connection. 
Let's start from listing apps that are available on your PC using `moonlight-qt list <YOUR_PC_IP_ADDRESS>`

```
jpomykala@raspberry:~ $ moonlight-qt list 192.168.0.120
00:00:00 - Qt Info: Unable to detect Wayland or X11, so EGLFS will be used by default. Set QT_QPA_PLATFORM to override this.
00:00:00 - Qt Info: Setting display mode by default. Set QT_QPA_EGLFS_ALWAYS_SET_MODE=0 to override this.
00:00:00 - Qt Critical: drmModeGetResources failed (Operation not supported)
Desktop
Steam Big Picture
jpomykala@raspberry:~ $ 
```
By default, moonlight have 2 apps, `Desktop` and `Steam Big Picture`, with this information we create a command that will look like this:

```
QT_QPA_EGLFS_KMS_CONFIG=eglfs.json moonlight-qt stream 192.168.0.120 'Steam Big Picture'
```

to start streaming Steam Big Picture automatically without needing any interaction.

## Waking up the PC

Waking up the PC is a nice part of this post. I've installed `etherwake` package that allows me to send a Magic Packet

```
sudo apt install etherwake
```

Using it is simple as that:

```
etherwake <PC_MAC_ADDRESS>
```

You don't need to add any IP address, mask or anything. It should just work. Here is copy and paste installation command:


## Listening for controller events

This is the most exciting part of this post. Whenever, I turn on my Xbox Controller I wanted to wake up my PC,
and run moonlight to start streaming the Steam Big Picture. For this I've used `udev` and `udev rules`. `Udev` allows 
me to detect whenever my Xbox Controller is connected or disconnected and run some custom scripts.

### Getting controller details

Lets start about getting some details about the controller as we need to know what to look for in `udev` rules. 
Make sure that your controller is connected and run the following command: `ls -l /dev/input`

```
$ ls -l /dev/input/
total 0
drwxr-xr-x 2 root root     100 May 19 14:53 by-path
crw-rw---- 1 root input 13, 64 May 19 13:17 event0
crw-rw---- 1 root input 13, 65 May 19 14:10 event1
crw-rw---- 1 root input 13, 66 May 19 14:10 event2
crw-rw---- 1 root input 13, 67 May 19 14:10 event3
crw-rw---- 1 root input 13, 68 May 19 14:10 event4
crw-rw---- 1 root input 13, 68 May 19 14:10 event5
crw-rw---- 1 root input 13, 63 May 19 13:17 mice
```

This will list all input devices connected to your Raspberry Pi and you should see a few `eventX` files. 
In my case, the controller was the last device, which is `event5`, so let's get more information to see if I was right.
Run the following command:
```
udevadm info --name /dev/eventX --attribute-walk
```

and replace `eventX` with the number you've found in the previous step. In my case it was `event5`:

```
$ udevadm info --name /dev/event5 --attribute-walk
looking at parent device '/devices/platform/axi/1000120000.pcie/1f00300000.usb/xhci-hcd.1/usb3/3-1/3-1:1.0/gip0/gip0.0/input/input8':
    KERNELS=="input8"
    SUBSYSTEMS=="input"
    DRIVERS==""
    ATTRS{capabilities/abs}=="3003f"
    ATTRS{capabilities/ev}=="20000b"
    ATTRS{capabilities/ff}=="107030000 0"
    ATTRS{capabilities/key}=="7cdb000000000000 0 0 0 0"
    ATTRS{capabilities/led}=="0"
    ATTRS{capabilities/msc}=="0"
    ATTRS{capabilities/rel}=="0"
    ATTRS{capabilities/snd}=="0"
    ATTRS{capabilities/sw}=="0"
    ATTRS{id/bustype}=="0006"
    ATTRS{id/product}=="02ea"
    ATTRS{id/vendor}=="045e"
    ATTRS{id/version}=="0408"
    ATTRS{inhibited}=="0"
    ATTRS{name}=="Microsoft Xbox Controller"
```

Look at line `ATTRS{name}=="Microsoft Xbox Controller"` is what we are looking for, this is the name of the controller that we are going to use in our `udev` rule.

### Writing `udev` rules

Now, we are going to create a rule that will execute a script whenever the controller is connected or disconnected. Create a file for a new rules:
```
sudo vim /etc/udev/rules.d/99-xbox-controller.rules
```

and add the following content:
```
ACTION=="add", SUBSYSTEM=="input", ATTRS{name}=="Microsoft Xbox Controller", RUN+="/home/jpomykala/moonlight/controller-connected.sh"
ACTION=="remove", SUBSYSTEM=="input", ATTRS{name}=="Microsoft Xbox Controller", RUN+="/home/jpomykala/moonlight/controller-disconnected.sh"
```

The first rule, will execute script at `/home/jpomykala/moonlight/controller-connected.sh` whenever the controller is connected, and it's name is `Microsoft Xbox Controller`.
The second rule will execute `/home/jpomykala/moonlight/controller-disconnected.sh` whenever the controller is disconnected.

The rules won't work until we reload them, but before we do this, let's create those scripts.
 

### Logging and debugging

Before we start writing scripts, lets create a place where we can store logs, in my case I've chosen: `/var/log/xbox-controller.log`:

```
touch /var/log/xbox-controller.log
chmod +0777 /var/log/xbox-controller.log
```

I've set the permissions to 0777, so I can do anything with this file, but you can set it to 0644 if you want to be more restrictive.

### When controller connects

[`controller-connected.sh` script](https://github.com/jpomykala/moonlight-pi/blob/main/controller-connected.sh) does a few things:

- creates a lockfile, because `udev` event fires 4 times, when my controller is connecting for some reason
- wakes my PC
- waits until I get a ping from my PC
- checks if the `moonlight-qt` is already running, and if not start it
- deletes a lockfile

### When controller disconnects

[`controller-disconnected.sh` script](https://github.com/jpomykala/moonlight-pi/blob/main/controller-disconnected.sh) is much shorter as it only force kills the moonlight. It also requires a lockfile as the rule we've created fires 2 times. 

### Wrapping up

Remember to make all files executable via:

```
chmod +x controller-connected.sh
chmod +x controller-disconnected.sh
```

Now, let's wrap this up and enable the rule by reloading them:

```
sudo udevadm control --reload-rules
```

To browse the logs use:
```
tail -f /var/log/xbox-controller.log
```

You can also check logs for `udev` via journalctl:
```
sudo journalctl -u systemd-udevd -f
```

Now your Raspberry Pi should wake up your PC and start streaming Steam Big Picture whenever you connect your Xbox Controller
and stop the streaming whenever you disconnect it. 🥳

## Further work

I plan to improve the final solution in a few ways:

- Wake on Lan part got naive assumptions; Whenever I've got ping from the PC I just wait a couple of seconds to make sure the Sunshine is ready to stream, this could be improved.
- Turning off PC; The current solution doesn't turn off the PC at all, so this could be added. I didn't find a way yet to do this without installing additional software on Windows.
- CEC and TV integration, I plan to use 'cec-utils' to automatically switch the TV output to the Raspberry Pi. For some reason, the CEC commands do nothing when I was executing them against my TV, so I give up for a moment.  

Additionally, I wanted to add a small speaker to the Raspberry Pi to play a sound when I launch 'moonlight-qt'. Furthermore, I would like to start some boot animation to make the startup process more visually appealing. This will add a more console feeling.
