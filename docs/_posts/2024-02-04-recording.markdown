---
layout: post
title: 録画
---

## yt-dlp

Youtubeでなくても使える場合があるのでとりあえずyt-dlpを試すのがよい。

### Youtube Live

Youtube Liveはデフォルトだと最新時刻からの録画となる。はじめから録画したい場合は`--live-from-start`を使う。

今回は以下のようなファイルが４つ生成された。

* title.f140.mkv.part
* title.f140.mkv.ytdl
* title.f248.mkv.part
* title.f248.mkv.ytdl

.ytdlファイルには`{"downloader": {"current_fragment": {"index": 1601}, "fragment_count": 1601}}`のようにメタデータが保存された。f140.mkv.partには音声が、f248.mkv.partには動画が保存された。これらは以下のコマンドで一つの動画ファイルに統合できた。

```
$ ffmpeg -i title.f140.mkv -i title.f248.mkv -c copy -f mp4 output.mp4
```

## OBS

注意点

* マウスポインタを消す
* FPSを揃える
* 解像度を揃える

### トラブルシューティング

録画時に間違った解像度を使ってしまった場合は以下のようにして変換する。

```
ffmpeg -i $input -vf "scale=1920:1080" -c:a copy $output
```

ファイルサイズが予想以上に小さくなるがそういうものらしい。

<https://superuser.com/questions/1608072/why-are-ffmpeg-processed-videos-per-default-multiple-times-smaller-in-file-size>
