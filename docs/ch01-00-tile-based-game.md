# タイルベースゲーム

このチュートリアルでは、LDtKレベルでタイルベースのゲームを作成します。
このゲームでは、倉庫番やスネークのようなタイルのグリッドに固定されます。

LDtKプロジェクトを作成し、プロジェクトをBevyにロードし、ゲームプレイを追加する手順を説明します。

またこのチュートリアルの完成系は`bevy_ecs_ldtk`リポジトリの`tile_based_game`にあります。

```sh
cargo run --example tile_based_game --release
```

## 必要なもの

以下のセットアップとインストールを行う必要があります。

- 互換性チャートで指定されたバージョンのBevyプロジェクト
- 互換性チャートで指定されたバージョンのLDtK。

またリソースをダウンロードする必要があります。

- 背景タイル、壁タイル、ゴールっぽいタイル
- プレイヤー用のタイルセット

このチュートリアルでは、[SunnyLand by Ansimuz](https://ansimuz.itch.io/sunny-land-pixel-game-art)のリソースを使用します。
しかし上記の目的に適したタイルがあれば、どのタイルセットを使用してもチュートリアルを進めることができます。

## 参考URL

https://trouv.github.io/bevy_ecs_ldtk/v0.9.0/tutorials/tile-based-game/index.html
