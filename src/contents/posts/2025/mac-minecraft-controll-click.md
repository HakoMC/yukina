---
title: MacのMinecraftでCtrlを押しながらクリックする
published: 2025-01-17
tags:
  - Java版
  - Mac

category: Minecraft
draft: false
---

Macで動作しているMinecraft Java Editionでコントロールキーを押しながらクリックをすると右クリックとなってしまうことへの対処法です。

## 2025/05/10 追記

このModを導入しましょう！
[MacOS Input Fixes](https://modrinth.com/mod/macos-input-fixes)

以下はModを使わない方法です。

---

注意:ソフトのインストールなどは全て自己責任で行なってください！

この記事はほぼこちらをの記事を日本語化したものとなっています。感謝。
[How to disable control to right click in Minecraft on Mac - Finlay Nathan](https://finlaynathan.com/blog/how-to-disable-control-to-right-click-in-minecraft-on-mac/)

## Karabiner-Elementsのインストール

[Homebrew](https://brew.sh)を使用してKarabiner-Elementsという、キー割り当ての変更などができるソフトをインストールします。[公式サイト](https://karabiner-elements.pqrs.org)からdmgファイルをダウンロードしてインストールしてもOK。

`brew install --cask karabiner-elements`

アプリケーションフォルダーに`Karabiner-Elements`が追加されているので、起動します。

すると、さまざまなダイアログが出てくるので、順番に許可していきましょう。

[公式サイト](https://karabiner-elements.pqrs.org/docs/getting-started/installation/)を見た方がわかりやすいと思いますが、日本語での説明も書いておきます。

まずは、Karabinerで使うキーボードの配列を設定します。日本語配列ならJIS、US配列ならANSI、UK配列などの場合はISOを選択します。

次に、ログイン項目を有効にします。システム設定→一般→ログイン項目と機能拡張へ進み、`Karabiner-Elements Non-Privileged Agents`と`Karabiner-Elements Privileged Daemons`の項目を有効にします。

その次に、入力監視を許可します。システム設定→プライバシーとセキュリティ→入力監視へ進み、`karabiner_grabber`を許可します。

最後に、システム拡張を有効にします。システム設定→一般→ログイン項目と機能拡張→ドライバ拡張機能のℹ️マークをクリックし、`.Karabiner-VirtualHIDDevice-Manager`を有効にします。

全て終わったら、タスクバーのKarabiner-Elementsのアイコンをクリックし、Restart Karabiner-Elementsでソフトを再起動します。

## Karabiner-Elementsの設定

インストールができたら、Karabinerの設定をします。

finderを開き、`⌘+⇧+G`で出てきたダイアログに`~/.config/karabiner/assets/complex_modifications`と入力し、Enterを押します。

そこに`minecraft.json`を作成し、次の設定を貼り付けるのですが、javaのパスが必要となるので、確認しておいてください。通常のランチャーを使っている場合は`^/Library/Java/JavaVirtualMachines/.+\\..+/Contents/Home/bin/java$`で行けるようです。私はPrism Launcherを使用しているので、インスタンスの編集→設定からJavaの設定の部分に書かれているパスを使いました。`/Users/ユーザー名/Library/Application Support/PrismLauncher/java/java-runtime-gamma/bin/java$`

```json ~/.config/karabiner/assets/complex_modifications/minecraft.json
{
  "title": "Minecraft Left Control Remap",
  "rules": [
    {
      "description": "Map Left Control to Right Application/Menu",
      "manipulators": [
        {
          "type": "basic",
          "from": {
            "key_code": "left_control",
            "modifiers": {
              "optional": ["any"]
            }
          },
          "to": [
            {
              "key_code": "application"
            }
          ],
          "conditions": [
            {
              "type": "frontmost_application_if",
              "file_paths": ["ここにjavaのパス"]
            }
          ]
        }
      ]
    }
  ]
}
```

指定したアプリケーションを使用しているときに`left_control`を押した場合`application`に置き換えるようです。

保存できたら、これをKarabinerで読み込みます。

Karabiner-Elementsを開き、左のバーの`Complex Modifications`から`Add predefined rule`をクリック、下に`Minecraft Left Control Remap`があるので、`Enable`をクリックします。

この状態で、Minecraftを開きます。

開けたら、キー割り当てでコントロールキーを使いたい動作を設定し直します。すると、日本語の場合は`メニュー`となります。これでコントロールキーを押した状態でもクリックでブロックを壊したり攻撃をしたりすることができるようになりました！

## 参考リンク

[How to disable control to right click in Minecraft on Mac - Finlay Nathan](https://finlaynathan.com/blog/how-to-disable-control-to-right-click-in-minecraft-on-mac/)

[Installation | Karabiner-Elements](https://karabiner-elements.pqrs.org/docs/getting-started/installation/)
