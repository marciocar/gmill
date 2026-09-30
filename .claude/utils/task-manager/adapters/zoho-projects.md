# 🟢 Zoho Projects Adapter

> Instância do padrão [SDAAL](../../../../docs/knowledge-base/concepts/specification-driven-ai-abstraction-layer.md).
> Transporte **único: REST API V3** (`TASK_MANAGER_TRANSPORT=mcp` cai para `api`: não há MCP do Zoho Projects).
> **Decisões de mapeamento:** [ADR zoho-projects-task-mapping](../../../../docs/technical-context/decisions/adr-zoho-projects-task-mapping.md) (SSOT, aceito 2026-09-30).
> **Referência da API:** [KB zoho-projects-api](../../../../docs/knowledge-base/platforms/zoho-projects-api.md).

⚠️ **Estado de validação:** as rotas, os verbos, os escopos e os formatos de resposta foram conferidos
contra a doc oficial V3 (<https://projects.zoho.com/api-docs>, baixada em 2026-09-30). Os **modos de
falha de autenticação foram medidos** contra os endpoints públicos (ver [Erros](#-erros-e-renovação-de-token)).
O **caminho feliz ainda não rodou contra um portal real**: faltam as credenciais da GMill. Pendências
P1–P5 do ADR, mais os itens `[A VERIFICAR]` abaixo.

---

## 📋 Configuração

### Variáveis de Ambiente

```bash
# Seleção do provider
TASK_MANAGER_PROVIDER=zoho-projects

# OAuth (Self Client) — obrigatórias
ZOHO_CLIENT_ID=1000.xxxxx
ZOHO_CLIENT_SECRET=xxxxx
ZOHO_REFRESH_TOKEN=1000.xxxxx.xxxxx     # vem só na troca inicial do grant code

# Data center — obrigatória (define os hosts; nunca hardcode o US)
ZOHO_ACCOUNTS_URL=https://accounts.zoho.com    # .eu · .in · .com.au · .jp · .uk · .sa · .ae · .sg · .com.cn · Canadá: https://accounts.zohocloud.ca

# Portal — obrigatória
ZOHO_PORTAL_ID=123456789

# Opcionais
ZOHO_DEFAULT_PROJECT_ID=1752587000000097024    # projeto padrão (createTask sem projectId; id puro)
ZOHO_DEFAULT_TASKLIST_ID=                      # sem ele, a task cai na lista geral do projeto
ZOHO_WEB_URL=https://projects.zoho.com         # base do link web da task [INFERIDO]

# ZOHO_ACCESS_TOKEN NÃO se guarda: é derivado (1 h) e renovado em runtime pelo adapter.
```

**Nunca** imprimir `ZOHO_CLIENT_SECRET`, `ZOHO_REFRESH_TOKEN` nem o access token em log, chat ou
comentário ([secret-handling-agent](../../../../docs/knowledge-base/concepts/secret-handling-agent.md)).

### Escopos do Self Client

```
ZohoProjects.portals.READ,ZohoProjects.projects.READ,ZohoProjects.tasks.ALL,
ZohoProjects.tasklists.READ,ZohoProjects.custom_fields.READ,ZohoProjects.users.READ
```

`custom_fields.READ` é necessário para resolver `status.id` via `global-statuses`. `users.READ`
resolve e-mail → ZPUID no filtro de `assignee`.

### Obter as credenciais

1. <https://api-console.zoho.com> → **Self Client** → gerar o *grant code* com os escopos acima.
   A validade do código é escolhida no console (minutos): troque-o logo.
2. Trocar pelo par de tokens **imediatamente** (o host é o do DC do portal):
   `POST {ZOHO_ACCOUNTS_URL}/oauth/v2/token` com `grant_type=authorization_code`.
3. Guardar o `refresh_token` no `.env`.
4. `ZOHO_PORTAL_ID`: `GET https://projects.zoho{dc}/api/v3/portals` (host do Projects, **não** o `api_domain`).

Guiado por `/meta:setup-integration` (opção Zoho Projects).

---

## 🚦 Transporte

| `TASK_MANAGER_TRANSPORT` | Comportamento |
|--------------------------|---------------|
| `api` (padrão) | REST V3 via `fetch` |
| `mcp` | **Não suportado.** Cai para `api` com aviso (não existe servidor MCP do Zoho Projects) |

---

## 🌐 Hosts, versões e autenticação

| O quê | Valor |
|---|---|
| Accounts (token) | `ZOHO_ACCOUNTS_URL` (por DC) |
| API | `https://projects.zoho{dc}` derivado de `ZOHO_ACCOUNTS_URL` (Canadá: `projects.zohocloud.ca`). **Não** o `api_domain` do token (`www.zohoapis.*` responde 404 para o Projects, medido) |
| Prefixo | `/api/v3/portal/{portal_id}`; **exceto users**: `/api/v3.1/portal/{portal_id}/users` |
| Header | `Authorization: Bearer {access_token}` (o Projects V3 documenta `Bearer`; o CRM usa `Zoho-oauthtoken`, não misturar) |

O prefixo de versão é **por endpoint**, nunca global (ADR, Decisão 1).

---

## 🆔 Formato de ID (ADR, Decisões 3 e 4)

Toda rota de task exige o `project_id`, e não existe rota de task só pelo portal. Por isso:

- **`taskId` da interface = `<project_id>.<task_id>`** (ex.: `1752587000000097024.1752587000000097101`),
  com 5 ou mais dígitos em cada lado (`1.0` ou `2026.09` não são id Zoho).
- O adapter **monta** o id composto ao normalizar e **decompõe** ao chamar a API.
- Um id numérico **puro** só é aceito se houver `ZOHO_DEFAULT_PROJECT_ID`. Sem ele, o adapter lança erro
  explícito pedindo o id composto.
- O `prefix` legível do Zoho (ex.: `CA1-T7`) **não** é endereçável por rota. Ele vai no nome exibido,
  não no id.

```typescript
function splitTaskId(taskId: string, defaultProjectId?: string): { projectId: string; taskId: string } {
  const id = taskId.trim();
  const composite = /^([1-9]\d{4,})\.([1-9]\d{4,})$/.exec(id);   // ids Zoho: 13-19 dígitos, sem zero à esquerda
  if (composite) return { projectId: composite[1], taskId: composite[2] };
  if (/^[1-9]\d{4,}$/.test(id)) {
    if (defaultProjectId) return { projectId: defaultProjectId, taskId: id };
    throw new Error(
      `❌ Id Zoho "${id}" sem projeto. Use o id composto <project_id>.<task_id> ` +
      `ou defina ZOHO_DEFAULT_PROJECT_ID.`
    );
  }
  throw new Error(`❌ Id "${id}" não é um id de task Zoho (esperado <project_id>.<task_id>).`);
}

const joinTaskId = (projectId: string, taskId: string) => `${projectId}.${taskId}`;
```

---

## 🔧 Implementação

### Cliente HTTP (token, erros, retry)

```typescript
interface ZohoConfig {
  clientId: string;
  clientSecret: string;
  refreshToken: string;
  accountsUrl: string;          // ZOHO_ACCOUNTS_URL
  portalId: string;             // ZOHO_PORTAL_ID
  defaultProjectId?: string;
  defaultTasklistId?: string;
  webUrl?: string;
}

class ZohoClient {
  private accessToken?: string;
  private expiresAt = 0;        // epoch ms
  private apiBase?: string;     // https://projects.zoho{tld}

  constructor(private cfg: ZohoConfig) {
    // S5: normaliza e valida o host de accounts (barra final, esquema, formato do DC)
    this.cfg = {
      ...cfg,
      accountsUrl: (cfg.accountsUrl ?? '').trim().toLowerCase().replace(/:443(?=\/|$)/, '').replace(/\/+$/, '')
    };
    if (!ZOHO_ACCOUNTS_HOSTS.includes(this.cfg.accountsUrl)) {
      throw new Error(
        `❌ ZOHO_ACCOUNTS_URL inválida: "${cfg.accountsUrl}". ` +
        `Esperado https://accounts.zoho.<dc> (.com, .eu, .in, .com.au, .jp, .uk, .sa, .ae, .sg, .com.cn) ` +
        `ou https://accounts.zohocloud.ca (Canadá).`
      );
    }
  }

  /** Renova o access token. O endpoint responde HTTP 200 MESMO NA FALHA: checar o corpo. */
  private async refresh(): Promise<void> {
    const res = await fetch(`${this.cfg.accountsUrl}/oauth/v2/token`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        refresh_token: this.cfg.refreshToken,
        client_id: this.cfg.clientId,
        client_secret: this.cfg.clientSecret,
        grant_type: 'refresh_token'
      })
    });
    const json: any = safeJson(await res.text());
    // Medido 2026-09-30: credencial inválida → HTTP 200 + {"error":"invalid_client"}
    if (json.error === 'Access Denied') {
      // Medido 2026-09-30: excesso de pedidos ao token → HTTP 400 + {"error":"Access Denied",
      // "error_description":"You have made too many requests continuously…"}. Não é credencial errada.
      throw new Error('❌ Zoho OAuth: limite de pedidos de token atingido (10 a cada 10 min por app). Aguarde e tente de novo.');
    }
    if (!res.ok || json.error || !json.access_token) {
      throw new Error(
        `❌ Zoho OAuth: falha ao renovar o token (${json.error ?? `HTTP ${res.status}`}). ` +
        `Confira ZOHO_CLIENT_ID/SECRET/REFRESH_TOKEN e se ZOHO_ACCOUNTS_URL é o DC do portal.`
      );   // nunca incluir os valores dos segredos na mensagem
    }
    this.accessToken = json.access_token;
    // renovação proativa ~5 min antes. S4: expires_in inválido ou curto demais → 3600
    // (o Zoho permite só 10 pedidos de access token a cada 10 min por app)
    const ttl = Number(json.expires_in);
    const seconds = Number.isFinite(ttl) && ttl >= 600 ? ttl : 3600;
    this.expiresAt = Date.now() + (seconds - 300) * 1000;
    this.apiBase ??= deriveProjectsBase(this.cfg.accountsUrl);
  }

  private async token(): Promise<string> {
    if (!this.accessToken || Date.now() >= this.expiresAt) await this.refresh();
    return this.accessToken!;
  }

  /**
   * Chamada à API. Regras (medidas e da doc):
   * - 401 INVALID_OAUTHTOKEN → token expirado/inválido: renova UMA vez e repete.
   * - 401 INVALID_TICKET → header ausente = bug do adapter: NÃO renova (evita loop), lança.
   * - 429 / bloqueio de rate limit → lança erro INFORMANDO o Retry-After (não espera nem repete);
   *   o bloqueio é de 10 min por endpoint.
   * - Erro tem dois formatos: {error:{title,status_code,details}} e {error:{code,message}}.
   */
  async call<T = any>(method: string, path: string, body?: unknown, retried = false): Promise<T> {
    const token = await this.token();
    const res = await fetch(`${this.apiBase}${path}`, {
      method,
      headers: {
        Authorization: `Bearer ${token}`,
        ...(body !== undefined ? { 'Content-Type': 'application/json' } : {})
      },
      body: body !== undefined ? JSON.stringify(body) : undefined
    });

    if (res.status === 204) return undefined as T;           // Delete Task
    const json: any = safeJson(await res.text());
    if (res.ok && !json?.error) return json as T;

    const err = json?.error ?? {};
    const title: string = err.title ?? err.code ?? `HTTP_${res.status}`;
    if (title === 'INVALID_OAUTHTOKEN' && !retried) {
      this.accessToken = undefined;                          // força refresh
      return this.call<T>(method, path, body, true);
    }
    if (res.status === 429) {
      const wait = res.headers.get('Retry-After');
      throw Object.assign(
        new Error(`❌ Zoho rate limit em ${path}. Retry-After: ${wait ?? '?'} s (o bloqueio pode durar 10 min).`),
        { status: 429, retryAfter: wait }
      );
    }
    const detail = err.details?.[0]?.message ?? err.message ?? '';
    // N2: erro TIPADO — quem decide por status/código lê os campos, nunca a mensagem (que traz o path)
    throw Object.assign(new Error(`❌ Zoho ${method} ${path}: ${title} ${detail}`.trim()), {
      status: res.status, code: err.code, title
    });
  }

  /**
   * Pagina uma listagem. `key` é a chave do array no envelope ({page_info, <key>: [...]}).
   * H1: a doc oficial mostra os DOIS formatos para a mesma listagem (array cru na amostra da
   * operação; envelope na seção de paginação/migração) → aceita ambos e LANÇA se não for nenhum,
   * em vez de devolver [] em silêncio.
   */
  async paginate<T = any>(path: string, key: string, params: Record<string, string> = {}, max = 1000): Promise<T[]> {
    const out: T[] = [];
    let prevPage: string | undefined;
    let sawPageInfo = false;
    const MAX_PAGES = 50;                              // S3: teto de chamadas, mesmo com has_next_page mentindo
    for (let page = 1; out.length < max && page <= MAX_PAGES; page++) {
      const qs = new URLSearchParams({ ...params, page: String(page), per_page: '200' });
      let json: any;
      try {
        json = await this.call('GET', `${path}?${qs}`);
      } catch (e) {
        // N5: sem page_info, a página além do fim pode vir como erro → fica com o que já leu
        if (page > 1 && !sawPageInfo) return out.slice(0, max);
        throw e;
      }
      if (json?.page_info) sawPageInfo = true;
      const items: T[] | undefined =
        Array.isArray(json?.[key]) ? json[key] : Array.isArray(json) ? json : undefined;
      if (!items) {
        throw new Error(`❌ Zoho GET ${path}: resposta sem "${key}" nem array (formato inesperado)`);
      }
      // guarda de repetição: servidor que ignora `page` devolveria a MESMA página para sempre.
      // N4: compara a página INTEIRA (não só o 1º item) — páginas distintas idênticas são implausíveis.
      const sig = items.length ? JSON.stringify(items) : undefined;
      if (page > 1 && sig !== undefined && sig === prevPage) return out.slice(0, max);
      prevPage = sig;
      out.push(...items);
      // S3: só segue com has_next_page === true (booleano). Sem page_info, segue até página vazia
      // (o servidor pode limitar per_page abaixo de 200 — contar 200 truncaria em silêncio)
      const next = json?.page_info ? json.page_info.has_next_page === true : items.length > 0;
      if (!next || items.length === 0) return out.slice(0, max);
    }
    console.warn(`⚠️ Zoho: listagem de ${path} truncada (${max} itens ou ${MAX_PAGES} páginas)`);
    return out.slice(0, max);
  }
}

/** DCs do Zoho (lista explícita: regex aceitava DC inexistente). Canadá usa zohocloud.ca. */
const ZOHO_ACCOUNTS_HOSTS = [
  'https://accounts.zoho.com', 'https://accounts.zoho.eu', 'https://accounts.zoho.in',
  'https://accounts.zoho.com.au', 'https://accounts.zoho.jp', 'https://accounts.zoho.uk',
  'https://accounts.zoho.sa', 'https://accounts.zoho.ae', 'https://accounts.zoho.sg',
  'https://accounts.zoho.com.cn', 'https://accounts.zohocloud.ca'
];

/** Host da API do Projects a partir do DC de accounts (validado em .com, .eu e zohocloud.ca). */
function deriveProjectsBase(accountsUrl: string): string {
  // accountsUrl já validada no construtor do ZohoClient. O api_domain do token (www.zohoapis.*)
  // NÃO serve para o Projects: medido 2026-09-30, zohoapis.com/api/v3/portals → 404,
  // projects.zoho.com/api/v3/portals → 401 (rota existe). Canadá: accounts.zohocloud.ca → projects.zohocloud.ca
  const suffix = new URL(accountsUrl).hostname.replace(/^accounts\.zoho/, '');   // ".com", ".eu", "cloud.ca"…
  return `https://projects.zoho${suffix}`;
}

/**
 * S9: JSON.parse perde precisão em inteiros acima de 2^53. A doc traz ids quase sempre como
 * string, mas há amostras com id numérico → cita inteiros longos antes de parsear.
 * N1: varredura que RESPEITA strings (regex corrompia descrição com "NF: 1234…"); cobre
 * valores em objeto, em array e negativos. Só inteiros com 16+ dígitos fora de string.
 */
function safeJson(text: string): any {
  if (!text) return {};
  let out = '';
  let inStr = false;
  for (let i = 0; i < text.length; i++) {
    const c = text[i];
    if (inStr) {
      out += c;
      if (c === '\\') { out += text[++i] ?? ''; continue; }
      if (c === '"') inStr = false;
      continue;
    }
    if (c === '"') { inStr = true; out += c; continue; }
    if (c === '-' || (c >= '0' && c <= '9')) {
      let j = i + (c === '-' ? 1 : 0);
      while (j < text.length && text[j] >= '0' && text[j] <= '9') j++;
      const isInt = !/[.eE]/.test(text[j] ?? '');
      const digits = j - i - (c === '-' ? 1 : 0);
      out += isInt && digits >= 16 ? `"${text.slice(i, j)}"` : text.slice(i, j);
      i = j - 1;
      continue;
    }
    out += c;
  }
  try {
    return JSON.parse(out);
  } catch {
    return { error: { title: 'INVALID_JSON', message: text.slice(0, 120) } };
  }
}
```

### ZohoProjectsAdapter

```typescript
/**
 * Adapter Zoho Projects implementando ITaskManager (API V3).
 * Mapeamentos: ADR docs/technical-context/decisions/adr-zoho-projects-task-mapping.md
 */
class ZohoProjectsAdapter implements ITaskManager {
  readonly provider: TaskManagerProvider = 'zoho-projects';
  readonly isConfigured: boolean;

  private client: ZohoClient;
  private statusCache?: Promise<Array<{ id: string; name: string }>>;   // cache por sessão (promessa)

  constructor(private cfg: ZohoConfig) {
    this.isConfigured = !!(cfg.clientId && cfg.clientSecret && cfg.refreshToken && cfg.accountsUrl && cfg.portalId);
    this.client = new ZohoClient(cfg);
  }

  private get base() { return `/api/v3/portal/${this.cfg.portalId}`; }

  // ═══════════════════════════════════════════════════════════════════════════
  // CRUD DE TASKS
  // ═══════════════════════════════════════════════════════════════════════════

  async createTask(input: CreateTaskInput): Promise<TaskOutput> {
    const projectId = input.projectId ?? this.cfg.defaultProjectId;
    if (!projectId) throw new Error('❌ projectId obrigatório (ou ZOHO_DEFAULT_PROJECT_ID) para criar task no Zoho');
    const raw = await this.client.call('POST', `${this.base}/projects/${projectId}/tasks`, this.toZohoTask(input));
    return this.normalizeTask(raw, projectId);
  }

  async getTask(taskId: string): Promise<TaskOutput> {
    const { projectId, taskId: tid } = splitTaskId(taskId, this.cfg.defaultProjectId);
    const raw = await this.client.call('GET', `${this.base}/projects/${projectId}/tasks/${tid}`);
    return this.normalizeTask(raw, projectId);
  }

  async updateTask(taskId: string, updates: UpdateTaskInput): Promise<TaskOutput> {
    const { projectId, taskId: tid } = splitTaskId(taskId, this.cfg.defaultProjectId);
    const patch: Record<string, unknown> = this.toZohoTask(updates as CreateTaskInput, /*partial*/ true);
    if (updates.status !== undefined) patch.status = { id: await this.resolveStatusId(updates.status) };
    const raw = await this.client.call('PATCH', `${this.base}/projects/${projectId}/tasks/${tid}`, patch);
    return this.normalizeTask(raw, projectId);
  }

  async deleteTask(taskId: string): Promise<boolean> {
    let ids: { projectId: string; taskId: string };
    try { ids = splitTaskId(taskId, this.cfg.defaultProjectId); } catch { return false; }   // id inválido → false
    try {
      await this.client.call('DELETE', `${this.base}/projects/${ids.projectId}/tasks/${ids.taskId}`);   // 204
      return true;
    } catch (e: any) {
      // só "não existe" vira false; auth, rate limit e rede PROPAGAM (não são "deleção negada")
      if (e?.status === 404 || e?.code === 6404 || e?.title === 'RESOURCE_NOT_FOUND') return false;
      throw e;
    }
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // SUBTASKS
  // ═══════════════════════════════════════════════════════════════════════════

  async createSubtask(parentId: string, input: CreateTaskInput): Promise<TaskOutput> {
    const { projectId, taskId: parentTid } = splitTaskId(parentId, this.cfg.defaultProjectId);
    const body = { ...this.toZohoTask(input), parental_info: { parent_task_id: parentTid } };
    const raw = await this.client.call('POST', `${this.base}/projects/${projectId}/tasks`, body);
    return this.normalizeTask(raw, projectId);
  }

  async getSubtasks(parentId: string): Promise<TaskOutput[]> {
    // Sem rota dedicada: filtro "Filter by Parent Task" da seção Task Filters da doc.
    const { projectId, taskId: parentTid } = splitTaskId(parentId, this.cfg.defaultProjectId);
    const filter = JSON.stringify({
      criteria: [{ field_name: 'parent_task', criteria_condition: 'is', value: [parentTid] }],
      pattern: '1'
    });
    const tasks = await this.client.paginate(`${this.base}/projects/${projectId}/tasks`, 'tasks', { filter });
    return tasks.map(t => this.normalizeTask(t, projectId));
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // COMENTÁRIOS
  // ═══════════════════════════════════════════════════════════════════════════

  async addComment(taskId: string, comment: string): Promise<CommentOutput> {
    const { projectId, taskId: tid } = splitTaskId(taskId, this.cfg.defaultProjectId);
    const raw = await this.client.call('POST', `${this.base}/projects/${projectId}/tasks/${tid}/comments`, { comment });
    // A resposta de Add Comment é um ARRAY com o comentário criado
    return this.normalizeComment(Array.isArray(raw) ? raw[0] : raw, comment);
  }

  async getComments(taskId: string): Promise<CommentOutput[]> {
    const { projectId, taskId: tid } = splitTaskId(taskId, this.cfg.defaultProjectId);
    const json: any = await this.client.call('GET', `${this.base}/projects/${projectId}/tasks/${tid}/comments`);
    return (json?.comments ?? []).map((c: any) => this.normalizeComment(c, c.comment ?? ''));
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // STATUS
  // ═══════════════════════════════════════════════════════════════════════════

  async updateStatus(taskId: string, status: TaskStatus): Promise<TaskOutput> {
    return this.updateTask(taskId, { status });
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // BUSCA (ADR, Decisão 2: só a rota por projeto; sem projectId, itera os projetos)
  // ═══════════════════════════════════════════════════════════════════════════

  async searchTasks(query: SearchQuery): Promise<TaskOutput[]> {
    const criteria: Array<Record<string, unknown>> = [];
    if (query.text) criteria.push({ field_name: 'name', criteria_condition: 'contains', value: [query.text] });
    if (query.status?.length) {
      // todos os status do portal que casam (ex.: "Closed" E "Completed" para done)
      const ids = (await Promise.all(query.status.map(s => this.resolveStatusIds(s)))).flat();
      criteria.push({ field_name: 'status', criteria_condition: 'is', value: [...new Set(ids)] });
    }
    if (query.priority?.length) {
      // [A VERIFICAR] a doc de filtros pede ID de opção para picklist; prioridade por nome não tem exemplo
      criteria.push({ field_name: 'priority', criteria_condition: 'is', value: query.priority.map(p => this.mapPriorityToZoho(p)) });
    }
    if (query.assignee) {
      criteria.push({ field_name: 'owner', criteria_condition: 'is', value: [await this.resolveZpuid(query.assignee)] });
    }
    // query.tags: nome do campo de filtro [A VERIFICAR — P5]; ignorado com aviso
    if (query.tags?.length) console.warn('⚠️ Zoho: filtro por tags ainda não suportado (P5 do ADR)');

    const params: Record<string, string> = { sort_by: 'DESC(last_modified_time)' };
    if (criteria.length) {
      params.filter = JSON.stringify({ criteria, pattern: criteria.map((_, i) => i + 1).join(' AND ') });
    }
    const limit = query.limit ?? 50;
    const projectIds = query.projectId ? [query.projectId] : (await this.getProjectList()).map(p => p.id);
    const out: TaskOutput[] = [];
    for (const pid of projectIds) {                                  // sequencial: respeita rate limit
      const tasks = await this.client.paginate(`${this.base}/projects/${pid}/tasks`, 'tasks', params, limit - out.length);
      out.push(...tasks.map(t => this.normalizeTask(t, pid)));
      if (out.length >= limit) break;
    }
    return out;
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // PROJETOS
  // ═══════════════════════════════════════════════════════════════════════════

  async getProjectList(): Promise<ProjectOutput[]> {
    // Get All Projects devolve um ARRAY cru
    const projects = await this.client.paginate(`${this.base}/projects`, 'projects');
    return projects.map((p: any) => this.normalizeProject(p));
  }

  async getProject(projectId: string): Promise<ProjectOutput> {
    const raw = await this.client.call('GET', `${this.base}/projects/${projectId}`);
    return this.normalizeProject(raw);
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // VALIDAÇÃO
  // ═══════════════════════════════════════════════════════════════════════════

  validateTaskId(taskId: string): boolean {
    const id = taskId.trim();
    return /^[1-9]\d{4,}\.[1-9]\d{4,}$/.test(id) || (/^[1-9]\d{4,}$/.test(id) && !!this.cfg.defaultProjectId);
  }

  getProviderFromTaskId(taskId: string): TaskManagerProvider | null {
    return this.validateTaskId(taskId) ? 'zoho-projects' : null;
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // HELPERS PRIVADOS
  // ═══════════════════════════════════════════════════════════════════════════

  /** Interface → corpo Zoho (Create/Update Task). */
  private toZohoTask(input: Partial<CreateTaskInput>, partial = false): Record<string, unknown> {
    const body: Record<string, unknown> = {};
    if (input.name !== undefined) body.name = input.name;
    const desc = input.markdownDescription ?? input.description;
    if (desc !== undefined) body.description = desc;          // [A VERIFICAR] render de Markdown no Zoho (P6)
    if (input.priority !== undefined) body.priority = this.mapPriorityToZoho(input.priority);
    if (input.startDate) body.start_date = toZohoDate(input.startDate);
    if (input.dueDate) body.end_date = toZohoDate(input.dueDate);
    if (input.assignees?.length) {
      // owners por e-mail, zpuid ou zuid (Create Task aceita os três)
      body.owners_and_work = {
        owners: input.assignees.map(a => (a.includes('@') ? { email: a } : { zpuid: a }))
      };
    }
    if (!partial && this.cfg.defaultTasklistId) body.tasklist = { id: this.cfg.defaultTasklistId };
    // tags: o Zoho pede ids de tag, não nomes → ignorado com aviso [A VERIFICAR]
    if (input.tags?.length) console.warn('⚠️ Zoho: tags por nome não suportadas (a API pede ids)');
    return body;
  }

  private normalizeTask(input: any, projectIdHint: string): TaskOutput {
    // H2: desembrulha {tasks:[...]} e REJEITA resposta sem id numérico (200 com corpo inesperado
    // não pode virar sucesso com id "…undefined")
    const raw = Array.isArray(input?.tasks) ? input.tasks[0] : Array.isArray(input) ? input[0] : input;
    if (!/^\d+$/.test(String(raw?.id ?? ''))) {
      throw new Error(`❌ Zoho: resposta de task sem id válido (${JSON.stringify(input)?.slice(0, 120)})`);
    }
    const projectId = String(raw?.project?.id ?? projectIdHint);
    const tasklistId = raw?.tasklist?.id;
    return {
      id: joinTaskId(projectId, raw.id),
      provider: 'zoho-projects',
      name: raw.prefix ? `[${raw.prefix}] ${raw.name}` : raw.name,
      description: raw.description ?? '',
      status: this.normalizeStatus(raw.status),
      statusRaw: raw.status?.name,
      statusColor: raw.status?.color_hexcode,
      priority: this.normalizePriority(raw.priority),
      url: this.taskUrl(projectId, tasklistId, raw.id),
      createdAt: raw.created_time ?? new Date().toISOString(),
      updatedAt: raw.last_modified_time ?? raw.created_time ?? new Date().toISOString(),
      dueDate: raw.end_date?.slice(0, 10),
      startDate: raw.start_date?.slice(0, 10),
      assignees: (raw.owners_and_work?.owners ?? []).map((o: any) => ({
        id: o.zpuid ?? String(o.zuid), name: o.name, email: o.email
      })),
      tags: (raw.tags ?? []).map((t: any) => t.name ?? String(t.id)),
      parent: raw.parental_info?.parent_task_id ? joinTaskId(projectId, raw.parental_info.parent_task_id) : undefined,
      projectId,
      projectName: raw.project?.name
    };
  }

  private normalizeComment(raw: any, text: string): CommentOutput {
    const by = raw?.created_by ?? {};
    return {
      id: raw?.id ?? String(Date.now()),
      text: (raw?.comment ?? text).trim(),
      author: { id: by.zpuid ?? by.id ?? 'unknown', name: by.name ?? by.full_name ?? 'Unknown' },
      createdAt: raw?.created_time ?? new Date().toISOString()
    };
  }

  private normalizeProject(raw: any): ProjectOutput {
    return { id: raw.id, name: raw.name, description: raw.description, workspaceId: this.cfg.portalId };
  }

  // ─── Status (ADR, Decisão 5) ────────────────────────────────────────────────

  /** Nome normalizado: minúsculas, sem acento, espaços colapsados. */
  private static norm(s: string): string {
    return s.normalize('NFD').replace(/[̀-ͯ]/g, '').toLowerCase().replace(/\s+/g, ' ').trim();
  }

  /** Tabela default (cada nome em UMA linha só → leitura determinística). */
  private static readonly STATUS_NAMES: Record<Exclude<TaskStatus, 'closed'>, string[]> = {
    backlog:     ['backlog'],
    todo:        ['open', 'to do', 'aberta', 'a fazer'],
    in_progress: ['in progress', 'em andamento'],
    review:      ['in review', 'review', 'em revisao'],
    done:        ['closed', 'completed', 'done', 'concluida', 'fechada'],
    canceled:    ['cancelled', 'canceled', 'cancelada']
  };

  /** Leitura: nome → interface; sem casamento, cai na categoria is_closed_type. */
  private normalizeStatus(status: any): TaskStatus {
    const name = ZohoProjectsAdapter.norm(status?.name ?? '');
    for (const [iface, names] of Object.entries(ZohoProjectsAdapter.STATUS_NAMES)) {
      if (names.includes(name)) return iface as TaskStatus;
    }
    return status?.is_closed_type ? 'done' : 'todo';
  }

  /**
   * Lista de status do portal, com cache por sessão. S7: guarda a PROMESSA (chamadas concorrentes
   * compartilham um fetch). N3: promessa REJEITADA sai do cache (um 429 não inutiliza a sessão).
   */
  private loadStatuses(): Promise<Array<{ id: string; name: string }>> {
    this.statusCache ??= this.client
      .paginate<{ id: string; name: string }>(`${this.base}/settings/global-statuses`, 'statuses', { module: 'tasks' })
      .catch(e => { this.statusCache = undefined; throw e; });
    return this.statusCache;
  }

  /** Busca: TODOS os status.id do portal que casam com o status da interface. */
  private async resolveStatusIds(status: TaskStatus): Promise<string[]> {
    const target = status === 'closed' ? 'done' : status;
    const wanted = ZohoProjectsAdapter.STATUS_NAMES[target as Exclude<TaskStatus, 'closed'>];
    const ids = (await this.loadStatuses()).filter(s => wanted.includes(ZohoProjectsAdapter.norm(s.name))).map(s => s.id);
    if (!ids.length) throw new Error(`❌ Nenhum status do portal casa com "${status}" (ajuste STATUS_NAMES)`);
    return ids;
  }

  /** Escrita: interface → status.id, pela lista paginada de global-statuses (cache por sessão). */
  private async resolveStatusId(status: TaskStatus): Promise<string> {
    const target = status === 'closed' ? 'done' : status;           // perda closed→done registrada no ADR
    const statuses = await this.loadStatuses();
    const wanted = ZohoProjectsAdapter.STATUS_NAMES[target as Exclude<TaskStatus, 'closed'>];
    const hit = statuses.find(s => wanted.includes(ZohoProjectsAdapter.norm(s.name)));
    if (!hit) {
      throw new Error(
        `❌ Nenhum status do portal casa com "${status}". Disponíveis: ` +
        statuses.map(s => s.name).join(', ') + '. Ajuste STATUS_NAMES no adapter.'
      );
    }
    return hit.id;
  }

  // ─── Prioridade (ADR, Decisão 6) ────────────────────────────────────────────

  private mapPriorityToZoho(p?: TaskPriority): string {
    const map: Record<TaskPriority, string> = { urgent: 'high', high: 'high', normal: 'medium', low: 'low' };
    return p ? map[p] : 'none';                                  // urgent→high: perda registrada
  }

  private normalizePriority(p?: string): TaskPriority | undefined {
    const map: Record<string, TaskPriority> = { high: 'high', medium: 'normal', low: 'low' };
    return p ? map[p.toLowerCase()] : undefined;                  // none → undefined
  }

  // ─── Usuários (v3.1) ────────────────────────────────────────────────────────

  private async resolveZpuid(assignee: string): Promise<string> {
    if (!assignee.includes('@')) return assignee;                  // já é zpuid
    const users: any[] = await this.client.paginate(`/api/v3.1/portal/${this.cfg.portalId}/users`, 'users');   // [A VERIFICAR] chave do envelope v3.1
    const u = users.find(x => x.email?.toLowerCase() === assignee.toLowerCase());
    if (!u) throw new Error(`❌ Usuário ${assignee} não encontrado no portal Zoho`);
    return u.zpuid ?? u.id;
  }

  /** Link web da task. [INFERIDO] a API não devolve link; formato a validar no portal real (P1). */
  private taskUrl(projectId: string, tasklistId: string | undefined, taskId: string): string {
    const web = this.cfg.webUrl ?? 'https://projects.zoho.com';
    return `${web}/portal/${this.cfg.portalId}#taskdetail/${projectId}/${tasklistId ?? ''}/${taskId}`;
  }
}

/** YYYY-MM-DD → ISO-8601 UTC exigido pela V3 (formato errado gera 400). */
function toZohoDate(d: string): string {
  return /^\d{4}-\d{2}-\d{2}$/.test(d) ? `${d}T00:00:00Z` : d;
}
```

---

## 📊 Mapeamento de Campos

| Interface | Zoho V3 | Nota |
|---|---|---|
| `id` | `<project.id>.<id>` | id composto (ADR, Decisão 3) |
| `name` | `[prefix] name` | o `prefix` (ex.: `CA1-T7`) é só exibição |
| `description` / `markdownDescription` | `description` | ≤ 80.000 chars; render de Markdown `[A VERIFICAR]` |
| `status` / `statusRaw` / `statusColor` | `status.{name, is_closed_type, color_hexcode}` | ver abaixo |
| `priority` | `priority` | ver abaixo |
| `dueDate` / `startDate` | `end_date` / `start_date` | ISO-8601 UTC na escrita |
| `assignees` | `owners_and_work.owners[]` | escrita por e-mail ou zpuid |
| `parent` | `parental_info.parent_task_id` | composto com o mesmo projeto |
| `projectId` / `projectName` | `project.{id,name}` | |
| `tags` | `tags[]` | leitura ok; escrita exige ids `[A VERIFICAR]` |
| `url` | montado pelo adapter | `[INFERIDO]`, sem link na API |

### Status Mapping

| Interface | Nomes aceitos no Zoho (normalizados) |
|---|---|
| `backlog` | backlog |
| `todo` | open, to do, aberta, a fazer |
| `in_progress` | in progress, em andamento |
| `review` | in review, review, em revisão |
| `done` | closed, completed, done, concluída, fechada |
| `closed` | — escrito como `done` (perda registrada); nunca lido |
| `canceled` | cancelled, canceled, cancelada |

Sem casamento na leitura: `is_closed_type=true` → `done`, `false` → `todo`. Na escrita, sem casamento
→ erro listando os status do portal. Os status podem depender do **layout** do projeto (P4).

### Priority Mapping

| Interface → Zoho | Zoho → Interface |
|---|---|
| `urgent` → `high` ⚠️ perda | `high` → `high` |
| `high` → `high` | `medium` → `normal` |
| `normal` → `medium` | `low` → `low` |
| `low` → `low` | `none` → `undefined` |

---

## 🚨 Erros e renovação de token

**Medido em 2026-09-30** contra os endpoints públicos, com credenciais inválidas:

| Situação | Resposta real | Tratamento |
|---|---|---|
| Refresh com credencial inválida | **HTTP 200** + `{"error":"invalid_client"}` | checar o **corpo**; erro sem vazar segredo |
| Excesso de pedidos de token | **HTTP 400** + `{"error":"Access Denied","error_description":"You have made too many requests continuously…"}` | mensagem própria de rate limit (não "confira credenciais") |
| API sem header de auth | 401, `title: INVALID_TICKET` | bug do adapter: **não** renova, lança |
| API com token inválido/expirado | 401, `title: INVALID_OAUTHTOKEN` | renova **uma vez** e repete |
| Task inexistente | 404, `{"error":{"code":6404,"message":"Resource Not Found"}}` (doc) | segundo formato de erro: tratado por `code`/`message` |
| Rate limit | 429 + `Retry-After` (doc) | erro com o tempo de espera; bloqueio de 10 min por endpoint |

- **Limites de token** (doc de OAuth da Zoho, <https://www.zoho.com/developer/oauth/token-limits.html>,
  não a doc do Projects): no máximo 10 pedidos de access token a cada 10 min **por app** (client) e 20 refresh
  tokens por usuário. Passar de 20 invalida o **mais antigo**, que pode ser o de outra integração. O cache (~55 min) evita
  renovar por chamada.
- **Grant code:** validade curta, escolhida no console. Trocar logo após gerar.

---

## 💬 Formatação de Conteúdo

Descrição e comentários vão como texto. O Zoho armazena rich text, e **se o Markdown renderiza não foi
verificado** `[A VERIFICAR]`. Até a validação real, prefira texto simples em `addComment`; os
comentários formatados com Unicode (padrão ClickUp) degradam bem.

**LGPD:** o repo proíbe tratar dado de paciente ou consumidor. Nada de dado pessoal sensível em nome,
descrição ou comentário de task. Se o DC do portal for fora do Brasil, os dados das tasks fazem
transferência internacional (pergunta aberta `Q_LGPD_ZOHO` no plano-grafo).

---

## ⚠️ Notas Operacionais

1. **Um portal por repo** (`ZOHO_PORTAL_ID`) e **um provider por vez** (decisão do maestro).
2. **Sem MCP:** `TASK_MANAGER_TRANSPORT=mcp` cai para `api` com aviso.
3. **`searchTasks` sem `projectId` itera todos os projetos**, sequencialmente, por causa do rate limit.
   Prefira passar `projectId`.
4. **Tags por nome** não são suportadas na escrita (a API pede ids).
5. **Deleção é permanente** (`DELETE` → 204). Diferente do Linear, que arquiva.

---

## 🧪 Exemplos de Uso

```typescript
const tm = getTaskManager();   // TASK_MANAGER_PROVIDER=zoho-projects

const task = await tm.createTask({
  name: 'Integrar chamado GLPI #1234',
  projectId: '1752587000000097024',
  priority: 'high',
  dueDate: '2026-10-15'
});
// task.id === '1752587000000097024.1752587000000099001'  (id composto)

await tm.createSubtask(task.id, { name: 'Mapear campos' });
await tm.addComment(task.id, 'Iniciado via /engineer:start');
await tm.updateStatus(task.id, 'in_progress');   // resolve status.id pelo nome
```

---

## 📚 Referências

- ADR de mapeamento (SSOT das decisões): [adr-zoho-projects-task-mapping](../../../../docs/technical-context/decisions/adr-zoho-projects-task-mapping.md)
- KB da API: [zoho-projects-api](../../../../docs/knowledge-base/platforms/zoho-projects-api.md)
- Doc oficial V3: <https://projects.zoho.com/api-docs>
- Interface: [interface.md](../interface.md) · Tipos: [types.md](../types.md) · Detector: [detector.md](../detector.md)

---

**Versão**: 1.0.0
**Criado em**: 2026-09-30
