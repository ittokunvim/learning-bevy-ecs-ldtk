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

## タイルベースの移動を実装

プレイヤーが`WASD`で移動できるように実装します。
`GridCoords`の値を操作することでプレイヤーの位置を変えることができます。

そして、移動した情報をプレイヤーエンティティの`GridCoords`コンポーネントに追加します。

```rust
fn main() {
    App::new()
        // ...
        .add_systems(Update, move_player_from_input)
        .add_systems(Update, bevy::window::close_on_esc)
        .run()
}

fn move_player_from_input(
    mut players: Query<&mut GridCoords, With<Player>>,
    input: Res<Input<KeyCode>>,
) {
    let movement_direction = if input.any_just_pressed([KeyCode::W, KeyCode::Up]) {
        GridCoords::new(0, 1)
    } else if input.any_just_pressed([KeyCode::A, KeyCode::Left]) {
        GridCoords::new(-1, 0)
    } else if input.any_just_pressed([KeyCode::S, KeyCode::Down]) {
        GridCoords::new(0, -1)
    } else if input.any_just_pressed([KeyCode::D, KeyCode::Right]) {
        GridCoords::new(1, 0)
    } else {
        return;
    };

    for mut player_grid_coords in players.iter_mut() {
        let destination = *player_grid_coords + movement_direction;
        *player_grid_coords = destination;
    }
}
```

