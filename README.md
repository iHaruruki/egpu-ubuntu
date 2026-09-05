# egpu-ubuntu

![Gitea Last Commit](https://img.shields.io/gitea/last-commit/iHaruruki/egpu-ubuntu?path=README.md)

## 🚀 Overview
- How to Use an eGPU on Ubuntu.
- Install **NVIDIA Driver**, **NVIDIA CUDA TOOLKIT**, **NVIDIA cuDNN**.

## 💻 Verification environment
| Types of parts | Model number |
| --- | --- |
| Graphics Cards | GeForce RTX 3070 Ti |
| eGPU Box | Razer Core X |
| CPU | AMD Ryzen™ 7 8845HS |
| OS | Ubuntu24.04.4 LTS |

## 🔎 Structure

| | | 
| --- | --- |
| Application Layer | PyTorch / TensorFlow |
| Library Layer     | cuDNN / cuBLAS / TensorRT |
| Runtime Layer     | CUDA Toolkit |
| Driver Layer      | NVIDIA Driver |
| Hardware Layer    | GPU Hardware |

## 🛠️ Setup

### セキュアブートの無効化
1. Intel NUCのBIOSへの入り方は起動時に`Esc`キーを連打する
2. BIOS画面で`Boot`>>`Boot/Secure Boot`と進み`Secure Boot`を**Disabled**にする

### UEFIがサードバーをロードすることを許可
BIOS画面で`Seurity`>>`Security Features`>>`Allow UEFI 3rd party driver loaded`にチェックを入れ有効化する

### eGPUの接続
1. eGPUとNUCをUSB Type-Cで接続する
2. eGPUの電源を入れる
3. NUCを起動

### Thunerbolt 3機器のuuidの確認
boltctlコマンドでThunderbolt3 機器(Razer CoreX)のuuidを調べる．
```bash
$ boltctl 
 ● Razer Core X
   ├─ type:          peripheral
   ├─ name:          Core X
   ├─ vendor:        Razer
   ├─ uuid:          <uuid>
   ├─ generation:    Thunderbolt 3
   ├─ status:        authorized
   │  ├─ domain:     52f78780-01ce-40ea-ffff-ffffffffffff
   │  ├─ rx speed:   20 Gb/s = 2 lanes * 10 Gb/s
   │  ├─ tx speed:   20 Gb/s = 2 lanes * 10 Gb/s
   │  └─ authflags:  none
   ├─ authorized:    xxxx
   ├─ connected:     xxxx
   └─ stored:        xxxx
      ├─ policy:     iommu
      └─ key:        no
```

### Nouveauドライバーの停止
NVIDIAのビデオカードをLinuxに接続するとNVIDIAドライバーをリバースエンジニアリングして実装されているOSS版ドライバーNouveauドライバーが有効になります．
```bash
lsmod | grep nouveau
```
有効になっている場合，nouveau項目が表示されます．
```bash
sudo sh -c "echo 'blacklist nouveau' > /etc/modprobe.d/blacklist-nouveau.conf"
sudo sh -c "echo 'options nouveau modeset=0' >> /etc/modprobe.d/blacklist-nouveau.conf"
sudo update-initramfs -u
```
再起動
```bash
sudo reboot
```

### NVIDIA Driver & NVIDIA CUDA Toolkit のセットアップ

> [!TIP]  
> NVIDIA Official Documentation.  
> [NVIDIA driver](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/latest/introduction.html)  

#### Linux System Requirements
| Distribution | Codename | Architecture |
| ------------ | -------- | ------------ |
| Ubuntu 24.04 LTS | ubuntu2404 | amd64 | 

#### Verify You Have a Supported Distribution of Linux

To determine which distribution and release number you’re running, type the following at the command line:
```bash
hostnamectl
```
Result
```bash
$ hostnamectl
 Static hostname: nuc40
       Icon name: computer-desktop
         Chassis: desktop 🖥️
      Machine ID: a657b15afa5f4e77b35166ba3a02310c
         Boot ID: b3163103c9a643979981208766799613
Operating System: Ubuntu 24.04.4 LTS              
          Kernel: Linux 6.8.1-1015-realtime
    Architecture: x86-64
 Hardware Vendor: GMKtec
  Hardware Model: NucBox K8 Plus
Firmware Version: NucBox K8 Plus 1.01
   Firmware Date: Wed 2025-02-19
    Firmware Age: 1y 6month 2w
```

#### Verify the System has the Correct Kernel Packages Installed

The version of the kernel your system is running can be found by running the following command:
```bash
uname -r
```
Result
```bash
$ uname -r
7.0.0-31-generic
```

1. Check [CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive)
2. Select Latest Release
3. Select Target Platform
3. Select Target Platform  
   - Operating System: Linux
   - Architecture: x86_64
   - Distribution: Ubuntu
   - Version: 24.04
   - Installer Type: deb(local)

Installation Instructions:
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-ubuntu2404.pin
sudo mv cuda-ubuntu2404.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/13.3.1/local_installers/cuda-repo-ubuntu2404-13-3-local_13.3.1-610.43.02-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2404-13-3-local_13.3.1-610.43.02-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2404-13-3-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-3
```
Driver Installer:
```bash
sudo apt-get install -y nvidia-open
```
Reboot
```bash
sudo reboot
```
Check
```bash
/usr/local/cuda/bin/nvcc -V
```

<!-- #### Select a driver version

> [!NOTE]  
> NVIDIA DATA CENTER DOCUMENTATION  
> [datacenter](https://docs.nvidia.com/datacenter/tesla/index.html)  

#### Choose an Installation Method 

Open [Ubuntu](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/latest/ubuntu.html#ubuntu-installation) link.


#### Preparation (Ubuntu)
1. Perform the [Pre-installation Actions.](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/latest/pre-installation-actions.html#pre-installation-actions)  
2. The kernel headers and development packages for the currently running kernel can be installed with:
```bash
sudo apt install linux-headers-$(uname -r)
```

#### Local Repository Enablement
1. Download the NVIDIA driver repository: ($version:610.57.04, $distro:ubuntu2404, $arch:amd64)
```bash
# wget https://developer.download.nvidia.com/compute/nvidia-driver/$version/local_installers/nvidia-driver-local-repo-$distro-$version_$arch.deb

wget https://developer.download.nvidia.com/compute/nvidia-driver/610.57.04/local_installers/nvidia-driver-local-repo-ubuntu2404-610.57.04_1.0-1_amd64.deb
```
where `$version` is the NVIDIA driver version.

2. Install local repository on file system:
```bash
# dpkg -i nvidia-driver-local-repo-$distro-$version_$arch.deb
sudo dpkg -i nvidia-driver-local-repo-ubuntu2404-610.57.04_1.0-1_amd64.deb
```
3. Enroll ephemeral public GPG key:
```bash
sudo cp /var/nvidia-driver-local-repo-ubuntu2404-610.57.04/nvidia-driver-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
```

#### Selecting a Branch or a Specific Driver version
```bash
# sudo apt install nvidia-driver-pinning-<branch>
sudo apt-get install -y nvidia-driver-pinning-610
```

#### Driver Installation
Open Kernel Modules
```bash
sudo apt install nvidia-open
```
Proprietary Kernel Modules
```bash
sudo apt install cuda-drivers
```

#### Compute-only (Headless) and Desktop-only (no Compute) Installation
##### Compute-only System

Open Kernel Modules
```bash
sudo apt -V install libnvidia-compute nvidia-dkms-open
```
Proprietary Kernel Modules
```bash
sudo apt -V install libnvidia-compute nvidia-dkms
```

#### Reboot the System
```bash
reboot
``` -->

<!-- ### NVIDIA CUDA Toolkit のインストール

> [!TIP]
> NVIDIA CUDA Toolkit provides a development environment 
> for creating high-performance, GPU-accelerated applications.  
> NVIDIA Official Documentation.  
> [NVIDIA CUDA Toolkit](https://developer.nvidia.com/cuda/toolkit)  

1. Ubuntuのバージョンを確認
```bash
uname -m  # アーキテクチャ（x86_64等）を確認
lsb_release -a  # ディストリビューション名とバージョンを確認
```
<details>
<summary>Version check</summary>

```bash
$ uname -mhttps://docs.nvidia.com/datacenter/tesla/index.html
x86_64

$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04.5 LTS
Release:	22.04
Codename:	jammy
```
</details>

2. CUDA Toolkit 13.0 Downloads  
Open link. [CUDA Toolkit 13.0 Downloads](https://developer.nvidia.com/cuda-13-0-0-download-archive)

3. Select Target Platform  
- Operating System: Linux
- Architecture: x86_64
- Distribution: Ubuntu
- Version: 22.04
- Installer Type: deb(local)

4. Install
Installation Instructions:
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/13.0.0/local_installers/cuda-repo-ubuntu2204-13-0-local_13.0.0-580.65.06-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2204-13-0-local_13.0.0-580.65.06-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2204-13-0-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-0
```

5. 環境変数の設定
```bash
export CUDA_HOME=/usr/local/cuda-13.0
echo 'export CUDA_HOME=/usr/local/cuda-13.0' >> ${HOME}/.bashrc
export LD_LIBRARY_PATH=/usr/local/cuda-13.0/lib64:${LD_LIBRARY_PATH}
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-13.0/lib64:${LD_LIBRARY_PATH}' >> ${HOME}/.bashrc
export PATH=/usr/local/cuda-13.0/bin:${PATH}
echo 'export PATH=/usr/local/cuda-13.0/bin:${PATH}' >> ${HOME}/.bashrc
source ${HOME}/.bashrc
```

6. インストールと設定の確認
```bash
cat /usr/local/cuda-13.0/version.json
nvcc --version
```

7. 再起動
```bash
sudo /sbin/shutdown -r now
```-->

### What is NVIDIA CUDA-X Libraries
[NVIDIA CUDA-X Libraries](https://developer.nvidia.com/cuda/cuda-x-libraries)  


### Install NVIDIA cuDNN (NVIDIA CUDA Deep Neural Network library)

> [!TIP]
> cuDNN provides highly tuned implementations for standard routines, 
> such as forward and backward convolution, attention, matmul, pooling, and normalization.  
> [NVIDIA cuDNN](https://developer.nvidia.com/cudnn)  
> [cuDNN 9.21.1 Downloads](https://developer.nvidia.com/cudnn-downloads)

1. Install `nvidia-pyindex`  
```bash
pip install nvidia-pyindex
```
2. Install `nvidia-cudnn`  
```bash
pip install nvidia-cudnn
```

### Install PyTorch
> [!TIP]
> [PyTorch](https://pytorch.org/get-started/locally/)

Install torch
```bash
pip3 install torch torchvision  # for cuda 13.0
```



### Dependency
```bash
pip install numpy==1.26.4
pip install --no-cache-fir torch==2.4.0 torchaudio==2.4.0 torchvision==0.19.0
python3 -C "import torch; print('PyTorch:', torch.__version__)" # Check version (PyTorch: 2.4.0+cu121)
```

### Uninstall
- [Removing the Driver](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/removing-the-driver.html)
- [Removing CUDA Toolkit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/#removing-cuda-toolkit)

## Removing CUDA Toolkit
```bash
sudo apt remove --purge "*cuda*" "*cublas*" "*cufft*" "*cufile*" "*curand*" "*cusolver*" "*cusparse*" "*gds-tools*" "*npp*" "*nvjpeg*" "nsight*" "*nvvm*"
```


## 👤 Authors

- **[iHaruruki](https://github.com/iHaruruki)** — Main author & maintainer

## 📚 References
#### NVIDIA
- [CUDA Toolkit 13.0 Downloads](https://developer.nvidia.com/cuda-13-0-0-download-archive)
- [NVIDIA CUDA-X](https://www.nvidia.com/ja-jp/technologies/cuda-x/)
- [eGPUでハイスペックLinuxデスクトップをDeep Learning Workstation化計画(eGPUセットアップ編)](https://qiita.com/y-vectorfield/items/8960c804441d2ebd605e)
- [GPUを使った機械学習の環境を作るためにすること/しないこと（Ubuntu 22.04/24.04編）](https://zenn.dev/yuyakato/articles/6915e735bc6aa5)
- [Ubuntu 22.04マシンでGPUを使えるようにする](https://qiita.com/tmasada/items/f77808c870c829c076fa)
- [GPU環境構築からyolo_rosを動かすまで その1](https://zenn.dev/nutechr/articles/2d2df996af2401)
- [NVIDIA ドライバ、NVIDIA CUDA ツールキット 11.8、NVIDIA cuDNN 8 のインストール（Ubuntu 上）](https://www.kkaneko.jp/tools/ubuntu/ubuntu_cudnn.html)
- [CUDA地獄ってなんだ？〜PyTorch環境構築の闘いを終わらせる完全ガイド〜](https://qiita.com/GeneLab_999/items/46eaac97fdd7a884e8d5)

#### About Documents
- [Shields.io](https://shields.io/)
