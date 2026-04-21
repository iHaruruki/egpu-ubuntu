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