---
layout: post
title: APRS Rx-only IGate with Raspberry Pi and DVB-T dongle
date: 2013-05-19 13:24
author: admin
comments: true
categories: [Allgemein, Amateurfunk, APRS, DVB-T, Raspberry Pi, rtl-sdr]
---
This step-by-step instruction is tested on the Raspberry Pi with the raspbian distribution and a Terratec Cinergy TStick RC with E4000 tuner and an ezcap 2.0 DVB-T dongle with R820T tuner. I recommend a USB hub with seperate power supply that supports more than 100mA on one USB port. The voltage at the raspbery has to be stable and not below 5.0 V, if not you can run into sync errors with the stick.

<b>It is recommended to update to version 1.0.0 or higher of pymultimonaprs, because if the connection to the internet is interrupted there will be no packets routed to the APRS-IS network anymore. Also the startscript and setup was updated.</b>

<b>Preparing the Raspberry Pi</b>
<pre>
sudo apt-get update
sudo apt-get upgrade
sudo raspi-config
mkdir ~/src
</pre>

<b>Avoid the Raspberry to load a kernel module</b>
Edit /etc/modprobe.d/raspi-blacklist.conf file and add the following lines
<pre>
blacklist dvb_usb_rtl28xxu
blacklist dvb_usb_v2
blacklist rtl_2830
blacklist rtl_2832
blacklist r820t</pre>
Then reboot the Raspberry or enter 'sudo rmmod dvb_usb_rtl28xxu'

<b>Installation of driver for the DVB-T dongle</b>
<pre>
cd ~/src
sudo apt-get install git build-essential cmake libusb-1.0-0-dev
git clone git://git.osmocom.org/rtl-sdr.git
cd rtl-sdr
mkdir build
cd build
cmake ../ -DDETACH_KERNEL_DRIVER=ON -DINSTALL_UDEV_RULES=ON
make
sudo make install
sudo ldconfig
</pre>

<b>Test the rtl dongle and determine possible gain settings</b>
<pre>rtl_test</pre>
Based on your antenna setup you have to choose a different gain, so that your signal is strong enough but not overloaded. I suggest to start rtl_tcp on the console and start SDRsharp, SDR-Radio, etc. on another computer and play with the gain settings.

<b>Installation of multimonNG decoder</b>
<pre>
cd ~/src
sudo apt-get install qt4-qmake libpulse-dev libx11-dev patch pulseaudio
git clone https://github.com/EliasOenal/multimonNG.git
cd multimonNG
mkdir build
cd build
qmake-qt4 ../multimon-ng.pro
make
sudo make install</pre>

<b>Installation of kalibrate tool</b>
<pre>
cd ~/src
sudo apt-get install libtool autoconf automake libfftw3-dev
git clone https://github.com/asdil12/kalibrate-rtl.git
cd kalibrate-rtl
git checkout arm_memory
./bootstrap
./configure
make
sudo make install</pre>

<b>Example of calibrating your DVB-T dongle with E4000 chip (remember PPM for later)</b>
<pre>
kal -s GSM900
kal -c 36</pre>
Channel 36 was the strongest in my region, yours can be different.

With the new R820T(2) Chip the kalibrate tool will give us often the wrong PPM value. You can use the command rtl_test -p instead, which will show you the right PPM value after some minutes runtime.

<b>Installation of APRS IGate software</b>
<pre>
cd ~/src
sudo apt-get install python2.7 python-pkg-resources
git clone https://github.com/asdil12/pymultimonaprs.git
cd pymultimonaprs
sudo python2 setup.py install
</pre>

<b>Prepare the startscript</b>
<pre>sudo cp pymultimonaprs.init /etc/init.d/pymultimonaprs
sudo chmod +x /etc/init.d/pymultimonaprs
sudo systemctl enable pymultimonaprs
</pre>

<b>Generate APRS-IS password for own CALL</b>
<pre>
cd ~/src/pymultimonaprs
./keygen.py CALLSIGN
Key for CALLSIGN: 31983
</pre>
Please only use your CALLSIGN without the SSID.

<b>Change configuration file (Call, password, position, gain, ppm, etc.)</b>
<pre>sudo nano /etc/pymultimonaprs.json</pre>
Do not use leading zeros in front of lat and lon parameters.

<b>To test if all is configured well and works fine enter the following (Strg+C cancels)</b>
<pre>rtl_fm -f 144800000 -s 22050 -p 18 -g 42.0 - | multimon-ng -a AFSK1200 -A -t raw -</pre>
<p>The 18 behind the -p option is my PPM and the 42.0 behind the -g option is one of the available gain settings of my stick</p>

<b>Start pymultimonaprs</b>
<pre>sudo /etc/init.d/pymultimonaprs start</pre>

Have fun.

<b>AddOn: Catch the ISS APRS Frames too</b>
Edit pymultimonaprs/multimon.py and change line 32-35 to
<pre>
proc_src = subprocess.Popen(
    ['rtl_fm', '-f', str(int(self.config['rtl']['freq'] * 1e6)), '-f', '145825000', '-s', '22050',
    '-p', str(self.config['rtl']['ppm']), '-g', str(self.config['rtl']['gain']), '-l', '10', '-'],
    stdout=subprocess.PIPE, stderr=open('/dev/null')</pre>

If you add more than one -f option, then the -l option is necessary (squelch).

After that build and install the code again.
<pre>
sudo python setup.py build
sudo python setup.py install
sudo /etc/init.d/pymultimonaprs stop
sudo /etc/init.d/pymultimonaprs start</pre>
