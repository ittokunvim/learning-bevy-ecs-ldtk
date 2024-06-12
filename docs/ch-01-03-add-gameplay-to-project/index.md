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

## `GridCoords`の値から位置を更新

今の時点ではプレイヤーは画面上では全く動きません。
`bevy_ecs_ldtk`は`GridCoords`エンティティの`Transform`を自動的に保持しません。

これはユーザーに任されており、`Transform`のアニメーションを自由に実装できます。

`bevy_ecs_ldtk`は、グリッドのセルのサイズがわかっていれば、結果の変換を計算するのに役立つユーティリティ関数を提供しています。

```rust
use bevy_ecs_ldtk::utils::grid_coords_to_translation;

fn main() {
    App::new()
        // ...
        .add_systems(Update, translate_grid_coords_entities)
        .add_systems(Update, bevy::window::close_on_esc)
        .run()
}

fn translate_grid_coords_entities(
    mut grid_coords_entities: Query<(&mut Transform, &GridCoords), Changed<GridCoords>>,
) {
    for (mut transform, grid_coords) in grid_coords_entities.iter_mut() {
        transform.translation =
            grid_coords_to_translation(*grid_coords, IVec2::splat(GRID_SIZE))
                .extend(transform.translation.z);
    }
}
```

## 衝突判定を追加する

先ほどプレイヤーを移動させることには成功しました。が今のままではプレイヤーが壁の中を移動できてしまいます。

タイルベースに衝突判定を実装するには、壁にコンポーネントを追加して位置を特定します。

壁のエンティティに新しいバンドルを作成しマーカーコンポーネントを追加します。

このバンドルに`LdtkIntCell`を追加し、`register_ldtk_int_cell`と壁の`intgrid`値を使ってアプリに登録します。
`IntGrid`エンティティは`GridCoords`を要求せずにスポーンします。

```rust
#[derive(Default, Component)]
struct Wall;

#[derive(Default, Bundle, LdtkIntCell)]
struct WallBundle {
    wall: Wall,
}

fn main() {
    App::new()
        // ...
        .register_ldtk_int_cell::<WallBundle>(1)
        // ...
        .run()
}
```

衝突処理を実装する方法はたくさんあります。
単純にプレイヤーが移動するたびに、全ての`Wall`エンティティをクエリして、`GridCoords`の値をチェックする方法もあります。

ここでは少し最適化された実装を行い、レベルがスポーンする際に壁の位置をリソースにキャッシュします。

値によって検索できる現在の壁の位置を格納するための`LevelWalls`リソースを作成します。
このリソースに壁の位置のための`Hash<Gridcoords>`フィールドを与えます。

レベルの幅と高さのフィールドも与えて、プレイヤーが範囲外に移動しないようにします。

次に、`in_wall`メソッドを実装します。
このメソッドは、指定された`grid_coords`がレベル境界の外側にあるか、`HashSet`に含まれている場合、`true`を返します。

```rust
use std::collections::HashSet;

fn main() {
    App::new()
        // ...
        .init_resource::<LevelWalls>()
        // ...
        .run()
}

#[derive(Default, Resource)]
struct LevelWalls {
    wall_locations: HashSet<GridCoords>,
    level_width: i32,
    level_height: i32,
}

impl LevelWalls {
    fn in_wall(&self, grid_coords: &GridCoords) -> bool {
        grid_coords.x < 0
            || grid_coords.y < 0
            || grid_coords.x >= self.level_width 
            || grid_coords.y >= self.level_height
            || self.wall_locations.contains(grid_coords)
    }
}
```

次に`LevelEvent::Spawned`をリッスンして、リソースに入力するシステムを追加します。

`HashSet`に入力するために、全ての壁の位置にアクセスする必要があり、現在のレベルの幅・高さを見つけるために`LdtkProject`データにアクセスする必要があります。

```rust
fn main() {
    App::new()
        // ...
        .add_systems(Update, cache_wall_locations)
        .add_systems(Update, bevy::window::close_on_esc)
        .run()
}

fn cache_wall_locations(
    mut level_walls: ResMut<LevelWalls>,
    mut level_events: EventReader<LevelEvent>,
    walls: Query<&GridCoords, With<Wall>>,
    ldtk_project_entities: Query<&Handle<LdtkProject>>,
    ldtk_project_assets: Res<Assets<LdtkProject>>,
) {
    for level_event in level_events.read() {
        if let LevelEvent::Spawned(level_iid) = level_event {
            let ldtk_project = ldtk_project_assets
                .get(ldtk_project_entities.single())
                .expect("LdtkProject should be loaded when level is spawned");
            let level = ldtk_project
                .get_raw_level_by_iid(level_iid.get())
                .expect("spawned level should exist in project");
            let wall_locations = walls.iter().copied().collect();
            let new_level_walls = LevelWalls {
                wall_locations,
                level_width: level.px_wid / GRID_SIZE,
                level_height: level.px_hei / GRID_SIZE,
            };

            *level_walls = new_level_walls;
        }
    }
}
```

最後に`move_player_from_input`関数に`LevelWalls`リソースを追加して、プレイヤーの目的地が壁の中にあるかどうかチェックします。

```rust
fn move_player_from_input(
    mut players: Query<&mut GridCoords, With<Player>>,
    input: Res<Input<KeyCode>>,
    level_walls: Res<LevelWalls>,
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
        if !level_walls.in_wall(&destination) {
            *player_grid_coords = destination;
        }
    }
}
```

上記のチェックを追加することで、プレイヤーは壁にぶつからなくなります。

