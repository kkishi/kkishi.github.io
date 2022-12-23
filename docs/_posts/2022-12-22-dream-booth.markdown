---
layout: post
title: DreamBooth
---

# 手順

<https://github.com/d8ahazard/sd_dreambooth_extension>と<https://www.youtube.com/watch?v=_GmGnMO8aGs>を参考にした。

## 前提条件

* 推奨: Python version>=3.10 (自分は3.8.10を使ったが、その場合コードを何箇所か書き換える必要があった）
* CUDA Toolkit
* 学習させる画像512x512を12枚。自分は顔写真を準備した。顔がはっきり写っているものを中心に、特に背景などは加工せずクロップと縮小をしただけだったがなんとかなった。

## Dreambooth extensionのインストール

Extensions > Available > Load from: をクリックして出る一覧の中から`Dreambooth`をインストールする。

NOTE: 自分は`webui-user.sh`で`export COMMANDLINE_ARGS="--listen"`としていたせいで`AssertionError: extension access disabed because of commandline flags`というエラーが出た。<https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/4215>を参考にしてその行をコメントアウトすると上手く行く。インストールが終われば戻してよい。

インストールが終わるとDreamboothタブが出る。また、`stable-diffusion-webui/extensions/sd_dreambooth_extension/`というディレクトリが作成されて、そこにコードなどが追加される。

## モデルの作成

Dreamboothタブ > Create Modelタブで以下を入力する。

* Name: nina
* Source Checkpoint: sd-v1-4.ckpt

Createをクリックしてしばらくすると、左上にあるModelセレクトボックスでninaが選択できるようになる。

## トレーニング

Modelセレクトボックスでninaを選択する。

`Concepts`タブ > `Concept 1`タブで以下を入力する。

* Dataset Directory: 画像の保存されているディレクトリへのパス
* Instance Prompt: Conceptを指し示すのに用いるPrompt

右上のTrainボタンをクリックする。正しく実行されれば20分ほどで学習が終わる(デフォルトだと画像一枚ごとに100ステップなので、合計1200ステップ)が、エラーにいくつか遭遇するので頑張って対処する。

### Pythonのバージョンによるエラー

自分はPython 3.8.10を使っていたために`TypeError: 'type' object is not subscriptable`というエラーが出た。<https://github.com/TheLastBen/fast-stable-diffusion/issues/914>を参考にコードを何箇所か書き換えると上手く行く。

### メモリ不足によるエラー

自分はRTX 3060 12GBを使用したが、メモリ不足で`max_split_size_mb`というエラーが出るので、Parametersタブ > Advancedで以下を設定する。

* Use 8bit Adam
* Mixed Precision > fp16
* Memory Attention > flash_attention

### CUDA Toolkitがインストールされていない事によるエラー

`NameError: name 'str2optimizer8bit_blockwise' is not defined`というエラーが出る。また、次のエラーが出る。

```
===================================BUG REPORT===================================
Welcome to bitsandbytes. For bug reports, please submit your error trace to: https://github.com/TimDettmers/bitsandbytes/issues
For effortless bug reporting copy-paste your error into this form: https://docs.google.com/forms/d/e/1FAIpQLScPB8emS3Thkp66nvqwmjTEgxp8Y9ufuWTzFyr9kJ5AoI
47dQ/viewform?usp=sf_link
================================================================================
CUDA_SETUP: WARNING! libcudart.so not found in any environmental path. Searching /usr/local/cuda/lib64...
WARNING: No libcudart.so found! Install CUDA or the cudatoolkit package (anaconda)!
CUDA SETUP: Loading binary /home/keisuke/projects/stable-diffusion-webui/venv/lib/python3.8/site-packages/bitsandbytes/libbitsandbytes_cpu.so...
 Scheduler, EMA Loaded.
 Allocated: 3.8GB
 Reserved: 3.9GB
```

確かにlibcudart.soが/usr/local/cuda/や/usr/lib/x86_64-linux-gnu/などに見当たらない。

#### 間違った解決方法

`sudo apt install nvidia-cuda-toolkit`で何らかのCUDA Toolkitがインストールされるが、これだと自分のドライバー(以前`sudo apt install nvidia-driver-515`で導入した)のCUDA Version (`11.6`だった、これは`nvidia-smi`コマンドで調べることができる)と互換性がないものがインストールされてしまう。具体的には、以下のエラーが出る。

```sh
===================================BUG REPORT===================================
Welcome to bitsandbytes. For bug reports, please submit your error trace to: https://github.com/TimDettmers/bitsandbytes/issues
For effortless bug reporting copy-paste your error into this form: https://docs.google.com/forms/d/e/1FAIpQLScPB8emS3Thkp66nvqwmjTEgxp8Y9ufuWTzFyr9kJ5AoI47dQ/viewform?usp=sf_link
================================================================================
CUDA SETUP: CUDA runtime path found: /usr/lib/x86_64-linux-gnu/libcudart.so
CUDA SETUP: Highest compute capability among GPUs detected: 8.6
CUDA SETUP: CUDA version lower than 11 are currenlty not supported for LLM.int8(). You will be only to use 8-bit optimizers and quantization routines!!
CUDA SETUP: Detected CUDA version 101
CUDA SETUP: TODO: compile library for specific version: libbitsandbytes_cuda101.so
CUDA SETUP: Defaulting to libbitsandbytes.so...
CUDA SETUP: CUDA detection failed. Either CUDA driver not installed, CUDA not installed, or you have multiple conflicting CUDA libraries!
CUDA SETUP: If you compiled from source, try again with `make CUDA_VERSION=DETECTED_CUDA_VERSION` for example, `make CUDA_VERSION=113`.
Exception importing 8bit adam: CUDA SETUP: Setup Failed!
Traceback (most recent call last):
  File "/home/keisuke/projects/stable-diffusion-webui/extensions/sd_dreambooth_extension/dreambooth/train_dreambooth.py", line 600, in main
    import bitsandbytes as bnb
  File "/home/keisuke/projects/stable-diffusion-webui/venv/lib/python3.8/site-packages/bitsandbytes/__init__.py", line 6, in <module>
    from .autograd._functions import (
  File "/home/keisuke/projects/stable-diffusion-webui/venv/lib/python3.8/site-packages/bitsandbytes/autograd/_functions.py", line 5, in <module>
    import bitsandbytes.functional as F
  File "/home/keisuke/projects/stable-diffusion-webui/venv/lib/python3.8/site-packages/bitsandbytes/functional.py", line 13, in <module>
    from .cextension import COMPILED_WITH_CUDA, lib
  File "/home/keisuke/projects/stable-diffusion-webui/venv/lib/python3.8/site-packages/bitsandbytes/cextension.py", line 41, in <module>
    lib = CUDALibrary_Singleton.get_instance().lib
  File "/home/keisuke/projects/stable-diffusion-webui/venv/lib/python3.8/site-packages/bitsandbytes/cextension.py", line 37, in get_instance
    cls._instance.initialize()
  File "/home/keisuke/projects/stable-diffusion-webui/venv/lib/python3.8/site-packages/bitsandbytes/cextension.py", line 27, in initialize
    raise Exception('CUDA SETUP: Setup Failed!')
Exception: CUDA SETUP: Setup Failed!
 Scheduler, EMA Loaded.
 Allocated: 3.8GB
 Reserved: 3.9GB
```

とりあえずアンインストールする。

```sh
$ sudo apt remove nvidia-cuda-toolkit 
$ sudo apt autoremove
```

#### CUDA Toolkitを公式ページからインストール

<https://developer.nvidia.com/cuda-11-6-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_local>の手順に従う。

NOTE: どうやらドライバーも同時にインストールされるらしい。後で調べたらバージョンが510に変更されていた。

インストール後はPCを再起動する必要がある。

## 画像の生成

トレーニングが成功すると`stable-diffusion-webui/models/Stable-diffusion/nina_1200.ckpt`のようなファイルが生成される。これはwebuiの左上の`Stable Diffusion checkpoint`セレクトボックスから選択できて、後はtxt2imgなどから通常通りに画像が生成出来る。

テスト用プロンプト:

* `Instance Prompt`, drawn by vincent van gogh
* photograph of `Instance Prompt`, portrait
