# egpu-ubuntu

## 🚀 Overview
- How to Use an eGPU on Ubuntu 22.04.
- Install **NVIDIA Driver**, **NVIDIA CUDA TOOLKIT**, **NVIDIA cuDNN**.

## 💻 Verification environment
| Types of parts | Model number |
| --- | --- |
| Graphics Cards | GeForce RTX 3070 Ti |
| eGPU | Razer core x |
| CPU | Intel® Core™ i7-1260P |
| OS | Ubuntu22.04 |

## :mag_right: Structure

| | | 
| --- | --- |
| Application Layer | PyTorch / TensorFlow |
| Library Layer     | cuDNN / cuBLAS / TensorRT |
| Runtime Layer     | CUDA Toolkit |
| Driver Layer      | NVIDIA Driver |
| Hardware Layer    | GPU Hardware |

## 🛠️ Setup

### セキュアブートの無効化
1. Intel NUCのBIOSへの入り方は起動時にF2キーを連打する
2. BIOS画面で`Boot`>>`Boot/Secure Boot`と進み`Secure Boot`を**Disabled**にする

### UEFIがサードバーをロードすることを許可
BIOS画面で`Seurity`>>`Security Features`>>`Allow UEFI 3rd party driver loaded`にチェックを入れ有効化する

### eGPUの接続
1. eGPUとNUCをUSB Type-Cで接続する
2. eGPUの電源を入れる
3. Ubuntu 22.04を起動

### Thunerbolt 3機器のuuidの確認
boltctlコマンドでThunderbolt 3機器(Razer CoreX)のuuidを調べる
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

### NVIDAドライバーのセットアップ
Ubuntuのaptにリポジトリを追加し,ドライバーをインストールします
```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
ubuntu-drivers devices
```
<details>
<summary>List of ubuntu-drivers devices</summary>

```bash
ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:07.2/0000:2c:00.0/0000:2d:01.0/0000:2e:00.0 ==
modalias : pci:v000010DEd00002482sv000010DEsd0000146Abc03sc00i00
vendor   : NVIDIA Corporation
model    : GA104 [GeForce RTX 3070 Ti]
driver   : nvidia-driver-580-server-open - distro non-free
driver   : nvidia-driver-570-open - distro non-free
driver   : nvidia-driver-580-server - distro non-free
driver   : nvidia-driver-590-server-open - distro non-free
driver   : nvidia-driver-470-server - distro non-free
driver   : nvidia-driver-565-open - third-party non-free
driver   : nvidia-driver-565 - third-party non-free
driver   : nvidia-driver-590-server - distro non-free
driver   : nvidia-driver-590 - distro non-free
driver   : nvidia-driver-470 - distro non-free
driver   : nvidia-driver-545 - distro non-free
driver   : nvidia-driver-535-server-open - distro non-free
driver   : nvidia-driver-535-server - distro non-free
driver   : nvidia-driver-590-open - distro non-free
driver   : nvidia-driver-545-open - distro non-free
driver   : nvidia-driver-580-open - distro non-free recommended
driver   : nvidia-driver-535-open - distro non-free
driver   : nvidia-driver-535 - distro non-free
driver   : nvidia-driver-570 - distro non-free
driver   : nvidia-driver-570-server-open - distro non-free
driver   : nvidia-driver-570-server - distro non-free
driver   : nvidia-driver-580 - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin
```
</details>

recommendedが付いているバージョンをインストールする
```bash
sudo apt install nvidia-driver-xxx-xxxx
```
再起動
```bash
sudo reboot
```

### GPUの状態を確認
```bash
nvidia-smi
```

<details>
<summary>nvidia-smi</summary>

```bash
Tue Apr 21 13:56:33 2026       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.126.09             Driver Version: 580.126.09     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3070 Ti     Off |   00000000:2E:00.0 Off |                  N/A |
|  0%   32C    P8              9W /  290W |     181MiB /   8192MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A            3564    C+G   ...c/gnome-remote-desktop-daemon        163MiB |
+-----------------------------------------------------------------------------------------+
```
</details>

### NVIDIA CUDA Toolkit のインストール
1. Ubuntuのバージョンを確認
```bash
uname -m  # アーキテクチャ（x86_64等）を確認
lsb_release -a  # ディストリビューション名とバージョンを確認
```
<details>
<summary>nvidia-smi</summary>

```bash
$ uname -m
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

### Dependency
```bash
pip install numpy==1.26.4
pip install --no-cache-fir torch==2.4.0 torchaudio==2.4.0 torchvision==0.19.0
python3 -C "import torch; print('PyTorch:', torch.__version__)" # Check version (PyTorch: 2.4.0+cu121)
```


## 👤 Authors

- **[iHaruruki](https://github.com/iHaruruki)** — Main author & maintainer

## 📚 References
- [CUDA Toolkit 13.0 Downloads](https://developer.nvidia.com/cuda-13-0-0-download-archive)
- [NVIDIA CUDA-X](https://www.nvidia.com/ja-jp/technologies/cuda-x/)
- [eGPUでハイスペックLinuxデスクトップをDeep Learning Workstation化計画(eGPUセットアップ編)](https://qiita.com/y-vectorfield/items/8960c804441d2ebd605e)
- [GPUを使った機械学習の環境を作るためにすること/しないこと（Ubuntu 22.04/24.04編）](https://zenn.dev/yuyakato/articles/6915e735bc6aa5)
- [Ubuntu 22.04マシンでGPUを使えるようにする](https://qiita.com/tmasada/items/f77808c870c829c076fa)
- [GPU環境構築からyolo_rosを動かすまで その1](https://zenn.dev/nutechr/articles/2d2df996af2401)
- [NVIDIA ドライバ、NVIDIA CUDA ツールキット 11.8、NVIDIA cuDNN 8 のインストール（Ubuntu 上）](https://www.kkaneko.jp/tools/ubuntu/ubuntu_cudnn.html)
- [CUDA地獄ってなんだ？〜PyTorch環境構築の闘いを終わらせる完全ガイド〜](https://qiita.com/GeneLab_999/items/46eaac97fdd7a884e8d5)