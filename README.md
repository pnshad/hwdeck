# hwdeck

A one-page hardware dashboard for the screen next to your machine.

hwdeck puts every sensor your Linux box exposes on a single screen: CPU cores, GPU, memory,
disks, network, every temperature channel, every fan with its PWM duty, and every voltage
rail. It shows hardware, not programs. It was made for old hardware brought back as a side
screen: a tablet running Linux on its framebuffer console, a Raspberry Pi display, a spare
laptop. So it asks very little:

- **16 colours and the stock console font.** Everything is drawn from block and box-drawing
  characters in the VGA (CP437) set, so it renders on a bare Linux VT with the kernel's
  default font, over SSH, or in any terminal.
- **Python 3 standard library only.** No pip install. It reads `/proc` and `/sys`, needs no
  root, and uses `sensors -j` (lm-sensors) and `nvidia-smi` when they are there.
- **One file.** Copy it, run it.

```
┌─┐1cpu┌───────────────────────────────────────────────────┐02:42:57┌───────────────────────────────┐workstation  up 0d 08:42┌─┐
│ 2%                                                                   ┌─┐11th Gen Intel Core i5-11400┌───────────────────────┐│
│                                                                      │ CPU ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■   2%  47°C 2.5G  ││
│                                                                      │ C0  ■■■■■■■■■■   0%  46°C  C6  ■■■■■■■■■■   1%  46°C ││
│                                                                      │ C1  ■■■■■■■■■■   0%  44°C  C7  ■■■■■■■■■■   4%  44°C ││
│                                                                      │ C2  ■■■■■■■■■■   1%  42°C  C8  ■■■■■■■■■■   0%  42°C ││
│                                                                      │ C3  ■■■■■■■■■■   4%  44°C  C9  ■■■■■■■■■■   3%  44°C ││
│                                                                      │ C4  ■■■■■■■■■■   2%  43°C  C10 ■■■■■■■■■■   0%  43°C ││
```

On screen every meter and graph is coloured. The full frames are in
[`examples/screen-btop.txt`](examples/screen-btop.txt) and
[`examples/screen-bunker.txt`](examples/screen-bunker.txt).

## Styles

The same data, five ways to draw it, plus a sixth that shows only heat and air. Pick one with `--style` or `style = "..."` in the config.

| Style | What it looks like |
|---|---|
| **`btop`** (default) | Coloured boxes with their titles in the top border, gradient graphs, `■` meters and per-core rows, in the visual language of [btop](https://github.com/aristocratos/btop). |
| **`bunker`** | One heavy chassis of LED meter columns with scale rails, stencil zone tags, amber readouts and a hazard-striped master alarm plate. |
| **`eq`** | Groups of equalizer bars with falling peak-hold caps, like an old media player's spectrum analyzer. |
| **`panel`** | A plant control panel: annunciator lamps that light by level, LED ladders and bracketed readouts. |
| **`glance`** | Three big numerals with load strips and heat tiles, for reading from across a room. |
| **`thermal`** | Every temperature sensor and every fan, one btop-style box per zone (CPU, fans, battery, board, skin & air, storage) with gradient meters and a history graph each. Channels that read nothing are listed raw in a "no sensor" strip. |

`btop`, `bunker`, `eq` and `panel` are laid out for 128×48 (a 1024×768 console with the
8×16 font); `glance` works from 100×40. A smaller terminal falls back to `glance`. `thermal` fits itself to
any size from 50×14: more columns on a wide screen, then it drops the summary box and trims
rows on a short one.

## What it reads

| Group | Sources |
|---|---|
| CPU | per-core load and clock, user / system / iowait / irq split, load average, context switches and interrupts per second |
| GPU | AMD via amdgpu sysfs (busy, VRAM, power and cap, temperatures, fan and PWM, core and memory clocks, core voltage) or NVIDIA via `nvidia-smi` |
| Memory, disks | used / available / cached / swap, per-mount usage, read and write rates |
| Network | per-interface bytes and packets per second |
| Temperatures | every hwmon channel: CPU package and cores, GPU, NVMe, DIMMs, board inputs, chipset, Wi-Fi, NIC; on Intel Macs every applesmc key (CPU, battery, memory, PCH, heatpipes, palm rest, airflow, ambient, Thunderbolt, SSD) |
| Cooling | every fan header's rpm, PWM duty and control mode; applesmc fans with min, max and target rpm |
| Power | every voltage rail the board reports, GPU power history |
| Services | systemd units you list, shown as green or red |

Channels that read nothing (unwired board inputs) are left out.

The `thermal` style reads every hwmon chip straight from `/sys/class/hwmon` (no lm-sensors
needed), including `applesmc` on Intel Macs, whose files sit on the platform device. Each
4-character SMC key gets a friendly name and a zone, with the raw key kept beside it; names
marked `?` are community guesses for that key. Readings of -127, -44, -0.25 or 0 are absent
sensors and are listed as "no sensor". Fans show rpm, % of max, min / max, the SMC target and
auto or manual mode. Alarm thresholds come from
the sensor's own critical value where it reports one.

## Run it

```bash
curl -LO https://raw.githubusercontent.com/pnshad/hwdeck/main/hwdeck
chmod +x hwdeck
./hwdeck
```

Ctrl-C to quit. For fan speeds, voltages and board temperatures install lm-sensors
(`sudo apt install lm-sensors`, then `sudo sensors-detect`); some boards need a driver
module, for example `nct6775`.

Options: `--style NAME` (`btop`, `bunker`, `eq`, `panel`, `glance`, `thermal`), `--config FILE` (default `~/.config/hwdeck/config.toml`),
`--interval SECONDS`, `--once` (draw one frame and exit). Every setting is optional; see
[`config.example.toml`](config.example.toml).

## As a wall or desk screen

Run hwdeck in a tmux session on the machine and have the small screen attach to it
read-only, so nothing on the display can type into your machine:

```bash
tmux new-session -d -s deck -x 128 -y 48 ./hwdeck
tmux set-option -t deck status off
# on the display (or over ssh from it):
tmux attach -r -t deck
```

With SSH you can go further and give the display a key that can only do that, in the
machine's `~/.ssh/authorized_keys`:

```
restrict,pty,command="tmux attach -r -t deck" ssh-ed25519 AAAA... display
```

On a bare Linux console, `setterm -blank 0` keeps the screen on, and
`echo 1 > /sys/class/graphics/fbcon/rotate_all` turns it to landscape.

## Notes

- **No braille, no truecolour.** btop's braille graphs are beautiful, but most console fonts
  don't have braille. Half-block graphs give two levels per cell and render everywhere.
- **Read-only by design.** hwdeck only reads `/proc`, `/sys`, `sensors` and `systemctl
  is-active`, plus any `[extra]` commands you add to your own config.
- **CPU package power** comes from the RAPL counter, which most kernels only let root read;
  without it that reading is left out.

## Licence

MIT. The `btop` style borrows the look of btop by Aristocratos; hwdeck shares no code with it.
