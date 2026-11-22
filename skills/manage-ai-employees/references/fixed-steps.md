# Fixed steps

## Tools that run before a Dozal starts

A Dozal's saved tool can be marked as a fixed step: on every new run the platform runs it before the Dozal starts, with the run's inputs, and hands the Dozal its output to check. The Dozal stays in charge: it uses a result that looks right and does the step itself when the tool failed or its result looks wrong. Use it for the lookups a Dozal repeats on every run (find the customer's record, read the last messages), never for a step that changes anything. Needs **Admin**.

```sh
dozaldevs get-dozal-workspace "$DOZAL"                                          # the tool ids
dozaldevs set-dozal-workspace-tool-fixed-step "$DOZAL" "$TOOL" --data fixed:=true --data order:=1
dozaldevs set-dozal-workspace-tool-fixed-step "$DOZAL" "$TOOL" --data fixed:=false   # optional again
```

Steps run in `order`. A tool that is set aside or not yet saved for good doesn't run; the Dozal is told and does that step itself. A person decides this, never a run: when a Dozal or a check suggests a tool, show the human what the tool does and ask first.
