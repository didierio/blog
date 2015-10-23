title
-----

How to: RaspberryPi as a motion camera

excerpt
-------

This blog post is a reminder.

It will help you to install and configure a RaspberryPi and the Pi Camera, with motion.

content
-------

This blog post is a reminder.
It will help you to install and configure a RaspberryPi and the Pi Camera, with motion.

## Install v4l

```
sudo apt-get install libgtk2.0-dev libjpeg9-dev
cd /usr/src
git clone git://git.linuxtv.org/v4l-utils.git
cd v4-utils
./bootstap.sh
autoreconf -vfi
make
sudo make install
```

## Install x264

```
cd /usr/src
git clone git://git.videolan.org/x264
cd x264
./configure --host=arm-unknown-linux-gnueabi --enable-static --disable-opencl
make
sudo make install
```

## Install ffmpeg with libfaac

```
sudo apt-get install libfaac-dev libmp3lame-dev libxvidcore-dev libgsm1-dev libtheora-dev libvorbis-dev
cd /usr/src
git clone git://source.ffmpeg.org/ffmpeg.git
cd ffmepg
./configure --enable-libmp3lame --enable-libtheora --enable-libx264 --enable-libgsm --enable-postproc --enable-libxvid --enable-libfaac --enable-pthreads --enable-libvorbis --enable-gpl --enable-nonfree
make
sudo make install
sudo ldconfig
```

## Enable /dev/video0

```
sudo modprobe bcm2835-v4l2
```

## Disable the red LED

Update the ``/boot/config.txt`` with:

```
disable_camera_led=1
```

And reboot (``sudo reboot``).

## Basic HTTP Live stream

```
raspivid -o - -t 0 -hf -w 640 -h 360 -fps 25|cvlc -vvv stream:///dev/stdin --sout '#standard{access=http,mux=ts,dst=:8090}' :demux=h264
```

And open "http://<ip>:8090" on VLC.

Standard definitions:

- SD Low: ``-w 480 -h 260 -fps 25 -b  800000``
- SD Medium: ``-w 640 -h 360 -fps 25 -b  1200000``
- SD High: ``-w 960 -h 540 -fps 25 -b  1800000``
- HD Ready: ``-w 1280 -h 720 -fps 25 -b  2500000``
- Full HD: ``-w 1920 -h 1080 -fps 25 -b  5000000``


## Resources

* [libfaac](https://packages.debian.org/sid/faac)
* [Replicate SPS/PPS in h264 bit streams](https://github.com/AndyA/psips)
* [Get_iplayer installation on Raspberry Pi with Debian Wheezy Beta](http://raspi.tv/2012/get_iplayer-installation-on-raspberry-pi-with-debian-wheezy-beta)
* [Raspberry Pi, Node.js, ffmpeg et la RaspiCam](http://eddy.martignier.ch/raspbian-node-js-ffmpeg-et-la-raspicam/)
* [Official V4L2 driver](https://www.raspberrypi.org/forums/viewtopic.php?t=62364)
* [The Raspberry Pi Camera Module](http://www.ics.com/blog/raspberry-pi-camera-module)
* [Streaming depuis la Raspberry Camera](http://blog.nicolargo.com/2013/05/streaming-depuis-la-raspberry-camera.html)
