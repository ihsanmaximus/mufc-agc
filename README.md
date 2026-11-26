# MUFC AGC — Project Workspace

This repository contains the MUFC Global Portal and its future AGC Engine.

## Project decisions
- Audience: global Manchester United fans
- Public language: English
- Design direction: minimalist editorial portal
- Brand: not selected
- Domain: deferred
- Long-term publishing target: autonomous publishing with mandatory quality gates
- Current implementation phase: Phase 1 — public portal foundation and manual CMS

## Documentation and agent skills
- `docs/`: architecture, AGC technology, implementation specification, admin workflows, data/security, AI boundaries, and QA.
- `.agents/skills/mufc-portal/SKILL.md`: project-specific architecture, scope, and implementation rules.
- `.agents/skills/absolute-ui/SKILL.md`: UI/UX design and frontend visual implementation guidance.
- `.agents/skills/absolute-ui/references/`: detailed UI guidance by topic.

For UI/frontend tasks, agents must use both skills. Project-specific decisions and `mufc-portal` remain authoritative; `absolute-ui` is a design guide and cannot change the approved stack or scope.

## Phase 1 boundary
Do not implement RSS collection, AI article generation, or automatic publishing in Phase 1.
