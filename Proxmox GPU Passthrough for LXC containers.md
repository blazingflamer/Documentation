# Guide to setting up Plex Media Server with LXC on Proxmox



|<img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/plex-new.png" width="128">|<img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/proxmox.png" width="128">|<img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/azure-container-service.png" width="128">|<img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/docker.png" width="128">|<img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/docker-compose.png" width="128">


This guide is taken and modified the below creators
- LXC on Proxmox with GPU passthrough 
[Lo-Res Diy](https://www.youtube.com/watch?v=-Us8KPOhOCY)
- [Jocke.no](https://jocke.no/2022/02/23/plex-gpu-transcoding-in-docker-on-lxc-on-proxmox/)
</br>

___
</br>

__Assumptions__
<details><summary> Click Here! </summary>

#### Please read before proeceeding </br>

* This guide assumes that you have already set up your proxmox server and you have an Nvidia GPU
* This guide assumes you have a basic debian distro understanding and grasp
* We are trying our best to make this as beginner friendly as possible

</br>

</p>
</details>

___


</br>

## Setting up the Proxmox Host
___
</br>

First step is to install the drivers on the host. Nvidia has an official Debian repo, that we could use. However, that introduces a potential problem; we need to install the drivers on the LXC container later without kernel modules. I could not find a way to do this using the packages within the official Debian repo, and therefore had to install the drivers manually within the LXC container. The other aspect is that both the host and the LXC container need to run the same driver version (or else it won’t work). If we install using official Debian repo on the host, and manual driver install on the LXC container, we could easily end up with different versions (whenever you do an apt upgrade on the host). In order to have this as consistent as possible, we’ll install the driver manually on both the host and within the LXC container.

Let us start with setting up your proxmox Host

Starting with the Proxmox Server 


We will pass a GPU through to a proxmox container and utilize it for video transcoding.

In the shell of the node

1. Check to see what kernel version you are running

```
uname -r
```

2. Install headers

```
apt update

apt-cache search pve-header

apt install pve-headers-*.*.*-*-pve
```

3. Blacklist Nouvea. We need to disable the Nouveau kernel module before we can install NVIDIA drivers

```
nano /etc/modprobe.d/blacklist.conf
```

add the text below to the blacklist.conf file

```
blacklist nouveau
```

use ctrl+o to save the file and use ctrl+x to exit </br>

Followed by

```
update-initramfs -u
```

and 

```
reboot
```

## Setting up the NVIDIA Card
___
</br>

4. Lets's Install Dependencies

```
apt install build-essential
```

5. Download Drivers

```
wget (your driver download link)
```

Please select the right linux driver according to your Nvidia card

Nvidia driver site: [Driver](https://www.nvidia.com/Download/index.aspx)

Let's make driver file executable

```
chmod +x (YOURDRIVERFILE)
```

6. Install the drivers

```
./(YOURDRIVERFILE)
```

Now let's make sure drivers load when restarted, we need to add some udev-rules. This is to make sure proper kernel modules are loaded, and that all the relevant device files is created upon boot.

```
nano /etc/modules-load.d/modules.conf
```

Add these lines

```
# Nvidia modules
nvidia
nvidia-modeset
nvidia_uvm
```
followed by 

Updating initramfs

```
update-initramfs -u
```

Now lets create udev rules

```
nano /etc/udev/rules.d/70-nvidia.rules
```

Add the below lines to the rules file
```
KERNEL=="nvidia", RUN+="/bin/bash -c '/usr/bin/nvidia-smi -L && /bin/chmod 666 /dev/nvidia*'"
KERNEL=="nvidia_modeset", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -m && /bin/chmod 666 /dev/nvidia-modeset*'"
KERNEL=="nvidia_uvm", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -u && /bin/chmod 666 /dev/nvidia-uvm*'"
```

Let's do a quick reboot
```
reboot
```

Now let's check that the drivers are running

```
nvidia-smi
```
7. LXC Container

Edit the conf container you want to passthrough too

```
nano /etc/pve/lxc/(YOURCONTAINERID).conf
```

Add these lines but make sure the path (numbers) are correct
```
ls -l /dev/nv*
```
``` 

# Allow cgroup access
lxc.cgroup2.devices.allow = c 195:0 rw
lxc.cgroup2.devices.allow = c 195:255 rw
lxc.cgroup2.devices.allow = c 195:254 rw
lxc.cgroup2.devices.allow = c 509:0 rw
lxc.cgroup2.devices.allow = c 509:1 rw
lxc.cgroup2.devices.allow = c 10:144 rw

# Pass through device files
lxc.mount.entry = /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry = /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
lxc.mount.entry = /dev/nvram dev/nvram none bind,optional,create=file
```

If you’ve come so far without any errors, you’re ready to reboot the Proxmox host. After the reboot, you should see the following outputs (GPU type/info will of course change depending on your GPU);

```
root@mercury:~# nvidia-smi
Wed Feb 23 01:34:17 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.47.03    Driver Version: 510.47.03    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA RTX A2000    On   | 00000000:82:00.0 Off |                  Off |
| 30%   36C    P2    4W /  70W |       1MiB /  6138MiB |     0%       Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

root@mercury:~# ls -l /dev/nv*
crw-rw-rw- 1 root root 195,   0 Oct  8 11:07 /dev/nvidia0
crw-rw-rw- 1 root root 195, 255 Oct  8 11:07 /dev/nvidiactl
crw-rw-rw- 1 root root 195, 254 Oct  8 11:07 /dev/nvidia-modeset
crw-rw-rw- 1 root root 509,   0 Oct  8 11:07 /dev/nvidia-uvm
crw-rw-rw- 1 root root 509,   1 Oct  8 11:07 /dev/nvidia-uvm-tools
crw------- 1 root root  10, 144 Oct  8 11:07 /dev/nvram

/dev/nvidia-caps:
total 0
cr-------- 1 root root 234, 1 Oct  8 11:07 nvidia-cap1
cr--r--r-- 1 root root 234, 2 Oct  8 11:07 nvidia-cap2
```
If the correct GPU shows from nvidia-smi, the persistence service runs fine, and all five files are available, we’re ready to proceed to the LXC container.

We have completed the NVIDIA GPU set up on the Proxmox Host

</br>

## LXC container
___
</br>


1. We need to add relevant LXC configuration to our container. Shut down the LXC container, and make the following changes to the LXC configuration file;

```
nano /etc/pve/lxc/101.conf
``` 
and add the following
```
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 509:* rwm
lxc.cgroup2.devices.allow: c 511:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```

The numbers on the cgroup2-lines are from the fifth column in the device-list above (via ls -alh /dev/nvidia*). 
For me, the two nvidia-uvm files changes randomly between 509 and 511, while the three others remain static as 195. I don’t know why they alternate between the two values (if you know how to make them static, please let me know), but LXC does not complain if you configure numbers that doesn’t exist (i.e. we can add all three of them to make sure it works).

We can now turn on the LXC container, and we’ll be ready to install the Nvidia driver. This time we’re going to install it without the kernel drivers, and there is no need to install the kernel headers.
```
wget NVIDIA-Linux-x86_64-510.47.03.run  https://us.download.nvidia.com/XFree86/Linux-x86_64/510.47.03/NVIDIA-Linux-x86_64-510.47.03.run
```
```
chmod +x NVIDIA-Linux-x86_64-510.47.03.run
```
```
./NVIDIA-Linux-x86_64-510.47.03.run --check
```

`answer "no" when it asks if it should update X config`

```
./NVIDIA-Linux-x86_64-510.47.03.run --no-kernel-module
```

At this point you should be able to reboot your LXC container. Verify that the files and driver works as expected, before moving on to the Docker setup.

```
root@docker1:~# ls -alh /dev/nvidia*
crw-rw-rw- 1 root root 195,   0 Feb 23 00:17 /dev/nvidia0
crw-rw-rw- 1 root root 195, 255 Feb 23 00:17 /dev/nvidiactl
crw-rw-rw- 1 root root 195, 254 Feb 23 00:17 /dev/nvidia-modeset
crw-rw-rw- 1 root root 511,   0 Feb 23 00:17 /dev/nvidia-uvm
crw-rw-rw- 1 root root 511,   1 Feb 23 00:17 /dev/nvidia-uvm-tools
```

```
root@docker1:~# nvidia-smi
Wed Feb 23 01:50:15 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.47.03    Driver Version: 510.47.03    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA RTX A2000    Off  | 00000000:82:00.0 Off |                  Off |
| 30%   34C    P8    10W /  70W |      3MiB /  6138MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```


## Docker container
Now we can move on to get the Docker working. We’ll be using docker-compose, and we’ll also make sure to have the latest version by removing the Debian-provided docker and docker-compose. We’ll also install the Nvidia-provided Docker runtime. Both these are relevant in terms of making the GPU available within Docker.

1. Remove debian-provided packages
```
apt remove docker-compose docker docker.io containerd runc
```

2. Install docker and docker compose from official repository
```
apt update
```
```
apt install ca-certificates curl gnupg lsb-release
```
```
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
apt update && apt install docker-ce docker-ce-cli docker-compose-plugin containerd.io -y
```

<!-- # install docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# install docker-compose bash completion
curl \
    -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose \
    -o /etc/bash_completion.d/docker-compose -->

3. Install nvidia-docker2
```
apt install -y curl distribution=$(. /etc/os-release;echo $ID$VERSION_ID) keyring_file="usr/share/keyrings/nvidia-container-toolkit-keyring.gpg"
```
```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | gpg --dearmor -o ${keyring_file}
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
  sed "s#deb https://#deb [signed-by=${keyring_file}] https://#g" | \
  tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
```
apt update && apt install nvidia-docker2 -y
```

4. Restart systemd + docker (if you don't reload systemd, it might not work)
```   
systemctl daemon-reload
```
```
systemctl restart docker
```
We should now be able to run Docker containers with GPU support. Let’s test it.

```
root@docker1:~# docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
Tue Feb 22 22:15:14 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.47.03    Driver Version: 510.47.03    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA RTX A2000    Off  | 00000000:82:00.0 Off |                  Off |
| 30%   29C    P8     4W /  70W |      1MiB /  6138MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
```

root@docker1:~# nano docker-compose.yml
```
```
version: '3.7'
services:
  test:
    image: tensorflow/tensorflow:latest-gpu
    command: python -c "import tensorflow as tf;tf.test.gpu_device_name()"
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
```

```
root@docker1:~# docker-compose up
```
```
Starting test_test_1 ... done
Attaching to test_test_1
test_1  | 2022-02-22 22:49:00.691229: I tensorflow/core/platform/cpu_feature_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
test_1  | To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
test_1  | 2022-02-22 22:49:02.119628: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1525] Created device /device:GPU:0 with 4141 MB memory:  -> device: 0, name: NVIDIA RTX A2000, pci bus id: 0000:82:00.0, compute capability: 8.6
test_test_1 exited with code 0
```

Yay! It’s working! Let’s add the final pieces together for a fully working Plex docker-compose.yml.

```
version: '3.7'

services:
  plex:
    container_name: plex
    hostname: plex
    image: linuxserver/plex:latest
    restart: unless-stopped
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    environment:
      TZ: Europe/Paris
      PUID: 0
      PGID: 0
      VERSION: latest
      NVIDIA_VISIBLE_DEVICES: all
      NVIDIA_DRIVER_CAPABILITIES: compute,video,utility
    network_mode: host
    volumes:
      - /srv/config/plex:/config
      - /storage/media:/data/media
      - /storage/temp/plex/transcode:/transcode
      - /storage/temp/plex/tmp:/tmp
  
  ```
And it’s working!



