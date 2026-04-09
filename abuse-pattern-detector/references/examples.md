# Exemplos de Análise — Referência de Calibração

Três exemplos completos para calibrar profundidade e formato de resposta.

---

## EXEMPLO 1 — Básico: App de Streaming com Trial

**Input:** *"App de streaming de música. Novos usuários ganham 30 dias grátis de premium (R$19,90/mês depois). Cadastro por email."*

---

**RESUMO EXECUTIVO:** 4 brechas identificadas. A mais crítica é trial infinito por account cycling — trivial de executar, pode impactar 10-15% dos trials. Recomendo ação imediata em #1 e #4.

---

### BRECHA #1 — Trial Infinito por Account Cycling
📊 **ALTA** | 🎯 TRIVIAL | Categoria: Abuso de Limites (Trial Abuse)

**Cenário:**
1. Cria conta com email1@gmail.com → usa 30 dias grátis
2. Expirou → cria com email2@gmail.com → mais 30 dias
3. Repete indefinidamente (Gmail aliases funcionam: user+1@, u.s.e.r@)

**Impacto:** Se 10% dos 10.000 trials/mês abusarem → ~R$19.900/mês em receita perdida

**Remediações:**
- 🔴 Exigir número de telefone no cadastro
- 🟡 Device fingerprinting — vincular trial ao dispositivo, não ao email
- 🟢 Trial com cartão obrigatório (não cobra, mas vincula identidade financeira)

**Monitorar:** Contas do mesmo device · `email+N@` · Trial conversion rate < 2%

---

### BRECHA #2 — Account Sharing
📊 **MÉDIA** | 🎯 FÁCIL | Categoria: Abuso de Identidade

**Cenário:** 1 pessoa assina premium → compartilha senha com 5 amigos → R$3,98/pessoa em vez de R$19,90

**Remediações:**
- 🔴 Limite de 1 sessão simultânea
- 🟡 Máximo de 3 dispositivos cadastrados
- 🟢 Plano família com preço intermediário como alternativa legítima

---

### BRECHA #3 — Email Alias Exploitation
📊 **MÉDIA** | 🎯 TRIVIAL | Categoria: Abuso de Identidade

**Cenário:** Gmail trata como mesmo email: `user@`, `u.s.e.r@`, `user+tag@`. Muitos sistemas não normalizam e permitem múltiplos cadastros.

**Remediação:**
- 🔴 Normalizar emails no cadastro: remover dots e aliases `+tag` para provedores conhecidos

---

### BRECHA #4 — Timer Client-Side
📊 **BAIXA** | 🎯 MODERADA | Categoria: Abuso de Fluxos

**Cenário:** Se expiração do trial for verificada client-side, alterar data/hora do device pode "voltar no tempo".

**Remediação:**
- 🔴 Verificar expiração SEMPRE server-side (nunca confiar no client)

---

| P | Brecha | Sev. | Facil. |
|---|--------|------|--------|
| P0 | Account Cycling | ALTA | TRIVIAL |
| P1 | Email Alias | MÉDIA | TRIVIAL |
| P1 | Account Sharing | MÉDIA | FÁCIL |
| P2 | Timer Client-Side | BAIXA | MODERADA |

---

## EXEMPLO 2 — Intermediário: E-commerce com Ecossistema Promocional

**Input:** *"E-commerce. Cupom 1ª compra (R$25 off em compras acima de R$50) + indicação (ambos ganham R$15) + frete grátis acima de R$99 + devolução 30 dias dinheiro de volta + cashback 5% no app. Login por email, sem verificação de identidade. API pública para parceiros."*

---

**RESUMO EXECUTIVO:** 8 brechas identificadas, 2 CRÍTICAS. A cadeia Referral+Coupon pode custar R$5k–15k/mês se explorada em escala. Ação imediata em #1 e #2.

---

### BRECHA #1 — Cadeia Referral + Coupon + Cashback + Refund
📊 **CRÍTICA** | 🎯 FÁCIL | Categoria: Chain Attack

**Cenário completo:**
```
Conta A cria link de indicação
Conta B (mesma pessoa, outro email) usa o link

Conta B: compra R$55
  - Cupom 1ª compra:   -R$25  → R$30
  - Crédito referral:  -R$15  → R$15 pago
  - Cashback 5%:       +R$0,75

Conta A faz o mesmo

Ambas pedem devolução nos 30 dias → recebem R$15 de volta cada
Cashback não é estornado (gap)

RESULTADO: gastou R$0, gerou R$1,50 em cashback + uso gratuito dos produtos por 30 dias
PREJUÍZO: R$80 em cupons + R$30 em referrals + frete + operacional
```

**Remediações:**
- 🔴 Crédito de referral só libera 31+ dias após 1ª compra sem devolução · Cupom 1ª compra e referral mutuamente exclusivos
- 🟡 Device fingerprint entre indicador e indicado · Cashback só credita após janela de devolução
- 🟢 Trust score: contas novas têm limite de benefícios simultâneos · Graph analysis de clusters

---

### BRECHA #2 — Refund com Cashback Retention
📊 **CRÍTICA** | 🎯 TRIVIAL | Categoria: Refund Abuse

**Cenário:** Compra R$200 → recebe R$10 cashback → devolve tudo → recebe R$200 de volta → mantém R$10. Repetível infinitamente com compras grandes.

**Remediação:**
- 🔴 Cashback só credita APÓS janela de devolução fechar (31 dias)
- 🟡 Estorno automático de cashback em caso de devolução total

---

### BRECHA #3 — API Abuse por Parceiros
📊 **ALTA** | 🎯 MODERADA | Categoria: Abuso de Fluxos

**Cenário:** API pública pode automatizar criação de contas e aplicação de cupons programaticamente em escala.

**Remediações:**
- 🔴 API de parceiros NÃO pode aplicar cupons de 1ª compra/referral
- 🟡 Rate limiting por API key + audit log detalhado

---

### BRECHA #4 — Frete Grátis Gaming
📊 **BAIXA** | 🎯 TRIVIAL | Categoria: Abuso de Políticas

**Cenário:** Compra item R$30 + item R$70 para atingir frete grátis → devolve o item de R$70 → fica com frete grátis.

**Remediação:**
- 🔴 Devolução parcial que reduza total abaixo de R$99 gera cobrança retroativa de frete

---

| P | Brecha | Sev. | Facil. | Impacto/mês |
|---|--------|------|--------|-------------|
| P0 | Cadeia Referral+Coupon | CRÍTICA | FÁCIL | R$5k–15k |
| P0 | Refund+Cashback | CRÍTICA | TRIVIAL | R$3k–10k |
| P1 | API Abuse | ALTA | MODERADA | R$1k–5k |
| P1 | Self-Referral massa | ALTA | FÁCIL | R$2k–6k |
| P2 | Cupom infinito | MÉDIA | FÁCIL | R$1k–3k |
| P3 | Frete grátis gaming | BAIXA | TRIVIAL | R$500–1k |

---

## EXEMPLO 3 — Avançado: Marketplace de Freelancers

**Input:** *"Plataforma de freelancers. Taxa 15% por transação. Escrow com auto-aprovação em 14 dias. Avaliações bidirecionais 1-5 estrelas. Badge 'Top Rated' (>4.8⭐ + >50 projetos + >90% completion). Programa de afiliados 5% da 1ª transação. Chat interno com regex anti-contato externo. Stack: React + Node.js + PostgreSQL + Stripe. API REST com JWT."*

---

**RESUMO EXECUTIVO:** 12 brechas em 5 clusters. As mais graves: fee avoidance via evasão de regex (CRÍTICA), rating manipulation ring (CRÍTICA), e auto-approval timer abuse (CRÍTICA). Risco de 20–40% de receita perdida por transações off-platform.

---

### CLUSTER 1: FEE AVOIDANCE

#### BRECHA #1 — Off-Platform via Evasão de Regex
📊 **CRÍTICA** | 🎯 FÁCIL

**Cenário:** O filtro regex detecta emails e telefones óbvios. Evasões triviais:
- `"meu zap: nove nove, meia um..."` — número por extenso
- `"me chama no W"` — acrônimo para WhatsApp
- `"moc.liamg@oaoj"` — escrita invertida
- Enviar imagem com contato (regex não processa imagem)
- Unicode homoglyphs: `а` (cirílico) vs `a` (latino)

**Impacto:** 1ª transação na plataforma → troca contato → todas as futuras são off-platform → 100% das taxas futuras perdidas.

**Remediações:**
- 🔴 NLP classifier em vez de regex · Bloquear imagens nos primeiros N projetos
- 🟡 OCR em imagens do chat · Taxa regressiva (15%→10%→7%) para incentivar fidelidade
- 🟢 Tornar a plataforma indispensável: disputas, garantias, portfólio verificado

---

### CLUSTER 2: MANIPULAÇÃO DE RANKING

#### BRECHA #2 — Rating Ring
📊 **CRÍTICA** | 🎯 MODERADA

**Cenário:**
```
5 freelancers criam contas "cliente" mútuas
Custo: 5 micro-projetos × R$50 × 15% = R$37,50
Resultado: 5 perfis com rating 5.0 perfeito + caminho para Top Rated
ROI: badge Top Rated vale centenas/mês em visibilidade
```

**Remediações:**
- 🔴 Peso da review proporcional ao valor da transação (review de R$5 vale pouco)
- 🟡 Graph analysis: detectar clusters onde "clientes" só avaliam freelancers específicos
- 🟢 PageRank-like: review de cliente com histórico longo e diverso vale muito mais

---

### CLUSTER 3: ABUSO DE ESCROW

#### BRECHA #3 — Auto-Approval Timer Exploit
📊 **CRÍTICA** | 🎯 FÁCIL

**Cenário:** Regra "se cliente não aprovar em 14 dias, libera automaticamente."
- Freelancer entrega trabalho de qualidade inferior → espera cliente desatento → dinheiro liberado
- Variante: marca como "entregue" sem entregar nada → timer de 14 dias → dinheiro liberado

**Remediações:**
- 🔴 Auto-aprovação só acontece se cliente ABRIU e VISUALIZOU a entrega · Notificações agressivas (email + push + SMS nos dias 7, 10, 13)
- 🟡 Período proporcional ao valor: R$50=7 dias · R$500=14 dias · R$5k=30 dias
- 🟢 Milestone-based delivery com aprovações parciais + AI review básico da entrega

---

#### BRECHA #4 — Dispute Abuse pelo Cliente (Trabalho+Reembolso)
📊 **ALTA** | 🎯 FÁCIL

**Cenário:** Cliente recebe trabalho de qualidade → faz download completo → abre disputa "não atende requisitos" → consegue reembolso + fica com o trabalho.

**Remediações:**
- 🔴 Marca d'água no trabalho até aprovação final
- 🟡 Escrow por milestones: liberação progressiva
- 🟢 Hash/timestamp das entregas como prova

---

### CLUSTER 4: ABUSO DE AFILIADOS

#### BRECHA #5 — Self-Referral + Transação Mínima
📊 **MÉDIA** | 🎯 FÁCIL

Freelancer cria conta de "cliente" própria → indica com link de afiliado → faz transação mínima → ganha 5% de afiliado sobre compras futuras reais. Custo baixo, benefício perpétuo.

**Remediação:** Device fingerprint entre indicador e indicado · Requisito de transação real do indicado acima de threshold

---

### CLUSTER 5: ABUSO TÉCNICO

#### BRECHA #6 — Race Condition no Escrow
📊 **ALTA** | 🎯 DIFÍCIL

Aprovação de milestone e abertura de disputa simultâneas podem causar double-release do escrow se o PostgreSQL não estiver com isolamento adequado.

**Remediação:**
- 🔴 Serializable isolation level nas transações de escrow · Distributed lock no `escrow_id`

---

**PRIORIZAÇÃO:**

```
SPRINT 1 (esta semana):
  #3 Auto-Approval Timer — reformular regra
  #1 Off-Platform Detection — upgrade para NLP
  #2 Rating Ring — peso por valor de transação

SPRINT 2 (próximas 2 semanas):
  #4 Dispute Abuse — marca d'água
  #6 Race Condition — fix de isolamento

SPRINT 3-4 (próximo mês):
  #5 Self-Referral — device fingerprint
  Graph analysis engine para ring detection

ROADMAP (trimestre):
  ML fraud scoring · Trust & Safety team · Taxa regressiva
```