# Voz do cliente — Grupo GMill

> ⚠️ **Esta é a camada mais fraca deste contexto.** Não existe base pública de depoimento de cliente
> da GMill: sem reviews de farmácia, sem NPS publicado, sem estudo de caso com cliente nomeado. O que
> existe é **voz do colaborador** (Glassdoor/Indeed) e **voz da empresa sobre si mesma** (imprensa,
> redes). Tudo abaixo é `[INFERIDO]` salvo onde a fonte estiver citada.
>
> **Prioridade de validação alta** — ver [index § Pendências](../index.md#pendências-de-validação).

---

## 1. Léxico do setor — como o cliente realmente fala

Sistemas de IA que atendam farmácias precisam entender o vocabulário do balcão, que **não é** o
vocabulário de supply chain.

| Nós dizemos | A farmácia diz | Observação |
|---|---|---|
| ruptura / stockout | **"faltou"**, "não veio", "cortou" | "corte" é o termo do pedido parcialmente atendido |
| fill rate | "veio completo?" | ninguém usa a expressão em inglês no balcão |
| SKU | "item", "apresentação" | "apresentação" tem sentido técnico (dosagem/embalagem) |
| positivação | — | termo **do vendedor**, não do cliente |
| genérico / similar / referência | mesmos termos | distinção legal e comercialmente central |
| prazo de pagamento | **"quantos dias?"** | frequentemente decide a compra |
| bonificação | "bonificado", "leva 10 paga 9" | mecânica promocional dominante |
| verba / rebate | "acordo", "campanha" | linguagem de rede, não de independente |
| pedido mínimo | "quanto tem que fechar?" | barreira real para o independente pequeno |

**Regra para IA:** responda no registro do interlocutor. Para P1 (dono de farmácia), *"faltaram 3 itens
do seu pedido"* comunica; *"fill rate de 87%"* não.

---

## 2. Temas recorrentes esperados no feedback `[INFERIDO]`

Derivados das dores estruturais do canal — **a validar com SAC/Ouvidoria real**. O grupo mantém
canal de **Ouvidoria** publicado (`gmill.com.br/portal/ouvidoria`), o que indica que existe registro
formal de reclamação — provavelmente a melhor fonte interna para substituir este capítulo inteiro.

| Tema | Hipótese de frequência | Como se manifesta |
|---|---|---|
| **Corte de pedido / ruptura** | 🔴 alta | "pedi 20 itens, vieram 14" |
| **Prazo e janela de entrega** | 🔴 alta | "prometeram terça, chegou quinta" |
| **Divergência de preço** | 🟡 média | preço do portal ≠ preço do representante — risco direto do modelo de canal duplo |
| **Crédito/limite** | 🟡 média | bloqueio em momento de necessidade |
| **Avaria e troca** | 🟡 média | logística reversa é ponto cego clássico |
| **Nota fiscal / tributário** | 🟢 baixa mas crítica | erro trava a entrada da mercadoria na farmácia |
| **Usabilidade do portal** | ❓ desconhecida | nenhum dado |

> ⚠️ **Risco de canal apontado pela própria estrutura:** havendo força de vendas grande (130+
> representantes, S8) **e** portal self-service (S1), a **divergência de preço entre canais** é quase
> inevitável sem governança única de pricing. A GMill mitigou isso do lado certo — centralizou
> precificação em plataforma algorítmica (S11) —, mas o sintoma merece monitoramento explícito.

---

## 3. Voz do colaborador (evidência real disponível)

Única voz efetivamente medida nas fontes públicas.

| Métrica | Valor | Fonte |
|---|---|---|
| Glassdoor — Millenium Comercial (geral) | **3,9/5** (38 avaliações) | S8 |
| Glassdoor — unidade Serra | **4,1/5** (16 avaliações) | S8 |
| Recomendam a empresa | **69%** | S8 |
| Remuneração e benefícios | 3,7/5 | S8 |

**Padrão qualitativo citado (Indeed):** *"a empresa é uma grande distribuidora de medicamentos, mas
mesmo sendo tão grande todos os setores são extremamente unidos; promove várias ações sociais entre
outros eventos"* (S8).

**Leitura:** clima acima da média do setor logístico, com **união entre áreas e ações sociais** como
tema espontâneo — e **remuneração como o ponto mais fraco** (3,7 é o menor subíndice). Isso conversa
com o discurso do CEO, que credita o salto de 2025 primeiro a **pessoas** e à reestruturação
comercial, antes da tecnologia (S4).

**Por que isso pertence ao contexto de negócio, e não só ao RH:** num modelo em que **a relação
comercial mora no representante**, clima e retenção do campo **são** variável de receita. Turnover de
representante é churn de cliente com atraso.

---

## 4. Voz da empresa sobre si mesma

| Canal | Mensagem | Fonte |
|---|---|---|
| **Slogan** | "No seu tempo" | S6 |
| **Bio Instagram** | "Distribuição no mercado farmacêutico! • Conectamos resultados diariamente" | S7 |
| **Valor citado** | "pessoas cuidam de pessoas" (pauta de Setembro Amarelo) | S6 |
| **Pauta social** | Setembro Amarelo, QualificarES (visitas técnicas de alunos à logística), ações sociais | S6, S8 |
| **Evento próprio** | "Feirão Legado" | S7 |
| **Marco celebrado** | 28 anos da Millenium (jul/2026) | S7 |

**Tensão a resolver:** *"No seu tempo"* promete **conveniência do cliente**, mas a comunicação
pública é quase toda **interna** (vagas, cultura, datas). O grupo comunica muito bem para quem
trabalha nele e quase nada para quem compra dele. Ver
[messaging-framework](../04-operations/messaging-framework.md).

---

## 5. O que fazer para esta página deixar de ser hipótese

Em ordem de retorno sobre esforço:

1. **Exportar 12 meses da Ouvidoria/SAC** e classificar por tema — substitui a tabela da seção 2 por dado.
2. **NPS por canal** (representante × televendas × portal) — mede a fricção de canal duplo.
3. **20 entrevistas**: 10 clientes ativos crescentes, 5 em queda, 5 inativos recentes. Os que caíram
   contam mais do que os satisfeitos.
4. **Motivo de corte por SKU** — transforma "faltou" em causa raiz (compra, previsão ou logística).

---

**Fontes**: S1, S4, S6, S7, S8, S11 — ver [tabela completa](../index.md#fontes-consultadas).
