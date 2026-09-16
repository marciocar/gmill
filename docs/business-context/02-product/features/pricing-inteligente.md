# Capacidade: Pricing inteligente

> **Status**: ✅ **ativo e creditado publicamente pela própria empresa como causa do crescimento.**
> É a única iniciativa do grupo com relação declarada entre ação e resultado.

## Propósito

Substituir a precificação **manual e subjetiva** por decisão algorítmica de preço, item a item,
considerando demanda, concorrência e custo.

## O antes (documentado pelo fornecedor — S11)

Palavras do CEO **Lucas Freire**: *"a precificação manual e baseada em percepções subjetivas era um
grande problema"*. Consequências listadas:

- dificuldade de acompanhar flutuação de mercado;
- erros frequentes e inconsistência na definição de preços;
- ausência de dado estruturado para decisão estratégica;
- desvantagem competitiva e **margem prejudicada**.

## O depois

**Plataforma de precificação inteligente da [Proffer](https://proffer.com.br/case-gmill-distribuidor-farmaceutico-e-proffer)** (S11):
algoritmos que determinam preço automaticamente a partir de demanda, concorrência e custo, com
implantação acompanhada e treinamento de equipe.

**Filipe De Boni**, Diretor de TI e Projetos: *"com a Proffer, conseguimos automatizar o processo de
precificação, tornando-o muito mais ágil e preciso"*.

**Resultados reportados** (S11) — **qualitativos, sem percentual divulgado**: aumento da margem de
contribuição em produtos estratégicos, incremento de vendas, melhora da margem de lucro, maior
participação de mercado.

## A confirmação de 2026 — e o que ela revela

Cinco anos depois, o CEO atribui o salto de **40% em 2025** a duas causas, nesta ordem (S4):

1. **Pessoas** — *"a principal razão desse crescimento foram as pessoas, tanto a reestruturação da
   área comercial quanto de todas as áreas internas"*;
2. **Pricing com IA** — *"implementamos uma arquitetura de pricing com metodologia eficiente,
   utilizando bastante tecnologia e inteligência artificial, o que nos tornou mais competitivos com
   maior rentabilidade"*.

> 📌 **Note a palavra escolhida: "arquitetura".** Não é "ferramenta" nem "sistema". Sugere que o
> pricing deixou de ser um software comprado e virou **uma camada de decisão da empresa** — com
> metodologia, dados e governança próprios. Se for isso mesmo, é o ativo digital mais valioso do
> grupo. `[INFERIDO]` — **validar o que exatamente compõe essa arquitetura hoje e o que é da Proffer.**

## Por que funciona tão bem neste negócio

Atacado farmacêutico é **o caso de uso canônico de pricing algorítmico**: margem fina, milhares de
SKUs, preço decidido muitas vezes ao dia, concorrência transparente e elasticidade muito diferente
entre curva A e cauda longa. Julgamento humano não escala nessa matriz — e erra de forma
sistematicamente conservadora.

## Métricas
Ver [metrics §5](../metrics.md#5-kpis-do-projeto-de-pricing--a-única-iniciativa-com-resultado-creditado).
O indicador crítico é o **desvio entre preço recomendado e preço praticado** — mede se o campo
respeita o algoritmo, que é onde este tipo de projeto vaza valor sem ninguém notar.

## Riscos

| # | Risco | Detalhe |
|---|---|---|
| **1** | **Dependência de fornecedor** | a vantagem competitiva creditada roda em plataforma de terceiro (R3 em [strategy](../strategy.md#6-riscos-estratégicos)) |
| **2** | **Coerência entre canais** | o preço do portal, do representante e do televendas tem que ser o mesmo, na mesma hora |
| **3** | **Aderência do campo** | representante que "dá um jeitinho" no preço desmonta o ganho, silenciosamente |
| **4** | **Guerra de preço** | algoritmo que só persegue concorrente entra em espiral de margem |
| **5** | **Transparência regulatória** | preço de medicamento tem teto (CMED/PMVG na ponta) e o setor é sensível a prática coordenada — **algoritmo de preço precisa de trilha de auditoria**. `[INFERIDO]` |

## Guideline para IA

- Preço é **dado comercial sensível**: nunca exponha tabela, margem ou regra de precificação fora de
  contexto autorizado.
- Ao propor evolução, ancore em **margem de contribuição e competitividade** — o vocabulário que a
  diretoria usa (S4) — e não em stack ou modelo.
- Respeite a ordem do discurso da casa: **pessoas primeiro, tecnologia depois**. Proposta que trate o
  time comercial como variável a ser eliminada não conversa com esta empresa.

---
**Fontes**: S2, S4, S11.
