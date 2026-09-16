# 🧅 gmill — Claude Code Rules

## 🎯 Identidade

Este é o repositório de contexto do **Grupo GMill** — distribuição farmacêutica atacadista
(Serra/ES) —, adotado pelo Sistema Onion em modo `greenfield`, carimbado `role: hub`.
O framework `.claude/` orquestra o ciclo de desenvolvimento com Claude Code nas dimensões
**produto**, **engenharia** e **compliance/governança**.

- **Papel `hub` (Camada 2).** Este repo é **autoridade de adoção local**: pode adotar e atualizar
  os próprios projetos (`/meta:adopt <projeto>`, `/meta:adopt --update <projeto>`). **Não** ganha a
  autoria do framework (Camada 1, só o core) nem a federação cross-empresa (Camada 3).
- **Cadeia:** core (`source`) → **este repo (`hub`)** → projetos adotados (`adopted`).
- **Proveniência do core** em `.claude/.onion-version` (`source_commit`); atualização via
  `/meta:adopt --update` rodado a partir do core (vendor-branch 3-way).

## 📊 O que já existe aqui — e o que ele ainda NÃO é

`docs/business-context/` **precede a adoção** e é a SSOT de negócio deste repo (15 documentos:
cliente, produto, mercado, operações). Ele foi gerado em **modo infer-from-evidence**, a partir de
fontes públicas, **sem entrevista com stakeholder** — e **ainda não foi validado por ninguém de
dentro da GMill**. Leia o aviso e a lista de pendências no próprio
[`docs/business-context/index.md`](docs/business-context/index.md) antes de usar qualquer número
para decisão. Marcações `[INFERIDO]` e `[TO BE COMPLETED]` são parte do contrato: não as apague ao
editar, promova-as com fonte.

## ⚖️ Escopo regulatório ATIVO (não é menção de mercado)

O contexto declara operação sob **AFE/AE ANVISA**, **SNCM** (rastreabilidade), **Portaria 344**
(controlados) e **cadeia fria**; e `04-operations/customer-communication.md` fixa a regra de que
**dado de paciente/consumidor é sensível na LGPD e não deve ser tratado** — se aparecer numa
conversa, não registrar nem repetir. Isso vale para qualquer artefato gerado aqui.

`docs/compliance-context/` está como **template vazio** (a adoção rodou em modo `greenfield` por
escolha). Para populá-lo com os frameworks aplicáveis: `/docs:build-compliance-docs`.

## 🔌 Task Manager — Detecção e Roteamento

Provider-agnóstico via SDAAL (`.claude/utils/task-manager/`). **Antes de operar com tasks**, carregue
o `.env` e leia `TASK_MANAGER_PROVIDER` (`jira` | `clickup` | `asana` | `linear` | `none`) +
`TASK_MANAGER_TRANSPORT` (`api` default | `mcp`). Delegue ao especialista do provider ativo
(`@jira-specialist`, `@clickup-specialist`) ou ao `@task-specialist`. Variável ausente → avisar em
pt-BR + sugerir `/meta:setup-integration`; nunca inventar valores.

## 🐙 Forge — Operações de Host Remoto

Abstraído via SDAAL (`.claude/utils/forge/`). `/git/*` e `/engineer:pr` **nunca** chamam `gh`/API
direto — passam pelo adapter. Git local (branch/merge/tag/push) é `git` direto.

## 📝 Diretrizes de Linguagem

Autoridade canônica: skill **`language-standards`**.
- **Chat, comentários, docs, READMEs, mensagens ao usuário**: Português brasileiro (pt-BR)
- **Código, variáveis, funções, nomes de arquivo/branch, logs**: Inglês
- **Commits**: prefixo Conventional em inglês + assunto e corpo em pt-BR

## 🛠️ Padrões Técnicos

- Comandos: `.claude/commands/` por categoria · Agentes: `.claude/agents/<categoria>/`
- Sessões: `.claude/sessions/<feature-slug>/` (kebab-case)
- **Spec as Code** — contextos L1+ deste repo (o L0 do framework vive em `docs/meta-specs/`):
  `docs/business-context/` (existente, SSOT) · `docs/technical-context/` · `docs/compliance-context/`
  · `docs/knowledge-base/`
- **Branches**: produto na principal; a evolução do framework chega pela branch de integração
  (ver `.claude/.onion-version`). A instalação entrou por `onion/adopt`; `onion/vendor` é a
  fonte-de-merge do `--update`.

## ⚠️ Setup após clonar (o gate NÃO viaja no clone)

O gate determinístico é um hook git nativo em `.githooks/pre-commit`, ativado por `core.hooksPath` —
**config local, não objeto git**. Um clone fresco traz o hook **inerte**. Quem clonar roda uma vez:

```bash
git config core.hooksPath .githooks
```

Conferir por **execução**, não por existência de arquivo: `bash .githooks/pre-commit`.

## 🚀 Entrada

- `/warm-up` — carrega o contexto do projeto
- `/onion` — ponto de entrada inteligente
- `/product:warm-up` — contexto de negócio (é onde está o material vivo deste repo)
- `/meta:co-evolve` — co-evolução com o core
