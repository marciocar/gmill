# Personas — Grupo GMill

> **Modo infer-from-evidence.** Nenhuma persona abaixo veio de entrevista. Todas foram derivadas do
> perfil de cliente declarado nas fontes (farmácias independentes, franquias e associativismo) e do
> desenho de canal visível no portal Millcompras. Marcadas `[INFERIDO]` onde extrapolam a fonte.
>
> Ver [`index.md` § Pendências de validação](../index.md#pendências-de-validação) itens 5 e 7.

**Regra de ouro do negócio:** o **cliente da GMill é a farmácia**, nunca o paciente. O consumidor final
só entra em cena na aposta do [marketplace Meds](../02-product/features/meds-marketplace.md) — e mesmo
lá, a farmácia é quem vende.

---

## P1 — Dono de farmácia independente (persona primária)

**Quem é.** Proprietário-operador de 1 a 3 lojas no ES ou RJ. Frequentemente farmacêutico de formação
ou família que herdou a loja. Toma sozinho a decisão de compra, muitas vezes no balcão, entre um
atendimento e outro.

| | |
|---|---|
| **Volume** | núcleo dos ~7.000 clientes ativos (S4) |
| **Decisor** | ele mesmo — ciclo de decisão de minutos, não semanas `[INFERIDO]` |
| **Frequência de compra** | alta e fracionada: repõe o que girou, não compra por planejamento `[INFERIDO]` |

**Objetivos**
- Não perder venda por ruptura — o cliente que não encontra o remédio vai na farmácia da esquina.
- Comprar barato o suficiente para sobreviver à guerra de preço com as grandes redes.
- Manter capital de giro respirando: prazo de pagamento vale tanto quanto desconto.

**Dores**
- **Capital de giro apertado.** Compra pouco e frequente porque não pode imobilizar estoque. `[INFERIDO]`
- **Assimetria de poder de compra.** Uma rede nacional compra em condições que ele nunca terá — é a
  razão econômica do associativismo existir (ver [industry-trends](../03-market/industry-trends.md)).
- **Falta de presença digital.** Foi exatamente a dor que a GMill nomeou ao lançar o Meds: farmácias
  "sem presença digital nem aplicativo de vendas" (S3).
- **Dependência de uma pessoa.** O representante comercial *é* a relação com o fornecedor. Se ele sai,
  o cliente pode ir junto. `[INFERIDO]`

**Contexto tecnológico**
- Tem ERP de farmácia, raramente integrado ao fornecedor. O Meds foi desenhado justamente para
  funcionar **sem exigir integração de ERP** (S3) — o que confirma que integração é barreira real.
- Usa WhatsApp para tudo, inclusive pedido. O portal Millcompras publica número de WhatsApp com DDD
  fixo de Serra (S1).

**Como a IA deve tratar esta persona**
- Responda em **português simples, sem jargão de supply chain**. "Ruptura" ele chama de "faltou".
- **Nunca** dê conselho clínico ou farmacológico. A persona é comerciante; a orientação técnica é do
  farmacêutico responsável e tem cerco regulatório.
- Priorize o concreto na ordem que ele decide: **tem? por quanto? chega quando? pago em quantos dias?**
- Presuma pressa. Resposta longa aqui é resposta perdida.

---

## P2 — Comprador de rede ou franquia

**Quem é.** Profissional de compras de uma rede regional ou franqueadora com dezenas de lojas.
Compra por planilha e contrato, não por relação.

| | |
|---|---|
| **Peso no negócio** | **franquias + associações = 60% do faturamento** (S4) — não é nicho, é o principal |
| **Decisor** | comprador executa; diretoria comercial define acordo e verba |
| **Ciclo** | negociação periódica (acordo comercial), com pedidos recorrentes dentro do acordo `[INFERIDO]` |

**Objetivos**
- Bater meta de margem da rede inteira, não de uma loja.
- Garantir **fill rate**: promoção anunciada em 40 lojas e produto que não chega é crise de marca.
- Padronizar mix entre lojas.

**Dores**
- **Pedido atendido pela metade.** Para rede, atendimento parcial é pior que recusa — desorganiza o
  encarte e a promessa ao consumidor. `[INFERIDO]`
- **Nota fiscal, prazo e rebate** têm que fechar com o acordo. Divergência vira retrabalho financeiro.
- Precisa de **previsibilidade de abastecimento**, não de oportunidade pontual.

**Como a IA deve tratar esta persona**
- Fale em **agregado**: cobertura por loja, fill rate do pedido, desvio contra o acordo.
- Tenha número. Esta persona não aceita adjetivo — aceita percentual e prazo.
- Escalone para o gerente de contas ao primeiro sinal de disputa contratual.

---

## P3 — Central de compras / associativismo

**Quem é.** Central que negocia em nome de dezenas ou centenas de farmácias independentes
associadas. Formalmente compra pouco; **influencia muito**.

**Por que importa mais do que o volume sugere:** ela é o mecanismo pelo qual P1 recupera poder de
compra. Perder a central é perder o acesso a um bloco inteiro de P1 de uma vez. `[INFERIDO]`

**Objetivos**
- Provar valor da associação para reter associados.
- Extrair condição comercial que o associado sozinho não obteria.

**Como a IA deve tratar esta persona**
- Trate como **canal**, não como conta. O sucesso dela é o que retém P1.
- Nunca ofereça a um associado, direto, condição melhor que a da central — desmonta a proposta de
  valor do canal. `[INFERIDO]` — **confirmar política comercial real.**

---

## P4 — Representante comercial GMill (persona interna)

**Quem é.** Campo. A Millenium isolada tem **mais de 130 representantes comerciais, 8 supervisores,
2 coordenadores distritais e 2 gerentes** (S8) sobre ~250 colaboradores — ou seja, **o campo é maior
que a empresa administrativa**. Esta é a estrutura que define a cara da companhia.

**Objetivos**: positivação (fazer o cliente inativo comprar de novo), volume e mix — geralmente
remunerados por comissão sobre isso `[INFERIDO]`.

**Dores**
- Chegar na loja **sem saber o preço certo** — era exatamente o problema que o projeto de pricing
  atacou: precificação "manual e baseada em percepções subjetivas" (S11).
- Concorrer com o próprio portal: se o cliente compra sozinho no Millcompras, quem fica com a
  comissão? Conflito de canal clássico. `[INFERIDO]` — **confirmar regra de comissionamento.**

**Como a IA deve tratar esta persona**
- Ferramenta de campo é ferramenta **de celular, offline-tolerante e de 3 toques**.
- Entregue a resposta que ele precisa **na frente do cliente**: preço válido, estoque real, limite de
  crédito, histórico do último pedido.

---

## P5 — Diretoria / TI (persona interna de decisão)

**Quem é.** Nomes públicos: **Lucas Freire e Freire** (CEO do grupo) e **Filipe De Boni** (Diretor de
TI e Projetos) (S2, S11).

**O que este par revela — e é o dado mais acionável deste documento:** a diretoria da GMill
**compra tecnologia como alavanca de margem, não como custo de TI**. Evidências convergentes:

- R$ 4 milhões num projeto de tecnologia para reestruturar canais de venda e aplicativos de equipe
  comercial e clientes (S2);
- substituição da precificação manual por plataforma algorítmica de terceiro (Proffer), com o CEO
  nomeando o problema em público (S11);
- em 2026, o CEO atribui o salto de 40% a "arquitetura de pricing com metodologia eficiente,
  utilizando bastante tecnologia e **inteligência artificial**" (S4).

**Como a IA deve tratar esta persona**
- Fale em **margem, giro e competitividade** — não em stack. O CEO descreveu um projeto de IA sem
  citar uma única tecnologia.
- **Traga o antes/depois.** A cultura declarada é de resultado atribuído, não de piloto.
- Cuidado com o ponto cego: o discurso público credita o crescimento primeiro a **pessoas**
  ("reestruturação da área comercial e de todas as áreas internas" — S4) e só depois à tecnologia.
  Proposta que ignore o fator humano da operação não conversa com esta diretoria.

---

## Quem **não** é persona

| Não-persona | Por quê |
|---|---|
| **Paciente / consumidor final** | não compra da GMill. Só aparece como demanda no Meds, e mesmo lá compra **da farmácia** |
| **Indústria farmacêutica** (EMS, Teuto, BioLab, Althaia…) | é **fornecedor**, não cliente — embora negocie verba e campanha, o que a torna stakeholder de peso (S1) |
| **Hospital / licitação pública** | nenhuma fonte indica atuação institucional. `[TO BE COMPLETED]` |

---

**Fontes**: S1, S2, S3, S4, S8, S11 — ver [tabela completa](../index.md#fontes-consultadas).
