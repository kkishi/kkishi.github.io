---
layout: post
title:  "PhotoPrism"
---

## インストール

### Prerequisites

* [Docker](https://docs.docker.com/engine/install/ubuntu/)
* [Docker compose](https://docs.docker.com/compose/install/)

### PhotoPrism

[公式ドキュメント](https://docs.photoprism.app/)を参照。[Docker Compose](https://docs.photoprism.app/getting-started/docker-compose/)を使った。`docker-compose.yml`を適切に書き換えてから`sudo docker-compose up -d`を実行すればよい。

PhotoPrismは複数アカウントをサポートしていないので、撮影者ごとにインスタンスを分けるのがよい。以下を参照。

* <https://github.com/kkishi/selfhosted/tree/master/photoprism/https>
* <https://github.com/kkishi/selfhosted/tree/master/traefik>

## トラブルシューティング

### This site can’t be reached (DNS\_PROBE\_FINISHED\_NXDOMAIN)

CNAMEレコードが正しく設定されていない。

### 404 page not found

Traefikがルーティングルールを見つけられなかった。

以下を確認する:
* サービスは起動しているか? (`docker-compose ps`)
* サービスは正しく動作しているか？ (`docker-compose logs`)
* TraefikのダッシュボードのHTTP Routers欄にサービスが表示されているか？

### Bad Gateway

サービスがTraefikに正常なレスポンスを返さなかった。

サービスが正しく動作しているか確認する。

### GoProで撮影した動画が表示されない

初回アクセス時にavcにTranscodeされるので、ファイルサイズによってはしばらく時間がかかる。例えば3.8Gのmp4ファイルの場合のログは次のようになった。TranscodingはCPUを使用した。

```sh
$ docker-compose logs
...
photoprism_1  | time="2022-12-27T08:44:12Z" level=info msg="libx264: transcoding GH010338.MP4 to avc"
photoprism_1  | time="2022-12-27T08:47:38Z" level=info msg="libx264: created GH010338.MP4.avc [3m25.560385646s]"

```

備考: [NVIDIA Container Toolkit](https://docs.photoprism.app/getting-started/advanced/transcoding/#nvidia-container-toolkit)を設定するとGPUが使用されて倍速ほどになった。

```sh
photoprism_1  | time="2022-12-27T12:02:08Z" level=info msg="h264_nvenc: transcoding GH010338.MP4 to avc"
photoprism_1  | time="2022-12-27T12:04:04Z" level=info msg="h264_nvenc: created GH010338.MP4.avc [1m55.514401506s]"
```

上記の機能はスポンサー限定なので注意。デフォルトでは以下のエラーが表示され、CPUが使用される。

```sh
photoprism_1  | time="2022-12-27T10:46:22Z" level=info msg="ffmpeg: hardware transcoding is available to sponsors only"
```

### GoProで撮影した動画のサムネイルが表示されない

一部の動画に対してサムネイルが生成されない。`docker-compose exec photoprism photoprism thumbs`を実行しても対象の動画ファイルが検出されていないように見える。

以下の行をdocker-compose.ymlに追加すると動的（UIでファイルを表示しようとしたときにサムネイルが無い場合）にもサムネイルが生成される。フラグ名からだと分かりづらいが、インデックス時に静的にサムネイルを生成する挙動は変わらない。

```sh
PHOTOPRISM_THUMB_UNCACHED: "true"
```
