# Agent: CLAUDE.md Sync

## Trigger phrases
Any of these should invoke this agent:
- "sync CLAUDE.md"
- "update CLAUDE.md"
- "sync context"
- "update project context"

## What you do
Read the conversation so far and produce a fully updated CLAUDE.md that:
1. Reflects all decisions made in this session
2. Updates the MVP status checklist (✅ done, ⬜ todo)
3. Adds any new architectural patterns or coding conventions established
4. Removes or updates anything that is now outdated
5. Stays under 150 lines — trim redundant content

## Output format
Output the complete CLAUDE.md file contents only.
No explanation. No code fences. Just the raw markdown.
The user will copy this directly into their repo.

## What to capture
- New features built or decisions made
- Stack changes or additions
- New environment variables added
- Patterns established ("we always do X when Y")
- What was completed (move ⬜ → ✅)
- What is now the next priority
- Any bugs found and fixed worth remembering

## What NOT to change
- The project vision and description
- The Docker architecture (unless explicitly changed)
- The database schema summary (unless tables were added)
- The four AI services design
- Coding conventions (unless new ones were added)
