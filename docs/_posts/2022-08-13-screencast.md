---
layout: post
title: Screencast
---

Ubuntu 20.04でscreencastする方法。


デフォルトだと30秒で録画が止まってしまうので、設定を変更する。

```
$ gsettings set org.gnome.settings-daemon.plugins.media-keys max-screencast-length 60
```

Ctrl+Alt+Shift+Rで録画が始まる。デフォルトでは~/Videosに保存される。

参考
* <https://help.ubuntu.com/stable/ubuntu-help/screen-shot-record.html>
* <https://askubuntu.com/questions/966633/how-to-make-screencast-on-ubuntu-17-10-record-longer-than-30s>

追記: OBSを使う方がよい。
