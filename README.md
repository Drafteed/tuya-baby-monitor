# Tuya / Acebell Baby Monitor (TL-501) Rooting and Customization

This repository provides information and a method to obtain root access on **Tuya / Acebell WiFi 2K Baby Monitor (TL-501 / TL-BM501D / JST31)** devices based on the Ingenic T31X SoC.

<img src="/images/device.jpg" width="500" alt="Acebell WiFi 2K Baby Monitor">

## ⚠ Warning

Proceed only if you fully understand what you are doing. Improper use may permanently damage your device.

Depending on your network configuration, this project may expose your camera directly to the public internet. You are solely responsible for securing and properly configuring your network. Make sure you understand the potential risks before continuing.

## Features

* No hardware modifications required
* Easy uninstall - simply remove files from the SD card
* Official mobile application remains fully functional
* Telnet/SSH access with SFTP support
* Local RTSP/WebRTC/HLS/MP4 streaming via [ingenic-stream-hack](https://github.com/Drafteed/ingenic-stream-hack) + [go2rtc](https://github.com/AlexxIT/go2rtc)
* Basic local PTZ control

## Installation

1. Download the repository (clone it or [download the ZIP archive](https://github.com/Drafteed/tuya-baby-monitor/archive/refs/heads/main.zip));
2. Prepare a microSD card and format it as **FAT32**;
3. Copy all files from the `SD_ROOT` directory to the root of the microSD card;
4. Power off the camera, insert the microSD card, then power it back on;
5. The camera should boot normally and perform its motor self-test;
   If everything booted correctly, the startup melody will **not** play after the motor test.

> Note: In some cases the camera may not detect the SD card immediately after power-on.
> If this happens, reboot the camera using the official Tuya mobile app.
> It is also recommended to disable video recording to the SD card.

## Root Access

### SSH / SFTP

```sh
ssh root@CAMERA_IP
```

### Telnet

```sh
telnet CAMERA_IP
```

## Local RTSP Stream

The device uses [ingenic-stream-hack](https://github.com/Drafteed/ingenic-stream-hack) to create a local **RAW TCP video stream (video only, no audio)** on port `12345`.

To convert this stream into RTSP/WebRTC/HLS/MP4/etc., it is recommended to use an external instance of [go2rtc](https://github.com/AlexxIT/go2rtc), for example running on a home server.

> Although go2rtc binaries are included in the `bin` directory and can theoretically run directly on the camera, this is **not recommended** due to limited device resources.

### Example `go2rtc` configuration

```yaml
streams:
  baby_monitor_local: tcp://192.168.1.25:12345/
```

Replace `192.168.1.25` with your camera's IP address.

## Local PTZ Control

Camera position (PTZ) is controlled via a [simple shell script](https://github.com/Drafteed/tuya-baby-monitor/blob/main/SD_ROOT/bin/ptz).

### Usage

```bash
ptz left|right|up|down|stop [steps] [delay_us]
```

### Examples

```bash
ptz up
ptz left
ptz left 100 4
ptz down 400 3
```

### Network control example

```sh
echo "right 100 4" | nc 192.168.1.25 2323
```

### Home Assistant example

```yaml
shell_command:
  baby_monitor_ptz: 'sh -c "echo \"{{ direction }} {{ steps }} {{ delay }}\" | nc 192.168.1.25 2323"'
```

Replace `192.168.1.25` with your camera's IP address.

## Home Assistant WebRTC Camera

You can use the Home Assistant [WebRTC Camera](https://github.com/AlexxIT/WebRTC) integration to display the stream card.

### Custom card example

```yaml
type: custom:webrtc-camera
url: baby_monitor_local
style: "video {aspect-ratio: 16/9; object-fit: fill;}"
background: true
muted: true
ptz:
  service: shell_command.baby_monitor_ptz
  data_left:
    direction: left
    steps: 100
    delay: 4
  data_right:
    direction: right
    steps: 100
    delay: 4
  data_up:
    direction: up
    steps: 100
    delay: 4
  data_down:
    direction: down
    steps: 100
    delay: 4
```

## Security Notes

⚠ **Important:** By default, both Telnet and SSH access are enabled **without a password**.

It is **strongly recommended** to secure your device immediately after obtaining root access.

### Add your SSH public key

Add your own SSH public key to:

```
/mnt/extsd/opt/wz_mini/etc/ssh/authorized_keys
```

Need help? Here's a [guide on using public key authentication](https://averagelinuxuser.com/how-to-use-public-key-authentication/).

### Disable password authentication in Dropbear

Edit the file:

```
/mnt/extsd/baby_monitor
```

Replace:

```sh
dropbear -B -R
```

With:

```sh
dropbear -R -s -g
```

This disables password authentication and allows login **only via SSH keys**.

### Disable Telnet (recommended)

To disable Telnet access, comment out the following lines:

```sh
telnetd -l /bin/sh &
```

Securing SSH and disabling Telnet is highly recommended, especially if your camera is accessible outside your local network.

## Special thanks

Thanks to [gtxaspec](https://github.com/gtxaspec) and the [wz_mini_hacks](https://github.com/gtxaspec/wz_mini_hacks) project for the inspiration and for some of the binaries used in this project.

Thanks to [AlexxIT](https://github.com/AlexxIT/go2rtc) for the [go2rtc](https://github.com/AlexxIT/go2rtc) and [WebRTC Camera](https://github.com/AlexxIT/WebRTC) projects.

## License

[The MIT License (MIT)](LICENSE)
