# ESP8266 RTOS SDK Toolchain for Windows WSL 1
Toolchain for development of RTOS applications for ESP8266 using [WSL 1 (Windows Subsystem for Linux)](https://learn.microsoft.com/en-us/windows/wsl/about) and programming using USB-UART converter. Using WSL makes it portable across multiple Windows devices. This setup is based on [Espressif's Get-Started guide](https://docs.espressif.com/projects/esp8266-rtos-sdk/en/latest/get-started/index.html) along with fixes to problems that occured due to absence of updates.

## Why WSL 1?
- WSL 1 directly maps connected COM**X** devices to /dev/ttyS**X** files
- WSL 2 works using [usbipd](https://learn.microsoft.com/en-us/windows/wsl/connect-usb), but:
    - After disconnecting the device you have to attach it manually again
    - Interacting with files that are being stored on Windows filesystem is slow ([stackoverflow](https://stackoverflow.com/questions/68972448/why-is-wsl-extremely-slow-when-compared-with-native-windows-npm-yarn-processing))
- [Oracle VirtualBox](https://www.virtualbox.org/) provide COM ports mapping for virtual machines, but:
    - After testing it using [AntiX 23.1 Core](https://antixlinux.com/download/) virtual machine, there is a significant delay that made it impossible to program an ESP
    - While receiving longer messages by UART (using monitor tool), part of the message is being lost
- [Docker](https://www.docker.com/) cannot be used on Windows, because as of today (15.07.2024) it doesn't support mapping USB devices to containers

## 1. WSL Setup
### 1.1 Create a folder to store new WSL machine somewhere.
For example `Documents/WSL_Distros/`

You can use this folder or create another one to manage multiple toolchain WSL's.

### 1.2 Open CMD in newly created folder
**Tip** You can open command line in given folder by typing *cmd* in file explorer url tab:
![CMD tip](assets/cmd_tip.png)

### 1.3 Install Debian WSL image
It can be achieved by using command below or by installing it using [Windows Store](https://apps.microsoft.com/detail/9msvkqc78pk6)
```
wsl --install -d Debian
```
Go through installation steps, newly created account doesn't matter as is the next step the machine is going to be exported and it will lose all users accounts.

### 1.4 Export Debian image
Export Debian image to *debianExport* file.
```
wsl --export Debian debianExport
```
Import previously exported image here to machine named Debian_ESP8266 using WSL 1.
```
wsl --import Debian_ESP8266 . debianExport --version 1
```
**(Optional)** You can delete debianExport file
```
del debianExport
```

## 2. Debian setup
This is essentially doing everything listed in [Get Started guide](https://docs.espressif.com/projects/esp8266-rtos-sdk/en/latest/get-started/index.html) and [Standard Setup of Toolchain](https://docs.espressif.com/projects/esp8266-rtos-sdk/en/latest/get-started/linux-setup.html#install-prerequisites) with slight fixes.
### 2.1 Login to WSL and get repository updates
Open WSL in current folder.
```
wsl -d Debian_ESP8266
```
Get updates and install them.
```
sudo apt-get update
sudo apt-get upgrade
```

### 2.2 Install tools
Slightly modified prompt from [here](https://docs.espressif.com/projects/esp8266-rtos-sdk/en/latest/get-started/linux-setup.html#install-prerequisites).
```
sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python3 python3-pip
```
### 2.3 Download ESP8266 Toolchain
Get appropiate link [here](https://docs.espressif.com/projects/esp8266-rtos-sdk/en/latest/get-started/linux-setup.html#toolchain-setup) and download it.
```
wget https://dl.espressif.com/dl/xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-amd64.tar.gz
```
Export it using command below. Files should be exported to `/xtensa-lx106-elf` directory.
```
tar -xzf ~/Downloads/xtensa-lx106-elf-linux64-1.22.0-100-ge567ec7-5.2.0.tar.gz
```
Clone Espressif's [ESP8266 RTOS SDK](https://github.com/espressif/ESP8266_RTOS_SDK).
```
git clone --recursive https://github.com/espressif/ESP8266_RTOS_SDK.git
```
Make virtual environment for Python.
```
python3 -m venv .venv
```

### 2.4 Modify .bashrc file for environmental variables
Go to home directory and edit .bashrc file using text editor of choice.
```
cd ~
sudo nano .bashrc
```
`wslpath` is a utility tool that converts Windows path to Linux path. Last line runs python environment on login.
```
export PATH="$PATH:$(wslpath 'C:\Users\ketow\Documents\WSL_Distros\Debian_ESP8266\xtensa-lx106-elf\bin')"
export IDF_PATH="$(wslpath 'C:\Users\ketow\Documents\WSL_Distros\Debian_ESP8266\ESP8266_RTOS_SDK')"
source $(wslpath 'C:\Users\ketow\Documents\WSL_Distros\Debian_ESP8266\.venv\bin\activate')
```
Reload .bashrc file.
```
source .bashrc
```
You should be in virtual environment
`(.venv) root@Vojtek:/mnt/c/Users/ketow#`, if not, login to WSL again.
### 2.5 Install Python packages
```
python3 -m pip install -r $IDF_PATH/requirements.txt
```
You are all set!

## 3. Usage
Create a folder where you want to store your project and open WSL. I'am going to use `C:\Users\ketow\Documents\Projects_ESP\test_project`.
```
wsl -d Debian_ESP8266
```
Copy hello_world example from IDF directory.
```
cp -r $IDF_PATH/examples/get-started/hello_world .
```
**(Optional)** Create a symlink in home directory for your projects for easier access and simpler path.
```
cd ~
ln -s $(wslpath 'C:\Users\ketow\Documents\Projects_ESP\test_project') rtos_test_project
```

Now you can follow along the rest of [Get Started guide](https://docs.espressif.com/projects/esp8266-rtos-sdk/en/latest/get-started/index.html#connect).