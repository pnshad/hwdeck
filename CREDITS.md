# Credits

hwdeck is MIT-licensed (see [LICENSE](LICENSE)). It vendors no third-party code.

| Project | Licence | How hwdeck uses it |
|---|---|---|
| [btop](https://github.com/aristocratos/btop) by Aristocratos | Apache-2.0 | the `btop` style borrows its look (boxes, graphs); no code is shared |
| [lm-sensors](https://github.com/lm-sensors/lm-sensors) | GPL-2.0 | `sensors -j` is read when installed (runtime tool, called by name) |
| NVIDIA `nvidia-smi` | proprietary | read when present (runtime tool) |
| `iw`, NetworkManager `nmcli`, systemd `resolvectl` | ISC / GPL-2.0 / LGPL-2.1 | network view, called by name |
| [Tailscale](https://github.com/tailscale/tailscale) | BSD-3-Clause | `tailscale status --json` in the network view, when installed |
| Linux `applesmc` driver | GPL-2.0 | its sysfs files are read; nothing is copied |
| [Nerd Fonts](https://github.com/ryanoasis/nerd-fonts) | MIT (glyphs: their own licences, e.g. Material Design Icons Apache-2.0) | icon code points in the views; the font is not shipped |

SMC key names come from public community lists of Apple SMC keys; the ones marked `?` are guesses.
