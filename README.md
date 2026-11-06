# pitch-freeze

A Pure Data patch for the [Pisound](https://blokas.io/pisound/) (Raspberry Pi audio HAT) that turns a guitar input into a MIDI-controlled pitch shifter with a live "freeze" looper.

Tested on a Pisound, including freezing with the hardware button.

## What it does

- **Pitch shift** — the guitar on input 1 is written to a 2-second delay line and read back by two overlapping, triangle-windowed delay taps (`vd~`), giving a classic granular pitch shifter. Incoming MIDI notes set the interval: note 60 (middle C) is unity, and every semitone above or below shifts by one semitone.
- **Freeze** — on a freeze trigger the input is captured into a 2-second buffer (`freezebuf`, 96000 samples) and looped with two crossfaded read heads, so the sound sustains without clicks. A 30 ms ramp crossfades between the live/pitch-shifted signal and the frozen loop.
- **Loop playback modes** — forward, reverse or ping-pong.
- **Loop size** — adjustable from 1000 to 95999 samples, and can be changed while frozen to alter the loop length live.
- **Wet/dry mix** — blends the processed signal with the dry guitar.
- Output goes to both channels (`dac~ 1` and `dac~ 2`).

## Controls

| Control | Source | Range |
| --- | --- | --- |
| Pitch | MIDI note on | note 60 = unity |
| Freeze on/off | UDP message `freeze` on port 3000, or the on-screen toggle | 0 / 1 |
| Freeze loop size | MIDI CC 1, or the "freeze loop size" slider | 1000 – 95999 samples |
| Wet/dry mix | MIDI CC 2, or the "wet/dry mix" slider | 0 – 1 |
| Loop playback mode | MIDI CC 3, or the radio buttons | 0 forward, 1 reverse, 2 ping-pong |

The UDP freeze message is meant to come from a script bound to the Pisound's hardware button, for example:

```bash
echo "freeze 1;" | pdsend 3000 localhost udp
```

## Requirements

- [Pure Data](https://puredata.info/) (vanilla 0.51 or newer; uses `expr`/`expr~`, which ship with vanilla)
- A Pisound or any other audio interface with a mono input
- A 48 kHz sample rate — the freeze buffer length and loop timing assume 48000 Hz
- Optional: a MIDI controller for notes and CC 1–3

## Running

Open the patch in Pd with DSP on:

```bash
pd -r 48000 -mididev 1 main.pd
```

On a Pisound you can also run it headless (`pd -nogui -r 48000 main.pd`) or launch it through Pisound's button/Patchbox setup.

## Files

- `main.pd` — the whole engine in a single patch.
