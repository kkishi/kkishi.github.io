---
layout: post
title: Stable Diffusion
---

# 環境

* ASUS GeForce TUF RTX 3060 O12G-V2-GAMING LHR
* Linux saginomiya 5.15.0-56-generic #62~20.04.1-Ubuntu SMP Tue Nov 22 21:24:20 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

# 手順

## GPUドライバーのインストール

備考: GPUを設置してPCを起動すると何らかのドライバーが使用されてディスプレイに映像が表示されるが、このままではwebuiのインストール時に`Torch is not able to use gpu`というエラーが出てしまうので、NVIDIAのドライバーを明示的に使用する必要がある。

```sh
$ sudo apt install nvidia-driver-515
$ sudo prime-select nvidia
$ sudo reboot
```

再起動後、ドライバーが正しく使用されている事を以下のコマンドで確認する。

```sh
$ nvidia-smi
Sun Dec 18 16:28:31 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.86.01    Driver Version: 515.86.01    CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:0A:00.0  On |                  N/A |
|  0%   42C    P8    10W / 170W |     27MiB / 12288MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      5703      G   /usr/lib/xorg/Xorg                 18MiB |
|    0   N/A  N/A      6025      G   /usr/bin/gnome-shell                7MiB |
+-----------------------------------------------------------------------------+

```


備考: 最初は`nvidia-driver-525`をインストールしたが、再起動後に以下の症状が出た。

* BIOS等のメニューは出るが、その後Ubuntuのロゴなどは出ずに、ブラックスクリーンが表示され、左上でカーソルが点滅する。
* ただし、ssh接続は受け付ける。

以下のコマンドで正常に起動できるようになった。

```sh
sudo prime-select intel
sudo apt remove nvidia-driver-525
```

## webuiのインストール

<https://github.com/AUTOMATIC1111/stable-diffusion-webui#installation-and-running>を参考にする。

```sh
$ sudo apt install wget git python3 python3-venv
```

備考: aptでインストールされるPythonのバージョンは`3.8.10`と、Required Dependenciesにある`3.10.6`より低かったが、特に問題は無かった。

```sh
$ cd ~/projects/
$ git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
$ cd ~/projects/stable-diffusion-webui/
```

**以降、`stable-diffusion-webui`内で作業するものとする。**

`webui-user.sh`の以下の行をuncommentする。

```sh
#python_cmd="python3"
```

ここで`./webui.sh`を実行すると`ERROR: Cannot activate python venv, aborting...`というエラーが出る。[issues#1120](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/1120)を参考に、以下のコマンドを実行する。

```sh
python3 -m venv venv/
```

再度`./webui.sh`を実行すると`AttributeError: module 'h11' has no attribute 'Event'`というエラーが出る。[issues#4833](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/4833)を参考に、以下のコマンドを実行する。

```sh
source venv/bin/activate
pip install --force-reinstall httpcore==0.15
```

再度`./webui.sh`を実行するとHTTPサーバーが起動しURLが表示される。
