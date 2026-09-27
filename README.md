# hwdeck

A one-page hardware dashboard for the screen next to your machine.

hwdeck puts CPU, GPU and memory load, every temperature sensor, fans, network and the
state of your services on a single screen, built to be read at a glance: three big
numbers, colour-graded load strips and a row of heat tiles. You see green, yellow or red
first and read the numbers only when you want to.

It was made for old hardware brought back as a side screen: a tablet running Linux on its
framebuffer console, a Raspberry Pi display, a spare laptop. So it asks very little:

- **16 colours and the stock console font.** Everything is drawn from block and
  box-drawing characters in the VGA (CP437) set, so it renders on a bare Linux VT with the
  kernel's default font, over SSH, or in any terminal.
- **Python 3 standard library only.** No pip install. It reads `/proc` and `/sys`, needs no
  root, and uses `sensors -j` (lm-sensors) and `nvidia-smi` when they are there.
- **One file.** Copy it, run it.

```
 W O R K S T A T I O N                                up 0d 6h   load 0.07                                    SUN 27 SEP  00:02

 ┌───────────────────────────────────────┐ ┌───────────────────────────────────────┐ ┌────────────────────────────────────────┐
 │ C P U                                 │ │ G P U                                 │ │ M E M                                  │
 │                                       │ │                                       │ │                                        │
 │  █  █ █   45°C                        │ │ ███ █ █   49°C                        │ │  █  ███ █ █   3.8 GB                   │
 │ ██    █                               │ │ █ █   █                               │ │ ██    █   █                            │
 │  █   █    0.8 GHz                     │ │ █ █  █    12 W                        │ │  █  ███  █    of 31 GB                 │
 │  █  █                                 │ │ █ █ █                                 │ │  █  █   █                              │
 │ ███ █ █   12 threads                  │ │ ███ █ █   0.0/8G vram                 │ │ ███ ███ █ █   swap 0.0G                │
 │ 11th Gen Intel Core i5-11400          │ │ XFX RX 5700 XT RAW II                 │ │ cache 9.7 GB                           │
 │ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄   │ │ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄   │ │ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄ ▄▄    │
 │ ██ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀   │ │ ██ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀   │ │ ██ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀ ▀▀    │
 └───────────────────────────────────────┘ └───────────────────────────────────────┘ └────────────────────────────────────────┘
  CPU                                                GPU                    SSD      RAM
  ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄   ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄   ▄▄▄▄▄▄   ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄ ▄▄▄▄▄▄
   45°    45°    43°    40°    43°    42°    43°      49°    49°    50°      43°      37°    36°    36°    37°
  ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀   ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀   ▀▀▀▀▀▀   ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀ ▀▀▀▀▀▀
```

(The top and the heat tiles of a real 128×48 frame; the whole frame is in [`examples/screen-128x48.txt`](examples/screen-128x48.txt).
On screen every block is coloured.)

## What is on the page

| Zone | What it shows |
|---|---|
| **Lead tiles** | CPU, GPU and memory: a big percentage, the temperature or size beside it, and a 12-segment load strip. The last lit segment is brighter; the strip turns yellow at 90 % and red at 97 %. The frame takes the state colour: dim when idle, green when working, yellow at 80 % load or within 15 °C of the sensor's critical point, red at 95 % or within 5 °C. |
| **History** | CPU and GPU load over the last few minutes, side by side. |
| **Heat tiles** | Every temperature sensor as a coloured tile, grouped by component (CPU, GPU, SSD, RAM, board, Wi-Fi, NIC…), plus fans in cyan (rpm in thousands, `off` when stopped). Noise channels (unwired board inputs, duplicate NVMe sensors) are hidden. |
| **Status bar** | Your systemd services as green or red squares, network down/up sparklines, root disk use, and optional one-line custom commands. |

GPU support: AMD (amdgpu, read straight from sysfs, including power, VRAM, clocks and fan)
and NVIDIA (through `nvidia-smi`). Without a GPU the tile says so and the rest still works.

## Run it

```bash
curl -LO https://raw.githubusercontent.com/pnshad/hwdeck/main/hwdeck
chmod +x hwdeck
./hwdeck
```

Needs a terminal of at least 100×40 (it is laid out for 128×48 and has a compact layout for
smaller). Ctrl-C to quit. For fan speeds and board sensors install lm-sensors
(`sudo apt install lm-sensors`, then `sudo sensors-detect`).

Options: `--config FILE` (default `~/.config/hwdeck/config.toml`), `--interval SECONDS`,
`--once` (draw one frame and exit). All settings are optional; see
[`config.example.toml`](config.example.toml).

## As a wall or desk screen

The idea is to run hwdeck in a tmux session on the machine and have the small screen attach to
it read-only, so nothing on the display can type into your machine:

```bash
tmux new-session -d -s deck -x 128 -y 48 ./hwdeck
tmux set-option -t deck status off
# on the display (or over ssh from it):
tmux attach -r -t deck
```

With SSH you can go further and give the display a key that can only do that, in the machine's
`~/.ssh/authorized_keys`:

```
restrict,pty,command="tmux attach -r -t deck" ssh-ed25519 AAAA... display
```

On a bare Linux console, `setterm -blank 0` keeps the screen on, and
`echo 1 > /sys/class/graphics/fbcon/rotate_all` turns it to landscape.

## Why these choices

- **Colour first, text second.** The screen is read from half a metre away while you work, so
  state is carried by colour and shape and the text stays small and sparse.
- **No braille, no truecolour.** btop's braille graphs are beautiful, but most console fonts
  don't have braille. Half-block graphs give two levels per cell and render everywhere.
- **Read-only by design.** hwdeck only reads `/proc`, `/sys`, `sensors` and `systemctl
  is-active`; the only commands it runs that you did not write are those.

## Licence

MIT.
