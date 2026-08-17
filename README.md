# Allium58 — ZMK config

ZMK firmware for a Lily58 (58 keys, split) on **nice!nano v2** controllers with
**nice!view** displays.

This keymap is deliberately kept at parity with the QMK
`dactyl_manuform/5x6` keymap in
[`blakedietz/qmk_userspace`](https://github.com/blakedietz/qmk_userspace) —
same layer names, same layer indices — so the two can be read side by side
without a translation table. See [Parity with the dactyl](#parity-with-the-dactyl)
for where and why they diverge.

## Repo layout

| Path | What |
|---|---|
| `config/lily58.keymap` | The keymap. Behaviors, combos, and all 7 layers. Heavily commented — the `DECISION:` blocks record why things are the way they are. |
| `config/lily58.conf` | Kconfig: display, pointing, soft-off. |
| `config/west.yml` | ZMK manifest (tracks `zmkfirmware/zmk` `main`). |
| `build.yaml` | GitHub Actions build matrix — left, right, settings_reset. |
| `firmware/` | Last built `.uf2` files, committed so a known-good build is always recoverable. |
| `boards/`, `zephyr/` | Stock user-config scaffolding. |

## Layers

Layer indices are **the order of the nodes in the keymap block**, not the value
of the `#define`s. The two must be kept in agreement by hand.

| # | Name | Reached by |
|---|---|---|
| 0 | Base | default |
| 1 | Game | `&to GAME` on Dev's top row, or hold `[` + `]` |
| 2 | Dev | hold Space |
| 3 | Media | hold `;` |
| 4 | Mouse | hold Enter |
| 5 | Sym | hold `V`, `M`, `)`, or `[` |
| 6 | Num | hold `B`, `N`, or Backspace |

Layers 0–4 mirror QMK's indices exactly. Sym and Num have no QMK counterpart,
so they are appended at 5–6 rather than interleaved — that keeps the shared
indices stable if either side grows a layer later.

`display-name` is what the nice!view and ZMK Studio show, so those strings are
user-facing, not just documentation.

### 0 — Base

```
=      1      2      3      4      5                    6      7      8      9      0      -
`      Q      W      E      R      T                    Y      U      I      O      P      \
Esc*   A      S      D      F      G                    H      J      K      L      ;*     '*
Shift  Z*     X*     C*     V*     B*    Meh    Meh     N*     M*     ,*     .*     /*     Shift
                     (      )*     Bksp* Space* Enter*  Tab    [*     ]
```

`*` = tap/hold key. Holds: Esc→Hyper, `'`→Hyper, `Z X C`→Ctrl Alt Gui,
`,`→Gui, `.`→Alt, `/`→Ctrl, `V`/`M`→Sym, `B`/`N`→Num, `;`→Media,
`)`→Sym, Bksp→Num, Space→Dev, Enter→Mouse, `[`→Sym.

Thumb row, against the dactyl's right thumb cluster:

| QMK | ZMK |
|---|---|
| `LT(_MOUSE, KC_ENT)` | `&lt MOUSE RET` |
| `KC_TAB` | `&kp TAB` |
| `KC_LBRC` | `&lt SYM LBKT` |
| `KC_RBRC` | `&kp RBKT` |

Enter carries the mouse layer, matching QMK. Tab was *not* moved to get
there — Enter was already the layer-tap in that position and was simply
repointed from the old Raise layer.

Enter stays on plain `&lt`, not `&ltq`. Backspace wants tap-then-hold repeat;
a repeating Enter key is mostly a way to fire off blank lines by accident.

### 1 — Game

**The left-hand alphas are intentionally rearranged. Do not "correct" them
back to QWERTY order.**

```
=      1      2      3      4      5                    6      7      8      9      0      -
`      Q      E      W      R      T                    Y      U      I      O      P      \
Esc    C      A      S      D      G                    H      J      K      L      ;*     '
Shift  Z      X      F      V      B     Meh    Meh     N      M      ,*     .*     /*     Shift
                     Alt    Ctrl   Bksp  Space  Enter   Tab    [      →Base
```

The WASD cross is shifted one column right of its QWERTY home:

|  | pinky | ring | middle | index |
|---|---|---|---|---|
| Base row 2 | Q | W | E | R |
| Base row 3 | A | S | D | F |
| **Game row 2** | Q | **E** | **W** | R |
| **Game row 3** | **C** | **A** | **S** | **D** |

On Base, WASD lands on pinky/ring/middle and leaves the index finger idle. On
Game it lands on ring/middle/index — the three strong fingers — and frees the
pinky for Shift. `C` takes the vacated pinky home slot; `F` drops to row 3
col 3. The payoff is that games keep their stock WASD bindings: the ergonomics
come from the firmware, so no game needs individual remapping.

Everything else follows the dactyl — home row mods and thumb layer taps drop
to plain keys so held keys register with no tapping-term latency, and the right
hand keeps its mods (as QMK does) since only the left hand is on WASD.

**Getting in and out:** hold `[` + `]` together (see [Combos](#combos)), or
`&to GAME` on Dev's top row. Out is the same chord, or the single-tap `→Base`
on the outer right thumb.

### 2 — Dev

Bluetooth select/clear on the number row, F1–F12, symbols on the left home
row, arrows on the right home row (`←↓↑→` on `HJKL`).

Board-management keys live here because they are ZMK-only concerns the dactyl
does not have: `&studio_unlock`, `&soft_off`, `&ext_power EP_ON`, `&caps_word`,
`&key_repeat`, `&to GAME`.

Only `&ext_power EP_ON` is present — `EP_OFF` and `EP_TOG` are deliberately
gone from this layer and from Media. The switched rail (nice!nano v2 P0.13)
feeds exactly one thing here, the nice!view, and there is no underglow to save
power on. So `EP_OFF` could only ever blank the display — and ZMK persists
ext-power state to settings, so a stray press survived every reboot and the
screen stayed dead until a `settings_reset` flash. `EP_ON` is kept as the
recovery key for exactly that case.

`&soft_off` moved off position 9 (a digit you hold-space past all the time) to
the far-right pinky, and requires a 5s hold.

### 3 — Media

Transport and volume on the right home and bottom rows, as on the dactyl.
Reached by holding `;`, which is where QMK puts `LT(_MEDIA, KC_SCLN)`.

### 4 — Mouse

Key-for-key the same positions the dactyl uses — move on `ESDF`, scroll on
`UI`, click on `JK` — so muscle memory carries between the two boards.

### 5 — Sym / 6 — Num

ZMK-only. The dactyl has the physical keys to keep symbols and numbers on its
base layer; at 58 keys this board does not, so they get layers.

## Behaviors

All four hold-taps exist to make tap-hold keys survive fast typing. QMK gets
this from `CHORDAL_HOLD` + `FLOW_TAP_TERM`; ZMK has no direct equivalent, so
it is spelled out per behavior.

| Label | Purpose | Key properties |
|---|---|---|
| `hml` / `hmr` | Home row mods, left / right | `balanced`, 200ms term, `require-prior-idle-ms 125`, `quick-tap-ms 200`, plus `hold-trigger-key-positions` enumerating the opposite hand |
| `rpl` | Layer taps sitting on letter keys | same idle guard as above, but holds a **layer** instead of a mod |
| `ltq` | Layer tap with quick-tap (backspace) | `tap-preferred`, 200ms, `quick-tap-ms 200`, **no** idle guard |
| `htl` | Hold-to-switch-layer, for the layer combos | `tap-preferred`, 400ms, `&to` on hold / `&none` on tap |

Notes worth keeping in mind before editing these:

- **`hold-trigger-key-positions` is ZMK's `CHORDAL_HOLD`.** A mod only engages
  if the next key pressed is on the opposite hand, so same-hand rolls like `zx`
  emit letters instead of Ctrl-X. QMK derives handedness from the matrix half;
  ZMK has no such helper, so the opposite hand is enumerated by key position —
  which means these lists must be updated by hand if the layout ever moves.
- **`ltq` has no idle guard, deliberately.** Idle guarding would force a tap
  whenever you reach for the key right after typing — which for backspace is
  precisely the common case, and would make the layer hold unreachable.
- **The global `&lt` override sets tapping-term only.** No `quick-tap-ms`:
  Space is `&lt DEV SPACE`, and a global quick-tap would force a literal space
  (swallowing the Dev layer) any time the key is held within `quick-tap-ms` of
  the previous space. Keys that genuinely want quick-tap opt in via `&ltq`.
- **`&soft_off` must be held, not tapped.** It sits on Dev, and Dev is reached
  by holding space — so any stray hold-space + keypress was one tap away from
  powering the board down.

## Combos

`[` + `]` (positions 56/57, the two outer right thumbs) held together toggles
between Base and Game — the same gesture in both directions.

```dts
combo_game { bindings = <&htl GAME 0>; key-positions = <56 57>; layers = <BASE>;
             timeout-ms = <50>; require-prior-idle-ms = <125>; };
combo_base { bindings = <&htl BASE 0>; key-positions = <56 57>; layers = <GAME>;
             timeout-ms = <50>; };
```

Three decisions are load-bearing here:

1. **Wrapped in `&htl`, not firing `&to` directly.** A combo fires the instant
   its keys go down, and `&to` is a sticky, whole-left-hand change. Firing one
   on a fast `[]` roll — or dropping out of Game mid-fight — would be
   miserable. The 400ms hold means a quick both-press is a no-op instead.
2. **`tap-preferred`, not `hold-preferred`.** `hold-preferred` resolves to the
   hold as soon as *any* other key is pressed, which would re-open exactly the
   accident this is meant to prevent. Only the timer may decide.
3. **`require-prior-idle-ms` on the Base→Game direction only.** On Base those
   keys are `[` and `]`, so the idle guard is what keeps a typed `[]` roll out
   of the combo. On Game the same guard would be actively wrong — you are
   pressing keys constantly, so it would make the way out unreachable exactly
   when it is wanted.

`&to BASE` also stays on Game position 57 as a single-tap escape hatch, so the
combo is not the only way out of a layer that has no other layer access at all.

## Key positions

Combos and `hold-trigger-key-positions` are written in these numbers.

```
  0  1  2  3  4  5              6  7  8  9 10 11
 12 13 14 15 16 17             18 19 20 21 22 23
 24 25 26 27 28 29             30 31 32 33 34 35
 36 37 38 39 40 41 42       43 44 45 46 47 48 49
          50 51 52 53       54 55 56 57
```

Left hand `0-5, 12-17, 24-29, 36-42` · right hand `6-11, 18-23, 30-35, 43-49`
· thumbs `50-57`. Positions 42/43 are the two inner bottom-row keys (Meh).

## Building

Every push builds via GitHub Actions (`.github/workflows/build.yml`, which
calls `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`). The
matrix in `build.yaml` produces three artifacts:

- `lily58_left nice_view_adapter nice_view` — built with
  `-DCONFIG_ZMK_STUDIO=y` and the `studio-rpc-usb-uart` snippet
- `lily58_right nice_view_adapter nice_view`
- `settings_reset`

Download from the run's artifacts, or:

```sh
gh run list --limit 1
gh run download <run-id> --dir /tmp/fw
cp "/tmp/fw/firmware/"*.uf2 firmware/
```

## Flashing

Double-tap the reset button on a nice!nano; it mounts as a `NICENANO` USB mass
storage volume. Drag the `.uf2` over and it reboots itself.

**For a keymap-only change you only need the left half.** The keymap lives on
the central side, which is the left on this config; the right half is a
peripheral and its rebuild is functionally identical.

### If a new keymap appears to do nothing

The left half builds with `CONFIG_ZMK_STUDIO=y`. Any edit ever made through
ZMK Studio is saved to settings and **takes precedence over the compiled
keymap**. Flash `settings_reset-nice_nano__zmk-zmk.uf2`, then flash the left
build again.

The same applies to a nice!view that will not light up — see the `EP_ON` note
under [Dev](#2--dev).

## Parity with the dactyl

The dactyl's `LAYOUT_5x6` is **64 keys** (32 a side: 4 rows of 6, plus 2 inner,
plus 6 thumb). This board is **58** (29 a side: 3 rows of 6, a row of 7, plus 4
thumb).

The 6-key deficit understates the problem. The main grid actually *gains* a key
per side here; the entire loss lands on the thumbs, **8 a side down to 4**. That
is the source of nearly every difference: the layer taps the dactyl parks on
dedicated thumb keys have to ride on letters and existing thumbs instead, which
is why `V`/`M`/`B`/`N` carry Sym and Num holds here and nothing like that
appears on the dactyl.

| QMK feature | ZMK equivalent |
|---|---|
| `CHORDAL_HOLD` | `hold-trigger-key-positions` on `hml`/`hmr` |
| `FLOW_TAP_TERM 150` | `require-prior-idle-ms = <125>` |
| `QUICK_TAP_TERM` (on by default) | `quick-tap-ms` — **off** by default in ZMK, hence `&ltq` |
| `MOUSEKEY_*` speed tuning | not a keymap concern; set on the input listener / `CONFIG_ZMK_POINTING` |

Gaps with no equivalent in either direction:

- **`QK_LOCK`** (QMK, on both Game thumb clusters — hold a key without holding
  it, e.g. lock W for auto-run). ZMK has no built-in equivalent; `&sk` covers
  modifiers only, not arbitrary keys. Those two positions carry BSPC/SPACE and
  RET/TAB instead.
- **`MS_ACL0/1/2`** (QMK mouse acceleration). ZMK sets pointer speed on the
  input listener rather than with runtime keycodes, so those three positions
  are left `&trans` rather than faked.
- **Symbols and numbers** live on the dactyl's base layer; here they need Sym
  and Num layers of their own.
- **Thumb keys**: the dactyl's 6+2 per side become 4.
- **ZMK-only**: Bluetooth profiles, `ext_power`, soft-off, ZMK Studio,
  `caps_word`, `key_repeat`, the nice!view display.
- **QMK-only**: Vial support (live keymap editing without reflashing).
- **Hyper/Meh**: the dactyl carries `MEH_HOLD`/`HPR_HOLD` pairs on its thumb
  clusters. This board has Meh on the two inner keys (42/43) and Hyper on the
  outer home-row pinkies (Esc and `'`).

## Formatting

The `bindings` grids are aligned by
[qmk.nvim](https://github.com/codethread/qmk.nvim) on save (`variant = "zmk"`);
the 14-column lily58 layout lives in `lua/plugins/qmk.nvim.lua` in the nvim
config.

It rewrites each `bindings = < ... >` block from the parsed behaviors, so **a
commented-out binding inside one of those blocks is dropped on save** — comment
things out between the layer nodes instead, as the existing blocks do.

Outside Neovim:

```sh
nvim --headless config/lily58.keymap -c 'write' -c 'qa'
```
