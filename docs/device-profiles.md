# Device profiles

A device profile tells the app about one MIDI device: its sounds, how it picks
banks, which controllers its mixer listens to and how to reset it. Each profile
is one YAML file.

Built-in profiles live in `resources/devices`. Put your own in the app's
Devices folder (Options > Show Device Profiles Folder) and restart the app. A
file with the same `id` as a built-in one replaces it.

A track uses a profile when its MIDI port name contains one of the profile's
`ports` entries. Ports with no match use General MIDI.

## Korg Pa

`korg-pa.yaml` matches MIDI ports whose name contains `Korg`, `Pa5X`, `Pa4X`,
`Pa3X`, `Pa1000`, `Pa700`, `Pa600`, `Pa300` or `Pa80`. It uses the GM bank on
channels 1-16 and drums on channel 10. Korg factory voices use bank select
values; add a sound add-on for your model to use them.

Korg Pa styles that are saved as Yamaha SFF1 or SFF2 files load as normal
styles. They keep their section markers and parts, and the app can fill the
Timeline from them.

## Format

```yaml
profile: 1                 # format version, always 1
id: roland-d-05            # unique, same as the file name
name: Roland D-05
maker: Roland
dialect: none              # gm, gs, xg or none
ports: ["D-05"]            # parts of the MIDI port name, case ignored
drumChannels: []           # channels that play drums (often [10])
programBase: 1             # 1 if the manual numbers programs 1-128, 0 for 0-127
mixer:                     # controller number for each mixer knob, or none
  volume: 7
  pan: none
  reverb: none
  chorus: none
reset:                     # optional SysEx, sent by "Reset device"
  - "F0 41 10 42 12 40 00 7F 00 41 F7"
notes: Free text for the user.
capabilities:              # optional, for AI > Fit to My Synths
  polyphony: 8             # notes at once over all parts (use the safe number)
  parts: 1                 # channels with different sounds at once
  lowest: 0                # MIDI note range the synth plays
  highest: 127
  controllers: [1, 7, 64]  # optional: the controllers it reacts to
  source: manual           # where the numbers come from: manual, estimate...

sounds:
  - group: "Preset 1"
    items:                 # [name, bank MSB, bank LSB, program]
      - ["Fantasia", 87, 8, 1]
      - ["Metal Harp", 87, 8, 2]
```

Use `-` for a bank byte the device doesn't want sent.

Without `capabilities`, the hardware check says that it doesn't know the
numbers. A patch with two tones, layers or a long release can use more
voices than the specification says, so give the safe number. The user can
change the numbers in the app; their numbers win over the profile.

## Sound editor

The `editor:` part lists the sound settings of the synth. The app makes a
sound editor from it (`Track > Synth Sound Editor...`): one page for each
`pages:` entry, a knob for a number, a list for `choices`. Each change goes to
the synth at once. When the synth sends the same message (you turn a knob on
it), the knob on the screen follows.

```yaml
editor:
  pages:
    - name: Filter
      params:
        - { name: Frequency, cc: 105 }                    # a controller, 0-127
        - { name: Mod Env Depth, cc: 107, offset: -64 }   # shows 0-127 as -64..63
        - { name: Type, nrpn: 62, choices: [Low pass, Band pass, High pass] }
        - { name: Chorus Type, nrpn: 104, choices: [Chorus, Phaser], values: [34, 35] }
```

A setting has a `name` and one message:

| Key | Message |
| --- | --- |
| `cc: 74` | Control change 74 |
| `nrpn: 62` or `nrpn: [1, 5]` | NRPN (MSB 0, LSB 62; or MSB 1, LSB 5), value on CC 6 |
| `rpn: 0` | RPN, value on CC 6 |
| `at: "1A"` | SysEx: the address (hex) after the page's `base`, sent with the `sysex: write` pattern |

And optional:

| Key | Meaning |
| --- | --- |
| `min`, `max` | The MIDI range. Default 0 and 127; with `choices`, as many as there are choices |
| `choices` | Names of the values min, min + 1, ... |
| `values` | The MIDI value of each choice, when they are not in a row (several switches on one NRPN) |
| `offset` | Added to the value the user sees: `-64` for -64..63, `1` for 1..100 |
| `unit` | Shown after the number, for example `dB` |
| `noteFrom` | Show the value as a note name; `min` is this note, for example `C1` |
| `default` | The value for a double-click on the knob |

### SysEx synths

For a synth with SysEx settings, give the message patterns. Hex bytes and
these fields: `{dev}` the device ID, `{addr}` the address, `{value}` the value,
`{size}` the size to read, `{sum}` a Roland check sum of the bytes from
`{addr}` up to it.

```yaml
editor:
  sysex:
    write: "F0 41 {dev} 14 12 {addr} {value} {sum} F7"
    read: "F0 41 {dev} 14 11 {addr} {size} {sum} F7"   # optional: Read from Synth
    device: channel        # {dev}: the MIDI channel - 1, or a number
    addressBytes: 3
    sizeBytes: 3
  sections:                # settings that repeat on more pages
    partial:
      - { name: TVF Cutoff, at: "0D", max: 100 }
  pages:
    - { name: Upper Partial 1, base: "00 00 00", section: partial }
    - { name: Upper Partial 2, base: "00 00 40", section: partial }
```

Addresses add up in 7-bit bytes, as in Roland manuals: `00 00 40` + `40` is
`00 01 00`.

### Sound names from the synth

With `names:`, the editor's `Get Sound Names` button reads the names of the
sounds stored in the synth. For each bank, the app selects it (bank select
and program 1), then reads each name with the `read` pattern.

```yaml
  names:
    first: "02 03 00"      # address of the first name
    stride: "00 03 40"     # from one sound to the next
    count: 64              # sounds in a bank
    length: 18             # characters in a name
    charset: " ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz1234567890-"  # empty: ASCII
    banks:                 # [group, bank MSB, bank LSB]; none: the bank the synth is on
      - ["Internal 1", 87, 0]
```

## Your own sound names

The names that the app reads from a synth, and names you or the AI assistant
give, go in a sound list file next to the profile: `<id>.sounds.yaml` in the
Devices folder. The built-in profile does not change.

```yaml
soundsFor: novation-xiosynth
sounds:
  - group: "Bank A"
    items:
      - ["Analog Bass", -, 0, 5]   # same bank and program as A005: renames it
  - group: "My Sounds"             # a new group
    items:
      - ["Big Pad", -, 1, 12]
```

## Audio interfaces

A device that is also a USB audio interface can name its inputs and outputs in
an `audio:` map. An audio track then shows these names in `Input:` in the
inspector.

```yaml
audio:
  devices: ["L6max"]           # parts of the audio device name, case ignored
  inputs:                      # in the order the computer lists them
    - "CH1"                    # one channel
    - { name: "CH5", channels: 2 }   # a stereo pair
  outputs:
    - { name: "USB 1/2", channels: 2 }
```

The app uses the names only when the open device has the same number of input
channels as the list. Otherwise it uses `In 1+2`, `In 1 (mono)` and so on.

## Zoom LiveTrak L6max

`zoom-l6max.yaml` matches only the `L6max Mixer Control Port` (Windows:
`MIDIIN2/MIDIOUT2 (ZOOM L6max)`). It does not match the `L6max MIDI I/O Port`:
that port goes to the MIDI IN/OUT jacks, so use the profile of the device on
the jacks. Do not use the `for L6 Editor Port`.

- The track faders send nothing to this port. The L6max takes its mixer
  settings on controller numbers that you set in ZOOM L6 Editor.
- `Track > Synth Sound Editor...` has the EQ controls with the factory numbers
  that the manual shows (EQ HI and MID FREQ for CH1-8, EQ MID and LO for
  CH1-4). The manual does not print the numbers for LEVEL, PAN, MUTE and the
  sends.
- The track sound list has Scene A-D (program change 0-3).
- In Multi Track USB mode the 14 inputs are named CH1-CH4, CH5-CH8 (stereo) and
  MASTER (stereo).

## Writing one with an AI

Give the AI this page and the device manual's MIDI implementation chart and
sound list, and ask for a profile file. Things to check in the manual:

- Bank select: which MSB and LSB values pick each bank.
- Whether program numbers are printed 1-128 or 0-127.
- Whether the device receives CC 7, 10, 91 and 93. Some use other numbers
  (the XioSynth takes volume on CC 119) or none at all.
- The reset message, if it has one (GM On, GS Reset, XG System On).
- For a sound editor: the controller, NRPN or SysEx address of each sound
  setting, with its range.

The app reports the line number when a profile has a mistake.

## Bundled profiles

| File | Source |
| --- | --- |
| `yamaha-psr-s970.yaml` | PSR-S970 Data List, Voice List |
| `roland-d-05.yaml` | D-05 Parameter Guide, Sound List and MIDI Implementation (sound editor: SysEx address map) |
| `novation-xiosynth.yaml` | XioSynth MIDI Implementation chart (sound editor: controller and NRPN lists) |
| `zoom-l6max.yaml` | L6max Operation Manual: USB MIDI ports, MIDI CC# Mapping picture, MIDI implementation chart, USB audio mode |
| `generic-gs.yaml`, `generic-xg.yaml` | GM sound set with GS and XG resets |
