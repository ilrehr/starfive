# Debian Trixie image for the Starfive Visionfive 2 1.3b with only official Debian repositories in APT sources without GPU support

The purpose of this image is to test if users who value free/open technologies could have a usable desktop-experience (including latest security patches) with this RISC-V board in 2026.

It is based on the 2025-10 release image from starfivetech (https://debian.starfivetech.com/).

Release v0.2 comes with Linux kernel 6.12.93 (released on june 9th 2026), patched by r0b0 (https://github.com/r0b0/visionfive2-linux). His repository comes preconfigured in this Release for further updates. v0.1 uses the 6.12.5 kernel from starfivetech.

Login-Information:
* User: user
* Password: starfive

### Status of GPU Support  

No support. I previously assumed gpu support because glxinfo outputs "direct rendering: Yes" but further tests with cpu intensive tasks while glxgears is running showed that the FPS is cut in half. But with an interface like XFCE it's comparable to the accelerated interface of the 2025-10 release's Gnome interface in terms of responsiveness. 480p video is also quite watchable.

However, it seems that the gpu drivers are not open/free software, it says so regarding the IMG_GPU module:  
https://github.com/starfive-tech/soft_3rdpart

This license for example is not free:  
https://github.com/starfive-tech/soft_3rdpart/blob/JH7110_VisionFive2_devel/wave511/firmware/LICENSE.txt

According to this website on the debian wiki the rendering performance of the gpu is too slow to be practically usable:  
https://wiki.debian.org/InstallingDebianOn/StarFive/VisionFiveV2


Tests:
```
user@starfive:~$ glxinfo | grep "direct rendering"
direct rendering: Yes
```

```
user@starfive:~$ glxgears
2 frames in 7.0 seconds =  0.286 FPS
291 frames in 5.0 seconds = 58.083 FPS
297 frames in 5.0 seconds = 59.392 FPS
291 frames in 5.0 seconds = 58.005 FPS
```

### Environment  

This package comes with XFCE on X11 as the default desktop environment. Desktopenvironments that rely on hardware acceleration are too slow with this version.

Some other packages that are preinstalled (from the official debian trixie repository):
- xfce-goodies (and some more things, installed by tasksel)
- libreoffice
- firefox-esr
- thunderbird
- vim
- vlc
- keepassxc

### Videos  

This image comes with 2 videos (480p, 720p) and a bashscript for fullscreen video in /home/user which lowers the screen resolution before running vlc and restores the old resolution when vlc closes. To play a video in fullscreen:

```
cd /home/user
./start_video.sh
```

start_video.sh can take the path of another video as input. The fluidity of the 480p video is almost at full speed when using the fullscreen script after some initial stutter. 720p is usable if the desktop resolution is lowered.

### Info on how it was made  

The image 2025-10 engineering-release image from starfivetech was installed on an SD card and used as a base.

The repositories in /etc/apt/sources.list were replaced with the non-snapshotted trixie main repository from debian.org. The starfivetech-repository from /etc/apt/sources.list.d/ was removed.

Some packages relying on the non-official repo were removed: vlc docker* firefox

The existing packages were upgraded: sudo apt update && sudo apt upgrade -y

Installation of the Desktop environment:
```
sudo tasksel (Desktop environment + XFCE)
sudo apt install firefox-esr thunderbird keepassxc vim
```

The 25.0.7 mesa packages were installed (can be triggered by just sudo apt install libgl1-mesa-dri=25.0.7-2 mesa-vulkan-drivers=25.0.7-2).

Temporarily the sid repository was added to the repository to install the mesa 26 packages (sudo apt -t sid install libgl1-mesa-dri mesa-vulkan-drivers...) and then removed + apt update.

Updates on the mesa packages or vlc could cause them not to work.

### Setup tips  

#### XFCE  

The action buttons item in the panel (the user-icon in the upper right of the taskbar) doesn't come with a reboot option. Just right-click it, select properties and check the reboot checkbox to enable it.

To disable auto-lock after some idle time and screen dimming/screen black after some idle time, open Application -> Settings -> Session and Startup, select Application Autostart and disable Screen Locker. Then klick on +Add and enter:  
Name: Disable Screen Sleep  
Description: Prevents DPMS and screensaver from blacking out the screen  
Command: xset s off -dpms

Click ok and reboot.  

If your screen brightness is too high, you can try:  
xrandr --output HDMI-1 --gamma 0.8:0.8:0.8  

You can also add this command to Session and Autostart.  

#### Firefox performance  

Lots of the sluggishness doesn't come from the rendering but from the type and amount of requests. Disabling the following settings might have a positive impact on the performance.  

about:config:
* network.dns.disableIPv6 = true  

For some reason the handshakes are sometimes very slow if firefox tries ipv6. Perhaps it waits for a timeout before trying ipv4? If that's the case you may want to disable ipv6 system-wide.  

General:  
* Check spelling as you type  
* All under Browsing (smooth scrolling, recommended extensions, ...)  

Privacy and Security:  

Some of these have security implications so be sure to know what you are doing!  

* Everything under Firefox Data Collection and Use (don't send any data)  
* Block dangerous and deceptive content (saves extra requests)  
* Query OCSP responder servers to confirm the current validity of certificates (saves some extra requests but it is less secure)  
* DNS over HTTPS

By default you send all your DNS requests to mozilla/cloudflare if you enable DoH. If you know your ISP is in your local area and you use their DNS, turning it off is actually a reasonable - and faster - choice. You can even further improve this if you use a small caching DNS like dnsmasq in your LAN. That way only domains that aren't previously visited are queried to an outside DNS.  

Search:  

Disable all autosuggestions  

#### Lowering resolution  

Lowering your screen resolution to 720p will make typing, window dragging and animations significantly more responsive/smoother. Lowering the system's font size or dpi under Applications -> Settings -> Appearance, using browser.uidensity 1 in Firefox's about:config, using compact mode in keepass, ... will give more space on the screen at the cost of blocky or blurry text.  

#### Streaming Videos  

While this is not technically streaming, on the github page of yt-dlp you can download the platform-agnostic zipimport executable which works on the visionfive 2. Check beforehand if accessing the video files in that way is allowed by the videosite you transfer your bits from. Stream to a tempfs directory (directory that only exists in ram) to avoid reproduction on persistent storage.  

This image is intended for testing purposes.
