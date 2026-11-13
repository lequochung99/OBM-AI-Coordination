# Agent Availability and Assignment

Update this file only when availability or assignment policy changes materially.

| Agent | Current role | Availability | Notes |
|---|---|---|---|
| ChatGPT | Coordinator, task author, reviewer | Available | Reads GitHub coordination artifacts and prepares complete task contracts |
| Cursor | Primary AI Engineer | Available | Investigation, implementation, focused validation, commit, and report |
| Codex | Hard implementation / independent reviewer | Temporarily unavailable | Rejoin the same GitHub workflow when quota is restored |
| Operator | UI and physical runtime acceptance | Available | Final visual and physical acceptance owner |
| DeepSeek/other | Optional investigation-only | Not assigned | Use only when clear net value is proven |

## Assignment rule

The active task must identify exactly one primary executor. A second agent is a reviewer, not a duplicate executor, unless the operator explicitly resets the task after failure.
