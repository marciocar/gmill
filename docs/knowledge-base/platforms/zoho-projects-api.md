# Zoho Projects API — Knowledge Base

> **Versão**: 1.1.0 | **Última atualização**: 2026-09-30 | **Categoria**: Platforms
> Referência técnica da **Zoho Projects REST API V3**: autenticação OAuth 2.0 (Self Client e
> fluxo web), geração/renovação de tokens, data centers, escopos, paginação, rate limit e erros.
> Tudo com fonte na documentação oficial da Zoho; o que extrapola a fonte está `[INFERÊNCIA]`.

---

## 📋 Metadata

| Campo | Valor |
|-------|-------|
| **Versão** | 1.1.0 |
| **Data de Criação** | 2026-09-30 |
| **Última Atualização** | 2026-09-30 (v1.1: endpoints de tarefa + webhooks/automação) |
| **Categoria** | platforms |
| **Versão da API** | V3 (`/api/v3/`) — a única viva desde 2026-01-01 |
| **Fontes Principais** | [1] <https://projects.zoho.com/api-docs> (V3, oficial) · [2] <https://www.zoho.com/accounts/protocol/oauth/self-client/overview.html> · [3] <https://www.zoho.com/accounts/protocol/oauth/self-client/authorization-code-flow.html> · [4] <https://www.zoho.com/accounts/protocol/oauth/web-apps/authorization.html> · [5] <https://www.zoho.com/accounts/protocol/oauth/multi-dc.html> · [6] <https://www.zoho.com/developer/oauth/token-limits.html> · [7] <https://www.zoho.com/projects/help/rest-api/zohoprojectsapi.html> (legado) · [8] <https://ascentbusiness.co.uk/zoho-projects-api-deadline-migrate-to-v3-by-31-december-2025/> · [9] <https://www.zoho.com/projects/taskautomation.html> · [10] <https://www.zoho.com/projects/zoho-flow-integrations.html> |

---

## 📋 Visão Geral

O **Zoho Projects** é o gerenciador de projetos SaaS da Zoho (projetos, tasklists, tasks, milestones,
bugs/issues, timesheets, documentos, fóruns). A API REST expõe esses módulos para integração.

| Ponto | Valor | Fonte |
|---|---|---|
| Versão vigente | **V3**: prefixo `/api/v3/`, JSON puro, datas ISO-8601, `PATCH` para update | [1] |
| Legado (`/restapi/`) | **desligado**: pré-V3 retorna 4xx desde 2026-01-01 (prazo 2025-12-31 23:59 UTC) | [8] |
| Autenticação | OAuth 2.0; header `Authorization: Bearer <access_token>` | [1] |
| Unidade raiz | **portal** (a organização no Zoho Projects); quase toda rota é `/api/v3/portal/{portal_id}/...` | [1] |
| Formato | JSON (request = response: o mesmo shape volta e vai, ex.: `assignee: {zpuid, name}`) | [1] |

### Hierarquia de recursos

```
Portal (portal_id)
 └─ Project (project_id)
     ├─ Milestone
     ├─ Tasklist
     │   └─ Task ── Subtask
     ├─ Issue / Bug
     ├─ Timesheet (timelog)
     ├─ Document · Forum · Event
     └─ Custom fields (api_name, ex.: cf_priority_level)
```

---

## 🎯 Casos de Uso

| Use quando | Não use quando |
|---|---|
| Sincronizar tasks/projetos Zoho com outro sistema (ERP, BI, service desk) | Precisa de reação em tempo real: prefira **Webhooks + Workflow Rules** nativos [1][9] a polling (ver [Tarefas e Automação](#-tarefas-e-automação)) |
| Extrair timesheets e horas para faturamento ou relatórios | Carga massiva recorrente: há **Data Backup/Portal Export** nativos (1 export a cada 24 h [1]) |
| Criar tasks programaticamente a partir de outro fluxo | O dado vive em outro produto Zoho (CRM, Books): o token com escopo Projects **não** serve lá [7] |
| Automação back-end sem usuário interativo (Self Client) | App multiusuário/multi-org público: use o fluxo web com `redirect_uri`, não Self Client |

---

## ⚡ Quick Start — Self Client (back-end, sem redirect)

O **Self Client** gera um grant token da própria organização, sem domínio nem redirect URL. É o caminho
certo para um job servidor ou script [1][2].

### 1. Criar o Self Client

1. Acessar o **Zoho API Console** do seu data center (ex.: <https://api-console.zoho.com>) [2].
2. **GET STARTED → Self Client → CREATE NOW → OK** [2].
3. Na aba **Client Secret**, copiar `client_id` e `client_secret` [1].

### 2. Gerar o grant code

1. Aba **Generate Code** → informar os **escopos separados por vírgula** (ver [Escopos](#-escopos-oauth)).
   Escopo inválido gera o erro *"Enter a valid scope"* [1].
2. Escolher a **Time Duration** (validade do código; o default é **3 minutos**) [3].
3. Descrição → **Create** → copiar o código. **Use-o imediatamente**, porque ele expira.

### 3. Trocar o grant code por tokens

```bash
# accounts-server-url depende do DC (tabela abaixo). US = https://accounts.zoho.com
curl -X POST "https://accounts.zoho.com/oauth/v2/token" \
  -d "grant_type=authorization_code" \
  -d "client_id=$ZOHO_CLIENT_ID" \
  -d "client_secret=$ZOHO_CLIENT_SECRET" \
  -d "code=<GRANT_CODE>"
```

Resposta [3]:

```json
{
  "access_token": "1000.xxxx",
  "refresh_token": "1000.yyyy",
  "api_domain": "https://www.zohoapis.com",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

> ⚠️ O `refresh_token` **só aparece nessa troca**. Guarde-o no `.env` na hora. Se perder, gere um
> novo grant code.

### 4. Renovar o access token (a cada ≤ 1 h)

```bash
curl -X POST "https://accounts.zoho.com/oauth/v2/token" \
  -d "grant_type=refresh_token" \
  -d "client_id=$ZOHO_CLIENT_ID" \
  -d "client_secret=$ZOHO_CLIENT_SECRET" \
  -d "refresh_token=$ZOHO_REFRESH_TOKEN"
```

### 5. Descobrir o `portal_id` e chamar a API

```bash
# Escopo: ZohoProjects.portals.READ
curl -s "https://projects.zoho.com/api/v3/portals" \
  -H "Authorization: Bearer $ZOHO_ACCESS_TOKEN"

# Listar projetos do portal (escopo ZohoProjects.projects.READ)
curl -s "https://projects.zoho.com/api/v3/portal/$ZOHO_PORTAL_ID/projects?page=1&per_page=100" \
  -H "Authorization: Bearer $ZOHO_ACCESS_TOKEN"

# Tasks de um projeto (escopo ZohoProjects.tasks.READ)
curl -s "https://projects.zoho.com/api/v3/portal/$ZOHO_PORTAL_ID/projects/$PROJECT_ID/tasks" \
  -H "Authorization: Bearer $ZOHO_ACCESS_TOKEN"
```

### Variáveis de ambiente sugeridas `[INFERÊNCIA]`

```bash
# .env (NUNCA versionar; o .gitignore deste repo já protege o .env)
ZOHO_DC=com                         # com | eu | in | com.au | jp | ...
ZOHO_ACCOUNTS_URL=https://accounts.zoho.com
ZOHO_PROJECTS_API_URL=https://projects.zoho.com
ZOHO_CLIENT_ID=1000.xxxx
ZOHO_CLIENT_SECRET=xxxx
ZOHO_REFRESH_TOKEN=1000.xxxx        # longa duração: é o segredo que importa
ZOHO_PORTAL_ID=12345678
# ZOHO_ACCESS_TOKEN não se guarda: é derivado (1 h) e renovado em runtime
```

---

## 🔐 OAuth 2.0 — Referência

### Fluxos disponíveis

| Fluxo | Quando | Refresh token? | Fonte |
|---|---|---|---|
| **Self Client: authorization code** | Script/job servidor da própria org | ✅ sim | [2][3] |
| **Self Client: client credentials** | Back-end que só precisa do token | ❌ não: repete a requisição a cada expiração | [2] |
| **Web app (server-based)** | App com usuários e `redirect_uri` | ✅ com `access_type=offline` | [4] |

### Fluxo web: URL de autorização [4]

```
GET {accounts-server-url}/oauth/v2/auth
  ?response_type=code
  &client_id=<CLIENT_ID>
  &scope=ZohoProjects.portals.READ,ZohoProjects.tasks.ALL
  &redirect_uri=<URI registrada no console>
  &access_type=offline      # obrigatório para receber refresh_token (default = online)
  &prompt=consent           # força a tela de consentimento
```

O redirect volta com `code`, `location` e `accounts-server`. Troque o `code` no `accounts-server`
**retornado**, não no US fixo [5].

### Validade e limites de tokens

| Item | Limite | Fonte |
|---|---|---|
| Access token | **1 hora** (`expires_in: 3600`) | [3] |
| Refresh token | Sem expiração até ser revogado | [6]* |
| Refresh tokens ativos por usuário/client | **20**; o 21º invalida o mais antigo | [6] |
| Access tokens ativos por refresh token | **10** | [6] |
| Pedidos de access token | **10 a cada 10 minutos** (throttle) | [6] |
| Authorization codes | 10 por usuário em 10 min | [6] |

\* Validade "ilimitada até revogação" vem de páginas de token-validity de outros produtos Zoho, que
seguem o mesmo servidor de contas. Para o Zoho Projects especificamente, é `[INFERÊNCIA]`.

**Consequências práticas:**
- **Faça cache do access token** e renove só perto do vencimento. Renovar a cada chamada estoura o
  throttle de 10/10 min.
- Não gere refresh token novo a cada deploy. O 21º derruba silenciosamente o mais antigo, e outra
  integração pode ser a vítima.

### Multi-DC: accounts × API por região [1][5]

| DC | Accounts server | Projects API base |
|---|---|---|
| US | `https://accounts.zoho.com` | `https://projects.zoho.com` |
| EU | `https://accounts.zoho.eu` | `https://projects.zoho.eu` |
| IN | `https://accounts.zoho.in` | `https://projects.zoho.in` |
| AU | `https://accounts.zoho.com.au` | `https://projects.zoho.com.au` |
| JP | `https://accounts.zoho.jp` | `https://projects.zoho.jp` |
| CA | `https://accounts.zohocloud.ca` | `https://projects.zohocloud.ca` |
| SA | `https://accounts.zoho.sa` | `https://projects.zoho.sa` |
| UK | `https://accounts.zoho.uk` | `https://projects.zoho.uk` |
| CN / UAE / SG | ver `serverinfo` | `projects.zoho.com.cn` / `.ae` / `.sg` |

- Descoberta dinâmica: `GET https://accounts.zoho.com/oauth/serverinfo` [5].
- A resposta de token traz `location` e `api_domain`: use-os para escolher a base, **nunca hardcode** [1].
- Um app multi-região precisa **habilitar Multi-DC no API Console** (Settings → toggle por DC). O
  `client_id` é comum; o `client_secret` pode ser comum ou por DC [5].

---

## 🔑 Escopos OAuth

Formato: `ZohoProjects.<módulo>.<OPERAÇÃO>`, com operação em `READ | CREATE | UPDATE | DELETE | ALL`. Os
escopos são separados por vírgula. `ALL` cobre as quatro operações [1].

| Módulos com escopo próprio (V3) [1] |
|---|
| `portals` · `projects` · `projectgroups` · `milestones` · `tasklists` · `tasks` · `bugs` · `timesheets` · `users` · `teams` · `clients` · `documents` · `forums` · `events` · `tags` · `status` · `custom_fields` · `extensions` · `integrations` · `leave` · `skillset` · `bulk` · `custom_functions` |

`ALL` aparece documentado para `projects`, `tasks` e `timesheets` [1]. Para os demais módulos, peça
as operações explícitas.

**Conjuntos mínimos sugeridos** `[INFERÊNCIA]`:

| Integração | Escopos |
|---|---|
| Leitura/BI | `ZohoProjects.portals.READ,ZohoProjects.projects.READ,ZohoProjects.tasks.READ,ZohoProjects.timesheets.READ,ZohoProjects.users.READ` |
| Sync bidirecional de tasks | acima + `ZohoProjects.tasks.ALL,ZohoProjects.tasklists.READ,ZohoProjects.milestones.READ` |

> Token com escopo de Zoho Projects **não** funciona no Zoho BugTracker, e o contrário também vale [7].

---

## 📐 Convenções da V3

### Paginação

`page` + `per_page`, com `page_info` na resposta [1]:

```json
{ "page_info": { "page": 1, "per_page": 100, "page_count": 100, "has_next_page": true },
  "projects": [ ... ] }
```

Itere enquanto `has_next_page == true`. (O legado usava `index` + `range`, com range ≤ 200 [7].)

### Datas, updates e custom fields [1]

| Área | V2 (`/restapi`, morto) | V3 (`/api/v3`) |
|---|---|---|
| Data | `05-26-2014`, epoch… | `2024-05-26` ou `2024-05-26T11:25:58.000Z` (ISO-8601) |
| Update | `POST` | **`PATCH`** |
| Custom field | `UDF_CHAR1` | `api_name` (ex.: `cf_priority_level`) |
| Filtro | vários params | objeto `filter.criteria[]` + `pattern` |

### Rate limit [1]

- **200 chamadas por janela de 2 minutos, por endpoint.**
- Headers em toda resposta: `RateLimit`, `RateLimit-Remaining`, `RateLimit-Window`, `RateLimit-Window-Unit`.
- Ao exceder, **aquele endpoint fica bloqueado por 10 minutos** e a resposta traz `Retry-After` (segundos).
- ⚠️ Fontes antigas citam 100 chamadas/2 min com lock de 30 min [7][8]. Isso é do legado; vale a doc V3.

### Erros [1]

```json
{ "error": { "status_code": "400", "instance": "/api/v3/portal/25450296/...",
  "method": "GET", "error_type": "FIELDS_VALIDATION_ERROR",
  "details": [ { "message": "Invalid module", "field_name": "module" } ],
  "title": "INVALID_PARAMETER_VALUE" } }
```

| Código | HTTP | Significado / ação |
|---|---|---|
| `INVALID_OAUTHTOKEN` | 400 | Token inválido/expirado: renove com o refresh token |
| `URL_RULE_NOT_CONFIGURED` | 400 | URL errada: confira prefixo `/api/v3/` e DC |
| `INVALID_INPUTSTREAM` / `INVALID_PARAMETER_VALUE` | 400 | Parâmetro ou payload inválido |
| `INVALID_METHOD` | 400 | Método HTTP errado (ex.: `POST` onde a V3 pede `PATCH`) |
| `INTERNAL_SERVER_ERROR` | 500 | Erro do servidor: `support@zohoprojects.com` |

Note: token expirado volta como **400, não 401**. Um cliente que só renova em 401 nunca renova.

---

## 🧩 Tarefas e Automação

### Endpoints de tarefa (V3) [1]

| Operação | Método e rota | Escopo |
|---|---|---|
| Criar tarefa | `POST /api/v3/portal/{portal_id}/projects/{project_id}/tasks` | `tasks.CREATE` |
| Atualizar tarefa | `PATCH /api/v3/portal/{portal_id}/projects/{project_id}/tasks/{task_id}` | `tasks.UPDATE` |
| Comentar na tarefa | `POST /api/v3/portal/{portal_id}/projects/{project_id}/tasks/{task_id}/comments` (`{"comment": "..."}`) | `tasks.CREATE` |
| Listar tasklists | `GET /api/v3/portal/{portal_id}/projects/{project_id}/tasklists` | `tasklists.READ` |

Campos principais do **Create Task** [1]:

| Campo | Regra |
|---|---|
| `name` | **Obrigatório**, até 10000 caracteres |
| `description` | Até 80000 caracteres |
| `tasklist.id` | Sem ele, a tarefa cai na lista geral |
| `parental_info.parent_task_id` | Cria como subtarefa |
| `status.id` | Default: aberta |
| `priority` | `none` · `low` · `medium` · `high` |
| `start_date` / `end_date` | ISO-8601 (`yyyy-MM-dd'T'HH:mm:ss'Z'` ou com offset) |
| `owners_and_work.owners[]` | Por `zpuid`, `zuid` ou `email` |
| `tags`, `teams` | Objetos `add`/`remove` com ids |
| Custom fields | Por `api_name` (ex.: `cf_glpi_ticket_id`) |

### Webhooks e Workflow Rules [9]

- **Webhook** = notificação HTTP de saída para um sistema terceiro. **Só dispara quando associado a
  uma Workflow Rule** (ou Business Rule) cujas condições casam [9].
- **Workflow Rule** = gatilho + condição + ações (atribuir, atualizar campo, e-mail, webhook, custom function) [9].
- Também há **Macro Rules** (atualização em lote) e **Blueprints** (máquina de estados de tarefa) [1][9].
- **Zoho Flow** é a alternativa low-code para ligar o Projects a apps externos [10].
- Autenticar o webhook de saída (segredo no header/URL) e validar no receptor fica por conta do
  integrador `[INFERÊNCIA]`. A disponibilidade de webhooks pode variar por plano `[INFERÊNCIA]`.

> Fluxo completo usando isto: [GLPI → Zoho: chamado vira tarefa](../patterns/glpi-zoho-ticket-to-task.md).

---

## 💡 Best Practices

1. **V3 sempre**: qualquer código, SDK ou exemplo com `/restapi/` está morto desde 2026-01-01 [8].
2. **Menor escopo possível**: os escopos aparecem na tela de consentimento [1].
3. **Base URL derivada do token** (`api_domain`/`location`), nunca fixa no US [1][5].
4. **Cache do access token + renovação proativa** (~55 min), respeitando 10 renovações/10 min [6].
5. **Backoff guiado por `Retry-After`** e leitura de `RateLimit-Remaining` antes de lotes [1].
6. **Renovar em `INVALID_OAUTHTOKEN`**, não em 401 (ver erros acima) [1].
7. **Datas em ISO-8601 UTC** no payload. Formato errado gera 400 [1][8].
8. **Custom fields por `api_name`**, descoberto via *Module Meta → Get Field Info* [1].

---

## ⚠️ Limitações e Gotchas

| Gotcha | Detalhe |
|---|---|
| Grant code expira rápido | Default de 3 min: gere o código e troque em seguida [3] |
| Refresh token some | Só vem na troca inicial (e, no fluxo web, só com `access_type=offline`) [3][4] |
| Teto de 20 refresh tokens | Regerar demais invalida o token de outra integração em silêncio [6] |
| Bloqueio de 10 min | O rate limit é por endpoint; um loop de polling tranca o endpoint inteiro [1] |
| Dois hosts na doc | A V3 usa `projects.zoho.com` como raiz; exemplos e URLs de export mostram `projectsapi.zoho.com` [1][7]. Padronize no primeiro e trate o segundo como alias `[INFERÊNCIA]` |
| Header de auth | Projects V3 documenta `Bearer`. Outros produtos Zoho (ex.: CRM) documentam `Zoho-oauthtoken`: não copie exemplos entre produtos `[INFERÊNCIA]` |
| Escopo por produto | O token do Projects não vale no BugTracker nem em outros apps Zoho [7] |
| Portal export | 1 por 24 h (`EXPORT_ONCE_IN_24_HOURS`) [1] |

---

## 🔗 Integração com o Sistema Onion

- **Não é um provider do SDAAL de task manager.** `TASK_MANAGER_PROVIDER` aceita `jira | clickup |
  asana | linear | none` (ver [task-manager-abstraction](../concepts/task-manager-abstraction.md)).
  Operar tasks do Zoho via `/product:task` exigiria um **adapter novo** em
  `.claude/utils/task-manager/adapters/`, caminho de `/meta:create-abstraction` + sinal upstream ao core.
- **Segredos:** `ZOHO_*` no `.env`, carregado com `set -a; source .env; set +a`. Doutrina em
  [secret-handling-agent](../concepts/secret-handling-agent.md). Nunca imprimir o token em log ou chat.
- **Configuração guiada:** `/meta:setup-integration` é o comando canônico para registrar as variáveis.
- **Integrações estudadas:** [GLPI → Zoho: chamado vira tarefa](../patterns/glpi-zoho-ticket-to-task.md) · KB irmã [glpi-api](glpi-api.md).
- **Frescor:** a API mudou de major (V2→V3) em 2025. Reverifique esta KB contra [1] antes de
  decisão (`/meta:kb-freshness`; doutrina [verify-external-for-current](../concepts/verify-external-for-current.md)).

---

## 🔗 Referências

1. Zoho Projects V3 API Documentation: <https://projects.zoho.com/api-docs>
2. Zoho Accounts, Self Client overview: <https://www.zoho.com/accounts/protocol/oauth/self-client/overview.html>
3. Zoho Accounts, Self Client authorization code flow: <https://www.zoho.com/accounts/protocol/oauth/self-client/authorization-code-flow.html>
4. Zoho Accounts, Web apps authorization: <https://www.zoho.com/accounts/protocol/oauth/web-apps/authorization.html>
5. Zoho Accounts, Multi-DC support: <https://www.zoho.com/accounts/protocol/oauth/multi-dc.html>
6. Zoho Developer, OAuth token limits: <https://www.zoho.com/developer/oauth/token-limits.html>
7. Zoho Projects, API legado (restapi): <https://www.zoho.com/projects/help/rest-api/zohoprojectsapi.html>
8. Ascent Business, "Migrate to V3 by 31 Dec 2025": <https://ascentbusiness.co.uk/zoho-projects-api-deadline-migrate-to-v3-by-31-december-2025/>
9. Zoho Projects, Task Automation (Workflow Rules, Webhooks, Macros, Blueprints): <https://www.zoho.com/projects/taskautomation.html>
10. Zoho Projects + Zoho Flow: <https://www.zoho.com/projects/zoho-flow-integrations.html>

---

*Pesquisado e gerado em 2026-09-30 (v1.1 no mesmo dia: tarefas + automação) via `/meta:create-knowledge-base`. Nenhuma chamada real à API foi feita: os exemplos seguem a doc oficial e não foram executados contra um portal.*
