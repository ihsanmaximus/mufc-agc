# Instructions for AI Coding Agents

Before any repository work, read:
1. `PROJECT_BRIEF.md`
2. `docs/01-architecture.md`
3. `docs/03-implementation-spec.md`
4. The relevant workflow, security, AI-boundary and QA documents
5. `.agents/skills/mufc-portal/SKILL.md`

For any task involving the public portal UI, admin UI, layout, styling, responsive behavior, components, typography, color, accessibility, or interaction design, also read:
- `.agents/skills/absolute-ui/SKILL.md`
- Only the relevant topic files under `.agents/skills/absolute-ui/references/` as needed.

## Instruction priority

1. Explicit owner decisions and approved project specifications in this repository are authoritative.
2. `mufc-portal` governs project architecture, scope, security, editorial rules, and implementation boundaries.
3. `absolute-ui` provides UI/UX and visual implementation guidance; it must not override the project's approved stack, architecture, brand decisions, content requirements, or scope.
4. If a generic skill recommendation conflicts with a project-specific decision, follow the project decision and document the conflict when material.

Treat these documents as the project contract. Implement only the requested phase/task. Inspect the existing repository before editing. Do not guess unresolved product decisions. Phase 1 is the public portal and manual CMS only; do not implement AGC collection, AI writing or auto-publishing unless the owner explicitly authorizes a later phase.

Follow the skills' applicable security, editorial, asset-rights, responsive-design, accessibility and completion-report requirements. If instructions conflict or a required decision is missing, ask one focused question before proceeding.
