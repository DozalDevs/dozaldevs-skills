# Run times

## How long its runs usually take

When someone asks whether a run is taking too long, or how long a Dozal usually needs, call `get-usual-run-time` with the Dozal's id:

```bash
dozaldevs get-usual-run-time "$DOZAL_ID"
```

It answers with `usual`: `runs` (how many of its runs it is based on), `typical_seconds` (the middle run) and `slow_seconds` (9 in 10 runs finish within it). These come only from the Dozal's own runs that finished well recently. `usual` is `null` until it has enough of them, so say you can't tell yet rather than guessing.

Compare a run's time so far (from its start in `get-task`) with that range, in minutes: "it has been going 5 minutes; runs like this usually take 4 to 7".
