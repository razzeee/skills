# Troubleshooting workspace playback

## A fixture goes dark unexpectedly

Check that the fixture's universe, address and mode match the workspace patch. At the dark step, inspect the dimmer, colour and shutter values in the DMX Monitor. A scene can set a colour while the fixture's dimmer is at zero. If the output differs from the scene, check other active functions, the Grand Master and the console's Blackout control before editing it.

For a recording, find the cue on a timestamped contact sheet. Watch a short clip of the transition to judge the fade. A frozen picture can have regular frame timestamps, so compare frames where the DMX output predicts a visible change. If only stills are reliable, describe what they show and leave motion unverified. If the DMX output matches the cue but the stage is dark, check where the beam lands on the rig.

## Change one use of a shared function

Find the occurrence on the timeline or by its position in the manual cue list. Follow the function references to the scene. If two placements share a chaser, copy the chaser for the target placement and point that placement to the copy. Copy the scene too. In the copied chaser, change only the target `<Step>`'s function ID to the new scene ID. Keep its step number and timing attributes.

Compare the original and edited DMX output before the transition, through the fade and into the next cue. Check that the other uses of the scene still match. For a `Random` chaser, replay the recorded step order and timing. If you cannot reproduce the run, compare only matching transitions.

## Rebuild a generated workspace

Keep the original `.qxw` for comparison. Confirm which generator revision and inputs produced it before editing the source. Rebuild in a separate directory. Compare fixture mappings, DMX output and cue times even if the XML formatting differs. For synced playback, check that both workspaces use the same audio file and offset.
