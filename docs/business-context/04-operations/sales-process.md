# Processo comercial — Grupo GMill

> **Modo infer-from-evidence.** A estrutura de canal (seção 1) é evidência; o processo (seções 2-5) é
> `[INFERIDO]` do padrão do atacado farmacêutico e do que o desenho de canal revela.
> Pendências nº 4, 5 e 6 do [index](../index.md#pendências-de-validação).

**Metodologia**: B2B distribuição — venda **transacional recorrente de alta frequência**, não venda
consultiva de ciclo longo. O contrato não é o evento; **o pedido é o evento**, e ele se repete toda
semana.

---

## 1. A estrutura de canal (evidência)

| Canal | Estrutura conhecida | Fonte |
|---|---|---|
| **Força de vendas** | **130+ representantes comerciais**, 8 supervisores, 2 coordenadores distritais, 2 gerentes — sobre ~250 colaboradores da Millenium | S8 |
| **Portal Millcompras** | e-commerce B2B com login, catálogo por marca e campanhas | S1 |
| **Televendas / Central de Relacionamento** | SAC + WhatsApp `+55 27 3182-1500` | S1 |
| **Eventos** | "Feirão Legado"; seção "Eventos" no portal | S1, S7 |
| **Canal associativista** | franquias e associações = **60% do faturamento** | S4 |
| **Tecnologia de canal** | R$ 4 mi para reestruturar canais e **aplicativos da equipe comercial e dos clientes** | S2 |

> 📌 **O dado estrutural mais revelador**: mais de metade das pessoas da Millenium está em campo. Esta
> é uma **empresa de força de vendas** que também tem portal — não o contrário. Qualquer iniciativa
> digital que não passe pelo representante compete com a espinha dorsal da companhia.

---

## 2. O funil, canal a canal

### 2.1 Canal associativista / franquia — 60% da receita

```
prospecção da CENTRAL → negociação de acordo comercial → habilitação da base
        → pedidos recorrentes dos associados → renovação periódica do acordo
```
Venda em **dois níveis**: ganha-se a central (acordo, verba, mix) e depois conquista-se cada loja
(execução). Acordo sem execução vira share que não se realiza. `[INFERIDO]`

### 2.2 Canal independente — a operação de campo

```
carteira do representante → roteiro de visita → tirada de pedido
        → positivação → reativação do inativo
```
**Positivação** é a métrica-mãe: fazer o cadastrado comprar no período. O roteiro tende a ser
semanal ou quinzenal, com prioridade para curva A da carteira. `[INFERIDO]`

### 2.3 Canal digital — complementar

```
login no portal → catálogo/campanha → carrinho → pedido
```
Sem captação de topo (o portal é login-first, S1): **o digital serve quem o campo já trouxe.** É
canal de *retenção e conveniência*, não de aquisição. Ver
[millcompras-portal](../02-product/features/millcompras-portal.md).

---

## 3. Qualificação

| Critério | Por quê |
|---|---|
| **Licença sanitária vigente** | pré-requisito legal para vender medicamento à farmácia |
| **Autorização para controlados** | Portaria 344 — habilita ou bloqueia parte do catálogo |
| **Crédito aprovado** | define o teto do relacionamento |
| **Localização na malha** | fora da rota, o custo de servir come a margem |
| **Potencial de compra** | tamanho da loja, giro, associação |

`[INFERIDO]` — **validar o processo real de cadastro e análise de crédito.**

---

## 4. Os três conflitos estruturais do modelo

Não são problemas hipotéticos: decorrem diretamente de ter, ao mesmo tempo, 130+ representantes
comissionados, um portal self-service e 60% da receita em canais organizados.

| # | Conflito | Sintoma | Mitigação |
|---|---|---|---|
| **C1** | **Campo × portal** | representante desestimula o cliente a usar o portal para não perder comissão | creditar a venda digital à carteira do representante — transforma o portal em ferramenta dele `[INFERIDO]` |
| **C2** | **Preço divergente entre canais** | cliente vê um preço no portal e ouve outro do representante | pricing centralizado (S11) resolve na origem; falta garantir que **todo canal leia a mesma fonte** |
| **C3** | **Central × loja** | condição da central × condição direta ao associado | política explícita de não-subcotação `[INFERIDO]` — **validar** |

> 💡 **C1 é o que mais trava digitalização em distribuidoras.** A solução conhecida não é técnica, é
> de remuneração: o pedido digital tem que **contar para o representante**. Enquanto o portal for
> concorrente do campo, o campo ganha — porque o campo é quem está na loja.

---

## 5. Retenção

A retenção aqui não é renovação de contrato, é **frequência**. O sinal de perda é a curva, não o
evento (ver [journey §7](../01-customer/journey.md)).

| Alavanca | Estado |
|---|---|
| **Fill rate** | a alavanca nº 1 — ruptura recorrente migra item, e item migrado não volta |
| **Campanhas de cadência** | `Promo Dia Mill`, `Promo A a Z` (S1) — dão motivo recorrente de retorno |
| **Eventos** | "Feirão Legado" (S7) — concentra volume e renova relação |
| **Relação de campo** | a mais forte e a mais frágil: sai com o representante |
| **Crédito** | prazo sustenta o independente descapitalizado |
| **Demanda entregue (Meds)** | seria a mais forte de todas — não confirmada (S3) |

---

## 6. O que instrumentar primeiro

1. **Mix de canal na receita** (campo × televendas × portal) — pendência nº 4, e o número que muda
   qualquer decisão digital.
2. **Positivação por representante e por carteira.**
3. **Lista de clientes com frequência em queda** — churn antes do churn.
4. **Desvio preço praticado × recomendado** — mede C2 e a aderência do campo.
5. **Fill rate por curva ABC** — a causa raiz de metade do resto.

---

**Fontes**: S1, S2, S3, S4, S7, S8, S11 — ver [tabela completa](../index.md#fontes-consultadas).
