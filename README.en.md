Sure! Here's the translated version of the document into English:

# Ubuntu 18.04/20.04 with Gnome and NoMachine Desktop

## Reference: Docker-GUI Code
Based on this repository, remove the dependency on the host's X11 service:
https://github.com/fadams/docker-gui

## Reference: Docker-Nvidia-Glx-Desktop
Refer to this code repository for GPU driver installation:
https://github.com/ehfd/docker-nvidia-glx-desktop

## Related Repositories:
https://github.com/gezp/docker-ubuntu-desktop

### 1. Overview
* **Image Purpose:** Build a Docker-based Ubuntu system supporting the native Gnome desktop, with installation methods closely resembling those on a host machine.
* **Project Goal:** Enable remote desktop connections (Lightdm+Gnome+NoMachine) on Ubuntu 18.04/20.04, using a virtual display (`DISPLAY=:0`) with support for OpenGL and Vulkan.  
* The project is based on the above repositories to create a Gnome remote desktop that does not rely on the host X service, primarily for remote work or running OpenGL/Vulkan-dependent programs (e.g., Carla, Unreal Engine 4).

**Supported Image Tags:**  
* Ubuntu 18.04: `18.04`
* Ubuntu 20.04: `20.04`

**Ubuntu 18.04 Remote Desktop:**
![Ubuntu 18.04 Image](img/ubuntu18.04-0.png)  
![About Ubuntu 18.04](img/ubuntu18.04-about.png)

**Ubuntu 20.04 Remote Desktop:**
![Ubuntu 20.04 Image](img/ubuntu20.04-0.png)  
![About Ubuntu 20.04](img/ubuntu20.04-about.png)

### 2. Basic Usage (Common for Ubuntu 18.04/20.04)
* **Quick Start Guide:** You don't need to clone the repository; the necessary scripts have been copied to the `/home` directory.

#### 2.1 Preparations
* **Required Environment:** Docker, nVidia-docker, NoMachine  
* **Notes:**
  1. This image requires the virtual display configuration script located in the `/home` directory, which will automatically install necessary drivers (supporting Tesla/Geforce GPUs).
  2. The container requires Docker's `--privileged=true` parameter to install the same version of the GPU driver as the host.

#### 2.2 Creating a Container
* **About GPU vs. CPU Rendering:**
```bash
In GPU mode, rendering may slow down if the server's GPU usage is high (e.g., with deep learning). Consider whether to use GPU rendering based on your needs (default: GPU, e.g., V100).
In CPU mode, rendering speed may increase, providing better visual interaction (default: llvmpipe).
```

* **docker pull:** Get the Ubuntu 18.04 or 20.04 image
```bash
docker pull colorfulsky/ubuntu-gnome-nomachine:18.04  
                        or                  
docker pull colorfulsky/ubuntu-gnome-nomachine:20.04  
```

* **docker run:** Create and run the container  
```bash
Method 1 (Recommended Script):  
wget https://raw.githubusercontent.com/ColorfulSS/docker-ubuntu-gnome-nomachine/master/2-remote-virtual-desktops/nx/ubuntu-18.04-gnome-nomachine/ubuntuSimple.sh  
# Alternative link if the above doesn't work:  
wget https://git.ustc.edu.cn/Colorful/file_download/-/raw/main/ubuntuSimple.sh
# Make the script executable
chmod +x ubuntuSimple.sh
# Execute the script and follow the prompts to enter the relevant information  
./ubuntuSimple.sh  

Note: After the script completes, it will remain on the command-line interface; terminate it with Ctrl+C.
```
![AutoScript Image](img/AutoScript.png)

```bash
Method 2 (Direct Execution):   
# Set the following parameters:  
# Image name to use:            IMAGE  
# Container name:               CONTAINER  
# NoMachine connection port:    NomachineBindPort   
# SSH connection port:          SshBindPort  
# Ubuntu system username:       CreateUserAccount  
# Custom mapped directory:      WorkSpaceBind  
docker run -d \
    --restart=always \
    -p $NomachineBindPort:4000 \
    -p $SshBindPort:22 \
    --privileged=true\
    --userns host \
    --device=/dev/tty0 \
    --name $CONTAINER \
    --ipc=host \
    --shm-size 2g \
    --security-opt apparmor=unconfined \
    --cap-add=SYS_ADMIN --cap-add=SYS_BOOT \
    -e CreateUserAccount=$CreateUserAccount \ # Set the username
    -e RenderType=$RenderType \               # Render mode (Gpu or Cpu)
    -v /sys/fs/cgroup:/sys/fs/cgroup \
    -v $WorkSpaceBind:/data \
    $IMAGE /sbin/init  

# Execute different commands based on GPU type  
# Tesla series: V100, A100, etc.   
docker exec -it $CONTAINER /home/Tesla-XorgDisplaySettingAuto.sh  
# GeForce series: 3090, 2080, etc.  
docker exec -it $CONTAINER /home/GeForce-XorgDisplaySettingAuto.sh  

```

* **docker exec:** Enter the container and execute the virtual display configuration script (for debugging)  
```bash
# Container name: CONTAINER, enter as root
docker exec -itd $CONTAINER bash  
```

#### 2.3 Connecting to the Container 
SSH Connection - For security reasons, SSH is not pre-installed. If needed, please install and configure login security manually.  
```bash
# Access the container via SSH
ssh username@host-ip -p xxxx  
```

**NoMachine Remote Desktop Connection:**
* Download the NoMachine software, use the host IP and custom port, enter your username and password to connect.
* To modify NX login policy to key authentication: https://knowledgebase.nomachine.com/AR03Q01020
![NoMachine Connection](img/Nomachine.png)

#### 2.4 Vulkan Installation (Optional)
* For programs requiring Vulkan support, run the installation script to install Vulkan automatically (test for compatibility).
* 1. Navigate to the `/home` directory.
* 2. Execute the `vulkan-ubuntu-18.04.sh` or `vulkan-ubuntu-20.04.sh` script.
* 3. Run `vulkaninfo` to test the installation.

#### 2.5 Display Resolution Settings (as needed)  
After connecting to the Ubuntu system via NoMachine, customize the display resolution to your preference. Search for "Display" after entering the system.  
**Ubuntu 18.04 Display Resolution Settings:**    
![Display Settings 18.04](img/ubuntu18.04-DisplaySetting.png)

**Ubuntu 20.04 Display Resolution Settings:**    
![Display Settings 20.04](img/ubuntu20.04-DisplaySetting.png)

#### 2.6 Connection Speed Optimization
Since a virtual display is used, connected through the local network, you need to optimize both network settings and system settings (users should adjust parameters as needed).  

##### 2.6.1 Ubuntu System Optimization
**Main Task:** Disable Gnome animations  
```bash
Command: gsettings set org.gnome.desktop.interface enable-animations false  
```

##### 2.6.2 NoMachine Optimization
**Main Task:** Improve speed via hardware decoding and network transmission settings.  
**NoMachine Connection Speed Improvement:**
* 1. Software settings: https://forums.nomachine.com/topic/recommended-settings-for-fast-local-lan-connections-only
* 2. Client hardware acceleration settings: https://knowledgebase.nomachine.com/FR04N03097  

#### 2.7 Chinese Language Support (Inside Docker Container)
Reference: https://blog.csdn.net/weixin_39792252/article/details/80415550  
**Chinese Language Support Script:** Go to `2-remote-virtual-desktops/nx/ubuntu-20.04-gnome-nomachine/language.sh` and run the script.  
```bash
# Install Chinese language support package:
sudo apt-get install language-pack-zh-hans
# Then, modify /etc/environment (append to the end of the file):
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh:en_US:en"
# Next, modify /var/lib/locales/supported.d/local (create if it doesn't exist, append at the end):
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK GBK
zh_CN GB2312
# Finally, execute the command:
sudo locale-gen
# If there is an issue with Chinese characters appearing as squares, install Chinese fonts:
sudo apt-get install fonts-droid-fallback ttf-wqy-zenhei ttf-wqy-microhei fonts-arphic-ukai fonts-arphic-uming
```

#### 2.8 Switching Rendering Mode/GPU Card  
* **Note:** The mode/GPU card switching scripts must be run from the command line, not from within the virtual desktop, as this may cause the script to fail and freeze the virtual desktop.

##### 2.8.1 Switching Rendering Mode  
```bash
# Download the rendering mode switching script:
wget https://raw.githubusercontent.com/ColorfulSS/docker-ubuntu-gnome-nomachine/master/2-remote-virtual-desktops/nx/ubuntu-20.04-gnome