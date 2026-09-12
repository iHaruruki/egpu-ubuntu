# egpu-ubuntu

## 🚀 Overview
- How to Use an eGPU on Ubuntu.
- Install **NVIDIA Driver**, **NVIDIA CUDA TOOLKIT**, **NVIDIA cuDNN**.

> [!IMPORTANT]
> 本ドキュメントで記載される情報は，可能な限り正確の情報であるように努めますが，必ずしも正確性を保証することはできません．誤情報が含まれる可能性があるため，必ずNVIDIAの公式ドキュメントを確認するようにしてください．  
> Be sure to check NVIDIA's official documentation.

## 💻 Verification environment
| Types of parts | Model number |
| --- | --- |
| Graphics Cards | GeForce RTX 3070 Ti |
| eGPU Box | Razer Core X |
| PC | GMKtec K8 Plus |
| CPU | AMD Ryzen™ 7 8845HS |
| OS | Ubuntu 24.04.4 LTS |

## 🔎 Structure

| Layer | Library |
| --- | --- |
| Application Layer | PyTorch / TensorFlow |
| Library Layer     | cuDNN / cuBLAS / TensorRT |
| Runtime Layer     | CUDA Toolkit |
| Driver Layer      | NVIDIA Driver |
| Hardware Layer    | GPU Hardware |

## Install version

- Driver: 610.43.02
- CUDA Toolkit: 13.3
- cuDNN: 9.24.1

## 🛠️ Setup

### BISO settings
1. GMKtec K8 PlusのBIOSへの入り方は起動時に`Esc`キーを連打する
2. BIOS画面で`Security`と進み`Secure Boot`を**Disabled**にする
3. BIOS画面で`Advanced`と進み`PCI Subsystem Settings`

| 設定項目 | 値 |
| --- | --- |
| Above 4G Decoding | Enabled | 
| Re-Size BAR Support | Enabled |
| SR-IOV Support | Enabled |
| IOMMU | Enabled |

4. BIOS画面で`Advanced` >> `AMD CBS`と進み`IOMMU`を**Enabled**にする

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

### Update kernel
```bash
sudo apt install linux-image-generic-hwe-24.04
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

### Install NVIDIA CUDA-X Libraries

> [!TIP]
> NVIDIA CUDA-X is a comprehensive collection of highly optimized, domain-specific libraries, frameworks, and microservices built on top of the core NVIDIA CUDA parallel computing platform.  
> [NVIDIA CUDA-X Libraries](https://developer.nvidia.com/cuda/cuda-x-libraries)  

#### Install NVIDIA cuDNN (NVIDIA CUDA Deep Neural Network library)

1. Check [cuDNN 9.24.1 Downloads](https://developer.nvidia.com/cudnn-downloads)
2. Select Target Platform  
   - Operating System: Linux
   - Architecture: x86_64
   - Distribution: Ubuntu
   - Version: 24.04
   - Installer Type: deb(local)
   - Configuration: FULL

Installation Instructions:
```bash
wget https://developer.download.nvidia.com/compute/cudnn/9.24.1/local_installers/cudnn-local-repo-ubuntu2404-9.24.1_1.0-1_amd64.deb
sudo dpkg -i cudnn-local-repo-ubuntu2404-9.24.1_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-ubuntu2404-9.24.1/cudnn-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cudnn
```

To install for CUDA 13, perform the above configuration but install the CUDA 13 specific package:
```bash
sudo apt-get -y install cudnn9-cuda-13
```

> [!NOTE]
> Check [NVIDIA cuDNN](https://docs.nvidia.com/deeplearning/cudnn/latest/)

## Uninstall

- [Removing the Driver](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/removing-the-driver.html)
- [Removing CUDA Toolkit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/#removing-cuda-toolkit)

## Removing CUDA Toolkit

```bash
sudo apt remove --purge "*cuda*" "*cublas*" "*cufft*" "*cufile*" "*curand*" "*cusolver*" "*cusparse*" "*gds-tools*" "*npp*" "*nvjpeg*" "nsight*" "*nvvm*"
```


## 👤 Authors

- **[iHaruruki](https://github.com/iHaruruki)** — Main author & maintainer

## 📚 References
参考にさせていただいたサイトの一覧．

**NVIDIA**

- [CUDA Toolkit 13.0 Downloads](https://developer.nvidia.com/cuda-13-0-0-download-archive)
- [NVIDIA CUDA-X](https://www.nvidia.com/ja-jp/technologies/cuda-x/)
- [eGPUでハイスペックLinuxデスクトップをDeep Learning Workstation化計画(eGPUセットアップ編)](https://qiita.com/y-vectorfield/items/8960c804441d2ebd605e)
- [GPUを使った機械学習の環境を作るためにすること/しないこと（Ubuntu 22.04/24.04編）](https://zenn.dev/yuyakato/articles/6915e735bc6aa5)
- [Ubuntu 22.04マシンでGPUを使えるようにする](https://qiita.com/tmasada/items/f77808c870c829c076fa)
- [GPU環境構築からyolo_rosを動かすまで その1](https://zenn.dev/nutechr/articles/2d2df996af2401)
- [NVIDIA ドライバ、NVIDIA CUDA ツールキット 11.8、NVIDIA cuDNN 8 のインストール（Ubuntu 上）](https://www.kkaneko.jp/tools/ubuntu/ubuntu_cudnn.html)
- [CUDA地獄ってなんだ？〜PyTorch環境構築の闘いを終わらせる完全ガイド〜](https://qiita.com/GeneLab_999/items/46eaac97fdd7a884e8d5)

**About Documents**

- [Shields.io](https://shields.io/)
