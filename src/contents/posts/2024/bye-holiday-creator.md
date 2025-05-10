---
title: ホリデークリエイターの削除に対応しよう
published: 2024-11-12
tags:
  - 統合版
  - アドオン

category: Minecraft
draft: false
---

Minecraft統合版1.21.20でホリデークリエイターの実験が削除されたので、私がした対応をまとめます。
この変更により、ブロックの回転やプレイヤーの設置方向に関する処理が大きく変わりました。
主に変更するのはビヘイビアのblocksフォルダ内のjsonファイルです。

## 1.21.20より前のjson

まず旧バージョンのjsonを見ていきましょう。例として縁石ブロックのjsonをここに貼ります。

```json BEH/blocks/enseki_center.json
{
  "format_version": "1.19.50",
  "minecraft:block": {
    "description": {
      "identifier": "hakomc:enseki_center",
      "properties": {
        "hakomc:facing": [2, 3, 4, 5]
      },
      "menu_category": {
        "category": "construction"
      }
    },
    "components": {
      "minecraft:collision_box": {
        "origin": [3, 0, -8],
        "size": [5, 6, 16]
      },
      "minecraft:destructible_by_explosion": {
        "explosion_resistance": 6400
      },
      "minecraft:destructible_by_mining": {
        "seconds_to_destroy": 6.0
      },
      "minecraft:flammable": {
        "catch_chance_modifier": 0,
        "destroy_chance_modifier": 0
      },
      "minecraft:friction": 0.4,
      "minecraft:geometry": "geometry.enseki_center",
      "minecraft:light_dampening": 0,
      "minecraft:light_emission": 0,
      "minecraft:loot": "",
      "minecraft:map_color": "#b0b0b0",
      "minecraft:material_instances": {
        "*": {
          "texture": "joukei1",
          "render_method": "opaque"
        }
      },
      "minecraft:selection_box": {
        "origin": [3, 0, -8],
        "size": [5, 6, 16]
      },
      "minecraft:on_player_placing": {
        "event": "hakomc:placed"
      }
    },
    "events": {
      "hakomc:placed": {
        "set_block_property": {
          "hakomc:facing": "query.cardinal_facing_2d"
        }
      }
    },
    "permutations": [
      {
        "condition": "query.block_property('hakomc:facing') == 2",
        "components": {
          "minecraft:rotation": [0, 270, 0]
        }
      },
      {
        "condition": "query.block_property('hakomc:facing') == 3",
        "components": {
          "minecraft:rotation": [0, 90, 0]
        }
      },
      {
        "condition": "query.block_property('hakomc:facing') == 4",
        "components": {
          "minecraft:rotation": [0, 0, 0]
        }
      },
      {
        "condition": "query.block_property('hakomc:facing') == 5",
        "components": {
          "minecraft:rotation": [0, 180, 0]
        }
      }
    ]
  }
}
```

minecraft:on_player_placingコンポーネントでプレイヤーがブロックを設置した時にhakomc:placedイベントを発火し、プレイヤーの向きに応じてジオメトリが回転する機能が付いています。

## 1.21.20以降のjson

```json BEH/blocks/enseki_center.json
{
  "format_version": "1.20.20",
  "minecraft:block": {
    "description": {
      "identifier": "hakomc:enseki_center",
      "states": {
        "hakomc:facing": [2, 3, 4, 5]
      },
      "menu_category": {
        "category": "construction"
      },
      "traits": {
        "minecraft:placement_direction": {
          "enabled_states": ["minecraft:facing_direction"]
        }
      }
    },
    "components": {
      "minecraft:collision_box": {
        "origin": [3, 0, -8],
        "size": [5, 6, 16]
      },
      "minecraft:destructible_by_explosion": {
        "explosion_resistance": 6400
      },
      "minecraft:destructible_by_mining": {
        "seconds_to_destroy": 6.0
      },
      "minecraft:flammable": {
        "catch_chance_modifier": 0,
        "destroy_chance_modifier": 0
      },
      "minecraft:friction": 0.4,
      "minecraft:geometry": "geometry.enseki_center",
      "minecraft:light_dampening": 0,
      "minecraft:light_emission": 0,
      "minecraft:loot": "",
      "minecraft:map_color": "#b0b0b0",
      "minecraft:material_instances": {
        "*": {
          "texture": "joukei1",
          "render_method": "opaque"
        }
      },
      "minecraft:selection_box": {
        "origin": [3, 0, -8],
        "size": [5, 6, 16]
      }
    },
    "permutations": [
      {
        "condition": "query.block_state('hakomc:facing') == 2 || q.block_state('minecraft:facing_direction') == 'north' ",
        "components": {
          "minecraft:transformation": {
            "rotation": [0, 270, 0]
          }
        }
      },
      {
        "condition": "query.block_state('hakomc:facing') == 3 || q.block_state('minecraft:facing_direction') == 'south' ",
        "components": {
          "minecraft:transformation": {
            "rotation": [0, 90, 0]
          }
        }
      },
      {
        "condition": "query.block_state('hakomc:facing') == 4 || q.block_state('minecraft:facing_direction') == 'west' ",
        "components": {
          "minecraft:transformation": {
            "rotation": [0, 0, 0]
          }
        }
      },
      {
        "condition": "query.block_state('hakomc:facing') == 5 || q.block_state('minecraft:facing_direction') == 'east' ",
        "components": {
          "minecraft:transformation": {
            "rotation": [0, 180, 0]
          }
        }
      }
    ]
  }
}
```

## 変更点

### フォーマットバージョン

まず初めにフォーマットバージョンが`1.19.50`から`1.20.20`に変わりました。1.19から1.20でproperties周りに関する書式が大幅に変わったのでそれに対応するためです。

### descriptionプロパティ

descriptionプロパティでは、`"properties"`が`"states"`に変更となりました。プレイヤーの向きに応じて回転させるなどの機能を持たせる場合は、後述する`"traits"`を使用することで`"states"`は不要となりますが、過去のバージョンで`"properties"`を使用していたブロックを最新バージョンでも使えるようにするためには、`"states"`が必要になります。

また、`"traits"`が追加となりました。これは、カスタムブロックでイベントやトリガーを定義したり管理したりすることなく、プレイヤーの向きに応じて回転させるなどの機能を持たせることができるようにするものです。この縁石ブロックでは、`"minecraft:placement_direction"`を使って、`"minecraft:facing_direction"`を有効化することで、プレイヤーの向きに応じて回転させることができるようにしています。[公式ドキュメント](https://learn.microsoft.com/ja-jp/minecraft/creator/reference/content/blockreference/examples/blocktraits?view=minecraft-bedrock-stable)を確認することをお勧めします。

### componentsプロパティ

componentsプロパティでは、`"minecraft:on_player_placing"`コンポーネントが1.20.20で削除され、この機能はtraitsに移されたので、この部分を削除しました。

### eventsプロパティ

eventsプロパティは全体を削除しました。これも先ほどのcomponentsと同様に、traitsに機能が移されたためです。

### permutationsプロパティ

permutationsプロパティでは、`"condition"`でmolang演算子"or"(||)を使って、descriptionプロパティで定義したstatesの`"hakomc:facing"`とtraitsの`"minecraft:facing_direction"`どちらかがそれぞれの値だった場合に、`"components"`を設定するようにしました。

この`"components"`も、`"minecraft:rotation"`が削除され`"minecraft:transformation"`へと変わったのでそれぞれ変更しました。

```json 旧rotation
"components": {
  "minecraft:rotation": [0, 0, 0]
}
```

↓これに変更

```json 新transformation
"components": {
  "minecraft:transformation": {
    "rotation": [0, 0, 0]
  }
}
```

## まとめ

ホリデークリエイターの削除に対する主な変更点は以下の通りです：

- フォーマットバージョンが`1.20.20`以上に
- `properties`が`states`に変更
- `traits`の導入
- `rotation`コンポーネントが`transformation`に変更

これらの変更に対応することで、最新バージョンでもプレイヤーの向きに応じて回転するブロックなどを作ることができます！

質問があれば下にあるコメント欄からどうぞ！

## 参考リンク

[Block Documentation - Block Traits | Microsoft Learn](https://learn.microsoft.com/ja-jp/minecraft/creator/reference/content/blockreference/examples/blocktraits?view=minecraft-bedrock-stable)

[Minecraft Block Wizard](https://www.blockbench.net/plugins/minecraft_block_wizard)

[Minecraftにオリジナルブロックを追加する | とかさんのブログ](https://tokamcwin10.blog.jp/article-37132278)
