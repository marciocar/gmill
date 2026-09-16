# Capacidade: Portal Millcompras (canal B2B digital)

> **Status**: ✅ ativo e verificável — [millcompras.com.br](https://www.millcompras.com.br/) (S1, acesso 2026-09-16)

## Propósito

Canal de **autoatendimento de pedido** para as farmácias clientes. É o e-commerce B2B do grupo: o
cliente entra, vê catálogo e condição, e fecha pedido sem depender da visita do representante.

⚠️ **Não é loja para consumidor final.** O nome "Millcompras" induz ao erro; o produto é um portal de
compras *das farmácias*, protegido por login.

## O que é observável hoje (S1)

| Elemento | Observação |
|---|---|
| **Acesso** | login-first — catálogo, preço e condição ficam atrás da autenticação |
| **Onboarding** | fluxo de "Cadastro" separado; há "Esqueci a Senha" |
| **Catálogo por marca** | BeeBee, BioLab, Avvio, Althaia, Airela, Vitamedic (EMS), Pharmacience (Taiff), Teuto |
| **Campanhas** | `Promo Dia Mill` e `Promo A a Z` — promoções nomeadas e recorrentes |
| **Seções institucionais** | Home, Grupo, Eventos, "Quero ser Gmill" (recrutamento) |
| **Atendimento** | Central de Relacionamento (SAC) + WhatsApp `+55 27 3182-1500` |
| **Ouvidoria** | canal formal publicado em `gmill.com.br/portal/ouvidoria` |

**Não observável sem login**: mecânica de preço, disponibilidade em tempo real, limite de crédito,
prazo de entrega por região, integração com ERP da farmácia, app mobile.

## Benefício de negócio

| Para quem | Benefício |
|---|---|
| **Farmácia (P1)** | compra fora do horário da visita, na hora que percebe a falta |
| **GMill** | pedido digital custa uma fração do pedido tirado por pessoa; libera o campo para o que só gente faz (negociação, reativação) `[INFERIDO]` |
| **Diretoria** | dados de navegação e carrinho — insumo natural para pricing e recomendação `[INFERIDO]` |

O investimento de **R$ 4 milhões em tecnologia para reestruturar canais de venda e aplicativos de
equipe comercial e de clientes** (S2) indica que este portal e o app do representante foram tratados
como **um mesmo projeto de canal**, não como iniciativas separadas.

## Métricas
`[TO BE COMPLETED]` — nenhuma pública. As que importam estão em
[metrics §6](../metrics.md#6-kpis-digitais-portal-e-marketplace). A primeira é **% da receita
originada no portal**: sem ela, não dá para dizer se o canal digital é alavanca ou vitrine.

## Problemas conhecidos e riscos

| # | Risco | Detalhe |
|---|---|---|
| **1** | **Conflito de canal** | portal self-service × 130+ representantes comissionados (S8). Sem regra clara de crédito da venda, o campo desestimula o digital. `[INFERIDO]` — **validar política de comissionamento** |
| **2** | **Divergência de preço entre canais** | o portal e o representante precisam falar o mesmo preço. Mitigado pela centralização em pricing algorítmico (S11), mas segue sendo o sintoma a monitorar |
| **3** | **Zero captação digital** | login-first significa nenhuma descoberta por busca. Apropriado para preço B2B; mas não há camada pública que atraia farmácia nova (ver [journey §1](../../01-customer/journey.md)) |
| **4** | **Dependência de integração** | se o portal não conversa com o ERP da farmácia, o cliente digita duas vezes. Foi a barreira que o próprio grupo reconheceu ao desenhar o Meds "sem exigir integração de ERP" (S3) |

## Guideline para IA

- Ao falar do Millcompras com cliente: é **"o portal"** ou **"o site de pedidos"**, nunca "e-commerce".
- Nunca exponha preço, condição ou limite de crédito sem contexto autenticado — **preço em B2B
  farmacêutico é confidencial e varia por acordo comercial**.
- Perguntas de acesso (senha, cadastro) resolvem-se pelo fluxo do próprio portal; disputa comercial
  escala para o representante da conta, não para o SAC.
- Para recomendação de compra, o eixo útil é **reposição do que girou**, não descoberta de novidade.

---
**Fontes**: S1, S2, S3, S8, S11.
