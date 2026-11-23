# Skill maintenance

When fixing an observed mistake in how an agent selects or follows a skill:

1. Capture the triggering request and any necessary input in the skill's
   `evals/evals.json`. Reuse an existing case if it already covers the mistake.
   Keep the expected behavior separate from the prompt. For an example, see
   [the GLib evals](glib-c-conventions/evals/evals.json).
2. Run the case in fresh sessions with the original and revised instructions.
   Keep the prompt, inputs, model, and tools the same. For a selection mistake,
   let the agent choose the skill in both runs.
3. Compare the observed actions or outputs against the same expectation.
   Report what failed before and what happened after. If the original run
   didn't reproduce the mistake or either run was blocked, say so.
