---
title: "Sistemas de autenticação modernos — estado da arte 2026"
date: 2026-09-30
type: research-synthesis
genre: landscape
kg: docs/evolution/research/auth-modernos-2026-09/auth-modernos-2026-09.kg.yaml
run_id: wf_238306ff-d73
tokens: 4561324
agents: 103
duration_min: 8.6
review_after: 2026-12-29
---

# Sistemas de autenticação modernos — estado da arte 2026

> **Projeção** do grafo [`auth-modernos-2026-09.kg.yaml`](auth-modernos-2026-09.kg.yaml) (radar exit 0).
> O grafo é a fonte; os ids entre crases são os nós. Pedido como aparte paralelo durante a SAC-58.
> Orçamento: `maxFetch` 15 · `maxVerify` 25. Foram 64 claims extraídas e 25 verificadas (18 confirmadas,
> 7 refutadas). Todo o resto está declarado em [Não verificados](#não-verificados).

## Resumo

O estado da arte de 2026 tem **quatro pilares**:

1. **Passkeys (WebAuthn/FIDO2) como fator padrão resistente a phishing.** A Thoughtworks as colocou em
   **Adopt** em abr/2026 (`E_TW_RADAR_PASSKEYS_ADOPT`), e a FIDO Alliance reporta 5 bilhões em uso
   (`E_FIDO_5B_PASSKEYS`).
2. **OAuth 2.1**, ainda em *draft* 16 e não RFC: acaba o Implicit, o PKCE S256 passa a ser obrigatório e
   o redirect URI tem que casar exatamente (`E_OAUTH21_NO_IMPLICIT_PKCE`, `E_OAUTH21_EXACT_REDIRECT`).
3. **Tokens sender-constrained** no lugar do bearer puro: **DPoP (RFC 9449)** ou mTLS. O OAuth 2.1
   recomenda isso como SHOULD (`E_DPOP_SENDER_CONSTRAINING`, `E_OAUTH21_SENDER_CONSTRAINED_SHOULD`).
4. **Proteger a sessão, não só o login.** O MFA não resiste a roubo de cookie via AiTM
   (`E_MFA_SESSION_GAP_AITM`). As respostas são tokens de vida curta, device binding e avaliação
   contínua via **SSF/CAEP** (`E_SSF_CAEP_CONTINUOUS_TRUST`).

**Ressalva que o Elenxo preservou:** a passkey só resiste a phishing se o **fallback fraco for eliminado**.
Existe um ataque de *downgrade* que simula um ambiente sem suporte e força SMS ou push
(`E_PASSKEY_DOWNGRADE_AITM`, `E_DOWNGRADE_FAKE_UNSUPPORTED_ENV`). A mitigação é começar pelas contas
privilegiadas e cortar os fallbacks (`E_FIDO2_PRIVILEGED_FIRST`).

## Achados confirmados

| Nó | Claim | Fonte (tier) | Voto |
|---|---|---|---|
| `E_DPOP_SENDER_CONSTRAINING` · `E_DPOP_REPLAY_DETECTION` | O DPoP prende access e refresh tokens ao cliente com uma prova JWT por requisição, o que detecta replay. Serve para browser e mobile sem mTLS | RFC 9449 (10) | 3-0 |
| `E_OAUTH21_NO_IMPLICIT_PKCE` · `E_OAUTH21_EXACT_REDIRECT` | OAuth 2.1: sem Implicit; PKCE S256 obrigatório; redirect URI exato (loopback pode variar a porta) | IETF draft 16 (9) | 3-0 |
| `E_OAUTH21_SENDER_CONSTRAINED_SHOULD` | Tokens sender-constrained (DPoP/mTLS) são **SHOULD**, não MUST | IETF draft 16 (9) | 3-0 |
| `E_FIDO_5B_PASSKEYS` · `E_FIDO_WORKFORCE_68PCT` | 5 bilhões de passkeys; 68% das organizações implantando para funcionários | FIDO Alliance (8), **parte interessada** | 3-0 |
| `E_PASSKEY_INDEX_36_26` · `E_PASSKEY_SUCCESS_93_VS_63` | 36% das contas com passkey e 26% dos logins com passkey; taxa de sucesso de 93% contra 63% | FIDO Passkey Index (8) | 3-0 / 2-1 |
| `E_TW_RADAR_PASSKEYS_ADOPT` · `E_PASSKEYS_PHISHING_RESISTANT` | Passkeys em Adopt; resistentes a phishing por construção, ao contrário de SMS OTP e TOTP | Thoughtworks Radar (8) | 3-0 |
| `E_MFA_SESSION_GAP_AITM` | MFA protege o login, não a sessão: AiTM rouba o cookie | WorkOS blog (5) | 3-0 |
| `E_PASSKEY_DOWNGRADE_AITM` · `E_DOWNGRADE_FAKE_UNSUPPORTED_ENV` · `E_FIDO2_PRIVILEGED_FIRST` | Downgrade de passkey força o fallback; mitigação: privilegiados primeiro e sem SMS/push | Proofpoint via imprensa (5) | 3-0 |
| `E_SSF_CAEP_CONTINUOUS_TRUST` | SSF/CAEP trocam eventos de segurança (SETs) em tempo real entre IdP e aplicações | KuppingerCole EIC26 (7) | 3-0 |
| `E_AUTHRIM_EDGE_SELF_HOSTED` · `E_AUTHRIM_MODERN_EXTENSIONS` | Há IdP self-hosted na edge (Cloudflare Workers) com PAR, DPoP, JAR e FAPI 2.0 | Repositório do projeto (4), autodeclarado | 3-0 |

## Mercado (`E_MERCADO_AUTH_2026_09`)

- **Capital:** **nenhum** sinal confirmado (rodada, M&A ou valuation). As candidatas (Okta–Permiso,
  1Password–Passage, SecureAuth–Cloudentity) ficaram **fora do orçamento de fetch**. Isso não prova que
  não existem.
- **Preço e TCO:** as cotações por MAU e a comparação Keycloak × FusionAuth × Auth0 foram **refutadas**
  (0-3).
- **Analistas convergem:** Thoughtworks (passkeys em Adopt), KuppingerCole (sessão estática → confiança
  contínua) e FIDO Alliance (adoção em escala).
- **Direção:** passkeys como padrão, tokens sender-constrained e sessão avaliada continuamente. **Quem
  ganha comercialmente não tem evidência confirmada.**

## Relevância para este repo

- **A pergunta "gerenciado × self-hosted" segue SEM resposta** (`E_LACUNAS_AUTH_2026_09`). Ela precisa de
  uma rodada `mode: 'primaries'` com fontes nomeadas (páginas de preço oficiais, NIST SP 800-63B-4).
- **GMill:** a pesquisa **não cobriu** autenticação regulada no Brasil. Exemplos: trilha de quem opera
  controlados (Portaria 344), ICP-Brasil/e-CNPJ e residência de dados do IdP (LGPD). Nada disso pode ser
  inferido das claims globais.
- **Ligação com a SAC-58 (adapter Zoho):** a integração Zoho usa **OAuth 2.0 com bearer puro e refresh
  token de longa vida guardado no `.env`**. Pelo estado da arte, esse é o modelo que o OAuth 2.1 quer
  substituir por tokens sender-constrained. O Zoho não oferece DPoP na doc lida. O risco residual é o
  refresh token, que é segredo estático: proteção do `.env` e rotação são obrigatórias.

## Não verificados

- **Refutados (7), a maioria por fonte fraca e não por evidência contrária:**
  - SAML/OIDC ficam estáticos após a emissão (0-3).
  - Passkeys-as-default como força principal de compra de CIAM (0-3).
  - Auth0 lançou OAuth 2.1 e tokens agente × humano em out/2025 (0-3).
  - Cotações por MAU (WorkOS/Entra/Cognito/Auth0) (0-3).
  - TCO Keycloak × FusionAuth × Auth0 (0-3).
  - Backup de passkey no Keychain não restaurável entre Macs (0-3).
  - SAML, SCIM e audit logs como requisito B2B (1-2).
- **Não verificados por orçamento (`maxVerify` 25): 39 claims.** As mais relevantes para revisitar:
  - NIST SP 800-63-4 e passkeys sincronizadas em AAL2;
  - especificações SSF/CAEP/RISC finalizadas em ago–set/2025, com interoperabilidade de 8 implementações;
  - Auth0 for AI Agents (GA nov/2025) e Auth for MCP (mai/2026);
  - preços a 1M MAU;
  - "o CIAM developer-first supera os legados";
  - PQC (FIPS 203/204/205).

  A lista completa está no retorno do run.
- **Fontes descartadas pelo orçamento (`maxFetch` 15): 43.** Entre elas estão o draft
  `oauth-browser-based-apps` (rev. 26), CSA e GitGuardian sobre OAuth em MCP, e os comparativos
  Keycloak × Authentik e Zitadel × Ory.

## Valeu a pena?

- **Custo:** 4.561.324 tokens · 103 agentes · 8,6 min · **21 nós** → **≈ 217k tokens por nó**. É cerca de
  3× o custo típico do censo (68–74k) e fica abaixo da varredura de referência (≈ 291k).
- **Rendimento:** 18 claims confirmadas, das quais 6 em fontes de tier 9–10 (RFC e IETF). A parte de
  **protocolo** sai sólida. A parte de **mercado e custo**, que é a pergunta "gerenciado × self-hosted",
  **não rendeu**: 7 refutadas por fonte fraca. Pela régua da skill, a próxima rodada é **`primaries`**
  (≈ 42k por nó), porque agora as lacunas têm nome.

## Perguntas abertas

1. Qual o TCO real e o risco de lock-in de um IdP gerenciado frente a um self-hosted?
2. O que o NIST SP 800-63B-4 final diz sobre SMS OTP e passkeys sincronizadas em cada AAL?
3. Como fica a identidade de agentes de IA e workloads (token exchange, delegação, autorização MCP)?
4. Como está a portabilidade de passkeys entre ecossistemas (CXP/CXF)?
5. Qual é a primeira decisão de auth da GMill (portal B2B, sistema interno, integrações ERP/SNCM) e que
   requisitos brasileiros se impõem?
6. Quando o OAuth 2.1 vira RFC, e o que muda em relação ao draft 16?
