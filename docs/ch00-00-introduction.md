# はじめに

## bevy_ecs_ldtkとは

`bevy_ecs_ldtk`はBevy用のECSフレンドリなLDtKプラグインです。
LDtKプロジェクトをアセットとして使用したり、レベルの生成、LDtKエンティティ／タイルにBevyコンポーネント／バンドルを挿入することができます。

このプラグインはECSフレンドリで、ECSの内部的な使用によってユーザに追加機能を提供します。
また、タイルマップのレンダリングに`bevy_ecs_tilemap`を使用します。

これらは全て人間工学に基づいたAPIであり、一般的な使用例に対して低ボイラープレートの解決策を提供します。

あまり一般的でない使用例については、このプラグインのECS構造を活用した戦略にも利用できます。

## このドキュメントについて

このドキュメント内は以下のような項目を提供することを目的としています。

- チュートリアル：簡単なゲームを1から最後まで作る。
- 解説：`bevy_ecs_ldtk`で採用されている概念と戦略についての説明。
- 使用ガイド：よくある問題の推奨される解決方法と、移行ガイド。

APIリファレンスでは、`docs.rs`にある`bevy_ecs_ldtk`のドキュメントを参照してください。

このドキュメントは「ドキュメントの大統一理論」を満たすことを目指しています。

以下の章のドキュメントは初心者が最初に読むのに適しており、ここではこれらを読み進めていきます。

- [Tile-based Game tutorial](ch01-00-tile-based-game-tutorial.md)
- [Level Selection explanation](ch03-00-level-selection-explanation.md)
- [Game Logic Integration explanation](ch04-00-game-logic-integration-explanation.md)

## その他リソース

このドキュメントは、BevyやLDtKを学ぶのに適していません。
これらについて学ぶ際は、公式サイトにてお願いしています。

`bevy_ecs_ldtk`のソースコードはGitHubで公開されています。
このリポジトリにはcargoのサンプルも含まれており、`cargo run --example example-name`で実行することができます。

実行の際にはバージョンに注意してください。バージョンが異なる場合、動作しない可能性があります。

## ライセンス

このドキュメントは、MITとApache2.0のデゥアルライセンスです。

## 参考URL

https://trouv.github.io/bevy_ecs_ldtk/v0.9.0/index.html
