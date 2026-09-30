---
title: Decisões técnicas (ADRs locais do hub GMill)
updated: 2026-09-30
---

# Decisões técnicas: ADRs locais

ADRs deste repo (hub GMill). Os ADRs **do framework Onion** vivem em
`docs/knowledge-base/decisions/` (`onion-adr-*`), vêm do core pelo vendor e não se editam aqui.

| ADR | Status | Escopo | Issue |
|---|---|---|---|
| [Mapeamento Zoho Projects V3 → ITaskManager](adr-zoho-projects-task-mapping.md) | aceito (2026-09-30) | adapter `zoho-projects` do Task Manager | SAC-59 (pai SAC-58) |

**Convenção:** `adr-<tema>.md` em kebab-case, com frontmatter `date`/`updated`/`status`. O `updated`
é o carimbo de frescor que o gate cobra em `docs/*-context/`. Marcações `[A VERIFICAR]`/`[INFERIDO]`
fazem parte do contrato: promova-as com fonte, nunca as apague.
