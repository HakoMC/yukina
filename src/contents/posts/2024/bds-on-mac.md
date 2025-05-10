---
title: M1 MacでMinecraft統合版サーバーを動かす
published: 2024-12-04
tags:
  - 統合版
  - サーバー
  - Mac
category: Minecraft
draft: false
---

Script APIの@minecraft/server-netを使いたかったので、Macbookで統合版サーバーを動かそうとしたときにやったことです。

## どうする

Minecraft Bedrock Dedicated Server (以下BDS)は、公式がリリースしている統合版向けのサーバーソフトウェアです。ですがこれは、WindowsとLinux向けにしかリリースされていません。しかもarm64アーキテクチャだとそのままでは動かせません。どうしよう！

## 作戦

だが私には経験がある。Nintendo SwitchにUbuntuをインストールしたとき、Box64を使ってBDSを動かしたことがあるのだ！ということで、Ubuntuの仮想環境を作り、そこでBDSを動かすことにします。

## 仮想環境の構築

M1 Macでは[UTM](https://mac.getutm.app/)という手軽にVMを作れるソフトがあるので、これを使うことにしました。

[Arm版Ubuntuのイメージ](https://ubuntu.com/download/server/arm)もダウンロードしておきましょう。私はUbuntu Server 24.04にしました。

`新規仮想マシンを作成`→`仮想化`→`Linux`と進み、ダウンロードしたイメージを選択します。メモリは4GBにしましたが、統合版なので負荷のかかるようなことをしなければ1GBや2GBでもそれなりに動くと思います。ストレージに余裕があれば、64GBで問題ないでしょう。

以上の設定ができたら仮想マシンを作成します。Ubuntu Serverが起動するので、普通にセットアップをしていきましょう。UTMではコピペができないので、sshで接続できるようにしておくといいです。

## Box64のインストール

Ubuntuのシェルが開けたら、忘れずに`sudo apt update && sudo apt upgrade -y`でパッケージの更新をしておきましょう。

更新が終わったら、[公式の手順](https://github.com/ptitSeb/box64/blob/main/docs/COMPILE.md)の通りにBox64をインストールします。

まずは必要なパッケージをインストールします。後で必要となるunzipも同時にインストールしておきましょう。
`sudo apt install make cmake zip unzip`

あとは以下のコマンドを順番に実行すればインストール完了です。

```bash
git clone https://github.com/ptitSeb/box64
cd box64
mkdir build; cd build; cmake .. -D ARM_DYNAREC=ON -D CMAKE_BUILD_TYPE=RelWithDebInfo
make -j4
sudo make install
sudo systemctl restart systemd-binfmt
```

## BDSのダウンロード

まずは適当なディレクトリを作成し、そこに移動します。

```bash
mkdir bedrock
cd bedrock
```

そして、BDSをwgetやcurlでダウンロードするのですが、なぜか普通に実行するとできません。(昔はできた)なので、User-Agentを偽装します。

[BDS-Versions](https://github.com/Bedrock-OSS/BDS-Versions)という便利なものがあるので、こちらのシェルスクリプトを実行することで、最新版をダウンロードすることができます。

```bash
#!/bin/bash

LATEST_VERSION=`curl -s https://raw.githubusercontent.com/Bedrock-OSS/BDS-Versions/main/versions.json | jq -r '.linux.stable'`

echo "The latest Linux BDS is [${LATEST_VERSION}]"

DOWNLOAD_URL=`curl -s https://raw.githubusercontent.com/Bedrock-OSS/BDS-Versions/main/linux/${LATEST_VERSION}.json | jq -r '.download_url'`

wget --user-agent="Mozilla/5.0" ${DOWNLOAD_URL}
```

執筆時の最新バージョンは`1.21.50.10`なので、`bedrock-server-1.21.50.10.zip`をダウンロードすることができました。

unzipで展開します
`unzip bedrock-server-1.21.50.10.zip`

お疲れ様でした！`LD_LIBRARY_PATH=. ./bedrock_server`を実行すれば、自動的にBox64を使用してBDSが起動します！

あとは`server.properties`をいじるなり、アドオンを導入するなり、好きな使い方をしましょう。中に`LD_LIBRARY_PATH=. ./bedrock_server`と書いた`start.sh`を作成すると、起動が少し楽になります。シェルを閉じても起動しておきたい場合はscreenコマンドを使いましょう。

## 参考リンク

[Minecraft Bedrock Server Download | Minecraft](https://www.minecraft.net/en-us/download/server/bedrock)

[AArch64 な Oracle Cloud Infrastructure で Minecraft統合版サーバーを動かす！【box86/box64】](https://ohayoyogi.hatenablog.com/entry/2021/12/13/231826)

[Box64: Linux Userspace x86-64 Emulator with a Twist](https://github.com/ptitSeb/box64)

[Bedrock-OSS/BDS-Versions: Auto updating BDS Versions ](https://github.com/Bedrock-OSS/BDS-Versions)

[UTM | Virtual Machines for Mac](https://mac.getutm.app/)
