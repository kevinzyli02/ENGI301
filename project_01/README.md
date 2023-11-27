<h1> GameBeagle: PocketBeagle turned into Gameboy</h1>

Hello! While the project is still being developed, here are the steps that I took to address some of the problems I faced. Please refer to the Li_ENGI301_project_01_proposal.pptx for more materials (BOM) and block diagrams. 
Please look at the Hackster page for more information! (https://www.hackster.io/kzl3/gamebeagle-a7b722#overview)

To run the programs, please utilize the `sudo ./run`. Change configure_pins if needed, but I found that following the Mouser Guide with device tree setup was more successful. 

<h2> Challenges </h2>
The biggest challenge I faced was actually getting an emulator to run. I tried my setup with Retroarch, visualboyadvance-m, desmume and Advance MAME with little success.

To install retroarch, I tried two methods using apt and snap store. In order to install using apt, sudo apt install retroarch, which did not have any issues installing. However, I would always encounter the same problems as seen below. I messed around with the retroarch.cfg file with little success. Similar problems occured when running through the snap store (https://snapcraft.io/install/retroarch/debian).
```python
debian@beaglebone:~$ retroarch
error: XDG_RUNTIME_DIR not set in the environment.
error: XDG_RUNTIME_DIR not set in the environment.
Segmentation fault
```
Next, I looked into visualboyadvance-m, which seemed to be the most promising emulator since it had the best compatibility in debian. I installed first through the snap store (https://snapcraft.io/visualboyadvance-mf), which yielded another segmentation fault. I tried editing the configurations for the snap store, which did not have much documentation. In order to manually connect, you can't really specify which device is required. Most of it is reliant on the system's default, which I could also not figure out. In the end, I gave up with snap store and tried to compile the game itself.
```python
debian@beaglebone:~ snap run visualboyadvance-m
mkdir: cannot create directory ‘/run/user/1000’: Permission denied
Segmentation fault
```
Compiling visualboyadvance-m from github turned out to be extremely frustrating as well. Following the directions (https://github.com/visualboyadvance-m/visualboyadvance-m), I tried to compile the device and ran out of memory. I was able to compile it overnight by creating a swap space on my hard drive using this link (https://www.cyberciti.biz/faq/linux-add-a-swap-file-howto/), but this took extremely long to finish. After compiling it, I could not find the actual visualboyadvance-m run file. After looking through the github and forum posts, I really could not figure out how to actually run the game, as visualboyadvance-m command did not work. Even after searching through the

Frustrated with the lack of GameBoy Advance emulators, I tried a few outside emulators such as Desmume. I was hopeful because Desmume had a lot more support than the other two emulators, and followed their github and wiki to compile it from source using these instructions. (https://wiki.desmume.org/index.php?title=Installing_DeSmuME_from_source_on_Linux). At least for desmume, I knew that it was a display issue even though the framebuffer had been setup. In the end, I decided to do a little more digging and found another guide with promise
```python
debian@beaglebone:~$ desmume

(desmume:5257): Gtk-WARNING **: 04:34:37.626: cannot open display:
```
<h2> Best Attempt: Advance MAME</h2>
Lastly, I tried Advance MAME using a guide from Mouser. What was unique about this guide was that they utilized a PocketBeagle and included its own device tree layout, hopefully solving the problem of the OS not recognizing the display's frame buffer. I had tried to setup the ILI 9488 frame buffer using the fbtft driver, but messed up while setting the device tree and ended up having to reinstall Debian on a new SD card. This page provided a detailed explanation of how to set this up on the BeagleBone Black.

Following their guide, I used their Device Tree Overlay Creation Guide in their github repository to setup the button and screen setups. I then installed Advanced MAME following their instructions with few problems ( I utilized -j1 while building the code as -j3 ran into issues for me). Even though button and displays were mapped in the device tree, I still ran into problems with the audio drivers. The guide had provided its own advmame.rc configuration file, but that did not fix the issue. Configuring the Advance MAME configuration file by removing the ALSA also did not eliminate the error code, and importantly, the video did also not initialize.
```
debian@beaglebone:~$ sudo advmame robby                                                                                                                                                                             
[sudo] password for debian:
AdvanceMAME - Copyright (C) 1999-2018 by Andrea Mazzoleni
MAME - Copyright (C) 1997-2003 by Nicola Salmoria and the MAME Team
ALSA lib pcm_dmix.c:1108:(snd_pcm_dmix_open) unable to open slave
ALSA lib pcm_dmix.c:1108:(snd_pcm_dmix_open) unable to open slave
No video modes available for the current game.
Failed to initialize the video
```
To fix these problems, I went into the advmame.rc files and removed these two lines, which resolved the first two errors.
```
device_alsa_device default
device_alsa_mixer channel
```

After looking through the AdvanceMAME documentation, I was able to start the program using the following command, which allowed any user to run the program as root. The manual gave some useful information for the required drivers for the program to run, so be sure to check it out.

```
chown root:root /usr/local/bin/advmame 
chmod u+s /usr/local/bin/advmame
```
With those changes, I was able to start the emulator by typing advmame robby, as seen below. The emulator only displays on part of the screen currently, and I am playing with the different configurations to see what I can do about that. I have tried adjusting device_video_overlaysize, display_resize, display_magnify, display_expand, and a few others with little success. The game runs however, I just need to make a few adjustments to be able to see the screen and use it properly. I recorded a video of what it looks like and will link it here.
