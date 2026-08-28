# Motokão — RAG Knowledge (extraído do design-system.json)

Fonte: motokao-system-design.zip (10 screenshots Instagram 709x1536 + Motoko sheet)

## Identidade
- Nome: Motokão Moto Center — Desde 1990 — Araras-SP — Rua Visconde do Rio Branco 144
- Nicho: 100-300cc carburadas e injetadas (CG/Biz/Pop/Bros/XRE/CB/Fazer)
- Tom: rápido, técnico, robusto, urbano, preciso, energético, amigável (ninja = precisão + confiança)
- Mascote canônico: Motoko — 3D CGI chibi ninja mecânico, faixa vermelha badge M, máscara preta, traje charcoal, sash/cachecol vermelho
- Anti-genérico: não usar gradiente automotivo genérico, não misturar trajes ninja, não trocar prova oficina por stock tech

## Paleta e Tokens
- color.brand.red.primary: #E30613 — uso: large promotional fields, headband, sashes — confiança: medium
- color.brand.red.deep: #9F0000 — uso: duotone shadows, background depth, dark red overlays — confiança: low
- color.brand.ink: #080B10 — uso: dark canvas, mascot suit, text shadow — confiança: medium
- color.brand.charcoal: #24262C — uso: technical panels, suit midtones, cards — confiança: medium
- color.brand.white: #F7F7F5 — uso: logo, headline, diagonal separators — confiança: medium
- color.accent.neon-amber: #FFC21A — uso: highlight-cover line icons, warm glow, small attention accents — confiança: medium
- color.accent.metal: #9EA5AE — uso: highlight-cover outer rings, cool technical details, subtle UI-like outlines — confiança: low
- color.platform.instagram-blue: #5D70FF — uso: platform action button in screenshot — confiança: medium

## Princípios visuais
- : 
- : 
- : 
- : 

## Componentes observados
- Profile shell: avatar, display name/category, post/follower/following metrics, bio, location/link
- Neon highlight cover: dark circular field, outer gray/metallic rings, inner red/orange ring, yellow line icon, label below
- Motoko mascot hero: red M headband, black mask, large brown eyes and heavy brows, black technical suit, red chest stripes

## Regras de produção
- Usar tokens.css exato (#E30613, #080B10, etc.), não aproximar
- Texto/preço no layout, não na IA; validar contraste; manter 7% safe area
- Motion real pendente (sem vídeo fonte); usar 180/420/720ms como placeholder
- Ver report.html para documentação viva completa
