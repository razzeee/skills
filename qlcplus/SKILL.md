---
name: qlcplus
description: "QLC+ (Q Light Controller Plus) lighting software — workspace files (.qxw), scenes, chasers, sequences, collections, EFX, RGB matrices, fixture definitions (.qxf), Virtual Console, and timing calculations. Use this skill whenever the user is working with QLC+ workspace XML, creating/editing scenes/chasers/shows, debugging timing or crossfade issues, generating fixture definitions, setting up Virtual Console widgets (cue lists, solo frames, buttons, sliders), troubleshooting HTP/LTP conflicts, fixing corrupted .qxw files, configuring QLC+ plugins (DMX USB, Art-Net, MIDI, E1.31), or asking any question where QLC+ is the software being used. Also trigger for SpeedModes, FadeIn/Hold/Duration, FixtureVal, RunOrder, or QLC+ function types. Do NOT trigger for general DMX hardware questions without QLC+ context, fixture buying advice, DAW-only questions, or custom protocol implementations."
---

# QLC+ Lighting Programming Skill

You are helping with QLC+ (Q Light Controller Plus) v5 lighting automation. This covers everything from fixture setup through show programming.

## Key Concepts

### DMX Fundamentals

- 512 channels per universe, values 0-255
- QLC+ supports unlimited universes (4 default)
- Fixtures are patched at a universe + start address

### HTP vs LTP (Critical for understanding conflicts)

| Rule                           | Channel types                                     | Behavior                            |
| ------------------------------ | ------------------------------------------------- | ----------------------------------- |
| HTP (Highest Takes Precedence) | Intensity, dimmer, color intensity (R/G/B/W/A)    | Highest active value wins           |
| LTP (Latest Takes Precedence)  | Pan, tilt, gobo, strobe, speed, all non-intensity | Most recently started function wins |

HTP means two active scenes both setting a dimmer will show the higher value. LTP means the last-started function controls non-intensity channels — order matters, and unexpected jumps can occur.

During crossfades: HTP levels transition smoothly. LTP levels may jump immediately (gobo) or transition (pan/tilt) depending on fade time.

### Fade Out Only Affects HTP

This is a common source of confusion. A scene's Fade Out time only fades intensity/HTP channels to zero. LTP channels (strobe mode, gobo position, pan/tilt) retain their values until another function overwrites them.

### Grand Master

Final master slider before DMX output. Two modes:

- **Reduce**: channels reduced by percentage (50% GM = all channels at 50% of current)
- **Limit**: channels cannot exceed the GM value (GM at 127 = max output 127)

Usually affects only Intensity channels, but can be set to affect all.

### Blackout

Sets ALL HTP channels in ALL universes to zero. Channels stay at zero regardless of running functions. When switched off, functions resume control.

### Palettes (v5)

A Palette abstracts a fixture feature (color, position, zoom). Can be used in Scenes to make looks fixture-independent. If you change a Palette, all Scenes using it update.

---

## Function Types

### Scene

A snapshot of channel values. Key properties:

- **Selective channel control**: Only enabled channels are affected. Disabled channels are NEVER touched. This enables layering.
- **Fade In**: Time to fade ALL channels (HTP AND LTP) to their target values. This is unique to Scenes — in other functions, Fade In only affects HTP.
- **Fade Out**: Time to fade HTP/intensity channels back to zero. **ONLY HTP channels are affected by Fade Out** — LTP channels retain their values.
- **No duration**: Scenes persist until stopped or overridden
- Monolithic scenes (all channels enabled) work well for chaser-driven workflows where only one scene is active at a time
- Partial scenes (only some channels) enable layering but require careful management
- **Crossfade blending**: When a chaser transitions between two scenes, it calls `setBlendFunctionID` so the new scene reads starting values from the previous scene — enabling true crossfade without snapping.

### Chaser

Runs steps sequentially. Each step is a Function (usually a Scene).

**The timing model (from source code):**

```
duration = fadeIn + hold
```

- `duration` is the TOTAL step time, not just hold
- `hold = duration - fadeIn`
- The step ends when `elapsed >= duration`
- On step end, `prevStepRoundElapsed = elapsed % duration` carries overshoot into the next step (prevents timing drift)
- **FadeOut is NOT included in duration** — it overlaps with the next step or runs after stop

**SpeedModes (on Chaser element) — precise behavior from source:**

| Mode        | Meaning                                                                                                                                                            |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Common**  | All steps use the chaser's own `<Speed>` values                                                                                                                    |
| **PerStep** | Each `<Step>` attribute is used directly                                                                                                                           |
| **Default** | Passes `Function::defaultSpeed()` (sentinel value UINT_MAX-1) to the child function, meaning the child uses its OWN fade times. The chaser does not override them. |

For Duration mode specifically: `Default` and `Common` behave identically — both use the chaser's `duration()`.

**Practical implications:**

- With `SpeedModes FadeIn="Default" Duration="PerStep"`: the step's Hold attribute IS the total step time. The child function's own FadeIn is used for crossfading. The Hold value is what contributes to the timeline.
- With `SpeedModes FadeIn="PerStep" Duration="PerStep"`: the step's FadeIn overrides the child function's fade. Duration = FadeIn + Hold.

**Run orders:** Loop, SingleShot, PingPong, Random
**Direction:** Forward, Backward

**Beat tempo:** `elapsedBeats` increments by 1000 on each beat. Duration in beats = multiples of 1000 (e.g., 4000 = 4 beats). Set `<Tempo Type="Beats"/>`.

**Infinite speed** (`Function::infiniteSpeed()`): Step never auto-advances. Used for manual cue lists. The runner checks: `if (duration != infiniteSpeed && elapsed >= duration)`.

### Sequence

A chaser bound to a single parent Scene — all steps control the same channels. Appears as a child of its parent Scene in the Function Manager.

**Sequence step values format** (inside `<Step Values="N">`):

```
fixtureID:channel,value,channel,value:fixtureID:channel,value,...
```

Fixture chunks separated by `:`, channel/value pairs by `,`. Only non-zero values saved.

### Collection

Runs multiple functions **simultaneously** (parallel, not serial). All members start in `preRun()`. Collection stops when all children finish (`m_runningChildren` becomes empty).

- Cannot contain itself (self-containment check in `postLoad`)
- Duplicate function IDs are rejected
- No speed settings of its own

### EFX

Automates pan/tilt (or RGB/dimmer) along mathematical paths. Algorithm types: Circle, Eight, Line, Line2, Diamond, Square, SquareChoppy, SquareTrue, Leaf, Lissajous.

**Propagation modes:** Parallel (all fixtures move together), Serial (offset start), Asymmetric (mirror offset).

### RGB Matrix

Graphic patterns/text on a grid of RGB fixtures. Requires a Fixture Group defining pixel layout. Has Fade In, Fade Out, Duration per frame. Extensible via ECMAScript RGB scripts.

### Show

Timeline-based function (Show Manager). Places functions on time tracks with precise timing. Best for pre-programmed sequences synced to music or a metronome.

**Show Manager key concepts:**

- Multitrack view similar to a DAW — each track is bound to a Scene
- Sequences on a track can only control channels of that track's Scene
- Functions can be placed at precise time positions, dragged, copied
- Supports BPM grid (4/4, 3/4, 2/2) with snap-to-grid for music sync
- Tracks support mute/solo states
- Playback always starts from cursor position, can resume

**Sequences vs Chasers in Show Manager:**

- Sequence steps are values of ONE Scene's channels (bound to track). Chaser steps can be ANY function.
- Sequences in Show Manager always play Forward. Chasers support all run orders.
- Sequence editing: new steps copy previous step's values (adjust differences). Chaser editing: pick existing functions.
- Sequences are better for extending shows (add track + sequence for new fixtures). Chasers require complex Collections for sync.

### Script

ECMAScript-based automation of QLC+ functions in sequential order.

---

## Workspace XML (.qxw) Structure

The .qxw file is XML. Common attributes on `<Function>`: `ID`, `Type`, `Name`, `Path` (folder), `Hidden`, `BlendMode`.

```xml
<Workspace>
  <Creator>...</Creator>
  <Engine>
    <InputOutputMap>...</InputOutputMap>
    <Fixture>...</Fixture>
    <FixtureGroup>...</FixtureGroup>
    <ChannelsGroup>...</ChannelsGroup>
    <Function ID="N" Type="Scene" Name="...">
      <Tempo Type="Time"/>
      <Speed FadeIn="0" FadeOut="0" Duration="0"/>
      <ChannelGroupsValues>id,level,id,level</ChannelGroupsValues>
      <FixtureVal ID="fixture_id">ch_num,value,ch_num,value,...</FixtureVal>
      <FixtureGroup ID="0"/>
      <Palette ID="0"/>
    </Function>
    <Function ID="N" Type="Chaser" Name="...">
      <Tempo Type="Time"/>
      <Speed FadeIn="0" FadeOut="0" Duration="0"/>
      <Direction>Forward</Direction>
      <RunOrder>SingleShot</RunOrder>
      <SpeedModes FadeIn="Default" FadeOut="Default" Duration="PerStep"/>
      <Step Number="0" FadeIn="0" Hold="8000" FadeOut="0" Note="">scene_id</Step>
    </Function>
    <Function ID="N" Type="Collection" Name="...">
      <Step Number="0">function_id</Step>
      <Step Number="1">function_id</Step>
    </Function>
    <Function ID="N" Type="EFX" Name="...">
      <PropagationMode>Parallel</PropagationMode>
      <Algorithm>Circle</Algorithm>
      <Width>127</Width>
      <Height>127</Height>
      <Rotation>0</Rotation>
      <Speed FadeIn="0" FadeOut="0" Duration="20000"/>
      <Direction>Forward</Direction>
      <RunOrder>Loop</RunOrder>
      <Fixture>...</Fixture>
    </Function>
  </Engine>
  <VirtualConsole>...</VirtualConsole>
</Workspace>
```

### Scene FixtureVal format

`<FixtureVal ID="fixture_id">channel,value,channel,value,...</FixtureVal>`

- Channel numbers are 0-indexed relative to the fixture
- Values are 0-255
- Empty FixtureVal (`<FixtureVal ID="3"/>`) means fixture participates but no values set

### Chaser Step format

`<Step Number="N" FadeIn="ms" Hold="ms" FadeOut="ms" Note="">function_id</Step>`

- Times in milliseconds
- function_id references the Function ID of the step content
- Tempo Type="Beats": values are multiples of 1000 (1000 = 1 beat)

---

## Virtual Console

### Button

Triggers a function. Modes: toggle, flash (held), blackout.

### Slider

Direct channel control or playback fader (controls HTP intensity of a running function).

### Cue List

- Can ONLY be assigned a Chaser
- Steps through with Next/Previous/Play/Stop
- Crossfade fader for manual transitions
- Set step duration to infinite (∞) for manual crossfade control
- Duration=0 causes frantic looping — always set explicit durations

### Solo Frame

Only ONE button active at a time. Starting a new function stops the previous. Essential for mutually exclusive looks.

### Frame

Visual grouping. Can collapse. No exclusive behavior.

---

## Fixture Definitions

Fixtures are defined in XML (.qxf) with:

- Manufacturer, Model, Type
- Physical properties (bulb, beam angle, dimensions)
- Channels with groups (Intensity, Pan, Tilt, Gobo, Color, Speed)
- Capabilities per channel (value ranges → names/functions)
- Modes (different channel configurations for the same fixture)

### Fixture Modes

Many fixtures have multiple modes (e.g., 8-bit vs 16-bit pan/tilt). Each mode defines which channels are active and their order.

### Heads

A head is an individual light output in a multi-head fixture (e.g., LED bar with 4 RGB segments). Heads share some properties but can be controlled individually.

---

## Input/Output & Plugins

QLC+ supports:

- **Art-Net** (network DMX)
- **E1.31/sACN** (network DMX)
- **DMX USB** (Enttec, FTDI adapters)
- **MIDI** (input/output for controllers)
- **OSC** (Open Sound Control)
- **HID** (joysticks, gamepads)
- **Loopback** (internal routing)

Input profiles map physical controller knobs/faders to QLC+ controls.

---

## Common Pitfalls

1. **Hold="0" on sub-chaser steps** — causes them to be skipped entirely
2. **SpeedModes FadeIn="PerStep" on main song chaser** — causes timing drift vs "Default"
3. **Fade Out expects LTP channels to fade** — they don't; only HTP channels fade
4. **Two scenes setting same LTP channel** — last-started wins, can cause unexpected jumps
5. **Cue List with duration=0 steps** — frantic looping with no visible result
6. **Forgetting channel enable/disable** — a scene with a channel disabled will never touch that channel, even if it looks like it should
7. **HTP conflicts when layering monolithic scenes** — if both scenes set intensity, highest wins, not the "current" one

## Timing Calculations

When programming song chasers, always verify:

- `sum(all step Hold values) == song duration` (for SingleShot chasers)
- Sub-chaser Hold = sum of sub-chaser's own steps' total durations
- FadeIn happens WITHIN the Hold window (with Default SpeedMode), not in addition to it

## Best Practices

- Use `SpeedModes FadeIn="Default" FadeOut="Default" Duration="PerStep"` for song chasers
- Give sub-chaser steps explicit Hold values matching their total runtime
- Never hold a single static scene longer than ~12s in performance
- Create front/rear intensity contrast for depth (not symmetric values)
- Use Solo Frames to prevent accidental scene stacking
- Keep strobe for punctuation, not atmosphere
