# Debian Trixie image for the Starfive Visionfive 2 1.3b with only official Debian repositories in APT sources and partial GPU support

The purpose of this image is to test if users who value free/open technologies could have a usable desktop-experience with (with latest security patches) this.

It is based on the 2025-10 release image from starfivetech (https://debian.starfivetech.com/).

Login-Information:
* User: user
* Password: starfive

Status of GPU Support:

What works:
- OpenGL and vulkan contexts (glxgears, vkcube for example)
- accelerated video with vlc

What doesn't work:
- hardware-accelerated interfaces (but with xfce it's still relatively responsive)

Tests:
user@starfive:~$ glxinfo | grep "direct rendering"
direct rendering: Yes

user@starfive:~$ glxgears
2 frames in 7.0 seconds =  0.286 FPS
291 frames in 5.0 seconds = 58.083 FPS
297 frames in 5.0 seconds = 59.392 FPS
291 frames in 5.0 seconds = 58.005 FPS

This package comes with XFCE on X11 as the default desktop environment. Desktopenvironments that rely on hardware acceleration are not practically usable.

Some other packages that are preinstalled (from the official debian trixie repository):
- xfce-goodies (and some more things, installed by tasksel)
- libreoffice
- firefox-esr
- thunderbird
- vim
- vlc
- keepassxc

This image comes with 2 videos and a bashscript for fullscreen video in /home/user.

Information on how this image was made:

The image 2025-10 engineering-release image from starfivetech was installed on an SD card and used as a base.

The repositories in /etc/apt/sources.list were replaced with the non-snapshotted trixie main repository from debian.org. The starfivetech-repository from /etc/apt/sources.list.d/ was removed.

Some packages relying on the non-official repo were removed: vlc docker* firefox

The existing packages were upgraded: sudo apt update && sudo apt upgrade -y

Installation of the Desktop environment:
sudo tasksel (Desktop environment + XFCE)
sudo apt install firefox-esr thunderbird keepassxc vim

The 25.0.7 mesa packages were installed (can be triggered by just sudo apt install libgl1-mesa-dri=25.0.7-2 mesa-vulkan-drivers=25.0.7-2).

Temporarily the sid repository was added to the repository to install the mesa 26 packages (sudo apt -t sid install libgl1-mesa-dri mesa-vulkan-drivers...) and then removed + apt update.

This image is for testing purposes only.
