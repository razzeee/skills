# Troubleshooting workspace playback

## A fixture goes dark unexpectedly

Match the fixture's universe, address and mode to the workspace patch. At the affected step, check the effective output from all active functions, including the master dimmer, colour channels and shutter. A scene can specify a colour while the dimmer is at zero. Check the Grand Master and console Blackout state before editing the scene.

If the symptom appears in a recording, a timestamped contact sheet can locate the cue. Use a short clip spanning the transition to judge motion. Compare decoded frames with the DMX changes you expect: a frozen preview can have regular frame timestamps. If only stills are reliable, report the sampled look and leave motion unverified. If DMX matches the cue but coverage is in doubt, check fixture aim and illumination on the rig.

## Change one use of a shared function

Locate the occurrence by timeline placement or manual trigger order. Follow the references down to the scene. If two placements reuse the same chaser, copying the scene alone is insufficient: give the target placement its own chaser, then copy and relink the child whose values need to change.

Compare effective output before, during and after the edited transition, including the release into the next cue. Verify that other occurrences still match. For a Random chaser, reproduce the actual step order and timing or limit the comparison to matching transitions.

## When the workspace is generated

Keep the original `.qxw` as the baseline. Identify the authoring source and its inputs before editing; an exported workspace alone does not identify the generator revision. Rebuild into a separate destination, then compare fixture mappings, effective DMX and cue timing. For synced playback, also verify the audio identity and offset. Differences in XML formatting alone do not establish a change in playback.
