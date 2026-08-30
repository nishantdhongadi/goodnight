# goodnight

One command to shut the desk down at the end of the day, instead of reaching
behind the monitor to unplug things.

macOS blanks an idle display to a screensaver long before it ever cuts power to
it, and USB peripherals stay lit the whole time. `goodnight` skips the idle
timer entirely and tells the machine to sleep *now*.

```
goodnight          # full system sleep (default, 5s countdown)
goodnight -d       # display sleep only, Mac keeps running
goodnight -m       # also send DDC standby to the monitor
goodnight -n 0     # skip the countdown
goodnight -h       # usage
```

## Install

```sh
git clone git@github.com:nishantdhongadi/goodnight.git
ln -s "$PWD/goodnight/goodnight" ~/.local/bin/goodnight   # anywhere on PATH
```

Optional, for `-m`:

```sh
brew install m1ddc
```

## Why explicit sleep

`pmset sleepnow` and `pmset displaysleepnow` are explicit commands, so they fire
even while something is holding a power assertion — a `caffeinate` process, a
video call, or an app like Amphetamine keeping the machine awake. You don't have
to quit those first, and you don't have to change your energy settings.

Full system sleep is the default on purpose. Display-only sleep is easily undone
by a HID wake event, so a nudged mouse or a brushed keyboard relights the panel
seconds later. Sleeping the whole machine also drops USB bus power, which is the
only lever most peripherals respond to.

## About `-m`

`-m` sends VCP `0xD6` = `4` (standby) over DDC/CI, for displays that keep their
backlight on through ordinary DPMS. It is deliberately *not* `5` (hard off):
on DisplayPort, a hard off can drop the link so macOS registers a disconnect and
reshuffles your windows, and DDC can't reach a powered-off controller to bring
it back — leaving you pressing the physical button, which is the thing this
script exists to avoid.

If your display ignores the standby command, `-m` says so and sleeps anyway.

## Caveats

**Keyboard and peripheral lighting.** Most RGB keyboards run their lighting in
firmware with no host-side control on macOS, particularly the common SINO WEALTH
controllers, which are neither QMK nor VIA and have no open protocol. For those,
lights go out only when USB bus power actually drops. Whether that happens on
sleep depends on your hub: a bus-powered hub loses power with the Mac, while a
self-powered one may keep feeding the port all night. If your lights survive
sleep, the fix is a hub with per-port switches or a smart plug — there isn't a
software answer.

**Waking back up on its own.** Wake-on-LAN and Power Nap will wake the machine
during the night, and every wake restores USB power and relights whatever went
dark. If things are off when you go to bed and lit by morning, check what woke it:

```sh
pmset -g log | grep -iE 'Wake from|DarkWake' | tail -20
```

and if that's the cause:

```sh
sudo pmset -a womp 0 powernap 0
```

Leave `womp` alone if you ever wake this machine remotely.

## Requirements

macOS (Apple Silicon or Intel), zsh. `m1ddc` only for `-m`.
