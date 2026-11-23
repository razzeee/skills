---
name: qlcplus
description: "Helps with QLC+ workspace files (.qxw), scenes, chasers, sequences, collections, EFX, RGB matrices, fixture definitions (.qxf), Virtual Console, and timing calculations. Use for creating or editing QLC+ shows, debugging timing and crossfades, generating fixture definitions, configuring plugins, repairing corrupt workspaces, or troubleshooting HTP and LTP conflicts. Also trigger for SpeedModes, FadeIn, Hold, Duration, FixtureVal, RunOrder, and QLC+ function types. Do not trigger for general DMX hardware questions without QLC+ context, fixture buying advice, DAW-only questions, or custom protocol implementations."
compatibility: Targets QLC+ 5 workspace and fixture formats. Playback and visual verification require QLC+; static XML editing does not require the application.
metadata:
  author: razzeee
  source: https://github.com/razzeee/skills/tree/main/qlcplus
---

# QLC+ Lighting Programming Skill

You are helping with QLC+ (Q Light Controller Plus) v5 lighting automation. This covers everything from fixture setup through show programming.

Confirm the target QLC+ version from the workspace's Creator metadata or the
user's environment before applying version-specific features. Preserve the
file's format when editing an existing workspace; verify v4 behavior against
that version's documentation or source instead of assuming v5 behavior.

## Key Concepts

### DMX Fundamentals

- 512 channels per universe, values 0-255
- QLC+ supports unlimited universes (4 default)
- Fixtures are patched at a universe + start address

### HTP vs LTP (Critical for understanding conflicts)

| Rule                           | Default channel types                             | Behavior                            |
| ------------------------------ | ------------------------------------------------- | ----------------------------------- |
| HTP (Highest Takes Precedence) | Intensity, dimmer, color intensity (R/G/B/W/A)    | Highest active value wins           |
| LTP (Latest Takes Precedence)  | Pan, tilt, gobo, strobe, speed, all non-intensity | Most recently started function wins |

Check the patched channel for a forced HTP or LTP setting before diagnosing a conflict or blackout. The override takes precedence over the fixture definition's channel group. For example, if a dimmer is forced to LTP, a later scene can lower it even while a brighter scene is active.

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
- **Fade In**: Fade-capable channels transition to their targets. If a channel snaps despite a nonzero fade, check its fade capability in the fixture profile.
- **Fade Out**: Time to fade HTP channels to zero. Fade Out does not affect LTP channels, which retain their values.
- **No duration**: Scenes persist until stopped or overridden
- Monolithic scenes (all channels enabled) work well for chaser-driven workflows where only one scene is active at a time
- Partial scenes (only some channels) enable layering but require careful management
- **Crossfade blending**: On channels shared by consecutive scenes, the previous scene supplies the starting value. The channel's fade capability still determines whether it transitions or snaps.

### Chaser

Runs steps sequentially. Each step is a Function (usually a Scene).

**The timing model:** For a finite XML step with `Hold`, the stored duration is the step's `FadeIn + Hold`. `FadeOut` can overlap the next step and does not add to that duration.

**SpeedModes:** Read `Duration`, `FadeIn` and `FadeOut` independently. Duration determines when the chaser advances; the fade modes determine how the child transitions. An explicit speed override from a parent takes precedence for that setting.

| Mode | Duration | FadeIn or FadeOut |
| --- | --- | --- |
| `Default` | Chaser's duration | Child function's own fade |
| `Common` | Chaser's duration | Chaser's fade |
| `PerStep` | Stored step duration | Step's fade |

**Practical implications:** With `Duration="PerStep"` and `FadeIn="Default"`, `<Step FadeIn="500" Hold="1500" ...>` occupies 2000 ms in time mode even though the child supplies the actual fade. With `Hold="0"` the same step still occupies 500 ms.

**Run orders:** Loop, SingleShot, PingPong, Random
**Direction:** Forward, Backward

**Beat tempo:** With `<Tempo Type="Beats"/>`, 1000 timing units represent one beat. A duration of 500 is half a beat; 4000 is four beats.

**Infinite speed** (`Function::infiniteSpeed()`): Step never auto-advances. Used for manual cue lists. The runner checks: `if (duration != infiniteSpeed && elapsed >= duration)`.

### Sequence

A sequence is a chaser bound to one parent scene. Every step controls the same channels, and the sequence appears under its parent scene in the Function Manager.

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

- Tracks hold scheduled functions. A track can omit `SceneID`; a Sequence requires a parent Scene and controls its channels.
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

Runs commands in QLC+'s own scripting language, such as starting a function or waiting before the next command. ECMAScript is used for RGB Matrix scripts.

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

`<Step Number="N" FadeIn="0" Hold="1000" FadeOut="0" Note="">function_id</Step>`

- Times in milliseconds for time-based playback, or in thousandths of a beat for beat-based playback
- function_id references the Function ID of the step content

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
- A zero-duration step can advance before its look is visible. At the last step, `SingleShot` ends and `Loop` wraps.

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

1. **Expecting Fade Out to affect LTP channels:** Only HTP channels fade out
2. **Two scenes setting the same LTP channel:** The scene started last wins, which can cause unexpected jumps
3. **Forgetting channel enablement:** A scene never changes a disabled channel
4. **Layering monolithic scenes:** When both scenes set intensity, the highest HTP value wins, not the value from the current scene

## Timing Calculations

- For a timed SingleShot chaser, sum its effective step durations under the active SpeedModes. On a Show track, add the placement offset to locate a step on the timeline.
- For nested chasers, check when the parent advances: the time it gives the child need not match the child's standalone runtime. Account for any overlapping fade or separately scheduled tail.
- When a step behaves differently in playback, see [Troubleshooting workspace playback](references/playback-and-rebuilds.md).

## Best Practices

- Choose hold length for the cue's purpose. A sustained look can give the next change emphasis.
- Create front/rear intensity contrast for depth (not symmetric values)
- Use Solo Frames to prevent accidental scene stacking
- Keep strobe for punctuation, not atmosphere
