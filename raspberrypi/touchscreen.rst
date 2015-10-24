title
-----

How to: RaspberryPi with touch screen

image
-----

http://media.didier.io/api/get/624bd68e5860a180c69d66ef318ea089

excerpt
-------

This blog post is a reminder.

It will help you to install and configure a touchscreen, in my case a Tontec 7 inches touch screen.

content
-------

## Full screen

Configure ``/boot/config.txt`` for full LCD display:

```
hdmi_force_hotplug=1
hdmi_group=2
hdmi_mode=1
hdmi_mode=87
hdmi_cvt 800 480 60 6 0 0 0
max_usb_current=1
```

## Calibrate touchscreen

Install ``X11`` dependencies:

```
sudo apt-get install libx11-dev libxext-dev libxi-dev x11proto-input-dev
```

Install ``xinput_calibrator``:

```
wget http://github.com/downloads/tias/xinput_calibrator/xinput_calibrator-0.7.5.tar.gz
tar -zxvf xinput_calibrator-0.7.5.tar.gz
./configure
make
sudo make install
```

Launch ``xinput_calibrator`` in the desktop mode (``startx``).
And paste  the resulting into ``/usr/share/X11/xorg.conf.d/99-calibration.conf``.

> In my case, the ``99-calibration.conf`` file looks like this:
> ```
> Section "InputClass"
>         Identifier              "evdev touchscreen catchall"
>         MatchProduct            "eGalax Inc. USB TouchController"
>         Driver                  "evdev"
>         Option "SwapAxes"       "1"
>         Option  "Calibration"   "1998 44 171 1908"
> EndSection
> ```

## Resources

* [Tontec 7 Inches Raspberry Pi LCD Touch Screen Display TFT Monitor AT070TN90 with Touchscreen Kit HDMI VGA Input Driver Board](http://www.amazon.com/gp/product/B00HNLXZHO)
* [Tuto : Branchement ecran LCD Tontec 7 pouces sur Raspberry PI 2](http://paradoxetemporel.fr/6480-tuto-branchement-ecran-lcd-tontec-7-pouces-sur-raspberry-pi-2.html)
* [How to connect a Tontec 7" touch screen](https://www.youtube.com/watch?v=ILBcgpWClD8)
