# Razer Blade 15 (2021) for Data Science

Here is how i set up my Razer Blade 15 (2021, RTX 3080, 4k) to work with Ubuntu.

- [Preparation](#preparation)
- [Installation](#installation)
- [Fixes](#fixes)
    - [Suspend Loop Bug](#suspend-loop-bug)
    - [Firefox for Touch](#firefox-for-touchscreen)
    - [Brightness](#brightness)
- [Miniconda](#miniconda)
- [Nvidia](#nvidia)
    - [CUDA and CuDNN](#cuda-and-cudnn)
    - [Nvidia On-Demand](#nvidia-on-demand)
- [TensorFlow](#tensorflow)
    - [Installation](#installation)
    - [TensorCores](#tensorcores)
- [Carla Autonomous Driving Simulator](#carla-autonomous-driving-simulator)
- [Nice-to-have](#nice-to-have)
    - [Face Unlock](#face-unlock)
    - [Gestures](#gestures)
    - [OpenRazer](#openrazer)
    - [Guake](#guake)

## Preparation

The original Razer wallpapers are located under: _C:\Windows\Web\Wallpaper_

If you want to use Ubuntu as single OS but still keep the recovery partition (partition 0) **DO NOT** remove the other partitions. The recovery tool will only install Windows 10 on partition 3 without checking or recreating any partitions. In that case you would need to manually create partition 1 (EFI=100MB), partition 2 (MSR=128MB) and a primary partition 3 for the C-Drive, where Windows 10 will be installed.

## Installation

The Intel WiFi AX210-module is only natively supported from kernel 5.10. To save the trouble from hassling with kernels and drivers, the easiest way is to install Ubuntu 21.04 (which is on kernel 5.11). Although it was launched just a couple of days ago, I haven't had any compatibility issues so far.

[Create a bootable USB-Stick](https://ubuntu.com/tutorials?q=%22create+a+bootable+usb+stick%22) with Ubuntu, boot, try, install, format the biggest partition as ext4, set mount point to "/" and install.

If you kept the partition as mentioned before, Ubuntu will recognize the system as dual boot and always load the grub bootloader on startup. To hide it, edit/add:

```markdown
sudo nano /etc/default/grub
GRUB_TIMEOUT="1"
GRUB_DISABLE_OS_PROBER="true"
GRUB_HIDDEN_TIMEOUT="0"
```

you might also want to fix the [suspend loop bug](#suspend-loop-bug) in the grub settings. Update grub.

```markdown
sudo update-grub
```

Reboot.

## Fixes

### Suspend Loop Bug

After the lid was closed, the notebook goes back to suspend after a couple of seconds and stays in that loop until a restart. To fix that update the following line in the grub config as described before:

```markdown
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash button.lid_init_state=open"
```

### Firefox for Touchscreen

Firefox isn't scrolling with touch. Change the following settings:

```markdown
sudo nano /etc/environment
MOZ_USE_XINPUT2=1
```

### Brightness

The OLED screen doesn't react on brightness changes (because it doesn't have a classic backlight as LCD-panels have). Install [ICC brightness](https://github.com/udifuchs/icc-brightness) to solve the problem.

## Miniconda

Install as usual:

```markdown
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
```

Create and activate environment:

```markdown
conda create --name ENV python==3.7
conda activate ai
pip install --upgrade pip
```

Deactivate and delete environment:

```markdown
conda deactivate
conda env remove -n ENV
```

## Nvidia

### CUDA and CuDNN

Activate Nvidia Proprietary Driver in "Software & Updates" (nvidia-driver-460 at the time of writing) and reboot. You can display information about the GPU and the CUDA Version (11.2 for me) with:

```markdown
nvidia-smi
```

Install the Nvidia Cuda Toolkit next:

```markdown
sudo apt install nvidia-cuda-toolkit
```

Download [Nvidia CuDNN](https://developer.nvidia.com/cudnn), extract the tarball and copy the files to the corresponding path:

```markdown
tar -xzvf cudnn-11.3-linux-x64-v8.2.0.53.tgz
sudo cp cuda/include/cudnn*.h /usr/lib/cuda/include/
sudo cp -P cuda/lib64/libcudnn* /usr/lib/cuda/lib64/
sudo chmod a+r /usr/lib/cuda/include/cudnn*.h /usr/lib/cuda/lib64/libcudnn*
echo 'export LD_LIBRARY_PATH=/usr/lib/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/lib/cuda/include:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

Another reboot and everything works.

### Nvidia On-Demand

The RTX 3080 supports _Nvidia On-Demand_, so your notebook will mainly use the Intel GPU and use the dedicated one only if needed. Go to _NVIDIA X Server Settings_ and set the _PRIME Profiles_ to _NVIDIA On-Demand_. You can specifiy which applications should use the Nvidia GPU under:

```markdown
sudo nano /etc/environment
```

for Vulkan applications add

```markdown
__NV_PRIME_RENDER_OFFLOAD=1 CarlaUE4.sh
```

for GLX applications add

```markdown
__NV_PRIME_RENDER_OFFLOAD=1
__GLX_VENDOR_LIBRARY_NAME=nvidia APPLICATION
```

To check which GPU is currently being used:

```markdown
glxinfo | grep vendor
```

In some cases Ubuntu switched back to Wayland for the desktop, which is not supported by Nvidia, so it would always rely on the Intel GPU. To always use Xorg uncomment the line:

```markdown
sudo nano /etc/gdm3/custom.conf
WaylandEnable=false
```

With Nvidia On-Demand my battery life increased under light load from two hours to nearly seven hours.

## TensorFlow

### Installation

Cuda 11.2 is not supported by TensorFlow 2.5. Use the nightly version of 2.6:

```markdown
pip install tf-nightly
```

### TensorCores

To use the _TensorCores_ on your GPU edit your Code:

```markdown
from tensorflow.keras import mixed_precision
mixed_precision.set_global_policy('mixed_float16')
```

I get roughly a 30% speedup. Nice.

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

## Nice-to-have

### Face Unlock

The Blade features an IR camera, to use face recognition instead of typing your password after _sudo_, istall [Howdy](https://github.com/Boltgolt/howdy)

```markdown
sudo add-apt-repository ppa:boltgolt/howdy
sudo apt update
sudo apt install howdy
```

and change the device path:

```markdown
sudo howdy config
device_path = /dev/video0
```

then add your face:

```markdown
sudo howdy add
sudo howdy test
```

### Gestures

If you're coming from a mac, you'll probably miss the gestures on the touchpad. The best I could find is [Touchegg](https://github.com/JoseExposito/touchegg), but much more customizable.

### OpenRazer

[OpenRazer](https://openrazer.github.io/) does currently not offer support for the Blade 2021. There seem to be a [workaround](https://github.com/openrazer/openrazer/issues/1442), but I haven't tested it so far.

### Guake

Always nice to have. Install with

```markdown
sudo apt install guake
```

and add "guake" to "Startup Application Preferences".
