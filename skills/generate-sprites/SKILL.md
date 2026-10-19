---
name: generate-sprites
description: Generate pixel art and game sprites with AI on the user's SpriteForge account - single sprites, themed sprite sets, animations, AI edits of an existing sprite, terrain / Wang / platformer tilesets and blended multi-terrain maps - and put them into their SpriteForge project or their game's files. Use when the user asks to generate, create or make sprites, pixel art, icons, tiles, tilesets, maps or animations, or to AI-edit or animate a sprite they have. Needs the signed-in SpriteForge connector.
---

# Generate pixel art with SpriteForge

The signed-in **SpriteForge** connector runs SpriteForge's AI pixel artist on the user's account. Every
generation has a fixed price, shown before it starts, paid from the user's SpriteForge credits;
anything that fails is refunded automatically. Results are saved into the user's SpriteForge library
(and appear in the studio), and you can download them as PNGs.

## The flow - ask before spending

1. **Plan the job yourself**: pick the type, canvas size, project, tier and any reference sprites (see
   below), and write a concrete prompt - subject, pose, materials, colours, mood. Don't ask the user to
   fill in settings they didn't mention.
2. **Quote it**: `quote_generation` with the same arguments returns the exact price, the balance and the
   usual time. Tell the user in one line what you'll make and what it costs, then **wait for their OK**.
3. **Generate**: `generate` with `max_cents` set to the quoted price and a fresh `request_id` (so a
   retried call can never start or charge a second job).
4. **Wait for it**: `get_job` with the `job_id`. Each call waits up to 25 s; keep calling while it says
   it's still drawing. When it's done it shows a preview and returns the new `asset_id`s and download
   links - the result is already in the user's library.

**When not to ask:** if the user has said to go ahead without asking ("just make them", "don't ask
me about prices", "spend up to $2"), skip the wait in step 2 - still quote first and keep `max_cents`
at the quoted price, or at their budget. Stay inside any budget they gave across the whole task, and
say what you spent at the end. (Whether the app itself asks before each tool call is the user's
permission setting, separate from this.)

If the balance is short, or `get_account` reports `signup_credit_pending`, give the user the studio link
from the tool's answer: new accounts get a one-time $0.25 starter credit by opening it once while signed
in, and credits are bought there.

## Choosing the setup

**Type** - what the user asked for:

| Ask | `type` | Needs |
|---|---|---|
| a sprite, item, icon, character | `single` | `subject`, `width`, `height` |
| a character or creature where anatomy matters | `phased` (2-3 staged passes) | `subject`, size |
| several matching sprites ("potion, scroll, key") | `set` (1-8, one shared palette) | `subjects`, size, optional `set_theme` |
| an animation (walk, idle, attack) | `animation` | `subject`, `frame_count` 1-8, `motion`; `asset_id` to animate an existing sprite |
| change an existing sprite ("give him a hat") | `edit` | `asset_id`, `prompt`; `mask_cells` to limit it to some pixels |
| top-down walls and floors | `terrain` (wall, transition, floor) | `subject` (wall), `subject_b` (floor), square size |
| an autotiling ground (grass, sand) | `wang` (16 tiles) | `subject`, square size |
| side-scroller ground tiles with slopes | `platformer` (49 tiles) | `subject` (earth), `subject_b` (top), square size |
| a small map of terrains blending into each other | `multiterrain` | `terrains` (2-6), `cols`/`rows`, square tile size |

**Tier** - if the user didn't pick one, choose it yourself; `get_generation_options` lists every tier's
price per call and usual time by size. Sensible defaults: **Sonnet** for most sprites up to 32px;
**Opus** for 48-64px canvases, detailed characters and final assets (it hits exact grids at every size);
**Haiku** for small, simple icons when cost matters (cheap but slow). The Low/High variants are the same
model thinking less or more. Multi-call types multiply the price (a 6-sprite set is 6 calls; an
animation is one call per frame).

**Size** - 16-32px for items and icons, 32-48px for characters, 16 or 32px for tiles; 64px is the
maximum. Width and height can differ for sprites; tiles are square.

**Project and style** - generations follow the project's style line, description, light, outline and
palette (`get_project` shows them). If the user describes a look for the whole game, set it once with
`update_project` (confirm the wording) instead of repeating it in every prompt. Use `project_id` to
target a project; by default the most recently used one is used.

**References** - pass up to 6 library sprites as `references` to hold a look: e.g. the hero sprite as a
`style` reference when making their items, so everything matches. `subject` references copy what's
depicted instead. Tile and map types use references for style only.

## Getting files in and out

- **Downloading (Claude Code):** save a result straight into the user's project:

  ```bash
  curl -sSf -o "assets/sprites/slime.png" '<png_url>'
  ```

  `png_url` is one frame at 1x with real transparency (`&frame=N` for frame N, `&scale=N` to enlarge);
  `sheet_url` is every frame of an animation or tileset in one PNG (`&cols=N` to change the layout).
  Links work without sign-in for 24 hours; `get_sprite_from_library` mints fresh ones for any sprite.
  Without a shell, give the user the link.
- **Uploading a file from disk (Claude Code):** don't paste base64. Let curl send the bytes to an edit
  link (PNG up to 128x128 and 256 colours; it's kept 24 hours after its last edit):

  ```bash
  curl -sS --data-binary @"assets/hero.png" -H 'Content-Type: image/png' \
    'https://www.spriteforge.tech/api/edit-session?name=hero'
  ```

  then `save_to_library` with the returned `id` (plus `project_id` and `name`). In chat apps,
  `upload_sprite` takes `png_base64` or `art` you drafted.
- To use one of the user's own images as a reference or to animate or AI-edit it, upload it first, then
  pass its `asset_id`.

## Good to know

- An animation of a library sprite is saved onto that sprite as a named animation (`animation_name`,
  default "idle"); one drawn from scratch becomes a new sprite.
- An `edit` updates the sprite in place and keeps its previous version in the sprite's history, unless
  `save_as_new` is set (animated sprites always get a new sprite for the edited still).
- If part of a multi-call job fails, what was drawn is delivered and the rest refunded; `get_job` says so.
- A multi-terrain map is one image of `cols x rows` tiles and must stay under about 160,000 pixels
  (16x12 tiles at 24px fits).
- Hand-tweaks after generating: `open_sprite_in_editor` with the `asset_id` opens the sprite in
  SpriteForge's editor - as a card in the conversation where the app shows them (`open_full_editor`
  with its `session_id` brings up the whole Edit studio there), otherwise as an edit link for the
  browser - and `save_to_library` with that `session_id` keeps the edit, the old version going into the
  sprite's history. In Claude Code, the SpriteForge Editor plugin adds pixel and tileset editor panes
  beside the conversation.
