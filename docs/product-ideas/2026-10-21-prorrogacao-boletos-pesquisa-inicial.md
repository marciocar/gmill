---
title: "Prorrogação de boletos — pesquisa inicial para o Financeiro"
date: 2026-10-21
status: draft
audience: Financeiro da GMill (contas a receber, crédito, gestor financeiro)
relates_to:
  - docs/product-ideas/2026-10-21-agente-prorrogacao-boletos.md
  - zoho task 2723266000000174001 (portal pessoal de teste)
review_after: 2027-01-31
---

# Prorrogação de boletos — pesquisa inicial para o Financeiro

> **O que é este documento.** É uma pesquisa de mesa: fontes públicas, legislação, manuais de banco e
> fóruns técnicos, consultados em 2026-10-21. Ainda **não conhecemos o processo da GMill**, e o
> objetivo é chegar à conversa com o Financeiro sabendo o que é padrão de mercado e o que é decisão
> interna. **Não é parecer jurídico.**
>
> Marcações: `[A VERIFICAR]` é fonte fraca ou não confirmada; `[INFERIDO]` é leitura nossa;
> ⚖️ é tema para o **jurídico**.

## Resumo em 6 pontos

1. **O status verdadeiro do boleto está no banco, não no ERP.** O ERP só sabe do pagamento, da baixa
   ou do protesto depois de processar o arquivo de retorno ou o aviso do banco.
2. **Prorrogar tem restrições do próprio banco.** No Itaú, boleto vencido não aceita alteração de
   vencimento, e o limite é de 3 alterações por boleto. Com protesto ou negativação em curso, é
   preciso sustar antes. Cada instrução pode ser tarifada.
3. **Cada banco é diferente.** Os códigos, os arquivos e as APIs mudam de banco para banco. A GMill
   precisa nos dizer com quais bancos e canais trabalha.
4. **Juros e multa de atraso seguem o contrato.** Se o contrato não disser nada, vale a taxa legal
   (Selic menos IPCA, Lei 14.905/2024). O teto de 2% de multa do Código de Defesa do Consumidor
   **em regra não vale** para farmácia que compra para revender.
5. ⚖️ **Cobrar "custo da prorrogação" é o ponto mais sensível.** Se for CDI mais spread, pode ser lido
   como juros remuneratórios, e para empresa que não é banco a Lei de Usura limita isso a 12% ao
   ano. Com a Selic em 13,75%, **só o CDI já passa do teto**.
6. **"Inadimplente recorrente" e as alçadas de aprovação não têm padrão de mercado.** São política
   da GMill, e é isso que precisamos definir juntos.

---

## 1. Ciclo de vida e status de um boleto registrado

| Situação | O que significa | Quem causa | Como a empresa fica sabendo |
|---|---|---|---|
| Registrado | Banco aceitou o boleto | Empresa (remessa ou API) | Retorno "entrada confirmada" |
| Rejeitado | Banco recusou o registro | Banco | Retorno com motivo |
| A vencer / vencido | **Não é status do banco**: é a data de hoje contra o vencimento `[INFERIDO]` | — | Calculado |
| Vencimento alterado | Prorrogação aceita | Empresa (instrução) | Retorno "vencimento alterado" |
| Liquidado | Pago (total ou parcial) | Pagador, por qualquer banco | Retorno ou webhook de pagamento |
| Baixado | Saiu da carteira sem pagamento, ou baixa por decurso | Empresa (instrução) ou banco | Retorno de baixa |
| Em protesto / protestado | Enviado ao cartório | Empresa (instrução) | Retorno de protesto |
| Negativado | Enviado a bureau (Serasa etc.) | Empresa (instrução) | Retorno de negativação |

- **Boleto registrado pode ser pago em qualquer banco, mesmo depois do vencimento,** com juros e multa
  calculados pela plataforma da CIP (Nova Plataforma de Cobrança). Fonte:
  [cartilha NPC Bradesco, 2018](https://assets.bradesco/content/dam/portal-bradesco/assets/corporate/pdf/solucoes-integradas/recebimentos/Cartilha_Nova_Plataforma_Cobranca_Ago18.pdf) `[A VERIFICAR: versão vigente]`
- **Baixa automática:** o Itaú baixa o título 120 dias após o vencimento.
  Fonte: [Itaú, manual CNAB 400](https://download.itau.com.br/bankline/layout_cobranca_400bytes_cnab_itau.pdf) (versão de 2017).
- **Arquivo de retorno:** o banco só gera retorno quando há evento. O Itaú oferece também uma posição
  mensal da carteira.
- **Webhook:** BB e plataformas de cobrança avisam pagamento em tempo real. O aviso pode chegar mais
  de uma vez, e o valor pago pode diferir do original por causa de juros, multa e desconto.
  Fontes: [Alterdata/BB](https://ajuda.alterdata.com.br/bimerbase/bimer-financeiro/baixa-automatica-de-boletos-via-api-do-banco-do-brasil-por-webhook), [Asaas](https://docs.asaas.com/docs/webhook-para-cobrancas).
- **Boleto com QR Code Pix:** permitido desde 03/02/2025
  ([Res. BCB 443/2024](https://www.normasbrasil.com.br/norma/resolucao-443-2024_470421.html)).

## 2. Como se prorroga, na prática

**Três caminhos:**

1. **Instrução de alteração de vencimento**, pelo arquivo de remessa (CNAB) ou pela API do banco.
2. **Baixar o boleto e emitir um novo** com a nova data. É o caminho quando a alteração não é aceita,
   por exemplo com o boleto vencido.
3. **Prorrogação "de papel"**, sem mexer no banco. Isso cria divergência entre o banco e o ERP e não
   é recomendado `[INFERIDO]`.

**Restrições conhecidas** ([Itaú, CNAB 400](https://download.itau.com.br/bankline/layout_cobranca_400bytes_cnab_itau.pdf), salvo indicação):

| Restrição | Efeito |
|---|---|
| Título já vencido | Alteração rejeitada ("alteração inválida para título vencido"). A rota é baixar e emitir novo |
| Vencimento já alterado | Pode ser bloqueado. O limite informado é de **3 alterações por boleto** `[A VERIFICAR]` |
| Protesto ou negativação em curso | Sustar antes, ou usar a instrução que altera e susta junto |
| Valor e vencimento ao mesmo tempo | Alteração de valor deve ir sozinha |
| Tarifa | A instrução pode ser tarifada (o retorno traz ocorrência de tarifa). Valores não encontrados |
| Boleto com Pix | Provavelmente exige atualizar também a cobrança Pix `[INFERIDO]` |

**APIs dos bancos são diferentes entre si** (fóruns técnicos, `[A VERIFICAR]`):

- O Bradesco exige PUT no endpoint de alteração.
- O BB altera um campo por chamada.
- O Sicoob tem relato de gravar o vencimento um dia antes do pedido.
- Itaú, Inter, Santander e Sicredi não foram confirmados.

**Tempo:** pelo arquivo, o efeito aparece no retorno do mesmo ciclo ou do dia útil seguinte
`[INFERIDO]`. A prorrogação só deve ser dada como feita **depois da confirmação do banco**.

## 3. Encargos: o que diz a lei e como o mercado calcula

### No atraso (mora)
- **Vale o que está no contrato ou nas condições gerais de venda.** Se não houver nada, vale a taxa
  legal: IPCA mais juros de mora de Selic menos IPCA, ou zero se der negativo (Lei 14.905/2024, que
  alterou o art. 406 do Código Civil).
  Fontes: [BMA](https://www.bmalaw.com.br/conteudo/direito-contencioso-arbitragem-e-solucao-de-conflitos/informativos-e-newsletters/nova-lei-regula-atualizacao-monetaria-e-juros/pdf), [Debit](https://www.debit.com.br/lei-14905-2024).
- **A multa de 2% do CDC em regra não se aplica:** a farmácia compra para revender e não é consumidora
  final ([Dizer o Direito](https://www.dizerodireito.com.br/2023/12/em-regra-o-cdc-nao-e-aplicado-para-os.html?m=1)).
  ⚖️ A exceção é farmácia muito pequena que prove vulnerabilidade.
- O padrão popular em boleto é **1% ao mês mais multa** ([Transfeera](https://transfeera.com/blog/calcular-juros-boleto/)).

### Na prorrogação pedida antes do vencimento
- **Não é atraso**, é dilação de prazo. Prorrogar não é novação: a dívida continua a mesma.
- ⚖️ **Cobrar pela prorrogação** é remunerar o prazo. Com CDI mais spread, pode ser lido como juros
  remuneratórios. A Lei de Usura (12% ao ano) vale para quem não é instituição financeira
  (Súmula 596 do STF). **Com a Selic em 13,75% (Copom, 16/09/2026), só o CDI já passa desse teto.**
  Alternativas a avaliar com o jurídico:
  - taxa fixa por prorrogação;
  - preço a prazo diferente;
  - encargo dentro do teto;
  - estrutura via instituição financeira (antecipação ou FIDC).
- ⚖️ **Aceite documentado:** para o encargo ser cobrável, convém um termo ou aditivo, que pode ser
  eletrônico, com a taxa, a base de cálculo e o novo vencimento.

### Fórmula
- Não há convenção única. ERPs oferecem dias corridos com base 360 ou 365, e dias úteis com base 252
  ([Senior](https://documentacao.senior.com.br/gestaoempresarialerp/5.10.3/manual-processos/financas/gestao-de-tesouraria/contratos-de-emprestimos-e-financiamentos.htm)).
- Duas formas usuais:
  - **simples:** `encargo = valor × taxa_mensal × dias_corridos / 30`
  - **composta (CDI):** `encargo = valor × [(1+CDI)^(du/252) × (1+spread)^(dc/365) − 1]`
- **Recomendação** `[INFERIDO]`:
  - escolher **um** método;
  - congelar a taxa na data da concessão;
  - registrar a fórmula, a taxa e o spread usados em cada decisão;
  - ler o CDI da série oficial (Banco Central ou B3), sem valor fixo no sistema.

## 4. Crédito no atacado farmacêutico

- **Prazos usuais:** 30 a 90 dias para farmácia
  ([Antecipa Fácil](https://antecipafacil.com.br/antecipacao-industria-farmaceutica-distribuidoras-medicamentos), fonte comercial, `[A VERIFICAR]`).
- **Inadimplência de empresas em nível recorde:** 9,2 milhões de CNPJs negativados em julho de 2026,
  segundo a Serasa, que atribui o quadro a juros altos
  ([Serasa, 01/09/2026](https://www.serasaexperian.com.br/sala-de-imprensa/indicadores/inadimplencia-chegou-a-9-milhoes-de-empresas-em-julho-e-dividas-somaram-238-bilhoes-revela-serasa-experian/)).
- **Sinais de risco no varejo farmacêutico:** recuperação judicial da Maxxi Econômica (RS), com plano
  aprovado em setembro de 2026 e deságios de 15% a 95%
  ([Jornal do Comércio](https://www.jornaldocomercio.com/economia/2026/09/1264948-maxxi-economica-aprova-plano-de-recuperacao-judicial-para-divida-de-rs-60-milhoes.html)),
  e falência de rede no MS `[A VERIFICAR]`. Fornecedor de rede em recuperação judicial vira credor
  com deságio. Isso é argumento para monitorar recuperação judicial por grupo econômico.
- **Associativismo:** a Febrafar reúne cerca de 15 mil lojas. Não achamos evidência de que redes
  associativistas garantam o crédito junto ao distribuidor.
- **Não há dado público** de inadimplência específica do atacado farmacêutico.
- **"Inadimplente recorrente" e alçadas:** não existe padrão. Boa prática é política escrita com
  faixas de risco, limite por grupo econômico, alçadas e gatilhos de revisão
  ([Antecipa Fácil](https://antecipafacil.com.br/artigo/padronize-politica-credito-aprove-mais)).

---

## 5. Roteiro de perguntas para o Financeiro

> Para a entrevista do item 1.1 (S1). Não traga para a conversa nomes de clientes, CPFs nem dados
> de sócios: só números agregados e regras.

### A. Bancos e canais
1. Com quais bancos a GMill emite boletos? Qual concentra o volume?
2. O registro e as instruções saem por arquivo (CNAB 240 ou 400), por API ou pelo internet banking?
   Quem opera?
3. Os boletos têm QR Code Pix?
4. Quanto custa hoje uma instrução de alteração de vencimento, e uma baixa com nova emissão?

### B. Status e dados
5. Onde a equipe vê a situação de um boleto hoje: no ERP, no banco ou numa planilha? Com que atraso?
6. O retorno do banco é processado todo dia? O ERP mostra protesto e negativação?
7. Fica registrado quantas vezes um boleto ou cliente já foi prorrogado?
8. Qual é o ERP, e existe API ou exportação dos títulos? O agente será um serviço separado.

### C. Processo e volume
9. Por onde chega o pedido de prorrogação: representante, televendas, WhatsApp, SAC ou e-mail?
10. Quantos pedidos por mês, e quanto tempo leva cada análise hoje?
11. Quem decide hoje, e quanto do que é pedido é aprovado?
12. Quando o boleto já venceu, vocês alteram ou baixam e emitem de novo?

### D. Política e alçadas
13. Existe política de crédito escrita? Podemos usá-la como base?
14. Quais são os limites de prazo máximo e de valor máximo de uma prorrogação?
15. O que é "inadimplente recorrente" para a GMill (quantos atrasos, de quantos dias, em quanto
    tempo)? E "bloqueado"?
16. Quem aprova o quê (analista, gerente, diretoria)? Rede associativista tem regra diferente?
17. Quando o certo é **reprovar** e quando é **encaminhar ao gestor**?
18. Vocês consultam Serasa ou Boa Vista? Monitoram recuperação judicial de clientes?

### E. Encargos e jurídico ⚖️
19. Hoje cobram algo pela prorrogação? Como é calculado (taxa fixa, % ao mês, CDI mais spread)?
20. O que as condições gerais de venda dizem sobre juros, multa e prorrogação?
21. O cliente assina ou aceita algum termo quando prorroga?
22. O jurídico já avaliou o custo de prorrogação frente à Lei de Usura (12% ao ano para quem não é
    banco)?
23. Podemos ter ~30 casos passados (sem dados de pessoa física) para testar o agente em modo sombra?

---

## Lacunas desta pesquisa

- **Bancos:** só o manual CNAB 400 do Itaú (de 2017) foi lido na fonte oficial. Os layouts de BB,
  Bradesco, Santander, Caixa e Sicoob, o CNAB 240 e as APIs oficiais de alteração não foram
  confirmados. **Confirmar com o gerente de cada banco que a GMill usa.**
- **Custos e regras:** não achamos valores de tarifa de instrução, prazo de processamento, impacto da
  alteração no Pix do boleto nem a regra da CIP sobre data-limite de pagamento.
- **Jurídico:** sem jurisprudência específica sobre custo de prorrogação entre empresas e Lei de Usura.
  O texto dos arts. 406 e 591 do Código Civil não foi conferido na fonte primária.
- **Setor:** não há dado público de inadimplência nem de prazo médio do atacado farmacêutico.
