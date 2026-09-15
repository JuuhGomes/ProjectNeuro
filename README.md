# Sistema Neuropsicológico — Juliana

Sistema de acompanhamento neuropsicológico longitudinal para TDAH Apresentação Combinada
Severa + Dupla Excepcionalidade (2e) verbal. Compara a evolução com o baseline de
Março/2026 através de check-ins diários/semanais com instrumentos clínicos validados.

## Arquivos

- **`index.html`** — Sistema principal. Check-ins diários/semanais rotativos (Digit Span,
  Stroop, Fluência, BRIEF-A, ASRS-18, Contexto & Garmin), HUD gamificado, histórico,
  gráficos SVG de evolução, ranking, portal de múltiplos perfis com PIN opcional.
- **`protocolo.html`** — Protocolo de avaliação formal para os checkpoints de 3/6/9 meses.
  Dois campos: avaliação completa (testes + questionários) e análise comparativa com
  geração automática de laudo. Usa Chart.js via CDN (não é standalone como o `index.html`).
- **`data/baseline.json`** — Relatório estruturado da avaliação baseline (Março/2026):
  perfil cognitivo (WAIS-IV), personalidade, TDAH (ASRS-18/BRIEF-A), histórico
  medicamentoso, dados Garmin, recomendações e metas de reavaliação.
- **`HANDOFF.md`** — Documento de transferência com todo o contexto clínico e técnico do
  projeto, arquitetura do sistema e lista de pendências.
- **`PROMPT-ORIGINAL.md`** — Especificação técnica original usada para desenvolver o
  `index.html` (restrições técnicas, modelo de dados, design system, critérios de aceite).

## Como usar

Abra `index.html` com duplo clique — funciona em `file://`, sem servidor, sem build,
sem dependências além das fontes do Google Fonts (com fallback `system-ui`).

## Restrições técnicas do `index.html`

- Arquivo HTML único (CSS + JS inline)
- Sem bibliotecas externas (sem React, Chart.js, D3, jQuery)
- Gráficos em SVG inline escritos à mão
- Funciona em `file://` — sem `import` de módulos, sem `fetch` de JSON local
- Dados em `localStorage`, com `try/catch` em toda leitura/escrita

Veja `HANDOFF.md` seção 6 para a lista de pendências (tooltip/crosshair nos gráficos,
filtros de período, tabela acessível, etc.).
