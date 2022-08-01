# Razer Blade 15 (2021) for Data Science

Here is how i set up my Razer Blade 15 (Early 2021, RTX 3080, 4k) to work with Ubuntu.

- [Installation](#installation)
	- [Which Version?](#which-version)
	- [Dual vs. Single Boot](#dual-vs-single-boot)
- [Fixes](#fixes)
	- [Suspend Loop Fix](#suspend-loop-fix)
	- [Grub Settings](#grub-settings)
	- [Keyboard Backlight](#keyboard-backlight)
	- [Older Ubuntu Versions](#older-ubuntu-versions)
- [Data Science](#data-science)
	- [CUDA and CuDNN](#cuda-and-cudnn)
	- [Nvidia On-Demand](#nvidia-on-demand)
	- [Conda and TensorFlow](#conda-and-tensorflow)
	- [Using TensorCores](#using-tensorcores)
	- [Carla Autonomous Driving Simulator](#carla-autonomous-driving-simulator)
- [Nice to Have](#nice-to-have)
	- [Better Sound](#better-sound)
	- [Face Unlock](#face-unlock)
	- [4K OLED](#4k-oled)
	- [Buetooth for AirPods](#bluetooth-for-airpods)

# Installation

## Which Version?

The Intel WiFi AX210-module is only natively supported from kernel 5.10. Initially it was the easiest to just install the latest version, but as pointed out in the issues older versions (at least 20.04 LTS) also ship with that or a newer kernel version.

[Create a bootable USB-Stick](https://ubuntu.com/tutorials?q=%22create+a+bootable+usb+stick%22) with Ubuntu

## Dual vs. Single Boot

**Important:** _If you want to use Ubuntu as the only OS, but still want to keep the original recovery partition containing Windows (partition 0) **DO NOT** remove the other partitions. The recovery tool will only install Windows 10 on partition 3 without checking or recreating any partitions. It took me way too long to find this out, but in case that happens to you, you would need to manually create partition 1 (EFI=100MB), partition 2 (MSR=128MB) and a primary partition 3 for the C-Drive, where Windows 10 will be installed._

If you want to keep the original Razer wallpapers, make sure to copy them before formatting Windows. You can find them located under: _C:\Windows\Web\Wallpaper_

For the single boot setup, boot from the USB stick and install it by formatting the biggest partition as ext4, set mount point to "/" and click install.

For using Ubuntu alongside Windows you can simply choose "Install Ubuntu alongside Windows Boot Manager" (e.g. like described [here](https://itsfoss.com/install-ubuntu-1404-dual-boot-mode-windows-8-81-uefi/)).

# Fixes

## Suspend Loop Fix 

After the lid was closed, the notebook goes back to suspend after a couple of seconds and stays in that loop until a restart. It can be fixed fairly easy by adding ```GRUB_CMDLINE_LINUX_DEFAULT="quiet splash button.lid_init_state=open"``` to the grub configuration, as is described in the next section.

## Grub Settings

#### Grub Font Size

Depending on whether you want to show the grub bootloader at startup or not, you might want to increase the font size of the grub menu (especially for the 4K display) or to hide it completely.

As the font of grub can be very tiny on a high res display, you can create a bigger one with the following command, as described [here](https://blog.wxm.be/2014/08/29/increase-font-in-grub-for-high-dpi.html). I chose a font size of 42 for the 4K panel. 

```markdown
sudo grub-mkfont --output=/boot/grub/fonts/DejaVuSansMono24.pf2 --size=42 /usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf
```

Next, open the grub settings with ```sudo nano /etc/default/grub``` and adjust it for either dual boot or single boot.

#### For Dual Boot

If you want to select the operating system with the UEFI boot manager by pressing F12 at startup, you can use the same settings as for the single boot setup. If you want to use grub I suggest following settings:
 
```markdown
GRUB_DEFAULT=saved  # boot the saved OS
GRUB_SAVEDEFAULT=true  # save the last booted OS
GRUB_TIMEOUT=2  # time it is shown (in seconds)
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash button.lid_init_state=open"  # suspend boot loop fix (see above)
GRUB_FONT=/boot/grub/fonts/DejaVuSansMono24.pf2  # choosing the created bigger font for 4k display
```

The first two lines will boot the last used OS, the third lines set the time in seconds the menu is shown (adjust to your own preferences), fourth line is the suspend loop fix and lastly, the previously created font.

Update grub with ```sudo update-grub``` and reboot your system  ```sudo reboot```.

#### For Single Boot

If you are using Ubuntu as a single OS as described before, it will still recognize the system as dual boot due to the recovery partition and always load the grub bootloader on startup (but can be hidden with the following settings). After opening the grub config file with ```sudo nano /etc/default/grub``` I used following settings:

```markdown
GRUB_DEFAULT=0  # boot first entry in the list (Ubuntu)
GRUB_TIMEOUT=1  # time until grub automatically loads the chosen OS (in seconds)
GRUB_DISABLE_OS_PROBER=true  # do not show grub (since it is single boot anyways)
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash button.lid_init_state=open"  # suspend boot loop fix (see above)
```

First line choses Ubuntu as boot partition. As grub will still run in background, the timeout is set to one second (setting it to zero it will be ignored and then it takes the standard time of ten seconds). The third line hides the display output, and the last line fixes the suspend loop.

Update grub with ```sudo update-grub``` and reboot your system  ```sudo reboot```.

## Keyboard Backlight

I added support for the Razer Blade Advanced Early 2021 to [OpenRazer](https://openrazer.github.io/). As my pull request got merged you can install it simply as described [here](https://openrazer.github.io/#ubuntu):

```markdown
sudo add-apt-repository ppa:openrazer/stable
sudo apt update
sudo apt install openrazer-meta
```

Don't forget to add your user to the plugdev group with ```sudo gpasswd -a $USER plugdev```.

For the GUI I use [Polychromatic](https://github.com/polychromatic/polychromatic/). Install with:

```markdown
sudo add-apt-repository ppa:polychromatic/stable
sudo apt install polychromatic
```

Reboot and you're ready to go!

Weirdly, when the tray applet is running and I put the laptop to sleep and wake again, the backlight is turned off until after login. Not sure if it is a bug, but it makes it difficult logging in in the dark. I just turned the applet of in startup scripts and that solved the problem.

## Older Ubuntu Versions

Here are the things I had problems with in Ubuntu 21.04, but worked fine with later versions.

### Firefox with Touch

Firefox isn't scrolling with touch. Open the environment file with:

```markdown
sudo nano /etc/environment
```

and add

```markdown
MOZ_USE_XINPUT2=1
```

### Brightness

The OLED screen doesn't react on brightness changes (because it doesn't have a classic backlight as LCD-panels have). Install [ICC brightness](https://github.com/tartansandal/icc-brightness) to solve the problem. 

```markdown
sudo apt install build-essential
sudo apt install liblcms2-dev
sudo make install
```

### Gestures

If you're coming from a mac, you'll probably miss the gestures on the touchpad. The best I could find was [Touchegg](https://github.com/JoseExposito/touchegg), but much more customizable. For the GUI I used [Touché](https://github.com/JoseExposito/touche). You might also want to check https://ubuntuhandbook.org/index.php/2021/06/multi-touch-gestures-ubuntu-20-04/


# Data Science

Now for the interesting part.

## CUDA and CuDNN

_To work at least TensorFlow 2.6 is needed. The configurations I tested and worked:_
- _Ubuntu 21.04: nvidia-driver-460 with CUDA 11.2 and TensorFlow 2.6 beta._
- _Ubuntu 21.10: nvidia-driver-470 with CUDA 11.4 and TensorFlow 2.6._
- _Ubuntu 22.04: nvidia-driver-510 and 515 with CUDA 11.6 and 11.7 and TensorFlow 2.8 and 2.9._

_Basically staying for the driver, CUDA and TensorFlow on the latest version worked fine for me. Installation procedures were the same for all versions._

Activate Nvidia Proprietary Driver in "Software & Updates" under "Additional Drivers" (nvidia-driver-515 at the time of writing) and reboot. You can display information about the GPU and the CUDA Version (11.7 for me) with:

```markdown
nvidia-smi
```

Install the Nvidia Cuda Toolkit next:

```markdown
sudo apt install nvidia-cuda-toolkit
```

Download [Nvidia CuDNN](https://developer.nvidia.com/cudnn), extract the tarball and copy the files to the corresponding path:

```markdown
tar -xf cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive.tar.xz  # edit to the downloaded version

sudo cp cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive/include/cudnn*.h /usr/lib/cuda/include/
sudo cp -P cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive/lib/libcudnn* /usr/lib/cuda/lib64/
sudo chmod a+r /usr/lib/cuda/include/cudnn*.h /usr/lib/cuda/lib64/libcudnn*
echo 'export LD_LIBRARY_PATH=/usr/lib/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/lib/cuda/include:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

Another reboot and everything should work. In one case, after reboot the Nvidia drivers were greyed out in _Software & Updates_ and TensorFlow was using the CPU. Running ```sudo ubuntu-drivers install``` and rebooting solved the problem.


## Nvidia On-Demand

The RTX 3080 supports _Nvidia On-Demand_, so your notebook will mainly use the Intel GPU and use the dedicated one only if needed. Go to _NVIDIA X Server Settings_ and set the _PRIME Profiles_ to _NVIDIA On-Demand_. You can also do this via Terminal with ```prime-select```, for example

```markdown
sudo prime-select on-demand
```

(also helpful if Intel only is greyed out)

You can specifiy which applications should use the Nvidia GPU under:

```markdown
sudo nano /etc/environment
```

for Vulkan applications add

```markdown
__NV_PRIME_RENDER_OFFLOAD=1
```

It can be tested by installing ```sudo apt install vulkan-tools``` and run ```vkcube```.

for GLX applications add

```markdown
__NV_PRIME_RENDER_OFFLOAD=1
__GLX_VENDOR_LIBRARY_NAME=nvidia
```

Testing OpenGL can be done by installing ```sudo apt install glmark2
``` and running ```glmark2```.

To check which GPU is currently being used:

```markdown
glxinfo | grep vendor
```

_For Ubuntu 21.04:_ In some cases Ubuntu switched back to Wayland for the desktop, which is not supported by Nvidia, so it would always rely on the Intel GPU. To always use Xorg uncomment the line:

```markdown
sudo nano /etc/gdm3/custom.conf
WaylandEnable=false
```

With Nvidia On-Demand my battery life increased under light load from two hours to nearly seven hours.

## Conda and TensorFlow

I like to use Miniconda for my work. Install as usual (if wished):

```markdown
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
```

Create and activate environment:

```markdown
conda create --name ENV python
conda activate ENV
pip install --upgrade pip
```

If you need to deactivate and remove environments you can do it with:

```markdown
conda deactivate
conda env remove -n ENV
```

Cuda 11.2 and later need at least TensorFlow 2.6:

```markdown
pip install tensorflow
```

## Using TensorCores

To use the _TensorCores_ on your GPU edit your Code:

```markdown
from tensorflow.keras import mixed_precision
mixed_precision.set_global_policy('mixed_float16')
```

I get roughly a 30% speedup. Nice.

## Matlab

Matlab doesn't scale automatically for the  4K-panel. In the command window run:

```markdown
s = settings;s.matlab.desktop.DisplayScaleFactor
s.matlab.desktop.DisplayScaleFactor.PersonalValue = 2
```

## Carla Autonomous Driving Simulator

[Carla](https://carla.readthedocs.io/en/0.9.11/start_quickstart/) is the only application so far that made minor troubles while installing under Ubuntu 21.04. 

When adding the repository replace _$(lsb_release -sc)_ with _focal_:

```markdown
pip install --user pygame numpy
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1AF1527DE64CB8D9
sudo add-apt-repository "deb [arch=amd64] http://dist.carla.org/carla focal main"
sudo apt update
sudo apt install carla-simulator
```

# Nice to Have

## Better Sound

_PulseEffect got updated and is called EasyEffects now. Installation got a little bit more complicated now._

The Blade has probably the worst sounding speaker I have heard since my first mobile phone in the mid 2000s! Luckily, there is [EasyEffects](https://github.com/wwmm/easyeffects)! If you haven't install Flatpak do so with 

```markdown
sudo apt install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

and then install EasyEffects

```markdown
flatpak install flathub com.github.wwmm.easyeffects
```

You need the run following commands as described [here](https://ubuntuhandbook.org/index.php/2022/04/pipewire-replace-pulseaudio-ubuntu-2204/):

```markdown
sudo apt install pipewire-audio-client-libraries libspa-0.2-bluetooth libspa-0.2-jack
sudo apt install wireplumber pipewire-media-session-
```

Reboot and run start the application. 

I spent way too much time tweaking the equalizer and the outcome is actually pretty good (screenshot still from PulseEffects, EasyEffects looks similar):

![PulseEffectsEQ](https://user-images.githubusercontent.com/34913182/119002831-7f2b3f00-b98d-11eb-9c53-ca6d91a5bbb6.png)

The .json file in this repo was exported from PulseEffect and does not work properly at the moment, I will look into it. Start service at login from the settings does not work at the moment. A workaround is explained in the [issues](https://github.com/flathub/com.github.wwmm.easyeffects/issues/3#issuecomment-908632842). Create the file ```com.github.wwmm.easyeffects.desktop``` in ```~/.config/autostart``` for example with:

```markdown
sudo nano ~/.config/autostart/com.github.wwmm.easyeffects.desktop
```

and add

```markdown
[Desktop Entry]
Name=EasyEffects
Comment=EasyEffects Service
Exec=/usr/bin/flatpak run --branch=stable --arch=x86_64 --command=easyeffects com.github.wwmm.easyeffects --gapplication-service
Icon=easyeffects
StartupNotify=false
Terminal=false
Type=Application
```

## Face Unlock

The Blade Advanced features an IR camera for Windows Hello. You can use it for face recognition, so you don't need to type your password after _sudo_ or on the login screen. Install [Howdy](https://github.com/Boltgolt/howdy) with

```markdown
sudo add-apt-repository ppa:boltgolt/howdy
sudo apt update
sudo apt install howdy
```

and in the setting ```sudo howdy config``` change the device path to ```device_path = /dev/video2``` (video0 is the normal camera, video2 the IR sensor) and set ```ignore_closed_lid = false```. Then add your face:

```markdown
sudo howdy add
sudo howdy test
```

There seems to be a [problem](https://github.com/boltgolt/howdy/issues/323) at some times, when the IR light doesn't turn on. Apparently, by booting from the bios the problem does not occur anymore.


## 4K OLED

I haven't heard so far, whether the OLED screen suffers of burn-in or not. But it doesn't hurt to take a little care. 

- **Black Wallpaper.** ou can remove the wallpaper and have a blank background. Use _dconf Editor_ or Terminal and set the background color to black with ```gsettings set org.gnome.desktop.background primary-color "#000000" ```, remove the wallpaper with ```gsettings set org.gnome.desktop.background picture-uri "" ``` for light mode and ```gsettings set org.gnome.desktop.background picture-uri-dark "" ``` for dark mode.

- **Hide Mouse Cursor.** To hide the mouse cursor when idling, install [Unclutter](https://manpages.ubuntu.com/manpages/trusty/man1/unclutter.1.html) with ```sudo apt install unclutter```. Then, add the command ```unclutter``` to the startup applications. Only works correctly with X11.

- **Hide the Dock.** One option is to go into the settings and set the dock to "auto-hide", then it will only appear when dragging the mouse to the side of the screen. To completely remove it you can install _Gnome Extensions_ from Ubuntu Software and turn the dock off. Then, it will only be visible in the activities overview when pressing the super key (the windows key in the case of the Blade).

- **Hide the Top Bar.** Lastly, to hide the top bar you can install the gnome extension [Hide Top Bar](https://extensions.gnome.org/extension/545/hide-top-bar/). Activate it and set the two Intellihide options off.

- **Hide Home and Trash Icon.** When gnome extension is installed, you can also remove the icons from the desktop. Just right-click in the desktop, choose settings and turn off the toggles for personal folder and trash.

## Bluetooth for Airpods

With Ubuntu 21.04 it didn't really work to connect Airpods to the notebook. A fix can be found [here](https://askubuntu.com/questions/922860/pairing-apple-airpods-as-headset).
