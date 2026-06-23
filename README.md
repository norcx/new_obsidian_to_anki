# better-obsidian-to-anki

[中文文档](README.zh-CN.md)

better-obsidian-to-anki is forked from [Obsidian_to_Anki](https://github.com/Pseudonium/Obsidian_to_Anki). The original project has not been actively maintained for a long time, so this fork keeps the existing Obsidian-to-Anki workflow and adds the features I wanted for my own study setup.

The main addition is source navigation: cards created in Anki can link back to the exact Obsidian block that generated them. This fork also supports path-based deck organization, so a note like `LinearAlgebra/matrix-derivatives.md` can sync to `LinearAlgebra::matrix-derivatives`.

![Path deck and source link demo](Images/demo.gif)

## What Changed

- Anki cards can jump back to the exact Obsidian block via `obsidian://open` links.
- Card IDs can be written as Obsidian block IDs, such as `^ID-1714433449297`.
- Decks can follow Obsidian file paths with `Use Path as Deck`.
- `Scan Directory` prefixes are removed from generated path decks.
- Moving or renaming a Markdown file can move existing Anki cards to the new path deck.
- Context can be prepended to fields with file and heading information.
- Math and code formatting are handled in a safer order so `$` inside code is not treated as LaTeX.

The upstream documentation is still useful for the basic note syntax, AnkiConnect setup, custom regex notes, media syncing, frozen fields, and deletion markers.

## Path Decks

With `Use Path as Deck` enabled:

```text
Obsidian file: root/hello/world.md
Anki deck:    root::hello::world
```

If `Scan Directory` is set, that prefix is removed:

```text
Scan Directory: anki
Obsidian file:  anki/LinearAlgebra/matrix-derivatives.md
Anki deck:      LinearAlgebra::matrix-derivatives
```

`Use Path as Deck` is enabled by default for newly generated settings. If you move or rename a Markdown file, the next scan treats the new path as changed and moves existing Anki cards to the newly generated deck.

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

Manual build:

```bash
npm install
npm run build
```

Copy these files into the Obsidian plugin folder:

```text
main.js
manifest.json
styles.css
```

## Recommended Settings

| Setting | Recommendation | Purpose |
| --- | --- | --- |
| `Add File Link` | Enable | Adds a link from Anki back to Obsidian. |
| `Add Card link` | Enable | Makes the source link target the exact Obsidian block. |
| `Use Path as Deck` | Enabled by default | Maps Markdown paths to Anki deck paths. |
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
