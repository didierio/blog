title
-----

Use PocketSphinx with a streamed microphone over TCP with GStreamer

content
-------

For a long time, I'm thinking about a speech recognition for my home.
I've checked available options:

- Google Speech API, Alexa, and others: pretty nice, but what I'm saying at home is highly personal. I don't want to share this data to a company, which can do what she want with.
- home made speech recognition: I have the control, everything is managed by me, but I need to build everything.

I choose the second option, because of privacy, and because it's more tricky to implement (challenge!). But launching PocketSphinx on a Raspberry Pi is really fun, but slow...: not enough computing unit to do that.

For every issue a solution! Especially when you are working on a Linux distribution.

## Stack

Libraries:

- ``pocketsphinx-5prealpha`` a lightweight Python speech recognition engine (based on the Java one ``sphinxbase``)
- ``gstreamer-1.0`` is perfect to transfer audio and video input over the network (the ``0.1`` one is still used, but deprecated).

Devices:

- A client: a RasbperryPi 3, with Raspbian 4.4, an USB sound card (no microphone connector as default), and a basic microphone plugged on it.
- A server: a simple computer, an Ubuntu 16.10 server, with a recent hardware configuration, if you want better performances.
- A private network (or a public, but be careful about encryption of your data), with the two devices connected.

During executions, you have to consider the server is ``192.168.0.32`` and the port ``3000`` is not already used (I read somewhere ``3000`` is not a conventional port for GStreamer).

## First test

To be sure GStreamer is working, we will just stream a fake WAV audio sample from the client to the ALSA output of the server.

### Server configuration

On the server, we open the tunnel:

```ssh
gst-launch-1.0 tcpserversrc host=192.168.0.32 port=3000 ! audio/x-raw, endianness=1234, signed=true, width=16, depth=16, rate=44100, channels=1, format=S16LE ! audioconvert ! audioresample ! alsasink
```

Explanations:

- ``gst-launch-1.0`` is the binary to launch GStreamer
- ``!`` is used to pipe and manipulate input with other libraries
- ``tcpserversrc host=192.168.0.32 port=3000`` will open a TCP server to catch the streamed sound
- ``audio/x-raw, endianness=1234, signed=true, width=16, depth=16, rate=44100, channels=1, format=S16LE`` explains what the server is waiting for: a WAV stream
- ``audioconvert`` and ``audioresample`` is used to convert and resample the sound (oh really!)
- ``alsasink`` send the stream to the speakers

### Client configuration

It's time to turn on the sound. On the client:

```ssh
gst-launch-1.0 audiotestsrc ! audio/x-raw, endianness=1234, signed=true, width=16, depth=16, rate=44100, channels=1, format=S16LE ! tcpclientsink host=192.168.0.32 port=3000
```

Explanations:

- ``audiotestsrc`` will generate a fake sound
- ``audio/x-raw, endianness=1234, signed=true, width=16, depth=16, rate=44100, channels=1, format=S16LE`` same as the server
- ``tcpclientsink host=192.168.0.32 port=3000`` send the sound to the server

You must ear a fake sound on your computer. But it is not generated from it. Creepy awesome!

If it not works, check the installed packages.
A good way is to use ``gst-inspect-1.0``.

### Stream microphone

To stream the microphone, the only thing to do is to replace ``audiotestsrc`` by ``alsasrc``.

```ssh
gst-launch-1.0 alsasrc ! audio/x-raw, endianness=1234, signed=true, width=16, depth=16, rate=44100, channels=1, format=S16LE ! tcpclientsink host=192.168.0.32 port=3000
```

## Get transcription with PocketSphinx

PocketSphinx allows you to pipe voice recognition from GStreamer.
The client will not change. His only job is to stream microphone, nothing more (dummy!).

On the server:

```ssh
gst-launch-1.0 tcpserversrc host=192.168.0.32 port=3000 ! audio/x-raw, endianness=1234, signed=true, width=16, depth=16, rate=44100, channels=1, format=S16LE ! audioconvert ! audioresample ! pocketsphinx ! fakesink
```

Explanations:

* ``pocketsphinx`` is the PocketSphinx pipeline, to make transcription
* ``fakesink`` replaces ``alsasink``, because we don't want the sound foldback on speakers anymore

I can speak, but I can't see anything about transcription.
You can use this Python code to see in your terminal what you are saying:

```py
import gi
gi.require_version('Gst', '1.0')
from gi.repository import GObject, Gst
GObject.threads_init()
Gst.init(None)

loop = GObject.MainLoop()

def element_message( bus, msg ):
        msg.get_structure().get_name()
        print "hypothesis= '%s'  confidence=%s\n" % (msg.get_structure().get_value('hypothesis'),msg.get_structure().get_value('confidence'))

pipeline = Gst.parse_launch('tcpserversrc host=192.168.0.32 port=3000 ! audio/x-raw, endianness=1234, signed=true, width=16, depth=16, rate=44100, channels=1, format=S16LE ! audioconvert ! audioresample ! pocketsphinx ! fakesink')

bus = pipeline.get_bus()
bus.add_signal_watch()
bus.connect('message::element', element_message)

pipeline.set_state(Gst.State.PLAYING)

loop.run()
```

## Go further

The encryption part must be enforced. The local network is relatively secured, mostly with Ethernet (WiFi not recommended, it's more easy to spoof), but not a fortress.

With GStreamer, you can also stream video... A new road to go...

## Resources

* [GStreamer documentation](https://gstreamer.freedesktop.org/)
* [GStreamer RTP and RTSP support](https://gstreamer.freedesktop.org/documentation/rtp.html)
* [PocketSphinx on Github](https://github.com/cmusphinx/pocketsphinx)
* [Using PocketSphinx with GStreamer and Python](http://cmusphinx.sourceforge.net/wiki/gstreamer)
* [How to use pocketsphinx (5prealpha) with gstreamer-1.0 in python?](http://stackoverflow.com/questions/35232989/how-to-use-pocketsphinx-5prealpha-with-gstreamer-1-0-in-python)
* [Training Acoustic Model For CMUSphinx [CMUSphinx Wiki]](http://cmusphinx.sourceforge.net/wiki/tutorialam)
* [Quelques mots sur la technologie de streaming [FR]](http://www.rap.prd.fr/pdf/technologie_streaming.pdf)
