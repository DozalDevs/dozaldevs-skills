# Follow-up work of a finished run

## A finished run's report or summary never arrived

A run can finish its work while what comes after it, its Slack report and its one-line summary on the run list, never arrives, because the platform gave up on that follow-up work. When the human says a finished run's report or summary is missing, run `rerun-end-of-run-work` with the run's task id. It tries only the follow-up work the platform gave up on, so it never sends the run's work twice. A `404` means nothing was given up: say so, and read the run's logs instead of calling it again.
