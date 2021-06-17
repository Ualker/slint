# Game Logic In Rust

We'll implement the rules of the game in Rust as well. The general philosophy of SixtyFPS is that merely the user
interface is implemented in the `.60` language and the business logic in your favorite programming
language. The game rules shall enforce that at most two tiles have their curtain open. If the tiles match, then we
consider them solved and they remain open. Otherwise we wait for a little while, so the player can memorize
the location of the icons, and then close them again.

We'll modify the `.60` markup inside the `sixtyfps!` macro to signal to the Rust code when the user clicks on a tile.
Two changes to <span class="hljs-title">MainWindow</span> are needed: We need to add a way for the MainWindow to call to the Rust code that it should
check if a pair of tiles has been solved. And we need to add a property that Rust code can toggle to disable further
tile interaction, to prevent the player from opening more tiles than allowed. No cheating allowed! First, we paste
the callback and property declarations into <span class="hljs-title">MainWindow</span>:


```60
...
MainWindow := Window {
    callback check_if_pair_solved(); // Added
    property <bool> disable_tiles; // Added

    width: 326px;
    height: 326px;

    property <[TileData]> memory_tiles: [
       { image: img!"icons/at.png" },
...
```

The last change to the `.60` markup is to act when the <code class="hljs-title"> signals that it was clicked on. We add the following handler:

```60
...
MainWindow := Window {
    ...
    for tile[i] in memory_tiles : MemoryTile {
        x: mod(i, 4) * 74px;
        y: floor(i / 4) * 74px;
        width: 64px;
        height: 64px;
        icon: tile.image;
        open_curtain: tile.image_visible || tile.solved;
        // propagate the solved status from the model to the tile
        solved: tile.solved;

        clicked => {
            // old: tile.image_visible = !tile.image_visible;
            // new:
            if (!root.disable_tiles) {
                tile.image_visible = !tile.image_visible;
                root.check_if_pair_solved();
            }
        }
    }
}
```

On the Rust side, we can now add an handler to the `check_if_pair_solved` callback, that will check if
two tiles are opened. If they match, the `solved` property is set to true in the model. If they don't
match, start a timer that will close them after one second. While the timer is running, we disable every tile so
one cannot click anything during this time.

Insert this code before the `main_window.run()` call:

```rust,noplayground
{{#include main_game_logic_in_rust.rs:game_logic}}
```

Notice that we take a [Weak](https://sixtyfps.io/docs/rust/sixtyfps/struct.weak) pointer of our `main_window`. This is very
important because capturing a copy of the `main_window` itself within the callback handler would result in a circular ownership.
The `MainWindow` owns the callback handler, which itself owns a reference to the `MainWindow`, which must be weak
instead of strong to avoid a memory leak.

These were the last changes and running the result gives us a window on the screen that allows us
to play the game by the rules.