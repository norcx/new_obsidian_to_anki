# better-obsidian-to-anki

[中文文档](README.zh-CN.md)

better-obsidian-to-anki is forked from [Obsidian_to_Anki](https://github.com/Pseudonium/Obsidian_to_Anki). The original project has not been actively maintained for a long time, so this fork keeps the existing Obsidian-to-Anki workflow and adds the features I wanted for my own study setup.

The main addition is source navigation: cards created in Anki can link back to the exact Obsidian block that generated them. This fork also supports path-based deck organization, so a note like `LinearAlgebra/matrix-derivatives.md` can sync to `LinearAlgebra::matrix-derivatives`.

![Path deck and source link demo](Images/demo.gif)

## What Changed

- Anki cards can jump back to the exact Obsidian block via `obsidian://open` links.
- Card IDs can be written as Obsidian block IDs, such as `^ID-1714433449297`.
- Decks can follow Obsidian file paths with templates such as `{path}`.
- `Scan Directory` prefixes are removed from generated path decks.
- Moving or renaming a Markdown file can move existing Anki cards to the new path deck.
- Context can be prepended to fields with file and heading information.
- Math and code formatting are handled in a safer order so `$` inside code is not treated as LaTeX.

The upstream documentation is still useful for the basic note syntax, AnkiConnect setup, custom regex notes, media syncing, frozen fields, and deletion markers.

## Deck Templates

The `Deck` setting can be a fixed deck name or a template. Supported variables:

```text
{path}    Obsidian file path without .md, using :: as separators
{folder}  Parent folder path, using :: as separators
{file}    File name without .md
```

Examples:

```text
Deck:          Default
Anki deck:     Default
```

```text
Deck:          {path}
Obsidian file: root/hello/world.md
Anki deck:     root::hello::world
```

```text
Deck:          {folder}::{file}
Obsidian file: LinearAlgebra/matrix-derivatives.md
Anki deck:     LinearAlgebra::matrix-derivatives
```

If `Scan Directory` is set, that prefix is removed:

```text
Scan Directory: anki
Deck:           {path}
Obsidian file:  anki/LinearAlgebra/matrix-derivatives.md
Anki deck:      LinearAlgebra::matrix-derivatives
```

New settings use `Deck = {path}` by default. Older path-deck configurations are migrated to `Deck = {path}`. If you move or rename a Markdown file, the next scan treats the new path as changed and moves existing Anki cards to the newly generated deck.

## Block Links

The upstream plugin writes note IDs like this:

```text
ID: 1714433449297
```

This fork can write them as Obsidian block IDs:

```text
^ID-1714433449297
```

When `Add File Link` and `Add Card link` are enabled, the plugin appends a source link to the configured Anki field. The link points back to the exact block:

```html
<a href="obsidian://open?...#^ID-1714433449297" class="obsidian-link">link</a>
```

For newly added notes, the plugin first creates the Anki note, receives the note ID, writes the block ID back to Markdown, and then updates the Anki field so the link points to the real block.

## Context And Formatting

When `Add Context` is enabled, file and heading context is placed before the card content. Headings are separated with line breaks and a horizontal rule:

```html
<br>course/chapter-5.md<br>Integrated Counters<br>Synchronous Counters<br><hr>
```

The Markdown formatter also protects code before converting math, cloze syntax, highlights, media, and links. Escaped dollar signs such as `\$` are restored to `$` in the final output.

## Installation

Requirements:

- Obsidian
- Anki
- AnkiConnect enabled in Anki
- AnkiConnect configured to allow Obsidian, usually with `app://obsidian.md`

### Install From Release Zip

1. Download `better-obsidian-to-anki-<version>.zip` from the [Releases](https://github.com/norcx/new_obsidian_to_anki/releases) page.
2. Extract the zip into your vault's plugin directory.
3. The final folder should look like this:

```text
<vault>/.obsidian/plugins/better-obsidian-to-anki/
  main.js
  manifest.json
  styles.css
```

On Windows, the path is usually similar to:

```text
D:\YourVault\.obsidian\plugins\better-obsidian-to-anki
```

Restart Obsidian, then enable `better obsidian to anki` in Community plugins.

### Install From Source

Clone this repository and build the plugin:

```bash
git clone https://github.com/norcx/new_obsidian_to_anki.git
cd new_obsidian_to_anki
npm install
npm run build
```

Create a plugin folder in your vault and copy these files into it:

```text
main.js
manifest.json
styles.css
```

For example:

```text
<vault>/.obsidian/plugins/better-obsidian-to-anki/
  main.js
  manifest.json
  styles.css
```

## Migrating From Obsidian_to_Anki

If you already used the upstream `Obsidian_to_Anki` plugin or an older copy of this fork, keep your existing `data.json`. That file stores plugin settings, note type field mappings, the added-media cache, and file hashes. Your Markdown files already contain Anki note IDs, so do not rewrite or remove those IDs during migration.

Recommended migration:

1. Close Obsidian.
2. Find the old plugin folder, commonly `<vault>/.obsidian/plugins/obsidian-to-anki-plugin/` for the upstream plugin, or an older fork folder such as `norcx-ob-to-anki/`.
3. Create or rename the target folder to `<vault>/.obsidian/plugins/better-obsidian-to-anki/`.
4. Keep or copy the old `data.json` into the target folder.
5. Replace `main.js`, `manifest.json`, and `styles.css` with the files from this fork's release zip or source build.
6. Disable or remove the old plugin folder so only one Obsidian-to-Anki plugin scans the vault.
7. Restart Obsidian and enable `better obsidian to anki`.
8. Open the plugin settings once after upgrading. Older path-deck settings are migrated to `Deck = {path}` there.

File handling summary:

| File | What to do | Why |
| --- | --- | --- |
| `data.json` | Keep or copy from the old plugin folder | Preserves settings, file hashes, media cache, and field mappings. |
| `main.js` | Replace with the new file | Contains this fork's plugin code. |
| `manifest.json` | Replace with the new file | Updates the plugin name, version, and metadata. |
| `styles.css` | Replace with the new file | Keeps the settings UI styles in sync. |
| Markdown notes | Leave unchanged | Existing note IDs link your Obsidian cards to Anki notes. |

If you migrate from the original upstream plugin, Obsidian may treat this fork as a separate plugin because the manifest ID can differ. Copying `data.json` into the new plugin folder is what carries your old settings into this fork.

## Recommended Settings

| Setting | Recommendation | Purpose |
| --- | --- | --- |
| `Add File Link` | Enable | Adds a link from Anki back to Obsidian. |
| `Add Card link` | Enable | Makes the source link target the exact Obsidian block. |
| `Deck` | `{path}` by default | Fixed deck name or template using `{path}`, `{folder}`, and `{file}`. |
| `Add Context` | Optional | Adds file and heading context before card content. |
| `ID Comments` | Usually disable for block IDs | HTML comments can hide IDs from Obsidian block recognition. |

## Custom Regex Compatibility

If your custom regex used this guard to avoid matching ID lines:

```regex
(?<!<!--)
```

use this with block IDs:

```regex
(?<!<!--)(?<!\^ID-)
```

Otherwise `^ID-...` lines may be captured as card content.

## Development

```bash
npm run build
npm test
```

The full test suite depends on the upstream Docker/Webdriver setup and a working Anki test environment.
