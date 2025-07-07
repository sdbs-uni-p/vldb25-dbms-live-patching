# System Preparation

Some experiments must be performed using the MMView Linux kernel (https://github.com/luhsra/linux-mmview; git hash `ecfcf9142ada6047b07643e9fa2afe439b69a5f0`). The MMView Linux kernel is an improved version of the original WfPatch Linux kernel (https://github.com/luhsra/linux-wfpatch). For our research, we used the newer MMView Linux kernel. Please note that the terms "MMView Linux kernel" and "WfPatch Linux kernel" can be used interchangeably.

For convenience, we provide a QEMU VM which is equipped (1) with the MMView Linux kernel and (2) all necessary software and libraries installed required for executing our scripts/tools. **However, to precisely reproduce our results, we recommend *not* using the QEMU VM. Instead, please reproduce them on a prepared server.**

> **NOTE: Our experiments were conducted directly on a prepared server, avoiding the overhead of virtualization. When replicating the results in the QEMU VM, we observed a 25% performance degradation compared to the direct server execution. This degradation introduces significant system noise, obscuring some measured effects. Consequently, we recommend conducting the experiments directly on a prepared server for more accurate results. However, we provide the QEMU VM for a quick start and ease of reproducibility.**

Below are the instructions for either preparing your own server or using the QEMU VM.

---

## Prepare own Server

In case you want to prepare a new server for the execution of all scripts, i.e. reproducing the results, please see the notes below on how to compile the MMView Linux kernel and what software is required. It is required to use ***Debian 11 (bullseye)*** as distribution (each distribution ships a different version of `gcc` and patch generation is sensitive to the `gcc` version used). So please follow the official instructions on how to install ***Debian 11 (bullseye)***. Once ***Debian 11 (bullseye)*** is installed, the steps below can be executed.

### 1. Software Installation

The following steps are required to prepare the system and install the necessary software:

```
# Add NOPASSWD to sudo group
vim /etc/sudoers

# Install software
apt-get update -y
apt-get install -y \
    sudo \
    vim \
    git \
    gcc \
    tmux \
    build-essential \
    flex \
    bison \
    libssl-dev \
    bc \
    elfutils \
    libelf-dev \
    dwarves \
    libncurses-dev \
    curl \
    zsh \
    openjdk-17-jdk \
    unzip \
    ccache \
    ninja-build \
    rsync \
    r-base \
    libmariadb-dev \
    libffi-dev
apt-get install -y --no-install-recommends\
    ca-certificates \
    cmake \
    gnutls-dev \
    libboost-all-dev \
    libevent-dev \
    libncurses5-dev \
    libpcre2-dev \
    libxml2-dev \
    pkg-config \
    psmisc \
    wget \
    zlib1g-dev \
    libcurl4-openssl-dev \
    libfreetype6-dev \
    libfontconfig1-dev

# Enable perf (used for MariaDB patch crawling)
echo -1 > /proc/sys/kernel/perf_event_paranoid

# Increase watchdog timeout
echo 60 > /proc/sys/kernel/watchdog_thresh

# Map /tmp to tmpfs
sudo vim /etc/fstab
  # Add line:
  tmpfs   /tmp    tmpfs   size=100%,mode=1777,nosuid,nodev   0   0

# Install Docker
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker `whoami`

```

### 2. Compile and Install MMView Linux Kernel

The following steps may be system-specific, as different systems may require different drivers and configurations. These steps serve as a general guide for compiling the MMView Linux kernel, but slight variations may be necessary depending on your system.

```
# Compile Linux Kernel (MMView)
git clone https://github.com/luhsra/linux-mmview.git
cd linux-mmview

# When compiling the kernel, disable the following options (list may not be complete):
# - CONFIG_USERFAULTFD
# - CONFIG_KSM
# - CONFIG_TRANSPARENT_HUGEPAGE
# - CONFIG_ACPI_NFIT
# - CONFIG_X86_PMEM_LEGACY
# - CONFIG_LIBNVDIMM

# ################################
# Compile kernel without modules #
# ################################
make localmodconfig
make menuconfig 
# Disable "enable loadable module support"
# Exit and save .config

# Set in .config
# - CONFIG_LOCALVERSION="-mmview-min"
# Disable the CONFIG_ options mentioned above

make -j
sudo make install
```

### 3. Kernel Switch

To reproduce our results, switching between the MMView Linux kernel and the regular Linux kernel is required, as data analysis must be performed using the regular kernel. In the README, we refer to two example scripts, `kernel-mmview` and `kernel-regular`, which are assumed to configure the respective kernel in GRUB. **These scripts are mentioned only in the README and are not used by any scripts in the reproduction package. Therefore, their existence is not required; the kernel can also be selected manually during boot.**
If you wish to use such scripts, they must be adapted to your specific system setup. You can refer to [vm-scripts/kernel-regular](vm-scripts/kernel-regular) and [vm-scripts/kernel-mmview](vm-scripts/kernel-mmview) as examples. In the README, we assume that these scripts are located in the home directory (`~`).

### 4. Python

The reproduction package requires Python 3.10. To install it, we use `pyenv`. See the steps below for installing `pyenv` and, subsequently, Python 3.10.

```
# Install pyenv
curl https://pyenv.run | bash

# Update .bashrc and .profile of user based on output of pyenv installation.
# Please note: Different to the output of pyenv, I added the following commands to .profile and .bashrc (slighlty modified in comparison to the pyenv output):

# .profile (or .zprofile):
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"

# .bashrc (or .zshrc)
eval "$(pyenv init -)"

# You may need to restart your shell for the changes to your shell configuration files to take effect.

# Now, install Python 3.10 via pyenv:
pyenv install 3.10.1
pyenv global 3.10.1
pip install pipenv
```

### 5. git

Git must be properly configured. Specifically, `user.name` and `user.email` must be set. The values for these can be arbitrary, for example:

```
git config --global user.name "Repro"
git config --global user.email repro@repro.repro
```

### 6. Getting Started

As the system is prepared now, we can prepare the reproduction pipeline. To do so, clone the repository and setup the utility tools:

```
# 0. This guide assumes that the repository will be cloned in the home directory:
cd ~

# 1. Clone Repository
git clone https://github.com/sdbs-uni-p/vldb25-dbms-live-patching.git dbms-live-patching

# 2. Setup utility tools
cd ~/dbms-live-patching/utils
./setup
```

Now, everything is prepared and the reproduction package can be executed. See the initial [README](..) for more details.

---

## QEMU VM

In case you do not want to prepare your own server (which is highly recommended; see the notes in the beginning), you can also use the prepared QEMU VM. To do so, you need to install QEMU and KVM (for best performance) on your system. Once QEMU and KVM are installed, the following commands get you started using the VM:

```
# 1. Clone Repository
git clone https://github.com/sdbs-uni-p/vldb25-dbms-live-patching.git dbms-live-patching

# 2. Go into the qemu directory
cd dbms-live-patching/qemu

# 3. Download and extract QEMU VM
./download-vm

# 4. Run VM
# Note: The script assigns *all available main memory*
# and *all CPU cores except for one core per CPU socket* 
# of the host system to the VM.
./run-vm

# 5. SSH into VM (see credentials below)
ssh repro@127.0.0.1 -p 2222

# 6. Inside the VM:
# Download the repository
./setup

# 7. Setup utility tools
cd ~/dbms-live-patching/utils
./setup
```

Now, everything is prepared and the reproduction package can be executed. See the initial [README](..) for more details.

### Credentials (username:password):

- repro:repro
- root:root

> **_NOTE:_** In order to enable easy reproduction, security best practices were neglected. Both users are in the `sudo` group without the need for a password (`NOPASSWD` in `/etc/sudoers`).

### Switching Kernel

To reproduce our results, we require both the unmodified Linux kernel and the MMView Linux kernel. The QEMU VM includes scripts facilitating the easy switching of kernels:

```
cd ~

# Enable MMView Linux kernel
./kernel-mmview
sudo reboot

# Enable unmodified Linux kernel
./kernel-regular
sudo reboot
```

