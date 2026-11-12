---
name: game-kit
description: Make the whole matching pixel-art kit for a game in one go with SpriteForge - hero, enemies, items, tiles and animations in one style - then save the PNGs and sprite sheets into the game's folder with a manifest. Use when the user asks for the art, assets, sprites or graphics for "this game" or "my game", an asset pack or art kit, to replace placeholder rectangles or emoji with real art, or when you are building a game with them and it has no art yet. Needs the signed-in SpriteForge connector. For one sprite or one tileset, use generate-sprites instead.
---

# A game's art kit with SpriteForge

One order, one price, one style: plan every asset the game needs, get a single OK on the total, then
generate the kit in an order that keeps it matching, and wire the files into the game. The mechanics of
each tool call (types, tiers, sizes, downloads) are in the `generate-sprites` skill; this skill is the
plan around them.

## 1. Read the game, then write the asset list

Look before asking. In a project folder, read the code and any design notes for:

- **The view** - top-down, 3/4 top-down, side view or isometric. It goes into every `perspective`.
- **The tile and sprite size** the code already assumes (a 16 px grid, 32 px characters). No size in the
  code yet: propose **16x16** for everything (cheapest, and where the artist is strongest), or 32x32
  characters on 16 px tiles for a richer look. One size per asset class, kept for the whole kit. A
  pickup the code draws smaller than its cell (an 8x8 apple in a 16 px tile) is still a 16x16 sprite;
  ask for the subject to fill most of the canvas and let the game scale or place it.
- **The background colour** the sprites will sit on. On a dark scene, say so in the style line ("bold
  light-edged silhouettes that read on a dark navy sky") so dark outlines don't vanish.
- **What exists as a placeholder** - coloured rectangles, emoji, missing-texture squares, entity names in
  the code (`Player`, `Slime`, `Coin`), states (`idle`, `run`, `jump`). Each one is a line in the list.
- **The engine** (Godot, Unity, Phaser, Pygame, LÖVE, plain canvas) and where it keeps images.

Then write the list, grouped the way it will be generated:

| Group | Typical content | How it's made |
|---|---|---|
| Hero | the player character, standing (this is also the idle pose) | `single` (or `phased` at 32 px and up) |
| Hero motion | the main way it moves: walk or run, 4 frames | `animation` of the hero's `asset_id` |
| Enemies | 2-6 creatures | `set`, hero as style reference |
| Items, props, HUD icons | pickups, chests, doors, hearts | `set` (up to 8 per call), hero as style reference |
| Ground, top-down | floor and walls | `terrain` (wall, transition, floor) or `wang` (one autotiling ground) |
| Ground, side view | grass over earth, with edges and slopes | `platformer` (49 tiles; a flat level still uses its top, fill and edge tiles) |

**Keep the first kit small** enough to see in the game today: the hero, **one** motion (if the code has
several states, animate the main locomotion one and list the others as left out), a few enemies, the
pickups the code already uses and one ground. More comes later, in the same project, and it will match.

Frame counts when the user gives none: walk or run 4, attack 3-4, hurt or jump 1-2 (a one-way motion
such as a jump or a death is `open_ended: true`).

Say plainly what the kit leaves out and why. Not made by SpriteForge at all: full-screen backdrops
(the canvas tops out at 64 px; keep a flat colour or a tiled ground), and pixel fonts or HUD digits
(use the engine's font).

No game yet, only an idea? Ask for the genre and view in one question, propose a starter list, go on.

## 2. One style, set once

Write the look as project settings instead of repeating it in every prompt:

- `description`: what the game is, in a sentence or two.
- `style`: one line, e.g. "chunky 16-bit fantasy, readable silhouettes, muted greens and greys with warm
  gold accents".
- `light` (e.g. "Top-left") and `outline` (e.g. "Dark-tinted").

Reuse the SpriteForge project that already carries the game's name (`list_projects`); otherwise create
one with `create_project` right away - it is free, spends nothing and only adds an empty project - and
show the user its style line in the quote message, where they can change it before anything is drawn.
**Pass that `project_id` on every `quote_generation` and `generate` call.** Without it a job lands in
whichever project was used last and takes that project's style.

## 3. One quote for the whole kit

`quote_generation` every job in the list (quotes are free), add them up, and give the user **one
message**: the style line, the list by group with each group's price, the size and tier, the total, how
long it will take, what was left out, and the balance from `get_account`. Then wait for one OK. Do not
ask per job.

- Default tier: **Sonnet** up to 32 px. **Opus** for a 48-64 px hero or a detailed boss. **Haiku** when
  the user asks for the cheapest kit and doesn't mind waiting.
- **The hero doesn't exist yet, so quote around it.** Quote the sets without `references` and add 1 cent
  per sprite for the style reference they will carry. Quote the hero's animation as if from scratch
  (`subject`, `width`, `height`, `frame_count`, no `asset_id`): the price is the same. An animation costs
  one call per frame plus a planning pass, so 4 frames are 5 calls.
- Add 10% to the total you ask for, named as "room for a redraw or two", and stay inside it.
- Time: jobs run up to three at once, so give the sum of the quotes' `eta_seconds` divided by two, plus
  a minute for looking at the hero - "about 3 minutes", not a per-job list.
- Balance short? Say what the balance covers (offer the 16 px version of the kit, or the first groups),
  and give the studio link from `get_account` for a top-up. New accounts get a free starter credit the
  first time they open the studio while signed in.
- The user gave a budget instead ("spend up to $5")? Fit the list to it, say what was left out, and go.

## 4. Generate in this order

1. **Hero first**, alone. Look at the preview `get_job` returns. If the hero is wrong, fix it now
   (`edit`, or one redraw) - every later sprite copies its look.
2. **Ground** can run at the same time as the hero; it takes its look from the project style.
3. **Enemies, then items and props**, each as a `set` with
   `references: [{ asset_id: <hero>, role: "style" }]` and a `set_theme`. Up to three jobs run at once
   per account; start them together and wait on all of them.
4. **Animations last**, on the approved hero: `type: "animation"` with the hero's `asset_id`, a short
   `subject` naming the character again (it is required even with `asset_id`), the `motion`,
   `frame_count`, and an `animation_name` ("walk", "run"). The frames are saved onto the hero sprite.

Give every `generate` call the `project_id`, its own `request_id` (e.g. `<game>-enemies-1`) and
`max_cents` = its quote (plus the reference cents).

**Check every preview.** A sprite that is blank, cut off or not the subject is a miss: redraw that one
as a `single` with a more concrete subject ("round glass flask full of red liquid, cork stopper, filling
most of the canvas") and a kit sprite as the style reference. Don't hand over a kit with a hole in it,
and don't redraw sprites that are merely not perfect - list those for the user instead.

## 5. Put it in the game

Download with `curl` as the `generate-sprites` skill shows, into the engine's usual place, lower-case
snake_case names from the code's own entity names:

```
assets/sprites/hero.png            one frame, 1x, transparent
assets/sprites/hero_walk.png       sheet_url of the animated hero (frames left to right)
assets/sprites/enemies/slime.png
assets/sprites/items/coin.png
assets/tiles/crypt.png             sheet_url of the tileset
assets/spriteforge-kit.json        the manifest
```

Write `spriteforge-kit.json` next to them: the project id and style line, and per file its
`asset_id`, size, frame count, frame size, the animation's name, and for tilesets the layout:

- `terrain` sheets: three tiles - wall, transition, floor.
- `wang` sheets: 4 columns; frame index = corner mask (NW=1, NE=2, SW=4, SE=8, so 15 is the solid tile);
  frames 16-19 are variants of the solid tile.
- `platformer` sheets: 49 tiles - the 47 blob-autotile tiles in ascending key order, then an up-slope and
  a down-slope. A tile's key adds N=128, E=64, S=32, W=16, NE=8, SE=4, SW=2, NW=1 for each solid
  neighbour, a corner counting only when both edges beside it are solid. For a level stored as a grid of
  solid cells, compute each cell's key and draw the tile at that key's rank; write the 47 keys, in order,
  into the manifest so the game code can look the frame up.

Future sessions read the manifest to extend the kit in the same style.

Then make the game show it:

- Replace the placeholders in the code with the files, at an **integer scale** with nearest-neighbour
  filtering - Godot: `texture_filter = Nearest` (project default for 2D); Unity: Filter Mode Point, no
  compression, Pixels Per Unit = the tile size; Phaser: `pixelArt: true`; canvas:
  `imageSmoothingEnabled = false`; CSS: `image-rendering: pixelated`.
- Slice sheets by the frame size in the manifest; play the walk at 6-10 fps; flip the sprite for the
  other direction instead of ordering a mirrored one.
- Run the game if you can, and look at it.

## 6. Hand over

One short message: what was made (count by group), where the files are, what it cost against the
quote, and anything you'd redraw with their OK. Offer the next step that fits the game - the states you
left out, an attack animation, the next enemy tier, a second biome - with its price. For a sprite the
user wants to nudge by hand, `open_sprite_in_editor` opens it in SpriteForge's editor.
