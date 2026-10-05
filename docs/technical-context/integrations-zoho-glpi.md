# Integrações Zoho Projects e GLPI — vínculos locais da GMill

> **Última Atualização**: 2026-10-05
>
> As KBs de referência são **vendorizadas** do core e chegam pelo `/meta:adopt --update`; não as edite
> aqui, porque a próxima atualização sobrescreve ou conflita. O que é específico da GMill fica neste
> arquivo, fora do manifesto do framework.

## KBs de referência (vendorizadas)

- [zoho-projects-api](../knowledge-base/platforms/zoho-projects-api.md) — API V3, OAuth, endpoints de tarefa
- [glpi-api](../knowledge-base/platforms/glpi-api.md) — API V1 e V2 do GLPI
- [glpi-zoho-ticket-to-task](../knowledge-base/patterns/glpi-zoho-ticket-to-task.md) — fluxo chamado → tarefa

Essas três KBs nasceram aqui (2026-09-30), foram absorvidas pelo core com scrub e voltaram corrigidas
por medição contra portal real. Desde 2026-10-05 este repo usa a versão do core.

## Regras locais que valem sobre essas KBs

- **LGPD — dado de paciente/consumidor não trafega.** A regra vem do `CLAUDE.md` deste repo e de
  [`04-operations/customer-communication.md`](../business-context/04-operations/customer-communication.md).
  Na integração GLPI → Zoho: filtrar CPF, nomes de pacientes e anexos **antes** do POST; não registrar
  esses dados em chamados do GLPI.
- **GLPI como ferramenta de TI do grupo:** se entrar, registrar aqui a versão da instância, V1/V2 em uso
  e as entidades. `[TO BE COMPLETED]`
- **Middleware do fluxo GLPI → Zoho:** decisão ainda não tomada; quando for, vira ADR em
  [`decisions/`](decisions/).

## Decisões relacionadas

- [ADR — mapeamento Zoho Projects V3 → ITaskManager](decisions/adr-zoho-projects-task-mapping.md)
  (SAC-58/SAC-59): fixa o id composto `<project_id>.<task_id>` e o escopo
  `ZohoProjects.custom_fields.READ` (resolve `status.id` ao mudar status). O passo de volta
  Zoho → GLPI do fluxo também precisa desse escopo se atualizar status.

## Exemplo de domínio

Tarefa típica que nasceria de um chamado: `[GLPI #4521] Implantar leitor SNCM no CD Serra`
(rastreabilidade SNCM é escopo regulatório ativo — ver o `CLAUDE.md`).
