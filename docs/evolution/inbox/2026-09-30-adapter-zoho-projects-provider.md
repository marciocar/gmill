---
title: 'Provider zoho-projects no Task Manager (SDAAL): adapter + integração nascidos no hub, pedido de promoção'
date: 2026-09-30
from: gmill (hub, role hub, pin cff9214c3b9a)
to: core (onion-evolve)
type: feature
severity: medium
flow: upstream
---

# Provider `zoho-projects` para o Task Manager — pedido de promoção ao core

## Por que chega aqui

A GMill usa o **Zoho Projects** online. O hub implementou o provider `zoho-projects` no SDAAL do Task
Manager (Linear SAC-58/59/60/61). O código vive em `.claude/utils/task-manager/`, que é **superfície
vendorizada**. Enquanto o core não absorver, o próximo `/meta:adopt --update` deste hub vai conflitar
nesses arquivos. O pedido é **promover o provider ao core**, para que ele volte pelo vendor.

## O que o hub fez (branch `feature/zoho-projects-provider` do `gmill`)

| Arquivo | Mudança |
|---|---|
| `.claude/utils/task-manager/adapters/zoho-projects.md` | **novo**: adapter completo sobre a API V3 |
| `.claude/utils/task-manager/detector.md` | config + transporte sempre `api` + regras de id **antes** de ClickUp/Asana/Jira |
| `.claude/utils/task-manager/{factory,types,interface,README}.md` | case, união `TaskManagerProvider`, `STATUS_MAPPING`, tabelas |
| `.claude/commands/common/prompts/task-manager-provider-detection.md` | valor + linha na tabela de variáveis (passo 0 dos comandos) |
| `.claude/skills/onion-validation/SKILL.md`, `.claude/commands/engineer/pr-update.md`, `.claude/commands/meta/setup-integration.md` | enumeração de providers / guia de setup |

**Decisões de mapeamento** (ADR local do hub, aceito): `docs/technical-context/decisions/adr-zoho-projects-task-mapping.md`.
Na promoção, o ADR precisa ser reescrito no formato `onion-adr-*`. Resumo:

- **API V3** (`/api/v3/`). users, phases e issues já estão em **`/api/v3.1/`**, então o prefixo de versão é **por endpoint**.
- **Toda rota de task exige `project_id`**, e não há rota de task pelo portal. Por isso o `taskId` da interface é o **id composto `<project_id>.<task_id>`**.
- **Detector:** `^\d+\.\d+$` → `zoho-projects` (nenhum outro provider usa ponto), mais o desempate `^\d{5,}$` pelo provider configurado. Os ids do Zoho têm **13 ou 19 dígitos** e colidem com ClickUp, Jira e Asana.
- **Status** customizáveis por portal: resolvidos por nome normalizado, com fallback em `is_closed_type`.
- **Prioridade** tem 4 níveis: `urgent→high` com perda registrada; `none→undefined`.

## Fatos MEDIDOS que valem para qualquer integração Zoho (não estão na doc)

1. **O endpoint de token responde HTTP 200 NA FALHA**: `POST /oauth/v2/token` com credencial inválida devolve
   `200` + `{"error":"invalid_client"}`. Adapter que confia no status HTTP aceita token inexistente.
2. **Os dois 401 são distintos:** `INVALID_OAUTHTOKEN` (renovar uma vez) × `INVALID_TICKET` (header ausente,
   que é bug e não deve renovar, senão entra em loop).
3. **O erro tem dois formatos:** `{error:{title,status_code,details}}` e `{error:{code,message}}`.

## Como foi validado

- **Dogfood por execução:** os blocos TypeScript do próprio `.md` foram extraídos e rodados (Node 22,
  `--experimental-transform-types`). O resultado foi **39/39**, incluindo a autenticação real contra os
  endpoints públicos com credencial falsa. O **teste de mutação** pegou os 2 bugs injetados (confiar no
  HTTP 200; retry sem guarda → loop).
- **Regressão do detector:** uma matriz de 7 providers × 13 ids, extraída do `detector.md` da `main` e da
  branch. **Nenhuma linha** de Linear, ClickUp, Asana, Jira ou `none` mudou para ids não compostos.
- **NÃO validado:** o caminho feliz contra um portal real, porque faltam as credenciais da GMill (P1–P6 do ADR).

## O que o hub deliberadamente NÃO editou (fica para a promoção)

Enumerações **ilustrativas** de providers em prosa, como "(ClickUp, Asana, Linear)", em cerca de 20
arquivos do core (`warm-up.md`, `product/task.md`, `product/feature.md`, `engineer/start.md`,
`engineer/hotfix.md`, `agents/meta/onion.md`, `skills/onion/SKILL.md`, KBs
`task-manager-abstraction`/`configuration-management`/`framework-story-points`/`onion-framework-identity`,
`docs/sdaal/sdaal.md`…). Editá-las no hub só aumentaria o conflito do `--update`. No core, basta um `grep`
por `linear | none` e `Asana, Linear`.

## Sinal lateral (achado de campo, baixo)

O hook `bash-empty-result-guard.sh` avisa `PR-SEM-PASSADA-ADVERSARIAL` ao abrir PR num repo **hub**, mas a
REGRA 56 (PR aberto carrega RESÍDUO da passada adversarial) **isenta** hub: `review-artifact-check.sh` saiu
rc=0 com `skip repo-derivado(role-adopted/hub)` no PR #2 do `gmill`. O hook não consulta o `role:` do
stamp. Guarda que grita no inócuo ensina a ser ignorada.

## Pedido

1. Promover o adapter e a integração ao core (e reescrever o ADR como `onion-adr-*`).
2. Atualizar as enumerações em prosa listadas acima.
3. Fazer o hook consultar o `role:` antes do aviso `PR-SEM-PASSADA-ADVERSARIAL`.
