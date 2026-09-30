# Ancestor

An [Omarchy](https://omarchy.org) theme: gold leaf, oxidised copper and lamp-black,
taken from a render of a fractured classical bust — a gilded crown, a verdigris
skull, a broken jaw catching the light out of the dark.

Dark mode. Warm near-black backgrounds, a bronze-gold accent, parchment text,
and a patina-green/teal range standing in for the cool end of the ANSI palette.

## Palette

| Role         | Colour                                            |
| ------------ | ------------------------------------------------- |
| Accent       | `#c89a63` gilt bronze                             |
| Background   | `#0e0c0a` → `#050504` warm void                   |
| Foreground   | `#d6d3c4` bone, `#f6d4a5` gold highlight          |
| Warm         | `#be5a3c` rust · `#b4793f` bronze · `#d5a26c` gold |
| Patina       | `#5f8b74` green · `#6fa39c` cyan · `#6e8e9c` blue  |

Hyprland's active border is a 45° bronze-to-gilt gradient.

## Install

```bash
omarchy theme install https://github.com/svukelic/omarchy-theme-ancestor.git
```

Then pick **Ancestor** from the theme switcher, or:

```bash
omarchy theme set ancestor
```

## Backgrounds

- `1-ancestor.jpg` — the source render
- `2-gilded-void.jpg` — bronze glow in the dark
- `3-verdigris.jpg` — patina glow in the dark

Cycle them with `omarchy theme bg next`.

## Notes

Terminal, Neovim, VS Code, btop, Chromium and the rest are generated from
`colors.toml` by Omarchy's own templates, so the theme ships colours rather
than per-app config.
