# Accurate Block Placement (Horizon native mod)

Hold right click with a block and every new spot you look at gets a block at once, however fast you move.

Vanilla places a block every four ticks while the use key is held: move fast and spots are skipped, move slowly and
extra blocks stack up. This mod places on each **fresh** target instead: a block you weren't already looking at and
didn't just place.

- **A new spot, a block**, for bridging, walls, floors and sweeps along a row.
- **Staring at one spot places nothing more.**
- **A single click stays a single block**: after the first click it waits out vanilla's four ticks, unless you turn
  the view (10 degrees), then places what you looked at meanwhile.
- **Building backwards**: the block you just placed counts as fresh again once you have stepped a full block back.
- Blocks that react to a click (doors, chests, buttons, levers...) get vanilla's click; scaffolding on scaffolding
  climbs as in vanilla.

Options > Mods > Accurate Block Placement: **Accurate placement** on or off, **Allowed on servers** (in your own worlds
it always works), and a toggle key (unbound by default). Some servers see fast, accurate placing as a cheat: check
their rules.

## Credits

A port of [Accurate Block Placement: Reborn](https://github.com/KadTheHunter/AccurateBlockPlacement-Reborn)
([Modrinth](https://modrinth.com/mod/accurate-block-placement-reborn)), MIT, by Clayborn, Flourick, schwar and
KadTheHunter. Its placement rules are rewritten in Rust for Horizon; see [LICENSE](LICENSE).

## Install

In the Horizon launcher: Mods → Accurate Block Placement → Install, then build the game with your native mods.
Native mods run with the game's full access to your computer: install them only from authors you trust.

## How it works

A [horizon-mixins](https://github.com/horizon-rust/horizon-mixins) bundle (`mixins/accurate_block_placement/`):

| Hook | Where | What it does |
|---|---|---|
| `native_frame` | Horizon's hook points | Every frame, while the use key is held with a block, what the crosshair hits; fresh targets are kept |
| `remote_handle_input` (HEAD) | `game/remote.rs` | Every tick, before the tick's clicks: the kept targets get their blocks through the game's own `startUseItem` (at most 4 a tick), and vanilla's repeat stays off |
| `native_key`, `native_mods`, `native_settings`, `native_settings_click` | Horizon's hook points | The toggle key, the Options > Mods row and its settings |

Sampling every frame and placing every tick means a fast sweep loses no spot, and the server sees placements tick by
tick. `remote_handle_input` is not one of Horizon's stable hook points, so the bundle is **pinned** to the Horizon
commit in `compatibility.json`.

| File | What it is |
|---|---|
| `mixins/accurate_block_placement/horizon-mixin.toml` | The bundle: one module and the hooks |
| `mixins/accurate_block_placement/runtime.rs` | The rules and state (`game::accurate_block_placement` in the game); also this crate's library, for its tests |
| `mixins/accurate_block_placement/*.rs` | One small helper per hook |
| `tools/package.py` | Packages `target/accurate_block_placement-<version>.hnmod` for a release |

## Build and check

```sh
cargo test                                   # the rules: fresh targets, the wait, sweeps, building backwards
horizon-mixins apply --workspace ../horizon --mods mixins --out ../stage   # Horizon at the pinned commit
(cd ../stage && cargo check --locked -p horizon-client --bin horizon-client)
python3 tools/package.py                     # target/accurate_block_placement-1.0.0.hnmod
```

License: MIT (see [LICENSE](LICENSE) for the original's notices).
