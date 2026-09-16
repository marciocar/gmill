# Business Context — Grupo GMill

> Contexto de negócio do **Grupo GMill** (distribuição farmacêutica atacadista, Serra/ES).
> Gerado por `/docs:build-business-docs` em **2026-09-16**, em **modo infer-from-evidence**:
> sem entrevista com stakeholder, tudo derivado de fontes públicas. Toda leitura nossa que
> extrapola a fonte está marcada `[INFERIDO]`; o que nenhuma fonte cobre está `[TO BE COMPLETED]`.

⚠️ **Este documento ainda não foi validado por ninguém de dentro da GMill.** Antes de usá-lo para
decisão, rode a lista de [Pendências de validação](#pendências-de-validação).

---

## Perfil de negócio

| Campo | Valor | Fonte |
|---|---|---|
| **Grupo** | Grupo GMill | S1, S6 |
| **Razão social (matriz)** | Millenium Comercial & Logop do Gmill Distribuição Ltda | S9 |
| **CNPJ (matriz)** | 02.632.609/0001-09 | S9 |
| **Sede** | Rua Basílio da Gama, 56 — Jardim Limoeiro, Serra/ES, 29.164-083 | S9 |
| **Atividade principal** | Comércio atacadista de medicamentos e drogas de uso humano (CNAE 4644-3/01) | S9 |
| **Formação do grupo** | março/2021, pela integração de LogOp + Millenium Comercial | S2, S4 |
| **Empresa mais antiga** | Millenium Comercial — desde **1998** (28 anos em jul/2026) | S7 |
| **Faturamento** | R$ 472,7 mi (2021) → **R$ 750 mi (2025, +40%)** → **meta R$ 1 bi (2026)** | S2, S4 |
| **Clientes** | ~7.000 farmácias ativas (2026); discurso de 2021 falava em 8.000 PDVs | S3, S4 |
| **Praça** | ES + RJ (2026); filiais CNPJ também em Conceição do Jacuípe/BA | S4, S9 |
| **Porte** | 201-500 colaboradores (LinkedIn); 500-600 em fontes de 2022 | S6, S2 |
| **Slogan** | "No seu tempo" | S6 |
| **Categoria de mercado** | Distribuidor farmacêutico **regional** de full-line + genéricos | S2, S10 |

### O que o grupo é, em uma frase

**O GMill é um distribuidor farmacêutico regional do Sudeste/Nordeste que vende para farmácias
independentes e redes associativistas, e que vem se verticalizando** — marketplace (Meds), logística
própria (BeeBee, transportadora, operador logístico) e indústria de suplementos (Médina).

### O que o grupo **não** é

- **Não é varejo.** O consumidor final não é cliente. O cliente é a farmácia.
- **"Millcompras" não é e-commerce de compras gerais.** É o **portal B2B de pedidos** das farmácias
  clientes — tem login, catálogo por marca e campanhas promocionais (`Promo Dia Mill`, `Promo A a Z`). (S1)
- **Não é nacional.** É regional, e essa é a posição competitiva escolhida, não uma limitação
  transitória. `[INFERIDO]` — ver [competitive-landscape](03-market/competitive-landscape.md).

---

## As empresas do grupo

| Empresa | O que faz | Status na evidência |
|---|---|---|
| **Millenium Comercial** (1998) | distribuição de similares e genéricos; é a marca comercial dominante | ✅ viva e crescendo (S4, S7) |
| **LogOp** (ex-Sudeste Farmacêutica, 2016) | produtos de saúde e correlatos (não-medicamento) | ✅ no nome da razão social (S9); sem cobertura recente |
| **BeeBee** | startup de logística, "Uber de fretes" — entrega sob demanda sem frota fixa | ⚠️ só cobertura de 2021 (S3); também aparece como **marca de produto** no catálogo (S1) |
| **Meds** | marketplace que conecta consumidor à farmácia independente mais próxima, entrega em até 30 min | ⚠️ só cobertura de 2021 (S3) — **status atual desconhecido** |
| **Transportadora própria** | integração logística | ✅ citada em 2026 (S4) |
| **Operador logístico** | operações logísticas internas / ambição de contract logistics | ✅ citada em 2026 (S4); ambição declarada em 2022 (S2) |
| **Médina** | indústria de suplementos, Elói Mendes/MG | ✅ citada em 2026 (S4) — **verticalização para indústria** |

---

## Camadas

### 1 — Cliente ([`01-customer/`](01-customer/))
- [**Personas**](01-customer/personas.md) — 5 personas: farmácia independente, comprador de rede/franquia, central de associativismo, e as duas internas (comercial de campo, TI/diretoria)
- [**Jornada**](01-customer/journey.md) — prospecção → primeira compra → positivação recorrente → crescimento de share of wallet → churn
- [**Voz do cliente**](01-customer/voice-of-customer.md) — terminologia do setor e sinais de satisfação/insatisfação

### 2 — Produto ([`02-product/`](02-product/))
- [**Estratégia**](02-product/strategy.md) — as 5 apostas visíveis e a tese de valor
- [**Métricas**](02-product/metrics.md) — KPIs do atacado farmacêutico e os que a GMill declara olhar
- Capacidades: [Portal Millcompras](02-product/features/millcompras-portal.md) · [Marketplace Meds](02-product/features/meds-marketplace.md) · [Pricing inteligente](02-product/features/pricing-inteligente.md) · [Malha logística](02-product/features/logistica-ultima-milha.md)

### 3 — Mercado ([`03-market/`](03-market/))
- [**Panorama competitivo**](03-market/competitive-landscape.md) — nacionais × regionais, e onde o GMill ganha
- [**Tendências do setor**](03-market/industry-trends.md) — consolidação, associativismo, marketplace de medicamentos, regulatório

### 4 — Operações ([`04-operations/`](04-operations/))
- [**Processo comercial**](04-operations/sales-process.md) — força de vendas, televendas, portal
- [**Framework de mensagem**](04-operations/messaging-framework.md) — brand voice e value props por audiência
- [**Comunicação com o cliente**](04-operations/customer-communication.md) — guidelines acionáveis para sistemas de IA

---

## Conflitos de evidência registrados

Fontes públicas se contradizem. Registro aqui em vez de escolher em silêncio.

| # | Conflito | Fontes | Tratamento adotado |
|---|---|---|---|
| **K1** | **Estados**: 3 (ES, BA, RJ) em 2021/22 × **2 (ES, RJ)** em 2026 — apesar de filial CNPJ ativa na BA | S2, S3 × S4, S9 | Uso **ES+RJ como núcleo** e trato BA como `[TO BE COMPLETED]` (ativa? reduzida? fora do recorte da matéria?) |
| **K2** | **Clientes**: 8.000 PDVs (2021) × **~7.000 farmácias ativas** (2026) | S3 × S4 | Bases provavelmente diferentes (*potencial atendível* × *ativa no período*). Uso **7.000 ativas** como número corrente |
| **K3** | **Porte**: 500 (2022) / 600 / 201-500 (LinkedIn) / 250 + 130 representantes (só Millenium) | S2, S6, S8 | Uso **~500 no grupo** como ordem de grandeza; os 250+130 são **da Millenium isolada** |
| **K4** | **Meds e BeeBee**: anunciados com força em 2021, **silêncio total** desde então | S3 × ausência | Documentados como **aposta não confirmada**, não como capacidade viva |
| **K5** | **Setor no LinkedIn**: "Retail" — contradiz o CNAE de atacado | S6 × S9 | Provável imprecisão de cadastro; vale o CNAE |

---

## Pendências de validação

Cada item abaixo é uma inferência ou lacuna que **uma conversa de 20 minutos com a GMill resolve**.

1. **Meds está vivo?** O marketplace saiu do piloto? Quantas farmácias usam? — bloqueia toda a seção de canal digital.
2. **BeeBee opera hoje?** É empresa de logística, marca de produto no catálogo, ou as duas coisas?
3. **Bahia**: ativa, reduzida ou encerrada? (K1)
4. **Mix de canal**: quanto do faturamento entra por força de vendas × televendas × portal Millcompras? — é o número que mais muda qualquer recomendação digital.
5. **Personas de compra**: quem assina a ordem numa farmácia independente × numa franquia × numa central de associativismo?
6. **KPIs com meta**: fill rate, ruptura, positivação, ticket médio, margem de contribuição — quais têm meta formal hoje?
7. **Concorrentes reais no ES/RJ**: quem tira pedido da GMill na prática?
8. **Objeção nº 1 na perda** — preço, prazo de entrega, crédito ou mix?
9. **Stack**: o Portal de Talentos roda em **Senior Sistemas** (`gmill.portaldetalentos.senior.com.br`) — o ERP também é Senior? `[INFERIDO]`
10. **Escopo regulatório ativo**: AFE/AE ANVISA, SNCM (rastreabilidade), Portaria 344 (controlados), cadeia fria/vacinas.
11. **Médina**: é aposta estratégica de marca própria ou investimento lateral?
12. **Restrições de confidencialidade**: o que deste contexto pode ser versionado e onde.

---

## Fontes consultadas

| id | Fonte | Data | Acesso |
|---|---|---|---|
| **S1** | [millcompras.com.br](https://www.millcompras.com.br/) — portal B2B | 2026-09-16 | ✅ |
| **S2** | [Panorama Farmacêutico — "GMill investe R$ 25 milhões na distribuição de genéricos"](https://panoramafarmaceutico.com.br/gmill-distribuicao-de-genericos/) | 2022-07-01 | ✅ |
| **S3** | [Panorama Farmacêutico — "Distribuidora GMill aposta em marketplace para farmácias regionais"](https://panoramafarmaceutico.com.br/distribuidora-gmill-aposta-em-marketplace-para-farmacias-regionais/) | 2021-09-03 | ✅ |
| **S4** | [Revista Conexão — "Distribuidora capixaba projeta faturar R$ 1 bilhão em 2026"](https://www.conexaoes.com.br/noticia/distribuidora-capixaba-de-medicamentos-projeta-faturar-r-1-bilhao-em-2026) | 2026-03-10 | ✅ |
| **S5** | [Folha Vitória — "Distribuidora de medicamentos capixaba mira R$ 1 bilhão"](https://www.folhavitoria.com.br/folha-business/distribuidora-de-medicamentos-capixaba-mira-r-1-bilhao-em-faturamento/) | — | ⛔ 403 (só via snippet de busca) |
| **S6** | [LinkedIn — Grupo GMill](https://www.linkedin.com/company/grupo-gmill/) | 2026-09-16 | ✅ |
| **S7** | [Instagram — @grupogmill](https://www.instagram.com/grupogmill/) | 2026-09-16 | ✅ |
| **S8** | [Glassdoor — Millenium Comercial](https://www.glassdoor.com.br/Avalia%C3%A7%C3%B5es/Millenium-Comercial-Avalia%C3%A7%C3%B5es-E2491190.htm) · [Indeed](https://br.indeed.com/cmp/Grupo-Gmill) | — | ⚠️ parcial (403 direto; via snippet) |
| **S9** | [Econodata](https://www.econodata.com.br/consulta-empresa/02632609000109-millenium-comercial-logop-do-gmill-distribuicao-ltda) · [Serasa Experian](https://empresas.serasaexperian.com.br/consulta-gratis/MILLENIUM-COMERCIAL-LOGOP-DO-GMILL-DISTRIBUICAO-LTDA-02632609000109) · [Solutudo](https://www.solutudo.com.br/empresas/es/serra/distribuidora-de-produtos-farmaceuticos/millenium-comercial-logop-do-gmill-distribuicao-ltda-5692783) | 2026-07 | ⚠️ parcial (403 direto; via snippet) |
| **S10** | [ABAFARMA](https://abafarma.com.br/) + pesquisa de mercado (Grupo SC, Profarma, Servimed) | 2026-09-16 | ✅ |
| **S11** | [Proffer — case Grupo GMill (pricing)](https://proffer.com.br/case-gmill-distribuidor-farmaceutico-e-proffer) | — | ✅ |
| **S12** | [Portal de Talentos GMill (Senior)](https://gmill.portaldetalentos.senior.com.br/) | 2026-09-16 | ⚠️ SPA sem conteúdo servido |
| **S13** | Varejo farmacêutico 2025-26: [Abradilan](https://abradilan.com.br/mercado/enquanto-1-224-independentes-fecham-as-portas-associativistas-vivem-expansao-historica/) · [Abradilan (expansão)](https://abradilan.com.br/mercado/1-600-novas-farmacias-no-brasil-associativistas-puxam-expansao/) · [Sincofarma SP](https://sincofarmasp.com.br/2026/02/18/farmacias-independentes-fecham-mais-do-que-abrem-e-indicam-nova-fase-do-varejo-farmaceutico-no-brasil/) · [Febrafar](https://febrafar.com.br/r-243-bilhoes-em-faturamento-o-varejo-farmaceutico-cresce-mas-nao-admite-mais-improviso/) · [ICTQ](https://ictq.com.br/guia-de-carreiras/1849-a-forca-do-associativismo-para-farmacias-independentes-competirem-com-grandes-redes) | 2026 | ✅ |

---

⏰ **Gerado**: 2026-09-16 · 🎯 **Status**: rascunho infer-from-evidence, aguardando validação
