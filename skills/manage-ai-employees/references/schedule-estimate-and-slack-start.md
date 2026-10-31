# Schedule estimates and Slack tag starts

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## What a schedule costs

`get-dozal-schedule` also answers `schedule_estimate`: runs a day and what that costs a day, from the employee's own recent runs. Tell the human in plain words ("about 6 runs a day, around $1.20 a day").

## A schedule above 24 runs a day

A schedule that runs more than once an hour (over 24 runs a day) needs the human's yes. If the change comes back `422` `FREQUENT_SCHEDULE_NEEDS_CONFIRM`, show them the `schedule_estimate` it carries, and only when they agree send the same change again with `--data confirm_frequent_schedule:=true`. Never add it up front. A change that leaves the schedule alone, like a rename, is never asked.

## Starting from a Slack tag

An employee can also start when someone tags the Slack app in a channel. Turn it on by setting `trigger_sources` with `slack_mention.channel_id`, and off by saving `trigger_sources` again without that key (calls in `../../operate-ai-employees/references/customer-api.md`). Read the answer's `warnings`: if the app isn't in the channel, the human has to add it (`/invite @<the app>` for a private one) before anything starts. Anyone in that channel can start it, so name the channel.
