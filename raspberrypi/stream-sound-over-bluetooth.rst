title
-----

Stream sound over bluetooth from your phone to a RaspberrryPi 3

content
-------

I have a Google Chromecast integrated to my triple play box.
I use Soundcloud as main music plateform, for a better sound experience.
But using Soundcloud with a Google Chromecast is uncertain: with no real reason, few titles can't be listen.

That's why I decided to implement my own system.

# How it works

I used:

* A phone, to select and play music I want to listen
* A RaspberryPi 3, which offers pretty good bluetooth connectivity (better than USB bluetooth dongles)
* A speaker, a soundbar or, in my case, an amplifier

The RaspberryPi 3 is plugged to the amplifier with an audio cable. I haven't tested yet the HDMI connectivity.

Inside the RaspberryPi 3, the bluetooth module will transmit audio to PulseAudio.

# Installation

The RaspberryPi 3 is running with a [Rasbpian Jessie](https://www.raspberrypi.org/downloads/raspbian/) distribution.

First, install dependencies (PulseAudio and bluetooth module):

```
sudo apt-get install pulseaudio bluez pulseaudio-module-bluetooth python-gobject python-gobject-2
```

In ``/etc/bluetooth/input.conf`` file, add ``Enable=Source,Sink,Media,Socket``.

In ``/etc/pulse/daemon.conf`` file, add ``resample-method = trivial``.

In ``/etc/bluetooth/main.conf`` file, add ``Class = 0x00041C``.

And then, ``reboot``.    

# Configuration

## Connect phone

Now, we have to pair and connect the phone. Activate bluetooth on it.

Go back to your RaspberryPi 3, launch ``sudo hciconfig hci0 piscan`` and ``pulseaudio -D``, to start PulseAudio server.

Then, in ``bluetoothctl``, launch those commands, with ``XX:XX:XX:XX:XX:XX`` the MAC address of your phone:

```
power on
agent on
default-agent
scan on
pair XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
scan off
exit
```

Check a sound card named ``bluez_card.XX_XX_XX_XX_XX_XX`` is configured in pulseaudio with ``pactl list cards short``.

Your phone must be connected to the RaspberryPi 3 with bluetooth. Check if everything is working, by playing a sound on your phone.
The sound must be transfered to the RasbperryPi 3. If not, check if PulseAudio is launched.

## Auto-connect 

But, the actual configuration doesn't allow to auto-connect the trusted phone.
When bluetooth connection is stopped, you have to reconnect it in your RasbperryPi 3 with ``bluetoothctl``.

It's because PulseAudio is not configured to be launched on start-up. To fix this, we have to start PulseAudio on start-up.

In ``/etc/dbus-1/system.d/pulseaudio-bluetooth.conf`` add this in ``busconfig`` section:

```
  <policy user="pulse">
    <allow own="org.pulseaudio.Server"/>
    <allow send_destination="org.bluez"/>
    <allow send_interface="org.bluez.Manager"/>
  </policy>
```

To the end of ``/etc/pulse/system.pa`` file, add:

```
.ifexists module-bluetooth-discover.so
load-module module-bluetooth-discover
.endif
```

In ``/etc/pulse/client.conf``, set ``autospawn`` to ``yes``.

In ``/etc/pulse/daemon.conf``, set ``allow-module-loading`` to ``true``.

Create ``/etc/systemd/system/pulseaudio.service`` file, with:

```
[Unit]
Description=Pulse Audio

[Service]
Type=simple
ExecStart=/usr/bin/pulseaudio --system --disallow-exit --disable-shm --exit-idle-time=-1

[Install]
WantedBy=multi-user.target
```

Then:

```
systemctl daemon-reload
systemctl enable pulseaudio.service
reboot
```

The phone will be connected automatically to your RaspberryPi 3 bluetooth connection.

# Next step

This solution is perfect to listen music when I'am alone.
But my friends can't connect their phones without my help. Because I have to pair their phones with ``bluetoothctl`` (highly geeky, but not pragmatic).

My next step is to use a push button, like Amazon Dash, to allow phone pairing without doing anything.

I will update this blog post when this feature has been done.

# Resources

* [Autostart PulseAudio on startup](https://github.com/davidedg/NAS-mod-config/blob/master/bt-sound/bt-sound-Bluez5_PulseAudio5.txt)
* [Setup Raspberry Pi 3 as bluetooth speaker](https://raspberrypi.stackexchange.com/questions/47708/setup-raspberry-pi-3-as-bluetooth-speaker)
* [Bluetooth headset - ArchWiki](https://wiki.archlinux.org/index.php/Bluetooth_headset#Headset_via_Bluez5.2FPulseAudio)
* [Bluetooth Issues! Blue-utils & bluez-simeple-agent](https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=133961)
* [Raspberry Pi - Bluetooth audio streaming](https://www.raspberrypi.org/forums/viewtopic.php?t=68779)
