# Ubuntu 14.04 (trusty) installation on Razer blade 2016
Instructions for installation of ubuntu trust on razer blade
* Installing Ubuntu 14.04 alongside windows 10 on Razer blade 2016
* Go to windows 10 disk management. Right click on the largest size volume and select "Shrink Volume". Shrink so that you have enough space for ubuntu
 (>40GB + your usage.).
* Download instructions and binaries from http://www.killernetworking.com/product-support/knowledge-base/17-linux
* Reboot the computer with Installer USB plugged in.
* Install ubuntu. Reboot
* Install killer wireless drivers from the saved instructions and killer
  wireless binaries. Over the existing files with files from the website.
* Install the latest version of bumblebee drivers from bumblebee ppa

```
    sudo apt-add-repository ppa:bumblebee/stable
    sudo apt-get update
    sudo apt-get install bumblebee bumblebee-nvidia
```
* Install latest nvidia-drivers

```
    sudo apt-get install nvidia-352 nvidia-367 libcuda1-367 nvidia-settings nvidia-modeprobe nvidia-prime
```
* Install the intel drivers and mesa opengl
```
    sudo apt-get install xserver-xorg-video-intel-lts-xenial \
             libgl1-mesa-dri-lts-xenial \
             libgl1-mesa-glx-lts-xenial \
             libglapi-mesa-lts-xenial \
             libgles1-mesa-lts-xenial \
             libglu1-mesa \
             mesa-common-dev-lts-xenial \
             mesa-utils \
             mesa-vdpau-drivers-lts-xenial
```
* Remove or disable gpu-manager. It is a [single script](http://bazaar.launchpad.net/~ubuntu-branches/ubuntu/trusty/ubuntu-drivers-common/trusty/view/head:/share/hybrid/gpu-manager.c) that tries to be oversmart. If it works for you then great, it didn't wor for me.
```
    sudo apt-get remove --purge ubuntu-driver-commons
```
* Set update-alternatives for x86_64-linux-gnu_gl_conf. Point the `ldd /usr/bin/glxgears` dependencies to the right ld.so.conf. Last two lines are for pointing LibraryPath in `/etc/bumblebee/bumblebee.conf` to the right location.
```
    sudo update-alternatives --set x86_64-linux-gnu_gl_conf /usr/lib/nvidia-367-prime/ld.so.conf 
    sudo update-alternatives --set i386-linux-gnu_gl_conf /usr/lib/nvidia-367-prime/ld.so.conf 
    sudo update-alternatives --set x86_64-linux-gnu_egl_conf /usr/lib/nvidia-367-prime/alt_ld.so.conf 
    sudo update-alternatives --set i386-linux-gnu_egl_conf /usr/lib/nvidia-367-prime/alt_ld.so.conf 
    sudo update-alternatives --install /usr/lib/nvidia-current nvidia-current /usr/lib/nvidia-367 367
    sudo update-alternatives --install /usr/lib32/nvidia-current nvidia-current32 /usr/lib32/nvidia-367 367
```
* Install upgraded kernel. I think kernel 4.4 should also work. Not sure if this is needed. [A blog suggested it](https://xipherzero.com/ubuntu-16-04-razer-blade-2016/). More kernel options [here](http://kernel.ubuntu.com/~kernel-ppa/mainline/)
```
    wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.8.6/linux-headers-4.8.6-040806_4.8.6-040806.201610310831_all.deb
    wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.8.6/linux-headers-4.8.6-040806-generic_4.8.6-040806.201610310831_amd64.deb
    wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.8.6/linux-image-4.8.6-040806-generic_4.8.6-040806.201610310831_amd64.deb
    sudo dpkg -i *.deb
```
* Fix bumblebee configuration and bumblebee modprobe configuration to look like this.
```
    dhiman@amacrine:~$ grep -i nvidia /etc/bumblebee/bumblebee.conf 
    # auto-detection is performed. The available drivers are nvidia and nouveau
    Driver=nvidia
    # Should the program run under optirun even if Bumblebee server or nvidia card
    # PMMethod: method to use for saving power by disabling the nvidia card, valid
    ## Section with nvidia driver specific options, only parsed if Driver=nvidia
    [driver-nvidia]
    KernelDriver=nvidia_367
    # colon-separated path to the nvidia libraries
    LibraryPath=/usr/lib/nvidia-current:/usr/lib32/nvidia-current
    # comma-separated path of the directory containing nvidia_drv.so and the
    XorgModulePath=/usr/lib/nvidia-current/xorg,/usr/lib/xorg/modules
    XorgConfFile=/etc/bumblebee/xorg.conf.nvidia
```
* Take care of the bumblebee bug when it fails to remove nvidia modules. [Bug](https://github.com/Bumblebee-Project/Bumblebee/issues/719)
```
    dhiman@amacrine:~$ tail -n 4 /etc/modprobe.d/bumblebee.conf 
    blacklist nvidia-experimental-355
    # Workaround to make sure nvidia-uvm is removed as well
    remove nvidia rmmod nvidia_drm nvidia_modeset nvidia_uvm nvidia
    alias nvidia-modeset nvidia_367_modeset
```
* Add module configuration to load bbswitch with off state. You can list options by `modinfo bbswitch`.
```
    dhiman@amacrine:~$ cat /etc/modprobe.d/bbswitch.conf 
    options bbswitch load_state=0
```
* Optionally add i915 options. You can list the module options by `modinfo i915`. More details [here](https://wiki.archlinux.org/index.php/Intel_Graphics)
```
    dhiman@amacrine:~$ cat /etc/modprobe.d/i915.conf 
    options i915 enable_rc6=1
```
* When you boot make sure that nvidia modules are not loaded
``` 
    lsmod | grep nvidia
```
* Make sure intel graphics support 3D acceleration
```
    glxgears
```
*  If it doesn't check if the opengl libraries are pointing to the right
   locations
```
    dhiman@amacrine:~$ ldd /usr/bin/glxgears 
        linux-vdso.so.1 =>  (0x00007ffc901ba000)
        libGL.so.1 => /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1 (0x00007f8497592000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f849728c000)
        libX11.so.6 => /usr/lib/x86_64-linux-gnu/libX11.so.6 (0x00007f8496f56000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8496b91000)
        libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f8496967000)
        libxcb-dri3.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-dri3.so.0 (0x00007f8496763000)
        libxcb-present.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-present.so.0 (0x00007f8496560000)
        libxcb-sync.so.1 => /usr/lib/x86_64-linux-gnu/libxcb-sync.so.1 (0x00007f849635a000)
        libxshmfence.so.1 => /usr/lib/x86_64-linux-gnu/libxshmfence.so.1 (0x00007f8496157000)
        libglapi.so.0 => /usr/lib/x86_64-linux-gnu/libglapi.so.0 (0x00007f8495f29000)
        libXext.so.6 => /usr/lib/x86_64-linux-gnu/libXext.so.6 (0x00007f8495d17000)
        libXdamage.so.1 => /usr/lib/x86_64-linux-gnu/libXdamage.so.1 (0x00007f8495b13000)
        libXfixes.so.3 => /usr/lib/x86_64-linux-gnu/libXfixes.so.3 (0x00007f849590d000)
        libX11-xcb.so.1 => /usr/lib/x86_64-linux-gnu/libX11-xcb.so.1 (0x00007f849570b000)
        libxcb-glx.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-glx.so.0 (0x00007f84954f3000)
        libxcb-dri2.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-dri2.so.0 (0x00007f84952ee000)
        libxcb.so.1 => /usr/lib/x86_64-linux-gnu/libxcb.so.1 (0x00007f84950cf000)
        libXxf86vm.so.1 => /usr/lib/x86_64-linux-gnu/libXxf86vm.so.1 (0x00007f8494ec8000)
        libdrm.so.2 => /usr/lib/x86_64-linux-gnu/libdrm.so.2 (0x00007f8494cba000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f8494a9c000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f8494897000)
        /lib64/ld-linux-x86-64.so.2 (0x0000555e63104000)
        libXau.so.6 => /usr/lib/x86_64-linux-gnu/libXau.so.6 (0x00007f8494693000)
        libXdmcp.so.6 => /usr/lib/x86_64-linux-gnu/libXdmcp.so.6 (0x00007f849448c000)
```
* Same for optirun glxgears (nvidia mode)
```
    dhiman@amacrine:~$ optirun ldd /usr/bin/glxgears 
        linux-vdso.so.1 =>  (0x00007ffd5fa44000)
        libGL.so.1 => /usr/lib/x86_64-linux-gnu/primus/libGL.so.1 (0x00007fda0d72d000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fda0d40a000)
        libX11.so.6 => /usr/lib/x86_64-linux-gnu/libX11.so.6 (0x00007fda0d0d5000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fda0cd10000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fda0caf1000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007fda0c8e9000)
        libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fda0c5e5000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fda0c3ce000)
        /lib64/ld-linux-x86-64.so.2 (0x0000563c2996e000)
        libxcb.so.1 => /usr/lib/x86_64-linux-gnu/libxcb.so.1 (0x00007fda0c1af000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fda0bfab000)
        libXau.so.6 => /usr/lib/x86_64-linux-gnu/libXau.so.6 (0x00007fda0bda6000)
        libXdmcp.so.6 => /usr/lib/x86_64-linux-gnu/libXdmcp.so.6 (0x00007fda0bba0000)
```
* After optirun exits the nvidia modules should be unloaded. `lsmod | grep nvidia` should not return anything.
* If intel drivers 3D acceleration breaks then, then gnome-session won't load
  because of a [bug](https://bugs.launchpad.net/ubuntu/+source/gnome-session/+bug/1251281). While you work on it, you will need to install xfce or lxde : `sudo apt-get install lubuntu-desktop`
* ```sudo mokutil --enable-verification``` to restore secure boot
* The display does not recover from suspend. Add acpi_sleep=s3_mode to the kernel parameters. Detailed explanation [here](https://www.kernel.org/doc/Documentation/power/video.txt)
```
dhiman@amacrine:~$ grep -2 CMDLINE /etc/default/grub
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="splash acpi_sleep=s3_bios"
GRUB_CMDLINE_LINUX=""

# Uncomment to enable BadRAM filtering, modify to suit your needs
```
# Cuda deviceQuery issues
* `optirun ./deviceQuery` leads to incompatible driver version error
* Try finding the missing libraries by running

```
LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH find /usr/local/cuda-8.0 -name '*.so' -exec ldd \{} \; | grep 'not found'
```
* In my case `libcuda.so.1` was missing which comes with package `libcuda1-367`
