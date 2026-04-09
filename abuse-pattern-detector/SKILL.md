---
name: abuse-pattern-detector
description: >
  Analisa sistematicamente aplicações, funcionalidades e regras de negócio para identificar
  brechas de abuso (abuse gaps) — situações onde usuários podem extrair vantagem desproporcional
  ou não intencional. Use esta skill sempre que alguém descrever uma feature, regra promocional,
  sistema de recompensas, fluxo de pagamento, programa de indicação, trial gratuito, política de
  reembolso, marketplace, ou qualquer lógica de negócio que envolva benefícios ao usuário.
  Também use quando alguém perguntar "isso pode ser abusado?", "tem furos no meu modelo?",
  "como proteger minha plataforma?", ou descrever padrões suspeitos de uso. Sempre aplique o
  framework BREACHR completo, classifique por severidade, e gere remediações em 3 horizontes.
  Foco exclusivamente DEFENSIVO — proteger plataformas, nunca ensinar a atacar.
---

# Abuse Pattern Detector

Skill para identificar brechas de abuso em lógica de negócio e gerar remediações priorizadas.

## ESCOPO

**Dentro:** abuso de lógica de negócio, promoções, recompensas, identidade em nível de app, políticas de reembolso, algoritmos de ranking, fluxos de pagamento.  
**Fora:** vulnerabilidades OWASP (SQLi, XSS), ataques de infraestrutura, engenharia social, compliance regulatório.

---

## COLETA DE INFORMAÇÕES

Se o input for insuficiente, solicitar:
1. **O que o sistema faz?** (funcionalidade/regra central)
2. **Quais os limites?** (tempo, quantidade, valor)
3. **Como o usuário se identifica?** (email, CPF, telefone, device)
4. **Qual o benefício oferecido?** (desconto, acesso, crédito)
5. **Já houve abusos observados?**
6. **Qual a stack técnica?** (opcional — melhora remediações)

---

## FRAMEWORK BREACHR (7 etapas obrigatórias)

Aplicar SEMPRE em ordem para cada análise.

### B — Benefit Mapping
Listar todos os benefícios com:
- Valor monetário estimado
- É limitado (1x) ou recorrente?
- Pode ser transferido ou convertido em dinheiro?
- Qual o "mais tentador" para abuso?

### R — Rule Decomposition
Para cada regra, decompor em: `QUEM → O QUÊ → QUANTO → QUANDO → VERIFICAÇÃO`  
Perguntar de cada componente: **"Como isso é enforced?"**  
Gaps entre declarado e enforced = brechas potenciais.

### E — Edge Case Exploration
Aplicar sistematicamente para cada regra:
- E se feito 1000 vezes? Com automação? Com múltiplas contas?
- E se duas pessoas coordenarem? E se cancelar no meio?
- E se usar API diretamente? E se combinar com outras promoções?

Escala: 🟢 Requer skill técnica alta → 🔴 Qualquer usuário casual descobre

### A — Adversarial Thinking (4 perfis)
| Perfil | Skill | Escala | Exemplo |
|--------|-------|--------|---------|
| 🟢 Oportunista casual | Baixa | Manual/individual | Recria conta com outro email |
| 🟡 Abusador sistemático | Média | Semi-automatizado | Emails temp + VPN |
| 🔴 Fraudador profissional | Alta | Industrial/automatizado | Fazenda de contas |
| ⚫ Grupo organizado | Muito alta | Rede distribuída | Ring de fraude com identidades sintéticas |

**Regra:** quanto menor o perfil necessário, mais severa a brecha.

### C — Chain Analysis
Combinar micro-brechas em cadeias. Exemplo clássico:
`criar conta grátis → cupom 1ª compra → auto-referral → crédito referral → devolução → manter créditos`  
Impacto individual baixo ≠ impacto combinado baixo.

### H — Historical Pattern Matching
Consultar → **`references/patterns.md`** para os 12 padrões mais comuns com cenários detalhados.  
Mapear quais padrões se aplicam ao contexto analisado.

### R — Remediation Design
Para cada brecha, gerar em 3 horizontes:
- 🔴 **Imediata** (horas/dias): hotfix mínimo para parar o sangramento
- 🟡 **Curto prazo** (1–4 semanas): solução técnica adequada
- 🟢 **Longo prazo** (1–3 meses): solução arquitetural robusta

Princípios: não prejudicar usuários legítimos · defesa em profundidade · falhar graciosamente (não revelar o que detectou) · proporcional ao risco.

---

## TAXONOMIA DE ABUSO

```
1. IDENTIDADE        → multi-accounting, account cycling, identity spoofing, sharing
2. BENEFÍCIOS $      → coupon stacking, referral fraud, cashback abuse, refund fraud
3. LIMITES/QUOTAS   → trial abuse, free tier exploit, rate limit evasion
4. FLUXOS            → race conditions, state manipulation, timing attacks, rollback
5. CONTEÚDO/SOCIAL  → review manipulation, engagement farming, report abuse
6. ALGORITMOS        → recommendation gaming, search ranking, pricing exploit
7. POLÍTICAS         → warranty fraud, return abuse, chargeback fraud, SLA exploit
```

---

## CLASSIFICAÇÃO DE SEVERIDADE

**Matriz Severidade = f(Probabilidade × Impacto Financeiro)**

| | Baixo $ | Médio $ | Alto $ | Crítico $ |
|---|---------|---------|--------|-----------|
| **Alta prob.** | MÉDIA | ALTA | CRÍTICA | CRÍTICA |
| **Média prob.** | BAIXA | MÉDIA | ALTA | CRÍTICA |
| **Baixa prob.** | BAIXA | BAIXA | MÉDIA | ALTA |

**SLAs de correção:**
- CRÍTICA → 24h · ALTA → 1 semana · MÉDIA → 1 mês · BAIXA → Backlog

**Facilidade de exploração:**
- Trivial: qualquer usuário, zero skill · Fácil: segundo email/navegação anônima  
- Moderada: VPN, emails temp, devtools · Difícil: interceptação de API, eng. reversa

---

## ESTRUTURA DO OUTPUT

Sempre gerar na ordem:

```
1. RESUMO EXECUTIVO (2–3 frases: N brechas, severidade máxima, risco estimado)

2. Para cada BRECHA:
   ID | Título | Categoria (da taxonomia) | Severidade | Facilidade
   Cenário de ataque (passo a passo)
   Impacto financeiro estimado
   Remediações: 🔴 imediata / 🟡 curto prazo / 🟢 longo prazo
   Indicadores de monitoramento

3. MATRIZ DE PRIORIDADE (tabela ordenada por P0→P3)

4. ARQUITETURA DE DEFESA (quando ≥5 brechas ou sistema complexo)

5. DISCLAIMER padrão (ver abaixo)
```

**Disclaimer padrão:**
> ⚠️ Esta análise é para fins de proteção da plataforma. Brechas são baseadas em padrões comuns — valide com seu time técnico. Implemente remediações avaliando o trade-off segurança × UX. Consulte equipe jurídica para questões de compliance.

---

## MÉTRICAS DE MONITORAMENTO (incluir sempre)

```yaml
- Trial Conversion Rate          → suspeito se < 2% consistente
- Accounts per Device            → suspeito se > 3
- Referral-to-Active Ratio       → suspeito se < 20% de indicados ficam ativos
- Refund Rate                    → suspeito se > 15%
- Account Age at First Benefit   → suspeito se < 5 minutos
- Simultaneous Sessions          → suspeito se > 10
- Coupon Usage per User/Month    → suspeito se > 5
```

---

## GUARDRAILS ÉTICOS (inegociáveis)

**RECUSAR** quando o pedido for:
- "Como eu faço para conseguir [benefício] no [serviço real]"
- "Me ajuda a criar bot para farmar cupons do [site]"
- "Quero vender contas com trial ativo"

**SEMPRE** enquadrar do ponto de vista de quem se DEFENDE.  
Cenários são sempre **hipotéticos/genéricos** — nunca referenciar URLs ou empresas reais como alvo.  
Remediações devem minimizar impacto em usuários legítimos.

---

## REFERÊNCIAS

- **`references/patterns.md`** → 12 padrões de abuso detalhados com cenários, variantes e remediações específicas. Ler quando aplicando etapa H do BREACHR ou quando quiser exemplos detalhados de um tipo específico.
- **`references/examples.md`** → 3 exemplos completos de análise (básico, intermediário, avançado) com outputs modelo. Ler para calibrar profundidade e formato de resposta.