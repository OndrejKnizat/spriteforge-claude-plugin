# SpriteForge for Claude

Make pixel art with SpriteForge's AI artist without leaving Claude. With your
[SpriteForge](https://www.spriteforge.tech) account, Claude can generate sprites, sets, animations and
tilesets, manage your projects and download the results into your game. For hand-editing without an
account - SpriteForge's pixel editor and tileset editor in the conversation - install the free
[SpriteForge Editor](https://github.com/OndrejKnizat/spriteforge-editor) plugin too.

## Install

- **Claude Code:** `/plugin marketplace add OndrejKnizat/spriteforge-claude-plugin`, then
  `/plugin install spriteforge@spriteforge`. Run `/mcp` and sign in to SpriteForge the first time.
- **claude.ai:** add the connector `https://www.spriteforge.tech/mcp` (Settings, Connectors) and sign in.

## What it does

The plugin's connector, **SpriteForge**, connects to your SpriteForge account when you sign in to it
(Claude asks you to authorize it the first time). With it Claude can:

- **generate** with your SpriteForge credits - single sprites, staged character sprites, themed sets
  of up to 8, animations (including animating one of your sprites), AI edits of a sprite, terrain,
  Wang and platformer tilesets, and blended multi-terrain maps. Claude writes the prompt and picks the
  setup (type, size, project, reference sprites) and, unless you name one, the quality tier. Try
  `/spriteforge:generate a slime enemy with a bounce animation`;
- list your projects, read or change their style settings (style, description, light, outline,
  palette) and create new ones - generations follow the project's style;
- browse your sprites, **download** them as PNGs with real transparency (one frame, or a whole
  animation or tileset as a sprite sheet) - in Claude Code straight into your project's files - and
  **upload** images into a project;
- open a library sprite in the editor and save your edits back (the previous version stays in the
  sprite's history), or keep a sprite you edited without an account;
- show your credit balance and storage.

**Spending is confirmed by default.** Before generating, Claude tells you what it will make and the
exact price (fixed up front; anything that fails is refunded) and waits for your OK. If you'd rather
it just went ahead, say so - "don't ask about prices", or give it a budget - and it will generate
without stopping, never above the price it quoted or your budget. Claude Code and the Claude apps can
also ask you to approve each tool call; allow the `generate` tool permanently there to remove that
prompt too.

It can't delete your sprites or projects, buy credits, manage API keys or change your account.
New accounts get a one-time $0.25 credit to try AI
generation, added the first time you open the SpriteForge studio while signed in - Claude will point
you there.

## What it sends and stores

- The SpriteForge connector, `https://www.spriteforge.tech/mcp`, signs you in through SpriteForge's
  sign-in provider (Clerk) and reads or writes only your own projects and sprites, when Claude calls its
  tools on your request. Generation prompts go to the AI model provider that draws the sprite, as in the
  studio.
- Download links name one of your sprites, work without sign-in and expire after 24 hours; anyone
  holding one can download that sprite until then.
- To bring a file from your disk into your library, Claude Code uploads it with a plain `curl` request to
  `https://www.spriteforge.tech/api/edit-session`, which keeps it under an unguessable edit link for 24
  hours after its last edit (7 days at most); anyone holding the link can view and edit that sprite.
- Library sprites you open in the editor are shown in SpriteForge's editor card in the conversation;
  the card is self-contained and loads nothing from other sites. Nothing else on your machine is read
  or sent.

Limits: uploads up to 128x128 pixels and 256 colours; pixels under 50% opacity become transparent.

## Components

- `skills/generate-sprites` - the generation flow: planning, choosing a tier, confirming the price,
  and getting files in and out; `commands/generate.md` - `/spriteforge:generate <what>`.
- `.mcp.json` - the **SpriteForge** connector (`/mcp`, OAuth sign-in): `get_account`, `list_projects`,
  `get_project`, `create_project`, `update_project`, `list_sprites`, `get_sprite_from_library`,
  `upload_sprite`, `get_generation_options`, `quote_generation`, `generate`, `get_job`,
  `open_sprite_in_editor`, `open_full_editor`, `open_tileset_editor`, `save_to_library`, and SpriteForge's editor cards (MCP
  Apps) with the two tools only they call, `load_art` and `save_art`.

## Links

- Privacy policy: https://www.spriteforge.tech/privacy.html
- Terms: https://www.spriteforge.tech/terms.html
- Support: support@spriteforge.tech
