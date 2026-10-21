---
title: "[Feature] Agente de IA para análise de prorrogação de boletos (Financeiro)"
type: feature
status: refined
origin: demanda real do Financeiro da GMill
collected_at: 2026-10-21
collected_by: /product:collect
refined_at: 2026-10-21
zoho_task_id: "2723266000000174001"  # MVP modo sombra, portal PESSOAL de teste, tasklist Financeiro — Prorrogação de Boletos
story_points: 13  # MVP modo sombra após refinamento (faixa 13–20); era 8
research: docs/product-ideas/2026-10-21-prorrogacao-boletos-pesquisa-inicial.md
---

# [Feature] Agente de IA para análise de prorrogação de boletos (Financeiro)

> **Histórico:**
> - Coletada via `/product:collect` em 2026-10-21.
> - No mesmo dia, o recorte MVP (modo sombra) foi criado no Zoho via `/product:task`, na task
>   `2723266000000174001`. O Zoho configurado é o portal pessoal de teste (ver `Q_ZOHO_CREDENTIALS`
>   em `docs/onion/graph/zoho-pendencias.kg.yaml`), não o da GMill.
> - Refinada via `/product:refine` com pesquisa de mesa, também em 2026-10-21.
>
> A base factual (com fontes) e o roteiro de perguntas ao Financeiro estão em
> [`2026-10-21-prorrogacao-boletos-pesquisa-inicial.md`](2026-10-21-prorrogacao-boletos-pesquisa-inicial.md).

## POR QUE

- **Hoje é manual.** As farmácias clientes pedem prorrogação de boleto e cada pedido é analisado à mão.
  O Financeiro quer análises rápidas, sempre pelo mesmo critério e financeiramente seguras.
- **Risco de crédito está alto.** A inadimplência de empresas está em nível recorde (Serasa, julho de
  2026) e há redes de farmácias em recuperação judicial. Prorrogar sem critério vira perda.
- **Prorrogar tem pegadinhas** que uma análise manual esquece:
  - restrições do banco (título vencido, limite de alterações, protesto);
  - tarifa de instrução;
  - risco jurídico do encargo cobrado.

## O QUE

**MVP em modo sombra.** O agente **recomenda** e o analista decide e executa no banco. Será um
**serviço separado do ERP**.

### Entradas (manual no MVP, preenchidas pelo analista)
- **Título:** CNPJ do cliente, banco/carteira, nosso número/seu número, valor, vencimento original e
  atual, nova data pedida.
- **Situação bancária:** status atual (em aberto, vencido, liquidado, baixado), **quantidade de
  alterações já feitas** e se há **protesto ou negativação** em curso.
- **Cliente:** histórico de pagamentos (agregado), limite de crédito, se está bloqueado e se há
  recuperação judicial no grupo econômico.
- **Taxa:** taxa de referência do dia (CDI ou Selic, informada no MVP) e spread aplicável.
- **Nada de dado de pessoa física** (sócio, CPF). Ver LGPD em `docs/business-context/04-operations/customer-communication.md`.

### Saída
- **Decisão:** Aprovar / Aprovar com encargos / Reprovar / Encaminhar para gestor
- **Rota bancária:** alterar vencimento **ou** baixar e reemitir (quando o título já venceu)
- **Novo vencimento:** data
- **Encargo financeiro:** valor e %, com a fórmula, a taxa e o spread usados
- **Justificativa:** até 3 linhas

### Regras duras (determinísticas, nunca decididas pelo LLM)
1. Cliente bloqueado, inadimplente recorrente, com recuperação judicial no grupo ou com pedido fora
   da política (prazo ou valor) **nunca** recebe "aprovar".
2. Título liquidado ou baixado: não há o que prorrogar.
3. Título vencido: não recomendar "alterar vencimento". A rota é "baixar e reemitir" (o Itaú rejeita
   a alteração de título vencido).
4. Limite de alterações atingido (3 no Itaú `[A VERIFICAR]` por banco): encaminhar.
5. Protesto ou negativação em curso: a recomendação inclui "sustar antes". Na dúvida, encaminhar.
6. Não alterar valor e vencimento na mesma instrução.
7. A tarifa de instrução entra no custo da prorrogação.

### Fora do MVP
- Execução da instrução no banco (CNAB ou API).
- Integração com o ERP.
- Canal integrado (WhatsApp, portal).
- Leitura automática do CDI.
- Autonomia de decisão.

### Métricas
- Tempo médio de análise.
- % de recomendações aceitas sem alteração. É a prévia do "% de decisões automáticas".
- Inadimplência após prorrogação.
- Encargo recuperado.

## COMO

- **Arquitetura:** o motor determinístico (cálculo + regras duras + matriz de decisão) produz a decisão
  e o encargo. O LLM **só** redige a justificativa, e uma guarda verifica que ele não altera nenhum
  campo do motor.
- **Fórmula configurável**, com um único método escolhido pelo Financeiro:
  - **simples:** `valor × taxa_mensal × dias_corridos / 30`
  - **composta:** `valor × [(1+CDI)^(du/252) × (1+spread)^(dc/365) − 1]`
  - A taxa é congelada na data da concessão. Fórmula, taxa e spread ficam no registro da decisão.
- **Regras bancárias por banco:** começar com 1 banco (o de maior volume, provavelmente com regras
  de Itaú `[A VERIFICAR]`) numa tabela de regras por banco. Os códigos e limites variam entre bancos.
- **Status confiável vem do banco**, via retorno CNAB ou webhook. O ERP só sabe depois de processar o
  retorno. No MVP o analista informa o status. Na integração futura, consultar a posição antes de
  recomendar.
- **Modo sombra:** registrar a recomendação contra a decisão real do analista. Usar ~30 casos
  históricos sem dado de pessoa física como regressão.

### Subtasks (Zoho)
- **S1 Spike: formalizar a política com o Financeiro e o jurídico.** É gate: S2 não começa antes da
  decisão sobre o modelo de encargo.
- **S2 Motor determinístico.** Recomendado dividir em:
  - S2a: cálculo (fórmula configurável, taxa congelada, CDI manual);
  - S2b: checagens bancárias.
- **S3 Agente de recomendação** (justificativa e guarda).
- **S4 Entrada manual, confirmação humana e modo sombra.** Inclui os campos de situação bancária.
- **S5 Métricas e relatório go/no-go.**

### Em aberto (dependem do Financeiro e do jurídico; ver o roteiro na pesquisa)
- `[TO BE COMPLETED]` Bancos e canal de registro e instrução (CNAB 240/400, API ou internet banking).
  Isso define as regras do S2b.
- `[TO BE COMPLETED]` **Fonte dos dados:** onde está o status (ERP, banco, planilha) e se o ERP tem
  API ou exportação. Confirmado: o agente é um **serviço separado**.
- `[TO BE COMPLETED]` **Parâmetros da política:** limites de prazo e valor, spread, definição de
  "inadimplente recorrente" e "bloqueado", e alçadas.
- `[TO BE COMPLETED]` A política escrita já existe ou precisa ser formalizada primeiro?
- `[TO BE COMPLETED]` **Reprovar × Encaminhar:** quando o agente reprova e quando encaminha?
- `[TO BE COMPLETED]` ⚖️ **Modelo de encargo frente à Lei de Usura** (12% ao ano para quem não é
  instituição financeira; Selic em 13,75%):
  - CDI + spread é viável?
  - As alternativas são taxa fixa, preço a prazo, encargo dentro do teto ou estrutura via
    instituição financeira.
  - É preciso aceite documentado (termo ou aditivo)?
- `[TO BE COMPLETED]` **LGPD:** confirmar que histórico e entrada não carregam dado de sócio pessoa
  física.
- `[TO BE COMPLETED]` **Canal:** por onde o pedido entra (WhatsApp, SAC, representante, portal
  Millcompras) e como a recomendação volta ao analista.

## 📊 ESTIMATIVA DE STORY POINTS

**Atual:** 13 pontos (MVP em modo sombra; faixa 13–20)

**Histórico de mudanças:**

| Data | Estimativa | Mudança | Motivo |
|------|------------|---------|--------|
| 2026-10-21 | 13 pontos | — | Criação inicial (`/product:collect`), escopo completo |
| 2026-10-21 | 8 pontos | −5 | Recorte MVP modo sombra (`/product:task`): sem ERP, sem canal, sem autonomia |
| 2026-10-21 | 13 pontos | +5 | Refinamento (`/product:refine`): dependência jurídica, checagens bancárias, fórmula configurável |

**Subtasks:** S1 = 5 (era 3) · S2 = 8 (era 5) · S3 = 2 · S4 = 3 · S5 = 2 · **soma 20** (era 15)

**Análise atual:**
- **Complexidade: alta.** As regras bancárias se multiplicam no motor: vencido, limite de alterações,
  protesto, tarifa e fórmula.
- **Risco: alto.** A Lei de Usura pode invalidar o modelo CDI + spread, e uma recomendação errada
  vira cobrança indevida.
- **Incerteza: alta.** Banco, canal e modelo de encargo dependem do Financeiro e do jurídico.

**Fatores que influenciaram a estimativa:**
- S1 ganhou a dependência do jurídico e o levantamento de bancos e canais. O roteiro de 23 perguntas
  já pronto atenua isso.
- S2 ganhou as checagens bancárias obrigatórias e a fórmula configurável com taxa congelada.
- A soma de 20 está no limite de épico: **dividir S2** em S2a (cálculo, ~3) e S2b (checagens, ~5).
- Para conter o escopo: começar com 1 banco e informar o CDI manualmente no MVP.
