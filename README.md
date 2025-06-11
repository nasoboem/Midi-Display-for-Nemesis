# MIDI Display for the Source Audio Nemesis Delay

I had a programmable MIDI controller putting on dust. I have three GRIDs (EN16, BU16 and EF44 first generation) from [Intech Studios](https://intech.studio/).
I met them at [Super Booth 2025](https://www.superbooth.com/) in Berlin  and had a look at their new lineup (generation 3) which now has modules with a display. Mine however does not have a display and neither has the [Nemesis](https://www.sourceaudio.net/nemesis-delay.html) from [Source Audio](https://www.sourceaudio.net/).
Therefore when I would program the MIDI controller I wouldn't know what I was doing on the Nemesis pedal. So I had the idea to build a display, a MIDI listener that would show me what kind of MIDI data I would send to my guitar pedal.
I hope that anybody can follow along and build their MIDI setup to unlock your guitar pedal true potential.
The Source Audio Nemesis is a very good candidate to get such an upgrade, because one can hardly access 1/3 of its features from the front panel. The rest is only accessible via the Neuron App or via MIDI. The MIDI capabilities of the Nemesis are virtually complete and the [documentation](https://www.sourceaudio.net/uploads/1/1/5/1/115104065/nemesis_midi_implementation_1_01.pdf) excellent. A very good starting point for the project.

So the project has several layers to it.

1. Setup of the Raspberry Pi so that it can detect MIDI data.
2. Programming the MIDI controller so that I could send MIDI data to the Nemesis.
3. Figuring out an easy and consistent way to present the MIDI data on the display.

First of credit where credit is due. The idea for the project came to me after I watched a video by the creator Floyd Steinberg – A DIY MIDI file player with Fluidsynth, Pirate Audio and Raspberry PI Zero 2. 
[![A DIY MIDI file player with Fluidsynth, Pirate Audio and Raspberry PI Zero 2](https://img.youtube.com/vi/ilmhX1j-ENU/0.jpg)](https://www.youtube.com/watch?v=ilmhX1j-ENU) 
He also does a phenomenal job of documenting his projects so it was easy to follow along and adapt his project to suit my needs.

Thanks to Floyd I had an easy start. The Pirate Audio header and a Raspberry Pi Zero he was using I had lying around from previous projects (I had actually two headers from Pirate Audio – unfortunately I killed both their display in the process. Note to self: Don’t mix strong magnets, like say from speakers, with OLED displays). In the end I was ordering a different display from BerryBase – https://www.berrybase.de for the finale version.

So the hardware for the project is:

    • Raspberry Pi Zero 2 WH - https://www.berrybase.de/raspberry-pi-zero-2-wh
    • 1.44" LCD Display HAT for Raspberry Pi - https://www.berrybase.de/1.44-lcd-display-hat-fuer-raspberry-pi (careful – there is a very similarly looking 1.3" OLED display from Waveshare that doesn’t work with my code – it took me several days to find the mistake. I was using the wrong files for the setup and was wondering why it wasn’t working.) You find the right setup files here: https://www.waveshare.com/wiki/1.44inch_LCD_HAT
    • Micro SD-Card for the Raspberry Pi OS
    • USB adapter (micro-USB-B to USB-A female)
    • Power cable for the Raspberry Pi
    • JustIn MIDI USB Interface MP-02 1x MIDI IN 1x MIDI OUT (any other should work too) 
    • Programmable MIDI controller (in my case EN16 - https://intech.studio/de/shop/en16?variant=en16-detent , BU16 - https://intech.studio/de/shop/bu16?variant=bu16-tactile and EF44 - https://intech.studio/de/shop/ef44?variant=ef44-detent from Intech Studio - https://intech.studio/)
      You would need also a MIDI USB host, that could share MIDI data send from the MIDI controller in my case via USB with the Nemesis delay pedal and the Raspberry Pi. I use the
    • RK006 - https://retrokits.com/shop/rk006/ from RetroKits - https://retrokits.com/ for my main MIDI hub in my setup.
    • MIDI cable TRS-A to DIN5 femal - https://retrokits.com/shop/trsa-din5-dongle/
    • MIDI cable TRS-A to DIN5 male - https://retrokits.com/shop/trsa-din5-male-15/

Man, written out it’s insane how much stuff I had laying around at home. I just bought the display, because I killed the ones I had. All the rest I already owned, because it’s part of my synth setup or from previous projects. Now that you spend 500 bucks, like me, we can begin.

##1. Setup the Raspberry Pi

In the beginning I was following Floyds recommendation but later I switched to the full 64 bit version. Not going with the headless version of the OS makes it more bloated, but saved me because I could fix an error in my CSV file that is coming up later.

###Writing the OS onto the SD card

So on top of all the gear mentioned above you now need a computer with a micro SD card reader and an internet connection via WiFi. Sorry.

First download the Raspberry Pi Imager here https://www.raspberrypi.com/software/ for your operating system and launch the Pi Imager.

    • Device: Raspberry Pi Zero 2 W (or your Pi)
    • OS: 64-BIT with Desktop
    • Storage: your micro SD card (careful, everything on the chosen device will be lost!!)

Hit NEXT and now we have to setup the Pi OS so we can interact via ssh with the Raspberry Pi. For that we have to customize the settings of the Pi OS. So, EDIT SETTINGS, on the GENERAL page set a username and password – remember these because you will need it later to ssh into the Pi. Set your WiFi and your timezone and keyboard layout. On the SERVICES page enable SSH and password authentication. Save the settings and apply them on the previous page. Write the OS onto the SD card.

Assemble the Pi and the LCD screen by putting it on the GPIO pins.

When its done put the SD card into the Pi and power it on. Give it some time to properly install the OS and then you should be able to ssh into the Pi via a terminal on your computer.

###Installing the Pi / Python

Terminal: 
	ssh pi@raspberrypi (or what ever user you set it to instead of pi)
	enter your chosen password and acknowledge the connection
	sudo apt update
	sudo apt upgrade
	sudo raspi-config - turn on spi, i2c (for the display) and vnc (for interacting with the desktop of the Pi via a VNC viewer)
	sudo apt-get install python3-rpi.gpio python3-spidev python3-pip python3-pil python3-numpy python3-rtmidi

###Installing the display

After all the installations ran trough now comes the installation of the display. Follow their instructions here. https://www.waveshare.com/wiki/1.44inch_LCD_HAT

As of the current description bookworm does not need any other installation except for the lgpio (may also already work, because we installed it for python with python3-rpi.gpio)

lgpio
sudo su
wget https://github.com/joan2937/lg/archive/master.zip
unzip master.zip
cd lg-master
sudo make install 
# For more information, please refer to the official website: https://github.com/gpiozero/lg
All the Python Installation we did already previously.

So its only the installation of the display left.

(In Bookworm, when I recall it correctly, it wasn’t necessary to install 7z, so you can omit the first command.)

Download Examples

sudo apt-get install p7zip-full -y
wget https://files.waveshare.com/upload/f/fa/1.44inch-LCD-HAT-Code.7z
7z x 1.44inch-LCD-HAT-Code.7z
sudo chmod 777 -R 1.44inch-LCD-HAT-Code
cd 1.44inch-LCD-HAT-Code/RaspberryPi/
To test the display:
cd python
sudo python main.py
sudo python key_demo.py
If you were able to see the test programs on the display you are good to go.
Now your Raspberry Pi should be able to receive and send MIDI data and show what ever you want on its display.

##2. Programming the the MIDI controller
