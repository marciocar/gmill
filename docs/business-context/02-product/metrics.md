# Métricas e KPIs — Grupo GMill

> **Modo infer-from-evidence.** Só os indicadores da seção 1 são públicos. As seções 2 em diante são
> **o painel que este negócio precisa ter**, derivado do padrão do atacado farmacêutico — não o painel
> que a GMill comprovadamente tem. `[INFERIDO]` salvo onde citada a fonte.
>
> Pendência nº 6 do [index](../index.md#pendências-de-validação): descobrir quais destes têm meta formal.

---

## 1. O que é público e verificável

| Métrica | Valor | Ano | Fonte |
|---|---|---|---|
| Receita | R$ 472,7 mi | 2021 | S2 |
| Crescimento | +32% | 2021 | S2 |
| Receita | **R$ 750 mi** | 2025 | S4 |
| Crescimento | **+40%** | 2025 | S4 |
| **Meta de receita** | **R$ 1 bi** | 2026 | S4 |
| Farmácias ativas | ~7.000 | 2026 | S4 |
| PDVs atendíveis | 8.000+ | 2021 | S3 |
| SKUs | 7.000 (2022) → 9.000 | 2022-… | S2, busca |
| Unidades de genéricos/mês | >10 milhões (~10% do mercado nacional) | 2022 | S2 |
| Share de canal | **franquias + associações = 60% da receita** | 2026 | S4 |
| Colaboradores | 500-600 (grupo) / 201-500 (LinkedIn) | 2022-2026 | S2, S6 |
| Estrutura comercial (Millenium) | 130+ representantes, 8 supervisores, 2 coordenadores distritais, 2 gerentes, ~250 colaboradores | — | S8 |
| Área construída | >15.000 m² em 3 CDs | — | busca |
| Glassdoor | 3,9/5 geral · 4,1/5 Serra · 69% recomendam | — | S8 |

### Métricas derivadas (cálculo nosso sobre as fontes)

| Derivada | Valor | Leitura |
|---|---|---|
| **Receita por farmácia ativa** | R$ 750 mi ÷ 7.000 ≈ **R$ 107 mil/ano** (~R$ 9 mil/mês) | ticket típico de farmácia independente; confirma que a base é pulverizada |
| **Receita por colaborador** | R$ 750 mi ÷ ~500 ≈ **R$ 1,5 mi/ano** | alto — coerente com atacado (baixa margem, alto volume por cabeça) |
| **Receita por representante** | R$ 750 mi ÷ 130 ≈ **R$ 5,8 mi/ano** | a carteira média vale ~R$ 480 mil/mês. **Mede o risco de perder um representante** |
| **Gap da meta 2026** | R$ 250 mi (+33%) | ver R2 em [strategy](strategy.md#6-riscos-estratégicos) |

> ⚠️ Derivadas misturam recortes (receita citada como "Millenium" em S4 × colaboradores do grupo em
> S2/S6). Servem como **ordem de grandeza**, não como número de gestão.

---

## 2. KPIs comerciais (o núcleo do atacado farmacêutico)

| KPI | Definição | Por que importa aqui |
|---|---|---|
| **Positivação** | % de clientes da carteira que compraram no período | métrica-mãe do canal; base de comissionamento típica |
| **Clientes ativos** | compraram nos últimos 30/60/90 dias | a diferença 8.000 → 7.000 (K2) vive aqui |
| **Ticket médio por pedido** | receita ÷ nº de pedidos | mede se o cliente consolida ou fraciona |
| **Itens distintos por pedido (mix)** | SKUs por pedido | **melhor proxy de share of wallet**: mix caindo = cliente comprando o resto em outro lugar |
| **Frequência de compra** | pedidos/cliente/mês | a derivada negativa antecede o churn em meses |
| **Share of wallet** | % da compra total do cliente | o número que ninguém tem e todos precisam |
| **Cobertura de visita** | % da carteira visitada no ciclo | produtividade do campo |
| **Novos clientes / reativados** | — | crescimento sem depender de share |

## 3. KPIs de serviço — onde a relação se ganha ou se perde

| KPI | Definição | Meta de referência do setor `[INFERIDO]` |
|---|---|---|
| **Fill rate (nível de serviço)** | % de itens do pedido atendidos | ≥ 95% |
| **Taxa de corte** | % de linhas cortadas | ≤ 5% |
| **Ruptura (stockout)** | % de SKUs em falta | ≤ 3% nos itens de curva A |
| **OTIF** | no prazo **e** completo | ≥ 90% |
| **Lead time** | pedido → entrega | D+1 na praça com CD |
| **Taxa de devolução/avaria** | — | ≤ 1% |
| **Acurácia de inventário** | — | ≥ 99% |

> 🎯 **Se for para instrumentar um único indicador primeiro, é o fill rate por curva ABC.** Ele
> explica simultaneamente satisfação (MV2 da [jornada](../01-customer/journey.md)), perda de receita
> imediata e migração de item para o concorrente. No atacado farmacêutico, corte é churn em câmera lenta.

## 4. KPIs financeiros

| KPI | Por que |
|---|---|
| **Margem de contribuição por SKU/cliente/canal** | é o alvo declarado do projeto de pricing (S11) |
| **Margem bruta** | fina por natureza no atacado; qualquer ponto conta |
| **Giro de estoque / cobertura em dias** | capital imobilizado é o maior ativo circulante do negócio |
| **DSO (prazo médio de recebimento)** | crédito é alavanca de venda e risco ao mesmo tempo |
| **Inadimplência por canal** | independente × rede têm perfis de risco distintos |
| **Custo logístico sobre receita** | mede se a verticalização (A2) está pagando |

## 5. KPIs do projeto de pricing — a única iniciativa com resultado creditado

O case (S11) reporta ganhos **qualitativos**: aumento de margem de contribuição em produtos
estratégicos, incremento de vendas, melhora de margem de lucro e maior participação de mercado —
**sem percentuais divulgados**.

Para transformar isso em painel:

| Métrica | O que responde |
|---|---|
| % de SKUs precificados por algoritmo vs. manual | cobertura real da automação |
| tempo de ciclo de reprecificação | agilidade (era o gargalo do "manual") |
| desvio preço praticado × preço recomendado | **aderência** — mede se o campo respeita o algoritmo |
| margem de contribuição antes/depois por família | o ganho, isolado |
| win rate em itens de alta elasticidade | se o preço está ganhando pedido |

> 💡 **A métrica mais reveladora é o desvio entre preço recomendado e praticado.** Ela mede a
> distância entre a arquitetura de pricing e a realidade do campo — e é onde projetos assim costumam
> vazar valor silenciosamente.

## 6. KPIs digitais (portal e marketplace)

| KPI | Status |
|---|---|
| % da receita originada no portal Millcompras | `[TO BE COMPLETED]` — **pendência nº 4, a mais importante** |
| clientes ativos no portal ÷ clientes ativos totais | `[TO BE COMPLETED]` |
| pedidos digitais sem toque humano | `[TO BE COMPLETED]` |
| adesão às campanhas (`Promo Dia Mill`, `Promo A a Z`) | `[TO BE COMPLETED]` (S1 confirma que existem) |
| farmácias ativas no Meds / pedidos / tempo de entrega | ⚠️ depende de o Meds existir (S3) |

## 7. KPIs de pessoas — que aqui são KPIs de receita

Justificativa em [voice-of-customer §3](../01-customer/voice-of-customer.md): se a relação comercial
mora no representante, **turnover de campo é churn com atraso**.

| KPI | Fonte/estado |
|---|---|
| turnover da força de vendas | `[TO BE COMPLETED]` |
| variação de positivação após troca de representante | `[TO BE COMPLETED]` — mede dependência pessoal da carteira |
| eNPS / recomendação | 69% recomendam (Glassdoor, S8) |
| tempo de preenchimento de vaga | Portal de Talentos em Senior (S12) |

---

## 8. Painel mínimo sugerido (se fosse começar amanhã)

Dez números, uma tela, revisão semanal:

1. Receita acumulada × meta de R$ 1 bi
2. Clientes ativos (30d) e positivação
3. **Fill rate curva A**
4. Taxa de corte por causa (compra / previsão / logística)
5. Ticket médio e itens distintos por pedido
6. Margem de contribuição por canal (independente × rede × central)
7. Desvio preço praticado × recomendado
8. % de receita originada no portal
9. DSO e inadimplência por canal
10. Top 20 clientes em queda de frequência — **a lista de churn antes do churn**

---

**Fontes**: S1, S2, S3, S4, S6, S8, S11, S12 — ver [tabela completa](../index.md#fontes-consultadas).
