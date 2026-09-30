---
title: "ADR — Mapeamento Zoho Projects V3 → ITaskManager (adapter zoho-projects)"
date: 2026-09-30
updated: 2026-09-30
implemented_by: .claude/utils/task-manager/adapters/zoho-projects.md (SAC-60/61)
type: adr
status: aceito — ratificado pelo maestro em 2026-09-30 (após 2 passadas de revisão independente)
decision-scope: engenharia / task-manager SDAAL / adapter zoho-projects
issue: SAC-59 (spike) · pai SAC-58
consumers: SAC-60 (adapter) · SAC-61 (integração no framework)
related:
  - ../../knowledge-base/platforms/zoho-projects-api.md
  - ../../knowledge-base/patterns/glpi-zoho-ticket-to-task.md
  - ../../../.claude/utils/task-manager/interface.md
---

# ADR: mapeamento Zoho Projects V3 → `ITaskManager`

## Contexto

A SAC-58 cria o provider `zoho-projects` para o Task Manager do Onion. Antes de escrever o adapter
(SAC-60), este spike fecha quatro perguntas: **versão e cobertura da API**, **hierarquia**,
**formato de ID** e **status/prioridade**.

**Fonte única:** a doc oficial da Zoho Projects API V3, <https://projects.zoho.com/api-docs>,
baixada em 2026-09-30. Ela é uma página de 18,9 MB e foi parseada em 523 operações (título,
verbo, rota, escopo). As âncoras `#<id>` abaixo apontam para a seção da operação nessa página.

> ⚠️ **Limite desta verificação:** tudo aqui foi checado **contra a documentação**, não contra a API
> real. Faltam o Self Client, o DC e o `portal_id` da GMill. A validação real fica como pendência
> (ver [Pendências](#pendências)).

## Decisão 1: versão da API (item 1.4)

**A V3 é a versão vigente.** O prefixo `/api/v3/` domina a doc (10.630 ocorrências). O legado V2
(`/restapi/`) aparece só na seção de migração V2→V3 (11 ocorrências). Alguns módulos já estão em
**`/api/v3.1/`** (588 ocorrências): users, phases e issues/links. **Nenhum** dos endpoints usados
pelo mapeamento abaixo (tasks, comments, projects, global-statuses) está em v3.1.

**Decisão:** o adapter usa `/api/v3/` para tudo que está na tabela. Se precisar resolver usuários
(ex.: `assignee` por id), usa `/api/v3.1/portal/{portal_id}/users`. O prefixo de versão fica
**por endpoint** no adapter, nunca global.

## Decisão 2: cobertura dos métodos (item 1.4)

**Todos os 12 métodos do `ITaskManager` que dependem de API têm endpoint na V3.** Os outros dois
são locais.

Prefixo comum: `https://projects.zoho{dc}/api/v3/portal/{portal_id}`. O host vem do DC de accounts, **não** do `api_domain` do token (`www.zohoapis.*` responde 404 para o Projects; medido na implementação, SAC-60)

| Método | Verbo + rota | Escopo OAuth | Âncora |
|---|---|---|---|
| `createTask` | `POST /projects/{pid}/tasks` | `tasks.CREATE` | `#tasks-create_task` |
| `getTask` | `GET /projects/{pid}/tasks/{tid}` | `tasks.READ` | `#tasks-get_task_details` |
| `updateTask` | `PATCH /projects/{pid}/tasks/{tid}` | `tasks.UPDATE` | `#tasks-update_task` |
| `deleteTask` | `DELETE /projects/{pid}/tasks/{tid}` | `tasks.DELETE` | `#tasks-delete_task` |
| `createSubtask` | `POST /projects/{pid}/tasks` com `parental_info.parent_task_id` | `tasks.CREATE` | `#tasks-create_task` |
| `getSubtasks` | `GET /projects/{pid}/tasks` com `filter={"criteria":[{"field_name":"parent_task","criteria_condition":"is","value":["{tid}"]}],"pattern":"1"}` | `tasks.READ` | `#tasks-get_project_tasks` + seção "Task Filters" |
| `addComment` | `POST /projects/{pid}/tasks/{tid}/comments` | `tasks.CREATE` | `#tasks-task_comments-add_comment` |
| `getComments` | `GET /projects/{pid}/tasks/{tid}/comments` | `tasks.READ` | `#tasks-task_comments-get_comments` |
| `updateStatus` | `PATCH /projects/{pid}/tasks/{tid}` com `status.id`, resolvido por `GET /settings/global-statuses?module=tasks` | `tasks.UPDATE` + `custom_fields.READ` | `#module_meta-modules-get_global_statuses` |
| `searchTasks` | `GET /projects/{pid}/tasks` com `filter.criteria` e `sort_by` (sem `projectId`: itera `getProjectList`) | `tasks.READ` | `#tasks-get_project_tasks` |
| `getProjectList` | `GET /projects` | `projects.READ` | `#projects-get_all_projects` |
| `getProject` | `GET /projects/{pid}` | `projects.READ` | `#projects-get_project_details` |
| `validateTaskId` · `getProviderFromTaskId` | local, sem API | — | — |

- **Paginação:** `page` + `per_page` (1–200, default 100); a resposta traz `page_info.has_next_page`.
- **`searchTasks` usa a rota por projeto, não a do portal.** O `GET /tasks` do portal não devolve o
  projeto no exemplo de resposta, e sem ele o `normalizeTask` não monta o id composto (Decisão 3).
  Sem `projectId`, o adapter itera os projetos, respeitando o rate limit. A rota de portal só entra
  se a pendência P2 mostrar que ela devolve o projeto.
- **Também não usa `/search`.** Ele exige o escopo extra `ZohoSearch.securesearch.READ` e o
  parâmetro obrigatório `module` (`all`, `tasks`…). O filtro de tasks já cobre o caso.
- **Mapeamento do `SearchQuery`:** `text` → `name contains`; `status` → filtro por `status`
  (ids resolvidos como na Decisão 5; `closed` filtra pelos nomes de `done`, igual à escrita);
  `assignee` → filtro `owner` `is` `<zpuid>` (seção "Filter by Owner"); `priority` → filtro por `priority`; `orderBy` → `sort_by`
  (`ASC(campo)`/`DESC(campo)`; campos: `id`, `name`, `start_date`, `end_date`, `created_time`,
  `last_modified_time`…, lista de "Available fields" em `#tasks-get_tasks`). O `assignee` chega como
  e-mail ou id e precisa virar ZPUID via `/api/v3.1/portal/{portal_id}/users`. `tags` fica
  `[A VERIFICAR]`: o nome do campo de filtro só se confirma no portal real.
- **Endpoints úteis fora da interface:** `bulk-tasks` (criar/atualizar em lote),
  `make-as-subtask`/`make-as-task`, `tasks/count`.

## Decisão 3: hierarquia → `projectId` / `taskId` (item 1.1)

Hierarquia do Zoho: **portal → project → tasklist → task → subtask** (o objeto task tem `depth`, e o
`parental_info` traz `parent_task_id` e `root_task_id`).

**Fato decisivo:** toda rota de task exige o `project_id`. Não existe `GET /portal/{id}/tasks/{tid}`.
A listagem por portal (`GET /tasks`) também **não** devolve o projeto no exemplo de resposta.
Só com o `task_id` não há como endereçar a task.

| Conceito da interface | Zoho | Origem |
|---|---|---|
| (implícito) | `portal_id` | `.env`: `ZOHO_PORTAL_ID`. Um portal por repo |
| `projectId` | `project_id` | Argumento ou `ZOHO_DEFAULT_PROJECT_ID` |
| `taskId` | **`<project_id>.<task_id>`** (id composto) | Gerado pelo adapter no `normalizeTask` |
| (opcional) | `tasklist.id` | `ZOHO_DEFAULT_TASKLIST_ID`. Sem ele, a task vai para a lista geral do projeto |

**Decisão: o `taskId` da interface é o id composto `<project_id>.<task_id>`.** O adapter monta esse
id ao normalizar e o decompõe ao chamar a API.

**Alternativas descartadas:**
- **Projeto fixo no `.env` como única fonte:** limita o repo a um projeto e quebra quando uma task
  de outro projeto é referenciada.
- **Descobrir o projeto pelo `task_id`:** não há rota que faça isso. Exigiria varrer projetos, com
  custo em chamadas e risco de rate limit (bloqueio de 10 min por endpoint).

**Custo aceito:** o id é longo, com cerca de 37 caracteres (ex.: `1752587000000097024.1752587000000097101`).
O `prefix` legível do Zoho (ex.: `RU1-T9`) **não** é endereçável por rota. O adapter pode exibi-lo
no nome ou na saída, mas não usá-lo como id.

## Decisão 4: formato de ID e detecção (item 1.3)

**Contexto:** o detector (`detector.md`) classifica `^\d{15,}$` como **Asana** e `^\d{5,14}$` como
**Jira** (só se o provider configurado for `jira`; senão, `null`). Os ids de task nos exemplos da doc
têm **13 ou 19 dígitos** (520 com 19 e 18 com 13, nas rotas de task). Ou seja, um id puro do Zoho
cairia **ora na regra do Asana, ora na do Jira**. Decisão do maestro: **um provider por vez** por repo.

**Decisão:**
1. **Regra primária:** `^\d+\.\d+$` → `zoho-projects`. O id composto é inequívoco (nenhum outro
   provider usa ponto) e a colisão com o Asana **desaparece**.
2. **Regra de desempate**, para id numérico puro (`^\d{5,}$`): se o `TASK_MANAGER_PROVIDER` for
   `zoho-projects`, retornar `zoho-projects`. Essa checagem roda **antes da regra do ClickUp**
   (`^[a-z0-9]{9}$`, que também casaria um número de 9 dígitos) e, portanto, antes das do Asana e do Jira. Com outro provider configurado, o comportamento atual
   não muda.
   É o mesmo mecanismo já usado para a colisão Jira × Linear (`PREFIX-NUM`). Nesse caso o adapter
   precisa do `ZOHO_DEFAULT_PROJECT_ID` para endereçar a task. Sem ele, erro explícito pedindo o id composto.
3. **Regressão obrigatória (SAC-61):** um id numérico de 16 dígitos com `TASK_MANAGER_PROVIDER=asana`
   continua resolvendo `asana`; um de 10 dígitos com `jira` continua `jira`; e `SAC-58` continua
   `linear`. O tamanho real dos ids do portal da GMill entra na pendência P1.

## Decisão 5: status (item 1.2)

**Fatos:**
- Os status do Zoho são customizáveis por portal.
- O status **embutido na task** traz `id`, `name` e **`is_closed_type`** (booleano).
- A lista de status vem de `GET /settings/global-statuses?module=tasks` (escopo
  `custom_fields.READ`). Ela é **paginada** (`page`/`per_page`), aceita `status_names` dentro do JSON
  `filter`, e cada item traz só `id`, `name` e cor, **sem** `is_closed_type`.
- Os status também parecem ligados a **layouts** (`/settings/layouts`), então um status global
  pode não valer para o layout de um projeto (pendência P4).

**Decisão:**
- **Leitura (Zoho → interface):** casar pelo **nome normalizado** (minúsculas, sem acento) usando a
  tabela default abaixo. Sem casamento, cair na **categoria** pelo `is_closed_type` do status
  embutido na task: `true` → `done`, `false` → `todo`. O nome original fica preservado em `statusRaw`.
- **Escrita (interface → Zoho):** resolver o `status.id` pelo nome alvo na lista de `global-statuses`,
  **paginando até o fim** e **com cache por sessão**. Se nenhum status casar, lançar erro explícito
  listando os status disponíveis. Nunca escolher um status às cegas.

| Interface | Nomes aceitos no Zoho (default, ajustáveis) |
|---|---|
| `backlog` | backlog |
| `todo` | open, to do, aberta, a fazer |
| `in_progress` | in progress, em andamento |
| `review` | in review, review, em revisão |
| `done` | closed, completed, done, concluída, fechada |
| `closed` | — (na escrita, usa os nomes de `done`) |
| `canceled` | cancelled, canceled, cancelada |

Cada nome aparece em **uma única linha**, então a leitura é determinística: "Closed" do Zoho vira
`done`. O `closed` da interface existe para providers com dois estados finais. No Zoho ele é
**escrito** como `done`, com a perda registrada, e nunca é lido.

`[A VERIFICAR]` Os nomes reais do portal da GMill. Só a doc e seus exemplos (`Open`, `Cancelled`)
foram vistos. A tabela é **default ajustável**, e a primeira execução real deve confrontá-la com a
lista viva.

## Decisão 6: prioridade (item 1.2)

O Zoho aceita `none | low | medium | high` (confirmado em Create e Update Task). Na leitura,
`none` vira `undefined`, porque `priority` é opcional na interface e inventar `normal` perderia a
informação de "sem prioridade".

| Interface → Zoho | Zoho → interface |
|---|---|
| `urgent` → `high` ⚠️ perda de nível | `high` → `high` |
| `high` → `high` | `medium` → `normal` |
| `normal` → `medium` | `low` → `low` |
| `low` → `low` | `none` → `undefined` (sem prioridade) |

A perda `urgent → high` é aceita e **documentada no adapter**. A alternativa seria um custom field
de prioridade, mas isso depende da configuração do portal e sai do escopo.

## Consequências

**Para a SAC-60 (adapter):**
- `normalizeTask` monta o id composto; toda chamada o decompõe (`split('.')`) e valida os dois segmentos.
- Cache de sessão para o access token (~55 min) e para os `global-statuses`.
- Escopos mínimos: `ZohoProjects.portals.READ`, `projects.READ`, `tasks.ALL`, `tasklists.READ`,
  **`custom_fields.READ`** (novo em relação à KB) e `users.READ`.
- Variáveis: `ZOHO_PORTAL_ID`, `ZOHO_DEFAULT_PROJECT_ID`, `ZOHO_DEFAULT_TASKLIST_ID` (opcional),
  além das de OAuth já listadas na KB.

**Para a SAC-61 (integração):**
- `types.md`: incluir `'zoho-projects'` na união `TaskManagerProvider` e nas tabelas `STATUS_MAPPING`
  e de prioridade, com as perdas `urgent→high` e `closed→done` marcadas.
- `detector.md`: a config do provider, a resolução de transporte (só `api`; sem MCP), a regra
  `^\d+\.\d+$` e o desempate `^\d{5,}$`, ambos **antes** das regras do ClickUp, do Asana e do Jira.
- `detector.md` (~l.245, aviso de incompatibilidade de provider): revisar para que um id numérico
  puro não gere falso alerta com `zoho-projects` configurado.
- `interface.md`: colunas `zoho-projects` nas tabelas de status e prioridade.

## Pendências

| # | O quê | Bloqueio |
|---|---|---|
| P1 | Validar tudo contra a API real (id composto, filtro `parent_task`, nomes de status) | Self Client, DC e `portal_id` da GMill |
| P2 | Confirmar se `GET /tasks` (portal) devolve o projeto em algum campo além do exemplo | Idem |
| P3 | Confirmar se o `root_task_id` de subtasks aninhadas afeta o id composto (é o mesmo `project_id`) | Idem |
| P4 | Confirmar se os status valem por layout de projeto e, se valerem, trocar `global-statuses` pelo detalhe do layout (`/settings/layouts/{id}`) | Idem |
| P5 | Nome do campo de filtro para `tags` (API de campos do módulo) | Idem |
| P6 | Se o Markdown da descrição/comentário renderiza no Zoho, e o formato do link web da task (a API não devolve link; o adapter o monta `[INFERIDO]`) | Idem |

## Histórico de revisão

| Data | Revisão | Mudanças |
|---|---|---|
| 2026-09-30 | Revisão independente (agente, só leitura, contra a doc baixada e o contrato `ITaskManager`) | 9 apontamentos, todos confirmados e aceitos: versões (V2 só na migração, v3.1 em users/phases/issues); `searchTasks` restrito à rota por projeto; ids de 13 e 19 dígitos (desempate estendido a `^\d{5,}$`); `is_closed_type` só no status embutido e `global-statuses` paginado; tabela de status sem nome duplicado; consequências da SAC-61 completas; mapeamento do `SearchQuery`; `/search` exige `module`; `none → undefined` |
| 2026-09-30 | Re-revisão das correções (mesmo agente) | As 9 correções confirmadas. Aceitos: desempate antes da regra do ClickUp; `assignee` via filtro `owner` (sai da P5); `closed` no `searchTasks` usa os nomes de `done`. **Rejeitado com evidência:** "campos de `sort_by` sem respaldo". A doc lista `id name start_date end_date completion_percentage created_time last_modified_time created_by is_completed` como "Available fields" em Get Tasks e Get Project Tasks |
