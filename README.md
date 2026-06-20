# Music Markdown Syntax

Syntax highlighting + lightweight **chord validation** for [Music Markdown](https://github.com/music-markdown/music-markdown)
files inside VS Code (and any editor that consumes TextMate / VS Code grammars).

It is a **grammar injection** into Markdown, so standard Markdown (headings, bold, lists, etc.)
keeps working untouched — this extension only adds rules for the music-specific lines.

## What it highlights

| Element | Example | Scope | Typical color (Dark+) |
| --- | --- | --- | --- |
| Chord-line marker | `c1:` | `constant.numeric.marker.chord.music-markdown` | orange |
| Valid chord | `G`, `Em`, `D7sus4`, `F#m`, `G/B`, `Gmaj7` | `entity.name.function.chord.music-markdown` | yellow |
| Annotation | `\|`, `(riff)` | `comment.line.annotation.music-markdown` | green/muted |
| **Invalid** chord | `Gmja7`, `H7` | `invalid.illegal.chord.music-markdown` | red |
| Lyric-line marker | `l1:` | `keyword.operator.marker.lyric.music-markdown` | green |
| Lyric text | `When you're down...` | `string.unquoted.lyric.music-markdown` | varies |
| Other voice marker | `t1:`, `r1:` | `keyword.operator.marker.voice.music-markdown` | green |

> Marker scopes are tuned so they resolve to distinct colors under the
> [An Old Hope](https://marketplace.visualstudio.com/items?itemName=dustinsanders.an-old-hope-theme-vscode)
> theme (chord marker orange, near the yellow chords; lyric marker green, near the blue lyric text).
> Under other themes the exact colors will differ — pin them via `editor.tokenColorCustomizations`
> (below) if you want them fixed.
| Markdown heading | `## Verse 1` | (unchanged — standard Markdown) | theme heading color |

The **validation** payoff: a chord line only colors tokens that pass the *exact* chord regex the
renderer uses. Mistype a chord and it turns red instead of yellow; forget the `l1:` prefix and the
lyric never gets tinted. Visual, instant feedback while you write.

## Fidelity to the renderer

The chord and annotation patterns are ported **verbatim** from
[`markdown-it-music@2.0.5`](https://github.com/music-markdown/markdown-it-music/blob/23e37862ddc07afcbf2ccff723b2d3d7937020fb/lib/chord.js)
(`lib/chord.js` → `CHORD_PATTERN` / `ANNOTATION_PATTERN`), which is the version the
`music-markdown/music-markdown` app actually depends on. The line-marker rule mirrors
[`parsers/verse.js`](https://github.com/music-markdown/markdown-it-music/blob/23e37862ddc07afcbf2ccff723b2d3d7937020fb/parsers/verse.js)
`voicePattern = /^([a-zA-Z-_]+)([0-9]*):\s(.*)/`, where only the `c` instrument is chord-validated
and every other prefix is raw text.

Two deliberate, documented deviations:

1. **`N.C.`** (no-chord) — the source regex leaves the dots unescaped (so they match any char);
   this grammar escapes them to literal `N.C.`, which is the obvious intent.
2. **Generic voice markers** are restricted to **lowercase** (`^[a-z][a-z0-9_-]*:`) to avoid lighting
   up ordinary prose with colons (e.g. `Note: see below`) in other Markdown files. The underlying
   parser also accepts uppercase; conventional song files only use lowercase markers, so this is safe
   in practice.

If you ever bump the `markdown-it-music` version in your renderer, re-verify these patterns against
the new release — the chord vocabulary or marker rules may have changed.

## Install (local / unpublished)

This isn't on the Marketplace. Symlink (or copy) it into your VS Code extensions folder:

```bash
ln -s "$(pwd)" ~/.vscode/extensions/music-markdown-syntax
```

Then reload VS Code (`Cmd+Shift+P` → **Developer: Reload Window**). Open any `.md` file with
`c1:` / `l1:` lines.

Or, to hack on it: open this folder in VS Code and press `F5` to launch an
**Extension Development Host** window with the grammar loaded.

## Debugging the grammar

`Cmd+Shift+P` → **Developer: Inspect Editor Tokens and Scopes**, then click any character. It shows
the exact scopes applied and which theme rule colored them — the REPL for TextMate grammars.

## Forcing exact colors (optional)

Colors come from your **theme**, not this grammar. If you want guaranteed, theme-independent colors,
add overrides to your VS Code `settings.json` keyed to the custom scopes:

```jsonc
"editor.tokenColorCustomizations": {
  "textMateRules": [
    { "scope": "entity.name.function.chord.music-markdown", "settings": { "foreground": "#E5C07B", "fontStyle": "bold" } },
    { "scope": "constant.numeric.marker.chord.music-markdown", "settings": { "foreground": "#EF7C2A" } },
    { "scope": "keyword.operator.marker.lyric.music-markdown", "settings": { "foreground": "#78BD65" } },
    { "scope": "string.unquoted.lyric.music-markdown",        "settings": { "foreground": "#98C379" } },
    { "scope": "invalid.illegal.chord.music-markdown",        "settings": { "foreground": "#E06C75", "fontStyle": "underline" } }
  ]
}
```

## License

MIT
