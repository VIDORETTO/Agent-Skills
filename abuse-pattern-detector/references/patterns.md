# Padrões de Abuso — Referência Completa

12 padrões com cenários, variantes e remediações. Usar durante a etapa H (Historical Pattern Matching) do framework BREACHR.

---

## PADRÃO #01 — Trial Abuse por Account Cycling

**Descrição:** Usuário cria conta → usa trial → abandona → cria nova conta → usa trial novamente.

**Variantes:**
- Mesmo email com alias (`user+1@gmail.com`, pontos ignorados pelo Gmail)
- Email temporário (10minutemail, guerrillamail, temp-mail)
- Múltiplos emails pessoais reais
- Email pessoal + email de trabalho

**Identificadores e força:**
| Sinal | Força | Motivo |
|-------|-------|--------|
| Email | Fraco | Criar novos é trivial e gratuito |
| Telefone | Médio | Chips baratos mas dão fricção |
| CPF | Forte (BR) | Nem sempre coletado no cadastro |
| Device fingerprint | Forte | Difícil evadir sem expertise |
| Browser fingerprint | Médio | Evadível com modo anônimo |
| IP address | Fraco | VPNs e IPs dinâmicos |
| Cartão de crédito | Forte | Trial geralmente não exige |

**Remediações:**
- 🔴 Exigir cartão válido no trial (não cobra, mas vincula identidade)
- 🟡 Device fingerprinting + período de cooldown por dispositivo
- 🟢 Trust score ML com múltiplos sinais + modelo de propensão a converter

**Indicadores:** Alta taxa de contas novas por dispositivo · Uso intenso nos últimos dias do trial · Contas criadas e deletadas rapidamente · Padrão `email+N@domínio`

---

## PADRÃO #02 — Referral Fraud Ring

**Descrição:** Usuário indica a si mesmo (contas falsas) ou forma anel de indicações mútuas.

**Variantes:**
- Auto-referral: conta A indica conta B (mesma pessoa)
- Ring: A→B→C→D→A todos indicam em círculo
- Compra de referrals em fóruns/grupos/Telegram
- Bot farms para criar contas e completar ações mínimas

**Sinais de alerta:**
- Conta indicada nunca faz ação real após signup
- Mesmo IP/device entre indicador e indicado
- Velocidade anormal de acúmulo de referrals
- Contas indicadas com perfis mínimos/incompletos
- Cluster de contas criadas em sequência com link de referral

**Remediações:**
- 🔴 Crédito só libera após ação real do indicado (compra, X dias de uso ativo)
- 🟡 Limite de referrals por conta + verificação de identidade para volume alto
- 🟢 Graph analysis: detectar clusters suspeitos + ML de detecção de rings

---

## PADRÃO #03 — Coupon / Promo Stacking

**Descrição:** Combinação de múltiplos descontos que juntos resultam em preço irreal ou negativo.

**Cenário de abuso (e-commerce):**
```
Produto:               R$100,00
Cupom 1ª compra -30%:  R$ 70,00
Cupom influencer -20%: R$ 56,00
Cashback parceiro 15%: R$ 47,60 efetivo
Frete grátis (+R$15):  R$ 47,60 total pago
Revenda por R$80:      LUCRO de R$32,40
```

**Variantes:**
- Cupom + cashback + frete grátis + promoção sazonal
- Cupom de influencer + cupom de primeira compra
- Erro de pricing + cupom válido
- Desconto de funcionário + promoção pública

**Remediações:**
- 🔴 Regra hard: máximo 1 cupom por pedido
- 🟡 Motor de promoções com regras de exclusão mútua explícitas
- 🟢 Price floor: nunca abaixo de X% do preço original independente de combinações

---

## PADRÃO #04 — Refund / Return Abuse

**Descrição:** Compra, usa/consome, e solicita reembolso alegando problema inexistente.

**Variantes por tipo de produto:**

*Físico:*
- "Wardrobing": compra roupa, usa em evento, devolve
- "DNA" (Did Not Arrive): alega não recebimento sendo que recebeu
- Danifica intencionalmente para acionar garantia

*Digital (SaaS, cursos, conteúdo):*
- Assiste/consome todo o conteúdo → pede reembolso no dia 29/30
- Exporta todos os dados → cancela → pede reembolso
- Usa API intensivamente → pede reembolso do período

**Remediações:**
- 🔴 Limitar reembolso se >X% do conteúdo foi consumido (para digital)
- 🟡 Histórico de reembolsos por usuário + risk scoring progressivo
- 🟢 Reembolso proporcional ao não-utilizado (pro-rata) + foto de entrega obrigatória

---

## PADRÃO #05 — Race Condition / Double Spend

**Descrição:** Múltiplas requisições simultâneas para gastar o mesmo saldo/crédito mais de uma vez.

**Variantes:**
- Duplo clique em "usar cupom" enviando 2 requests paralelos
- Múltiplas abas: resgatar saldo em dois pedidos simultaneamente
- API race: enviar N requests paralelos de resgate via script
- Timing entre débito e confirmação no banco de dados

**Cenário técnico:**
```
T=0ms: Request A lê saldo = R$50 ✓
T=1ms: Request B lê saldo = R$50 ✓ (lock ainda não aplicado)
T=5ms: Request A debita R$50, saldo = R$0
T=6ms: Request B debita R$50, saldo = -R$50 ← PROBLEMA
```

**Remediações:**
- 🔴 Mutex/lock pessimista no saldo durante transação
- 🟡 Idempotency keys em TODAS as operações financeiras
- 🟢 Event sourcing com validação transacional ACID + isolamento Serializable

---

## PADRÃO #06 — Free Tier / Freemium Abuse

**Descrição:** Usar o tier gratuito de forma que substitua completamente o plano pago.

**Variantes:**
- Múltiplas contas free = equivalente a 1 conta paga com mais recursos
- Usar free tier via API para volume acima do planejado
- Criar nova organização/workspace quando o atual enche
- Abusar de armazenamento gratuito como "HD infinito" rotativo
- Compartilhar credenciais de conta free com equipe inteira

**Remediações:**
- 🔴 Rate limiting agressivo no free tier vs pago
- 🟡 Feature gating: funcionalidades-chave exclusivas do plano pago
- 🟢 Usage-based billing com free tier como amostra genuína (não substituto)

---

## PADRÃO #07 — Pricing / Currency Arbitrage

**Descrição:** Explorar inconsistências de preço entre regiões, moedas, plataformas ou momentos.

**Variantes:**
- VPN para comprar na região mais barata (diferença pode ser 60-80%)
- Arbitragem entre web e mobile pricing (App Store/Play Store)
- Explorar delay de atualização de taxa de câmbio
- Comprar gift cards em desconto e usar como pagamento
- Usar preço de estudante/nonprofit indevidamente

**Remediações:**
- 🔴 Vincular preço ao país do meio de pagamento (não do IP)
- 🟡 Verificação periódica de elegibilidade (estudante, ONG) a cada 12 meses
- 🟢 Pricing dinâmico com anti-arbitrage checks + detecção de VPN

---

## PADRÃO #08 — Engagement / Metric Farming

**Descrição:** Gerar engajamento artificial para ganhar vantagens em sistemas que premiam engajamento.

**Variantes:**
- Click farms para aumentar views/likes/plays
- Bots para completar "desafios" e ganhar recompensas
- Auto-play/auto-scroll para gaming de watch time ou read time
- Grupos de "engagement mútuo" (pods)
- Manipular métricas de gamification para badges e ranking

**Remediações:**
- 🔴 Validação de ação real (CAPTCHA, interação humana verificável)
- 🟡 Anomaly detection em padrões de engajamento (timing perfeito, sessões 24/7)
- 🟢 Delayed rewards + multi-signal validation antes de liberar benefício

---

## PADRÃO #09 — Account Sharing / Credential Abuse

**Descrição:** Compartilhar credenciais de uma conta paga entre múltiplos usuários.

**Variantes:**
- Senha compartilhada em equipe inteira (1 seat, 20 usuários)
- Revenda de acesso a conta premium em grupos/fóruns
- Compartilhamento de tokens/sessões via API
- "Conta empresarial" de 1 seat usada por departamento inteiro

**Sinais:**
- Login de múltiplos IPs/geolocalizações simultâneos (países diferentes)
- Sessões ativas muito acima do padrão para perfil individual
- Padrão de uso 24/7 que não faz sentido para pessoa física

**Remediações:**
- 🔴 Limite de sessões simultâneas ativas (ex: máximo 3)
- 🟡 Device trust: alerta e verificação em novo device não cadastrado
- 🟢 Behavioral biometrics + concurrent session rules + plano família como alternativa legítima

---

## PADRÃO #10 — Policy Loophole Exploitation

**Descrição:** Usar a LETRA da política contra o ESPÍRITO dela.

**Variantes:**
- "Satisfação garantida ou dinheiro de volta" sem limite de usos → cicla mensalmente
- "Troca grátis" usada como guarda-roupa rotativo
- "Acesso vitalício" com termos ambíguos sobre transferência
- "Uso ilimitado" interpretado como permissão para revenda
- Explorar jurisdição diferente para escapar de restrições geográficas

**Remediações:**
- 🔴 Revisão jurídica dos termos com cenários de abuso explicitados
- 🟡 Cláusula de "fair use" com definição quantitativa (ex: máximo 2 trocas/ano)
- 🟢 Termos dinâmicos com enforcement automatizado + registro de histórico por usuário

---

## PADRÃO #11 — Fee Avoidance em Marketplaces

**Descrição:** Mover transações para fora da plataforma para evitar taxa de intermediação.

**Métodos de evasão de filtros de contato:**
- Escrever número por extenso: "nove nove, meia um..."
- Usar Unicode homoglyphs (а cirílico vs a latino)
- Enviar imagem com contato embutido (burla regex de texto)
- Links para portfólio/redes sociais que contêm contato
- Acrônimos conhecidos: "me chama no W", "meu insta é X"
- Escrita invertida ou com espaços: "moc.liamg @ oaoj"

**Impacto:** Primeira transação na plataforma → troca de contato → todas as futuras são off-platform → perda de 100% da receita futura desse par.

**Remediações:**
- 🔴 NLP classifier em vez de regex puro para detectar contatos
- 🟡 OCR em imagens enviadas no chat + taxa regressiva que incentiva ficar na plataforma
- 🟢 Tornar a plataforma tão valiosa (disputas, garantias, reputação) que sair não compense

---

## PADRÃO #12 — Rating / Ranking Manipulation

**Descrição:** Criar avaliações falsas para melhorar ranking próprio ou prejudicar concorrentes.

**Variantes:**
- Ring de avaliações mútuas (A avalia B, B avalia C, C avalia A)
- Contas "cliente" próprias para auto-avaliação
- Compra de reviews em fóruns especializados
- Avaliar concorrente com 1 estrela via conta falsa

**Cenário de ring (marketplace):**
```
5 vendedores criam contas de "cliente" mútuas
Custo: R$50 × 5 × 15% taxa = R$37,50 total
Resultado: 5 vendedores com rating 5.0 perfeito + badge "Top Rated"
ROI: badge vale centenas/mês em visibilidade
```

**Remediações:**
- 🔴 Peso da review proporcional ao valor da transação
- 🟡 Graph analysis: detectar clusters de "clientes" que só avaliam os mesmos vendedores
- 🟢 PageRank-like: review de cliente com histórico longo e diverso vale mais · Economic incentive: tornar fake reviews inviáveis financeiramente