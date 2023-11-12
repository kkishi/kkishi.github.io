---
layout: post
title: Archlinux
---

## インストール時の雑多な注意点

* X220でarchinstallを使う場合Wi-Fiだと失敗しがちなのでEthernetケーブルを使用する。
* インストール後の起動時にDHCPクライアントが無いせいでネットワークに繋げず困るので、`dhcpcd`を初期インストールしておき`systemctl enable dhcpcd.service`などする。

## Gnome

以下を参考にする。

<https://www.youtube.com/watch?v=8nlo7LewC5Q>
