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

## Views

A view puts one part of the machine on the whole screen, for a hotkey or a Touch Bar tap.
Pick one with `--view`; each is one flag:

```bash
hwdeck --view cpu        # per-core load and clock, the CPU split, temperatures, throttling
hwdeck --view memory     # used / available / cached / buffers / swap, and the top memory users
hwdeck --view fans       # every fan: rpm, % of max, target, min / max, mode, the CPU temperature it follows
hwdeck --view battery    # charge, health, power in and out, the charge limit, battery temperatures
hwdeck --view power      # what the machine draws: battery watts, RAPL domains, AC, who uses the CPU
hwdeck --view network    # Wi-Fi SSID, band, signal and link rate, IPv4, live down / up, Tailscale
hwdeck --view thermal    # every temperature sensor and fan by zone (the same as --style thermal)
```

The views are drawn in the visual language of the Touch Bar strip rather than btop's: a pure
black screen, one accent colour per subject (CPU mint, fans cyan, memory violet, power yellow,
network lime, temperatures cyan warming through amber to red, battery red at empty, yellow at half, green at
the charge limit), grey labels, and one rounded frame in the accent. Under its top edge sits
the hero: the main number as a 7-segment LED readout (lit segments in the accent, unlit ones
faintly visible; the same size in every view, five rows, the unit on its bottom row), its
facts beside it, its rows and the history graph. Sections follow in
columns, each with its own heading and graph where one says something. Headings and the main
facts carry a Nerd Font icon.

They want a terminal with 256 colours and a Nerd Font, and use 24-bit colour when
`COLORTERM=truecolor` (foot sets it); on the kernel console (`TERM=linux`) the icons are left
out. Outside `--once` a view also turns the terminal's own background black (OSC 11), so
foot's padding matches, and gives it back on exit.

| View | What it shows |
|---|---|
| **`cpu`** | Total load as the LED readout with cores, clock, package temperature, load average and throttle state beside it; per-core load, clock and core temperature; the user / system / iowait / irq split; load history. `temp`: coretemp plus the CPU SMC keys, with the hottest one's history. `throttle`: thermal-throttle event counts and time since boot (core and package), headroom to the CPU's critical temperature, ACPI passive cooling steps, the TCC offset, idle injection, package watts (RAPL, root-only on most kernels). `clock`: average clock against the maximum with its history, turbo, the scaling cap, governor and energy preference, intel_pstate perf limits, min / max / base, driver. |
| **`memory`** | Used share as the readout; used, available, cached, buffers, free, shared and swap meters. `top`: resident memory per program name, summed over its processes, from `/proc/*/statm`. `swap`: every swap device and zram's compression ratio, with history. `cache`: page cache, buffers, reclaimable slab, active / inactive file pages, dirty and writeback, with history. |
| **`fans`** | One section per fan, side by side: rpm as the LED readout, % of max, the SMC target, min and max, auto or manual mode, the CPU temperature the fan loop is answering to, and an rpm history. |
| **`battery`** | Charge as `charge_now / charge_full` (the kernel's `capacity` divides by the design capacity, which on a worn pack reads low; it is shown beside it), status, time to full or empty, the charge limit, the AC adapter, and a charge history. `power`: watts in or out, current, voltage, with a watts history. `health`: `charge_full` against `charge_full_design`, cycle count, chemistry and model. `temp`: the battery's hwmon sensor and the SMC keys `TB0T`, `TB1T`, `TB2T`, `TBXT`, with history. |
| **`power`** | The whole system's draw as the readout, in whole watts: the Apple SMC's `PSTR` key, the mean of the last three samples, on AC or battery alike. It comes from a root service on the Mac that writes the SMC's power keys every 2 s to `/run/mac13-smc-power.json` (`{"t": unix, "watts": {key: W}}`, world-readable; `HWDECK_SMC_POWER` points elsewhere); older than 10 s counts as stale. Without it the readout falls back to the battery's watts (`current_now` x `voltage_now`) and the view says "SMC power service not running" (or "stale") in one line. Beside it: source, adapter watts in, battery watts and charge, CPU package watts. `where it goes`, full width: system `PSTR`, adapter in `PD0R`, CPU package `PCPT`, CPU cores `PCPC`, graphics `PCPG` and battery / platform `PPBR?` (a guess), each named with its raw key and a meter, over the system's history. `top`: CPU share per program since the last frame, from `/proc/*/stat`, as a cheap stand-in for who is drawing the power. `rapl`: the RAPL domains in watts, or "locked" while `energy_uj` is root-only. `ac`: the adapter, current and voltage. |
| **`network`** | The Wi-Fi signal as the readout, in % on the Touch Bar strip's scale `2 x (dBm + 100)` (clamped 0-100, the mean of the last three samples so it does not flicker; `--` and "no wifi" without a link). Beside it: the SSID, dBm, band and link rate, the IPv4 address, and the down and up rates, smoothed over about 5 s, each in a unit that changes only after the rate has wanted another one for 4 s and always `ddd.d` wide. Down / up graphs side by side under it (raw, not smoothed). Wi-Fi comes from `iw dev <if> link` (no root), else `/proc/net/wireless` and `nmcli -t`; the interface is the default route's (or `net_interface` in the config). On a taller window: `wifi` (signal, band, channel, frequency, rx / tx rates), `tailscale` (this node's tailnet address, peers online, exit node, from `tailscale status --json` at most every 15 s; the frame shows its state too), `ip` (address, gateway, DNS servers, via `resolvectl dns` behind systemd-resolved's stub) and `since boot` (bytes down / up per interface). |
| **`thermal`** | The hottest sensor as the readout with the fans and sensor count beside it, then one section per zone (CPU, fans, battery, board, skin & air, storage; each fan with rpm, % of max and the temperature its loop follows, the SMC's `TC0F` on a Mac) with a meter per sensor in its temperature colour, and the channels that read nothing in a "no sensor" strip. |

**Burn-in protection.** The views are meant to stay up for hours, and their frames and labels
do not move. So every 3 minutes the whole drawing shifts one cell, round (0,0), (1,0), (1,1),
(0,1) and back, inside a spare column and row kept free for it (every cell of a frame moves
together, so nothing jumps mid-frame). After 30 minutes the chrome (frame, headings, labels)
drops to about two thirds of its brightness; values, meters, readouts and graphs stay full.
Restarting the view brings it back. Both are on by default; `--no-orbit` or `orbit = false`
in the config turns them off. `HWDECK_ORBIT_SECONDS` and `HWDECK_DIM_SECONDS` override the
3 minutes and 30 minutes, for a quick check.

The charge limit is read from `charge_control_end_threshold`, on the battery or, on Intel
Macs, on the `applesmc` platform device. Batteries that report energy (`energy_now`,
`power_now`) work the same as those that report charge.

The views are laid out for a grid of equal windows on one laptop screen, about 66 x 27 each
(the fans live in thermal there; `--view fans` still works on its own and fits 66 x 12), and
use the space they get: cores and
meters pair up on a short window, sections pack into two columns, and thermal fits all its
sensors (33 on a MacBook Pro 14,2) in two columns at that size. They fit any size from
50 x 10: on a short screen the readout gives way to a plain row, long lists end in "+N more"
and the last sections drop. Graphs fill in as hwdeck runs; a `--once` frame has one sample.

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

Options: `--style NAME` (`btop`, `bunker`, `eq`, `panel`, `glance`, `thermal`), `--view NAME`
(`cpu`, `memory`, `fans`, `battery`, `power`, `network`, `thermal`; wins over `--style`), `--no-orbit` (views: no burn-in protection), `--config FILE` (default
`~/.config/hwdeck/config.toml`), `--interval SECONDS`, `--once` (draw one frame and exit). Every setting is optional; see
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

- **No braille.** btop's braille graphs are beautiful, but most console fonts don't have
  braille. Half-block graphs give two levels per cell and render everywhere. The styles keep
  to 16 colours; only the views use 256 or 24-bit colour and Nerd Font icons.
- **Read-only by design.** hwdeck only reads `/proc`, `/sys`, `sensors` and `systemctl
  is-active`, plus any `[extra]` commands you add to your own config.
- **Hardware, not programs**, with two exceptions: the `memory` view lists the programs using
  the most memory (two small reads per process each refresh) and the `power` view the ones
  using the most CPU (one read per process).
- **CPU package power** comes from the RAPL counter, which most kernels only let root read;
  without it that reading is left out.

## Licence

MIT. The `btop` style borrows the look of btop by Aristocratos; hwdeck shares no code with it.
