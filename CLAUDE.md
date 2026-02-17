# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


## Work Methodology
- Create an implementation plan and write it to docs/ before starting work on a feature or a fix
- Create a validation checklist and write it to docs/ before starting work on a feature or fix
- Use the validation plan to guide the implementation. Cross off validation items as you complete them.
- Update progress.md as you work, write down in words what you're working on now and what you just did.
- Work according to the TDD methodology: create unit tests and (relatively few) Playwright tests, before you begin implementing.
- Use the tests to verify your work after completing each item on the validation list.
- After completing planning, don't immediately proceed to implementation. Stop, let the user review the plan before asking to implement.
- Document major decisions in `docs/decisions.md`
- If you notice any inconsistency or outdated information in `CLAUDE.md`, update `CLAUDE.md` as part of the same task.