# Playback evidence and reproducible rebuilds

## Locate the occurrence

Record the QLC+ version, workspace revision, active take and renderer. Match the workspace patch (universe and address) and fixture profile to the mode actually selected on the fixture. For synced playback, identify the audio file and align its clock with the show offset.

Trace the placement through nested chasers to the active scene, accounting for SpeedModes, holds, fades, direction, repeats and shared references. Include overlapping functions and separately scheduled tails. If a section label is missing, inspect effective DMX through the intended end, including the audio end when applicable. A missing navigation marker may need a metadata edit even when the lighting plays correctly.

## Choose evidence for the question

- **Where should I look?** Start with offline contact sheets for the whole run and individual sections, including the opening, quiet passages, transitions and ending. Plot both mean and peak output per time bucket: the mean hides brief accents; the peak can make an entire bucket look bright. Check that the preview's lens display does not saturate and hide level changes.
- **What happened here?** Inspect audience-view frames before, during and after the event. After seeking, wait for the preview to settle and compare its DMX values with the expected time bucket. Label the actual output and any experimental A/B controls separately; keep the rig, camera and exposure fixed between takes.
- **Did it move smoothly?** Record a short clip covering entry, hold and release. Use a full capture when the question requires continuous coverage. Check the saved file has data, the expected streams and duration, and no unexplained gaps in encoded frame timestamps. A frozen image can still have regular timestamps: compare decoded frames where the cue should visibly change, then watch the transition for smoothness. Identical frames during a static hold are expected. Warming up a browser renderer may help startup stalls; apply the same checks on retry. If recording still fails, use source-checked paused frames for static looks and report motion as unverified. After an interruption, keep verified captures and retry only missing or failed items.
- **What does the evidence establish?** Identify the workspace, take, renderer, rig profile, audio (if used) and timestamp for each observation. Distinguish what you saw, measured, heard or checked on the rig. Stills sample moments; waveform data and the presence of an audio stream do not establish that someone listened.

## Interpret the look before changing it

For an apparent blackout, trace the source scenes, overlapping controls and console Blackout state. Inspect colour, master dimmer and shutter or gate together, then follow the relight. In a music-synced cue, listen across the rest, lyric or transition: the same dark interval can frame a deliberate pause or obscure a performer. Brightness and audio-correlation scores locate candidates, not intent.

Preserve the character of each piece, including purposeful quiet passages and sustained endings. Choose hold lengths by purpose. When following an instrument, use measured attacks and envelopes rather than imposing periodic swells, flashes or blackouts. Treat a mixed stem as a timing proxy: it does not isolate an instrument or transcribe a riff.

To clarify an exchange between sides, try lowering the weaker side on each accent before raising the peaks. Check that hues, timing, wide chorus or finale looks, rear accents and continuity at cue boundaries survive. For a local dark dip, a floor on the existing dimmer envelope can preserve its accent and hue better than a flat replacement. Compare effective channel and colour samples across the dip, fade and release, and check repeated occurrences, section starts and the tail against the original.

Judge illumination on performers as well as fixture lenses. Mounting, aim, performer positions, camera exposure and ambient light affect the result. A simulated beam reaching centre stage is a prediction; confirm centre visibility from the audience position in rehearsal.

## Rebuild a generated workspace

1. Preserve the original workspace, exports and review evidence. Parse the actual workspace strictly as the playback baseline. A snapshot establishes what was exported; it does not establish which generator or inputs produced it. If a manifest fails validation, report the mismatch rather than bypassing it or claiming the current generator reproduced the export.
2. Identify the generator revision, configuration and input files. If the generator uses audio, verify that each required mix or stem is materialized and hash its contents; matching names or pointer files are insufficient. Build dependencies first, then regenerate into an empty, isolated destination. Keep bulky generated outputs out of source control unless the project tracks them.
3. Align the runs by actual step order, trigger timing and timebase as described in [Review and revise a cue](../SKILL.md#review-and-revise-a-cue). Compare effective output arrays, fixture mappings and profiles, cue timings and tails. For synced playback, also compare audio identities. XML formatting differences alone do not imply different playback; matching DMX alone does not establish matching audio or coverage. For a requested cue edit, identify which differences are intended and verify the rest within the matched coverage.
4. For a local rebuild, finish by reporting the comparison. If deployment is requested, publish a rehearsal candidate separately from production. When using a served preview, verify that it loads the candidate workspace, fixture profile and any associated audio before collecting evidence. Check the result on the physical rig before promotion.
