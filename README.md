# PyBlueZ Examples

Example Bluetooth tasks using the Python
[PyBluez](https://pybluez.github.io/)
module. Tested using BlueZ 5 on:

* Raspberry Pi 2 with CSR bluetooth 4.0 USB adapter
* Raspberry Pi 3 / 4 (on-board Bluetooth)
* laptop with Ubuntu
* Windows (Bluetooth classic, non-BLE only)


## Linux

For Ubuntu <= 18.04 we use system Python 2.7 for ease of library install.
If you have Anaconda/Miniconda, you can alternatively use conda-forge libraries.

1. from Terminal:

   ```sh
   apt install python-pip python-bluez libbluetooth-dev libboost-python-dev libboost-thread-dev libglib2.0-dev bluez bluez-hcidump

   adduser lp $(whoami)
   ```
2. setup Python code:

   ```sh
   python -m pip install -e .
   ```
3. check that your Bluetooth devices are not blocked (should say "no"):

   ```sh
   rfkill list
   ```


## Scan for bluetooth devices from Python

```sh
python bluetooth_scan.py
```

If no Bluetooth devices found in the PyBluez device scan, try each of the following:

```sh
hcitool scan
```

and:

```sh
bluetoothctl

scan on
```

If the second way finds devcies but not the first, you may have a chipset issue.
I have noted this with Marvell hardware on Ubuntu 18.04.
I did not look into a resolve for this, as I usually use other hardware.

---

If you get error

> OSError: No such device

check that there is a Bluetooth adapter available:

```sh
hciconfig dev
```

The bluetooth adapter may need to be enabled:

```sh
hciconfig hci0 up
```

## Non-Python Bluetooth examples

These example use Bluez directly from Terminal (without Python)

### Bluetooth pairing

using Bluez5 bluetoothctl agent:

```sh
hciconfig hci0 up  # enables bt on computer
hcitool scan       # gets UUID of devices in pairing mode
hcitool dev        # get BT adapter uuid

bluetoothctl       # starts interactive prompt
scan on            # scans for UUID of device (BT and BLE) in pairing mode
pair uuid          # where "uuid" is what you found with scan
trust uuid
connect uuid       # after pairing, this is how you connect in the future
```

## Notes

If you get the error

> Creating device failed: org.bluez.Error.AuthenticationRejected:
> Authentication Rejected

then edit `/etc/bin/bluez-simple-agent`,
[changing](http://www.wolfteck.com/projects/raspi/iphone/)
"KeyboardDisplay" to "DisplayYesNo"

Also try:

```sh
bluez-test-device trusted <speaker uuid> yes
```

If connected but lacking sound, try editing `~/.asoundrc`,
[pasting in](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=570468):

```
pcm.btspkr {
   type plug
   slave {
       pcm {
           type bluetooth
           device "AA:BB:CC:DD:EE:FF"
           profile "auto"
       }
   }
   hint {
       show on
       description "BT Speaker"
   }
}
ctl.btspkr {
  type bluetooth
}

pcm.btspkr_softvol
{
   type softvol
   slave.pcm "btspkr"
   control.name "Bluetooth"
   control.card 0
}

# Using bluetooth as default :
pcm.!default {
    type plug
    slave.pcm "btspkr_softvol"
}
```

## Bluetooth connect

```sh
hcitool cc <uuid>
```

I sometimes saw in Ubuntu that it disconnects after a second, maybe
because system bluetooth menu is overriding with "off"?

## Errors

> Cannot open shared library
> /usr/lib/arm-linux-gnueabihf/alsa-lib/libasound_module_pcm_bluetooth.so

```sh
apt install bluez-alsa
```

---

> bt_audio_service_open: connect() failed: Connection refused (111)

1. edit `/etc/bluetooth/audio.conf`, pasting in:

   ```ini
   [general]
   Enable=Sink,Source,Socket
   Disable=Media

   AutoConnect=true
   SCORouting=PCM
   ```
2. then:

   ```sh
   service bluetooth restart
   ```

## Set Bluetooth speaker as default audio device

First test it works with:

```sh
mpg321 -a bluetooth  myfile.mp3
```

or:

```sh
mplayer -ao alsa:device=bluetooth myfile.mp3
```

Then, list your audio ALSA devices with:

```sh
aplay -L
```

and you can use:

```sh
alsamixer
```

## References

<https://bitbucket.org/OscarAcena/pygattlib>

<https://wiki.archlinux.org/index.php/bluetooth_keyboard>

<https://github.com/oz123/dude/blob/master/bin/speakersswitcher.sh>

<http://blog.scphillips.com/2013/01/sound-configuration-on-raspberry-pi-with-alsa/>

<https://wiki.debian.org/Bluetooth/Alsa>

<http://www.raspberrypi.org/forums/viewtopic.php?f=63&t=92900>

<http://samtuke.com/2014/10/manually-pair-bluetooth-devices-on-linux-via-cli-nfc-workaround/>

<http://www.correderajorge.es/bluetooth-on-raspberry-audio-streaming/>

<http://www.ioncannon.net/linux/1570/bluetooth-4-0-le-on-raspberry-pi-with-bluez-5-x/>

<https://gist.github.com/dustywilson/8267078>

<http://www.correlatedcontent.com/blog/bluetooth-keyboard-on-the-raspberry-pi/>
