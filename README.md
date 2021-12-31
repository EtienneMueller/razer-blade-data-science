# Razer Blade 15 (2021) for Data Science

Here is how i set up my Razer Blade 15 (Early 2021, RTX 3080, 4k) to work with Ubuntu.

- [Installation](#installation)
	- [Which Ubuntu Version?](#which-ubuntu-version)
	- [Dual vs. Single Boot](#dual-vs-single-boot)
- [Fixes](#fixes)
	- [Suspend Loop Fix](#suspend-loop-fix)
	- [Grub Settings](#grub-settings)
	- [OLED and Touch](#oled-and-touch)
	- [Keyboard Backlight](#keyboard-backlight)
- [Data Science](#data-science)
	- [Miniconda](#miniconda)
    - [CUDA and CuDNN](#cuda-and-cudnn)
    - [Nvidia On-Demand](#nvidia-on-demand)
	- [Installing TensorFlow](#installing-tensorflow)
    - [Using TensorCores](#using-tensorcores)
	- [Carla Autonomous Driving Simulator](#carla-autonomous-driving-simulator)
- [Nice to Have](#nice-to-have)
	- [Sound](#sound)
	- [Face Unlock](#face-unlock)
    - [Gestures](#gestures)

# Installation

## Which Ubuntu Version?

The Intel WiFi AX210-module is only natively supported from kernel 5.10. To save the trouble from hassling with kernels and drivers, the easiest way is to install Ubuntu 21.04 (which is on kernel 5.11) or later. I used it for a couple of months and I haven't had any compatibility issues so far. 

I recently switched to 21.10 and in the beginning a couple of applications didn't work for me, as they weren't supported. But by the time of writing everything works well so far. More things work out of the box and installation was way easier than before. I would recommend 21.10.

[Create a bootable USB-Stick](https://ubuntu.com/tutorials?q=%22create+a+bootable+usb+stick%22) with Ubuntu

## Dual vs. Single Boot

**Important:** _If you want to use Ubuntu as the only OS but still keep the original recovery partition containing Windows (partition 0) **DO NOT** remove the other partitions. The recovery tool will only install Windows 10 on partition 3 without checking or recreating any partitions. It took me way too long to find this out, but in case that happens to you, you would need to manually create partition 1 (EFI=100MB), partition 2 (MSR=128MB) and a primary partition 3 for the C-Drive, where Windows 10 will be installed._

If you want to keep the original Razer wallpapers, make sure to copy them before formatting Windows. You can find them located under: _C:\Windows\Web\Wallpaper_

For using Ubuntu alongside Windows you can simply choose "Install Ubuntu alongside Windows Boot Manager" as usual (e.g. like described [here](https://itsfoss.com/install-ubuntu-1404-dual-boot-mode-windows-8-81-uefi/)).

Boot from the USB stick and boot, try, install, format the biggest partition as ext4, set mount point to "/" and install.

# Fixes

## Suspend Loop Fix 

After the lid was closed, the notebook goes back to suspend after a couple of seconds and stays in that loop until a restart. It can be fixed fairly easy by adding ```GRUB_CMDLINE_LINUX_DEFAULT="quiet splash button.lid_init_state=open"``` to the grub configuration, as is described in the next section.

## Grub Settings

Depending on whether you are using a dual or single boot setup, you might want to increase the font size of the grub menu (especially for the 4K display) or to hide it completely.

#### Dual Boot

As the font of grub can be very tiny on a high res display, you can create a bigger one with the following command, as described [here](https://blog.wxm.be/2014/08/29/increase-font-in-grub-for-high-dpi.html). I chose a font size of 42 for the 4K panel. 

```markdown
sudo grub-mkfont --output=/boot/grub/fonts/DejaVuSansMono24.pf2 --size=42 /usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf
```

Next, open the grub settings with ```sudo nano /etc/default/grub```. For the dual boot setup I adjusted the following settings:
 
```markdown
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
GRUB_TIMEOUT=1
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash button.lid_init_state=open"
GRUB_FONT=/boot/grub/fonts/DejaVuSansMono24.pf2
```

The first two lines will boot the last used OS, the third lines set the time in seconds the menu is shown (adjust to your own preferences), fourth line is the suspend loop fix and lastly, the previously created font.

Update grub with ```sudo update-grub``` and reboot your system  ```sudo reboot```.

#### Single Boot

If you are using Ubuntu as a single OS as described before, it will still recognize the system as dual boot due to the recovery partition and always load the grub bootloader on startup. After opening the grub config file with ```sudo nano /etc/default/grub``` I used following settings:

```markdown
GRUB_DEFAULT=0
GRUB_TIMEOUT=1
GRUB_DISABLE_OS_PROBER=true
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash button.lid_init_state=open"
```

First line choses Ubuntu as boot partition. As grub will still run in background, the timeout is set to one second (setting it to zero it will be ignored and then it takes the standard time of ten seconds). The third line hides the display output, and the last line fixes the suspend loop.

Update grub with ```sudo update-grub``` and reboot your system  ```sudo reboot```.

## OLED and Touch

The 4K OLED touchscreen looks nice, but has a couple of problems right out of the box.

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

**21.04 only.** _In 21.10 the brigthness works out of the box, no net for third party software._

The OLED screen doesn't react on brightness changes (because it doesn't have a classic backlight as LCD-panels have). Install [ICC brightness](https://github.com/tartansandal/icc-brightness) to solve the problem. 

```markdown
sudo apt install build-essential
sudo apt install liblcms2-dev
sudo make install
```

### Bluetooth for Airpods

With Ubuntu 21.04 it didn't really work to connect Airpods to the notebook. A fix can be found [here](https://askubuntu.com/questions/922860/pairing-apple-airpods-as-headset).

### Remove Static Elements

I haven't heard so far, whether the OLED screen suffers of burn-in or not. But it doesn't hurt to take a little care. 

- **Black Wallpaper.** ou can remove the wallpaper and have a blank background. Weirdly it isn't always just black, it depends which wallpaper was chosen before. Run ```gsettings set org.gnome.desktop.background picture-uri "" ```

- **Hide Mouse Cursor.** To hide the mouse cursor when idling, install Unclutter with ```sudo apt-get install unclutter```. Then, add the command ```unclutter``` to the startup applications.

- **Hide the Dock.** One option is to go into the settings and set the dock to "auto-hide", then it will only appear when dragging the mouse to the side of the screen. To completely remove it you can install _Gnome Extensions_ from Ubuntu Software and turn the dock off. Then, it will only be visible in the activities overview when pressing the super key (the windows key in the case of the Blade).

- **Hide the Top Bar.** Lastly, to hide the top bar you can install the gnome extension [Hide Top Bar](https://extensions.gnome.org/extension/545/hide-top-bar/). Activate it and set the two Intellihide options off.

- **Hide Home and Trash Icon.** When gnome extension is installed, you can also remove the icons from the desktop. Just right-click in the desktop, choose settings and turn off the toggles for personal folder and trash.

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

# Data Science

Now for the interesting part.

## Miniconda

Install as usual (if wished):

```markdown
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
```

Create and activate environment:

```markdown
conda create --name ENV python==3.7
conda activate ENV
pip install --upgrade pip
```

If you need to deactivate and remove environments you can do it with:

```markdown
conda deactivate
conda env remove -n ENV
```

## CUDA and CuDNN

_For 21.04 I used the nvidia-driver-460 with CUDA 11.2 and TensorFlow 2.6 beta, under 21.10 I only tested nvidia-driver-470 with CUDA 11.4 and TensorFlow 2.6 so far. Installation procedures were the same for both._

Activate Nvidia Proprietary Driver in "Software & Updates" (nvidia-driver-470 at the time of writing) and reboot. You can display information about the GPU and the CUDA Version (11.2 for me) with:

```markdown
nvidia-smi
```

Install the Nvidia Cuda Toolkit next:

```markdown
sudo apt install nvidia-cuda-toolkit
```

Download [Nvidia CuDNN](https://developer.nvidia.com/cudnn), extract the tarball and copy the files to the corresponding path:

```markdown
tar -xzvf cudnn-11.4-linux-x64-v8.2.4.15.tgz
sudo cp cuda/include/cudnn*.h /usr/lib/cuda/include/
sudo cp -P cuda/lib64/libcudnn* /usr/lib/cuda/lib64/
sudo chmod a+r /usr/lib/cuda/include/cudnn*.h /usr/lib/cuda/lib64/libcudnn*
echo 'export LD_LIBRARY_PATH=/usr/lib/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/lib/cuda/include:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

Another reboot and everything works.

## Nvidia On-Demand

The RTX 3080 supports _Nvidia On-Demand_, so your notebook will mainly use the Intel GPU and use the dedicated one only if needed. Go to _NVIDIA X Server Settings_ and set the _PRIME Profiles_ to _NVIDIA On-Demand_. You can specifiy which applications should use the Nvidia GPU under:

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

## Installing TensorFlow

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

## Sound

The Blade has probably the worst sounding speaker I have heard since my first mobile phone in the mid 2000s! Luckily, there is [PulseEffects](https://github.com/wwmm/pulseeffects)! Install with

```markdown
sudo apt install pulseeffects
```

I spent way too much time tweaking the equalizer and the outcome is actually pretty good:

![PulseEffectsEQ](https://user-images.githubusercontent.com/34913182/119002831-7f2b3f00-b98d-11eb-9c53-ca6d91a5bbb6.png)

It will flood your applications menu will a lot of LSP plugins, to hide them I found a solution on [Reddit](https://www.reddit.com/r/linux4noobs/comments/g40e3a/unwanted_lsp_plugins_showing_up/fo2y3bi/?utm_source=share&utm_medium=web2x&context=3)

```markdown
echo "[Desktop Entry] Hidden=true" > /tmp/1
find /usr -name "*lsp_plug*desktop" 2>/dev/null | cut -f 5 -d '/' | xargs -I {} cp /tmp/1 ~/.local/share/applications/{}
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

## Gestures

If you're coming from a mac, you'll probably miss the gestures on the touchpad. The best I could find is [Touchegg](https://github.com/JoseExposito/touchegg), but much more customizable. Install it with

```markdown
sudo add-apt-repository ppa:touchegg/stable
sudo apt update
sudo apt install touchegg
```

For the GUI I use [Touché](https://github.com/JoseExposito/touche), which can be installed from Flatpak with

```markdown
flatpak install flathub com.github.joseexposito.touche
```

Check: https://ubuntuhandbook.org/index.php/2021/06/multi-touch-gestures-ubuntu-20-04/
