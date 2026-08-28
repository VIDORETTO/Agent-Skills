# Motokão — handoff do system design

## Entrega

Este pacote contém:

- design-system.json: contrato normalizado em schema v2, com evidências, tokens, componentes, padrões, experiência, efeitos, lacunas e níveis de confiança.
- report.html: documentação viva e responsiva do sistema, usando os próprios tokens.
- tokens.css: exportação CSS dos tokens semânticos.
- assets/motoko-character-sheet.png: prancha única de continuidade do mascote Motoko.
- prompts/motoko-character-sheet.txt: prompt utilizado para a geração da prancha.
- evidence/: as dez capturas originais fornecidas, preservadas como evidência.

## Método e fidelidade

Aplicação da skill extract-web-design-system do repositório:
https://github.com/VIDORETTO/system-designs/tree/main/skill

A análise é screenshot-only. As imagens têm 709×1536 px e incluem o chrome do Instagram. A identidade de marca foi separada do chrome da plataforma. O pacote diferencia observed, computed, inferred e recommended, além de high, medium e low confidence.

## Principal fonte de verdade

Para o mascote, use assets/motoko-character-sheet.png como referência canônica. A versão 3D CGI é a identidade principal; a ilustração plana observada em uma publicação foi registrada como exceção secundária.

Para produção final:

1. use o logotipo vetorial original, que não foi fornecido;
2. use fontes aprovadas, pois a família exata não pode ser comprovada pelas capturas;
3. aplique textos e preços em layout, não dentro do gerador de imagens;
4. valide contraste sobre cada fotografia/crop;
5. defina e registre os timings reais de motion a partir de um vídeo ou protótipo.

## Lacunas que permanecem

- nenhum vídeo ou screen recording original;
- nenhum desktop/tablet ou comparação de breakpoints;
- nenhum DOM, CSS, arquivo de fonte, logo vetorial ou token calibrado;
- nenhum estado hover/focus/pressed/loading;
- nenhum teste de acessibilidade, áudio, legenda ou reduced motion;
- nenhuma confirmação do destino do QR code/link.

## Como abrir

Abra report.html em um navegador. Os caminhos são relativos; mantenha a estrutura do pacote intacta para visualizar as evidências e a prancha do mascote.
