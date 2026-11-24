# What each job needs, and where to read it

Generated from the platform's own list of jobs. Do not edit by hand.
Find the job, load the skill, read the section it names. Every command runs as `dozaldevs` followed by the command and its ids.
A command a job does not list yet is still callable: `dozaldevs help` lists them all.

| Job | Skill | Section | Commands |
| --- | --- | --- | --- |
| Hire a Dozal by chatting with the Hiring Dozal | `hire-ai-employee` | SKILL.md, "How the chat works" | `get-hiring-chat` `send-hiring-message` `start-hiring-chat` |
| Change what a Dozal does by chatting in an edit chat | `manage-ai-employees` | SKILL.md, "Pause, reactivate, rename, edit" | `get-open-hiring-change-chat` `start-hiring-chat` |
| Ask the assistant a question | `operate-ai-employees` | SKILL.md, "7. Routing: pick the right job skill" | none yet |
| Change a website by chatting with the assistant | `manage-ai-employees` | references/site-drafts.md, "Live drafts of a website" | none yet |
| Call a command and read one run or the list of Dozals | `operate-ai-employees` | SKILL.md, "3. How to call" | `get-task` `list-dozals` |
| Check which apps are connected after a person approves a connect | `operate-ai-employees` | SKILL.md, "6. What only a person can do" | `list-app-connections` |
| Read or change something in a connected app without a Dozal | `operate-ai-employees` | SKILL.md, "7. Routing: pick the right job skill" | `run-app-action` |
| Read what a photo, PDF, spreadsheet or text file says | `operate-ai-employees` | references/customer-api.md, "Reading a photo or a file" | `hand-over-file` `read-file` |
| Get a website's saved login from the organization's connected 1Password | `operate-ai-employees` | references/customer-api.md, "Signing in with a login saved in 1Password" | `get-saved-login` |
| Make a picture (a flyer, a post, an illustration) without a Dozal | `operate-ai-employees` | SKILL.md, "3. How to call" | `make-picture` |
| Send DozalDevs feedback about a failed call, with the person's yes | `operate-ai-employees` | references/2026-10-31-2230-feedback.md, "When something goes wrong: send feedback" | `check-feedback-for-dozal` `confirm-feedback-item` `get-feedback-item` `preview-feedback-item` |
| Decide a run that is waiting for an action | `operate-ai-employees` | references/customer-api.md, "Runs take time" | `decide-wait-action` `get-wait` |
| Find and call a connected app's actions directly | `operate-ai-employees` | references/customer-api.md, "Using your connected apps directly" | `get-app-action` `list-app-actions` `list-app-calls` `search-actions` |
| List the executions of a run | `operate-ai-employees` | references/customer-api.md, "The run's logs" | `list-executions` |
| Make a chart, diagram or document into a picture, PDF or Word file | `operate-ai-employees` | references/customer-api.md, "Charts, diagrams and documents as pictures and files" | `download-assistant-chat-card` `make-card-file` `report-assistant-chat-card-render-failure` |
| Share a page at a link anyone can open | `operate-ai-employees` | references/customer-api.md, "Sharing a page at a public link" | `publish-page` `update-page` |
| Change a Dozal's settings (such as the Slack tag that starts it) | `operate-ai-employees` | references/customer-api.md, "Starting a run from a Slack tag" | `update-dozal` |
| Check the held change, then hire from the chat or throw it away | `hire-ai-employee` | SKILL.md, "Hiring the Dozal" | `discard-hiring-chat` `get-wait-dozal-change` `hire-from-hiring-chat` `submit-wait-decisions` |
| List the abilities a Dozal can have | `hire-ai-employee` | SKILL.md, "What a Dozal can do" | `list-dozal-abilities` |
| Restart a hiring chat that refuses a send | `hire-ai-employee` | SKILL.md, "When a send is refused" | `restart-hiring-chat` |
| Run a Dozal (dry run first) | `run-ai-employee` | SKILL.md, "Check first with a dry run" | `run-dozal` |
| Save, preview and apply an organization's setup | `manage-ai-employees` | SKILL.md, "Saved setups" | `apply-latest-org-snapshot` `apply-org-snapshot` `delete-org-snapshot` `get-org-snapshot` `get-org-snapshot-apply` `list-org-snapshots` `preview-org-snapshot-apply` `save-org-snapshot` |
| Hand one Dozal's work to another | `manage-ai-employees` | SKILL.md, "Handoffs" | `create-handoff` `delete-handoff` `get-handoff-graph` `list-dozal-handoffs` `update-handoff` |
| Copy a Dozal's code to this computer with git | `manage-ai-employees` | references/copying-code.md, "Copying a Dozal's code" | `create-code-access` |
| Name and follow a change your own AI proposed | `manage-ai-employees` | references/copying-code.md, "Following your suggestion" | `get-code-proposal` `list-code-proposals` `update-code-proposal` |
| Remove a Dozal | `manage-ai-employees` | SKILL.md, "Removing an employee" | `delete-dozal` |
| Read what runs cost and the bill | `manage-ai-employees` | SKILL.md, "Usage and bills" | `get-billing` `get-cost` `get-run-cost` `list-run-costs` |
| Read, resume and run a Dozal's schedule | `manage-ai-employees` | SKILL.md, "Schedules" | `get-dozal-schedule` `list-schedules` `resume-dozal-schedule` `run-schedule-now` |
| Give a Dozal the tools of an app its steps need | `manage-ai-employees` | references/app-access.md, "Giving an employee an app it needs" | `give-dozal-app-access` |
| Connect Slack, GitHub and other apps | `manage-ai-employees` | SKILL.md, "Connections" | `list-integrations` `list-integrations-needing-attention` `start-app-connection` `start-github-install` `start-slack-install` |
| List members and invitations, and revoke an invitation | `manage-ai-employees` | SKILL.md, "Members and invitations" | `list-invitations` `list-members` `revoke-invitation` |
| List recent runs | `manage-ai-employees` | SKILL.md, "Review recent runs" | `list-tasks` |
| Open, read and close a website's live draft | `manage-ai-employees` | references/site-drafts.md, "Live drafts of a website" | `close-site-draft` `get-site-draft` `open-site-draft` |
| Start, open and stop an agent box | `manage-ai-employees` | references/agent-boxes.md, "Agent boxes" | `create-agent-box-link` `get-agent-box` `get-agent-box-availability` `list-agent-boxes` `start-agent-box` `stop-agent-box` `stop-agent-boxes` |
| Turn a Dozal's access to the organization's API on or off | `manage-ai-employees` | references/agent-boxes.md, "Organization access for a Dozal" | `set-dozal-org-access` |
| Save an edit chat and read what the save changed | `manage-ai-employees` | references/checking-and-undoing.md, "What a save changed" | `apply-hiring-chat` |
| List a Dozal's edits and revert one | `manage-ai-employees` | references/checking-and-undoing.md, "Undo a change" | `list-dozal-edits` `revert-dozal-edit` |
| See what teammates suggested for a Dozal, and approve or decline it | `manage-ai-employees` | references/checking-and-undoing.md, "Review what teammates suggested" | `get-dozal-looked-after-by` `list-dozal-team-suggestions` `decide-dozal-proposal` |
| See exactly what a Dozal is told | `manage-ai-employees` | references/checking-and-undoing.md, "See exactly what it's told" | `preview-dozal-brain` |
| See activity, deliverables, track record and what waits for approval | `monitor-and-review-ai-employee-work` | SKILL.md, "Activity and live status" | `count-tasks` `create-assistant-chat-browser-live-view-ticket` `create-run-browser-live-view-ticket` `get-assistant-chat-browser-status` `get-run-browser-status` `get-track-record` `list-deliverables` `list-recent-decisions` `list-silent-dozals` `list-task-pending-approvals` |
| See how long a Dozal's runs usually take | `monitor-and-review-ai-employee-work` | references/run-times.md, "How long its runs usually take" | `get-usual-run-time` |
| Approve or reject a run that waits for a person | `monitor-and-review-ai-employee-work` | SKILL.md, "Approve or reject (when awaiting_human)" | `decide-task` |
| Read a run's conversation, events and status log | `monitor-and-review-ai-employee-work` | SKILL.md, "Why a run went wrong: read the run's logs" | `get-task-conversation` `get-task-events` `get-task-status-log` |
| Read pages of a website with its saved sign-in | `operate-ai-employees` | references/customer-api.md, "Reading a website with its saved sign-in" | `read-browser-site-pages` `get-browser-site-read` `run-browser-keep-alive-visit` |
| Look, click and type on a website with its saved sign-in | `operate-ai-employees` | references/customer-api.md, "Browsing a website with its saved sign-in" | `open-browser-site-session` `take-browse-session-step` `end-browse-session` |
| Turn a short recording of speech into text | `operate-ai-employees` | references/customer-api.md, "Turning speech into text" | `transcribe-dictation` |
| Talk out loud with the chat: hear a spoken turn, speak a reply | `operate-ai-employees` | references/customer-api.md, "Talking out loud with the chat" | `get-live-voice` `hear-live-voice` `speak-live-voice` |
| Make a Dozal's saved tool run before it starts, or optional again | `manage-ai-employees` | references/fixed-steps.md, "Tools that run before a Dozal starts" | `set-dozal-workspace-tool-fixed-step` |
