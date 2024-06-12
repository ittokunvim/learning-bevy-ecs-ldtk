# ゲームプレイをプロジェクトに加える

このセクションでは、前のセクションで作成したプロジェクトにゲームプレイを統合します。
これにはタイルベースの移動、衝突、レベル遷移が含まれます。

## プレイヤーにマーカーコンポーネントと`GridCoords`を追加

タイルベースの移動とタイルベースの構造を処理するには、BevyのWorld変換だけでなく、タイル空間のエンティティの位置を処理する必要があります。

`bevy_ecs_ldtk`は、これに適したコンポーネントを提供しており、`LdtkEntity`派生と統合されています。
`GridCoords`コンポーネントを`PlayerBundle`に追加し、`#[grid_coords]`属性を与えます。

`Player`エンティティは、グリッド空間におけるエンティティの位置と一致する値を持つ`GtidCoords`コンポーネントと共に生成されます。

また将来のシステムでより簡単に照会できるように、`Player`にマーカーコンポーネントを与えます。
`bevy_ecs_ldtk`は、特に指定がない限り、コンポーネントを生成するときにこのデフォルトの実装を使用します。

```rust
#[derive(Default, Component)]
struct Player;

#[derive(Default, Bundle, LdtkEntity)]
struct PlayerBundle {
    player: Player,
    #[sprite_sheet_bundle]
    sprite_sheet_bundle: SpriteSheetBundle,
    #[grid_coords]
    grid_coords: GridCoords,
}
```

