# HANDOFF — Sistema Neuropsicológico Juliana
**Documento de transferência para Claude Code**
Gerado em: 15/09/2026 | Projeto: `Saúde Mental Retorno ✨`

---

## 1. CONTEXTO DO PROJETO

### Quem é a usuária
- **Juliana** — Analista Sênior DP e Gestão de Pessoas, 5 anos de experiência
- MBA Liderança e Gestão de Pessoas + Extensão Finanças Comportamentais
- Vive em Florianópolis
- **Diagnóstico:** TDAH Apresentação Combinada Severa + Dupla Excepcionalidade (2e) verbal
- Em recuperação de burnout desde início de 2026 após mudança de emprego

### Objetivo do projeto
Criar um **sistema de acompanhamento neuropsicológico longitudinal** que:
1. Realiza check-ins diários/semanais com instrumentos clínicos validados
2. Compara resultados com o baseline de Março/2026
3. Gera histórico, gráficos de evolução e ranking gamificado
4. Suporta múltiplos perfis (mesma família/aparelho)

---

## 2. AVALIAÇÃO BASELINE — MARÇO 2026

### Perfil Cognitivo (framework WAIS-IV)

| Índice | Escore | Percentil | Classificação |
|--------|--------|-----------|---------------|
| ICV (Verbal) | 125–130 | 95–98 | **Muito Superior ⭐** |
| IMO (Memória) | 120–125 | 91–95 | Superior |
| IRP (Perceptual) | 115–120 | 84–91 | Médio-Superior |
| IVP (Velocidade) | 105–110 | 63–75 | Médio |
| **QIT Total** | **118–123** | **88–94** | **Médio-Superior/Superior** |

**Subtestes baseline:**
- Digit Span Forward: 7 | Backward: 6
- Stroop: 8/8 em 15s
- Atenção (CPT): 11/11
- Fluência Semântica: 11 itens
- Aritmética: 4,75s/item
- Gap Funcional: 30–40 pts (QI ideal ~125 vs estresse ~85–95)

### TDAH — ASRS-18 (OMS)

| Dimensão | Escore | % | Classificação |
|----------|--------|---|---------------|
| Desatenção | 22/36 | 61% | Moderada |
| **Hiperatividade** | **35/36** | **97%** | **SEVERA 🔴** |
| Total | 57/72 | 79% | Severo |

### BRIEF-A (Funções Executivas)

| Domínio | % Comprometimento |
|---------|------------------|
| Flexibilidade Cognitiva | **88% SEVERA 🔴** |
| Org/Planejamento | 83% |
| Controle Emocional | 75% |
| Iniciação | 75% |
| Memória de Trabalho | 75% |
| Inibição | 63% |
| **Total** | **76%** |

**Paradoxo central:** Flexibilidade 100% em testes / 88% comprometida no cotidiano → TDAH + rigidez compensatória.

### Impacto Funcional (0=nenhum, 10=máximo)
- Trabalho: 6/10
- Relações: 6/10
- **Bem-estar: 8/10 🔴** ("motor ligado" permanente)

### Personalidade

| Instrumento | Resultado |
|-------------|-----------|
| Big Five — Abertura | 2–3/10 (muito baixa) |
| Big Five — Conscienciosidade | 9/10 (muito alta, raro em TDAH) |
| Big Five — Extroversão | 6–7/10 |
| Big Five — Amabilidade | 3,7/10 |
| Big Five — Neuroticismo | 5/10 |
| DISC Dominância | 56% |
| DISC Influência | 12,5% |
| DISC Estabilidade | 17,5% |
| DISC Conformidade | 20,95% |
| Sensorial Dunn — Sensibilidade | 42/70 (acima da média ⚠️) |
| Sensorial Dunn — Exploração | 41/70 (abaixo da média) |

### Garmin — Baseline (Fev–Mar 2026)
- HRV: 32ms (meta: ≥45ms)
- Sleep Score: 81/100 (meta: ≥85)
- FC Repouso: 60bpm
- Body Battery: +53pts
- Estresse pico: sextas 36–40 (coincide com crash Ritalina 18h)

### Medicação Atual
- Ritalina LA 10mg · 1x/dia · 6h30
- Crash marcado às 18h
- **Subtratamento evidente** (hiperatividade 97% não controlada)
- Histórico: falhas com Strattera, Bupropiona, Metilfenidato 10mg 2x, Ritalina LA 20mg

---

## 3. REAVALIAÇÃO — SETEMBRO 2026

**Data:** 03/09/2026 (6 meses após baseline)

**Estado no dia:**
- Sleep Score: 77 (abaixo baseline 81)
- Body Battery: 77
- HRV: 32ms (estável, ainda subótimo)
- Energia: 7–8/10
- Burnout: recuperação ótima, mudança de trabalho
- Estressor atual: necessidade de mudança de casa (gerindo bem)
- Medicação: sem mudanças (Guanfacina não iniciada)
- Decisão Medicina/Psiquiatria: em análise

---

## 4. ARQUITETURA DO SISTEMA

### Modelo C — Híbrido (adotado)

```
DIÁRIO (2 min) — contexto rápido
  → 3 itens ASRS rotativos (ciclo de 6 dias)
  → Energia + sono + HRV

SEMANAL (rotação, 5–8 min) — um instrumento por dia:
  Segunda   → Contexto & Garmin (dados fisiológicos completos)
  Terça     → Digit Span (Memória de Trabalho)
  Quarta    → Stroop + Aritmética (Controle Inibitório + Velocidade)
  Quinta    → Fluência + Analogias (Verbal + Semântica)
  Sexta     → BRIEF-A (Funções Executivas — 6 domínios)
  Sábado    → ASRS-18 (TDAH completo)
  Domingo   → Síntese Semanal (score composto automático)

SCORE SEMANAL COMPOSTO → gráfico longitudinal → comparação com baseline
```

### Score por Instrumento

| Instrumento | Fórmula |
|-------------|---------|
| Contexto & Garmin | Energia 25% + Sono 20% + HRV 30% + Sleep 15% + Estresse 10% |
| Digit Span | Forward 50% + Backward 50% (base: fwd 7, bwd 6) |
| Stroop + Aritmética | Stroop 50% + Acertos arit 30% + Velocidade 20% |
| Fluência + Analogias | Fluência 60% + Analogias 40% (base: 11 itens, 4/5) |
| BRIEF-A | 100 − % comprometimento (base: 76%) |
| ASRS-18 | 100 − % total (base: 79%) |
| Síntese Semanal | Média dos 6 dias |

### Metas de Reavaliação

| Marco | ASRS Hiper | BRIEF Flex | HRV | Sleep | Sofrimento |
|-------|-----------|-----------|-----|-------|-----------|
| 3m Jun/2026 | <80% | <75% | >35ms | — | <6/10 |
| 6m Set/2026 | <70% | <65% | >40ms | >85 | <5/10 |
| 9m Dez/2026 | <60% | <55% | >45ms | >85 | <4/10 |

---

## 5. ARQUIVOS DESENVOLVIDOS

### 5.1 sistema_neuro.html ← **ARQUIVO PRINCIPAL**
**Publicado em:** https://claude.ai/artifact/SVmxDy3E43v1DhY3arqNJ8
**Tamanho:** ~111 KB | Arquivo HTML único, sem dependências de servidor

#### Funcionalidades implementadas:
- **Portal de perfis** — múltiplos usuários no mesmo aparelho, PIN opcional, escape XSS
- **HUD gamificado** — score semanal, rank (S/A/B/C/D), barra de progresso, streak
- **Grade semanal visual** — 7 dias com instrumento do dia, score se feito, hoje destacado
- **Check-in por instrumento** — wizard em etapas com cronômetros integrados
- **Tela de Level Up** — quando score semanal bate recorde
- **4 abas:** Hoje · Histórico · Evolução · Mais
- **Gráficos SVG inline** — sem biblioteca, relê cores CSS por tema
- **Histórico** — cards por dia, agrupamento por semana, detalhe ao clicar
- **Ranking** — top semanas, records por dimensão
- **Exportação** — JSON e CSV por perfil
- **Tema claro/escuro** — sem piscar, persiste no localStorage, respeita preferência do sistema
- **Persistência** — localStorage com try/catch, migração de dados legados

#### Design system (baseado em "Sistema de Despertar"):
```
Fontes: Chakra Petch (títulos/números) + Rajdhani (corpo)
Cores:  --void:#04070f | --cyan:#5ec7ff | --gold:#f2c14e
        --ok:#46e0a0  | --danger:#ff4d6a | --violet:#a06bff
Forma:  clip-path chanfrado (zero border-radius, exceto toggle de tema)
Gráficos: --s1:#2f95db | --s2:#b8851a | --s3:#8f5cf0 | --s4:#1fae79
```

#### Chaves de localStorage:
```js
"neuro.tema"          // preferência de tema (global, não por perfil)
"neuro.users"         // [{id, nome, pin, visto}]
"neuro.sessao"        // id do perfil logado atualmente
"neuro.save.<id>"     // estado completo de cada perfil
"neuro.sistema.v1"    // chave legada (migração automática)
```

#### Estrutura de dados por perfil:
```js
{
  v: 1,
  semanas: [
    {
      id: "2026-W38",         // YYYY-Www
      dias: {
        "2026-09-15": {
          score: 74,
          instrNome: "Digit Span",
          dados: { digit: { fwd: 7, bwd: 6, score: 74 }, contextoRapido: {...} },
          em: 1757952000000   // timestamp
        }
      },
      scoreGeral: null,       // preenchido no domingo (síntese)
      sinteseEm: null
    }
  ],
  streak: 0,
  melhorStreak: 0,
  recordes: { scoreGeral: null }
}
```

---

### 5.2 protocolo_neuro_v4.html
**Tamanho:** ~95 KB | Protocolo de avaliação formal (checkpoints 3/6/9 meses)

#### Funcionalidades:
- **Dois campos independentes:** Campo 1 (avaliação) + Campo 2 (análise/laudo)
- **Múltiplas sessões** — baseline Mar/2026 pré-carregado, Set/2026 criado automaticamente
- **Persistência** — localStorage com autosave (800ms debounce)
- **Testes:** Digit Span, Stroop (cronômetro 15s), Fluência (60s), CPT, TMT-A/B, Aritmética, Analogias
- **Questionários:** ASRS-18 completo, BRIEF-A com score por domínio, Ruminação, Big Five (escalas clicáveis), Sensorial Dunn
- **Comparação:** tabela baseline vs atual com delta colorido (verde/vermelho)
- **Laudo:** interpretação automática por domínio (melhora/estabilidade/piora)
- **4 gráficos SVG:** QI por índice, ASRS, Garmin, Impacto Funcional
- **Exportação:** CSV e JSON
- **Modo apresentação** — esconde campos editáveis
- **Farmacologia** — tabela de medicamentos com eficácia/tolerabilidade

---

### 5.3 relatorio_juliana_mar2026.json
**Tamanho:** ~17 KB | Relatório completo em JSON estruturado

Seções: `meta`, `perfil_cognitivo`, `personalidade`, `tdah`, `historico_medicamentoso`, `dados_fisiologicos_garmin`, `fenomenos_especificos`, `diagnostico_sintetico`, `recomendacoes`, `prognostico`, `metas_reavaliacao`

---

### 5.4 relatorio_juliana_mar2026.csv
**Tamanho:** ~9 KB | 72 linhas | Colunas: Domínio, Subdimensão, Instrumento, Escore, Referência, Classificação, Percentil, Observação

Domínios: COGNITIVO, PERSONALIDADE, SENSORIAL, TDAH, FUNÇÕES EXECUTIVAS, FISIOLÓGICO, MEDICAÇÃO, RECOMENDAÇÕES, METAS

---

## 6. O QUE AINDA FALTA IMPLEMENTAR

### Prioridade Alta
- [ ] **Reavaliação Set/2026 completa** — Digit Span em andamento no momento da transferência; completar todos os 7 instrumentos e gerar laudo comparativo
- [ ] **Instrução de Síntese Domingo** — lógica existe mas não testada com semana completa real
- [ ] **Tooltip hover nos gráficos** — crosshair + tooltip especificado no prompt mas não implementado no SVG inline atual
- [ ] **Gráfico acumulado (Parte 2.4)** — barras de score por dia dentro da semana

### Prioridade Média
- [ ] **Filtros de período nos gráficos** — botões 7d/30d/90d/Tudo estão na UI mas a lógica `getDadosFiltrados()` precisa ser conectada ao render de cada gráfico
- [ ] **Filtro por área/instrumento** — botões existem, lógica pendente
- [ ] **Tabela acessível** — "Ver tabela" abre div mas renderTabela() ainda não implementada completamente
- [ ] **Detalhe do dia melhorado** — abrir detalhe de um dia a partir da grade semanal
- [ ] **Exportação de backup do perfil** — `exportarBackupPerfil()` implementada mas não testada

### Prioridade Baixa
- [ ] **Nível 2 login (Supabase)** — só se usuários forem usar aparelhos diferentes; documentado no prompt mas não implementado
- [ ] **Migração automática de dados legados** — função existe, precisa de teste com dados reais
- [ ] **PIN change** — não há como alterar PIN após criação
- [ ] **Import colar código** — mencionado no prompt, não implementado

---

## 7. REQUISITOS TÉCNICOS

### Restrições obrigatórias (do prompt original)
- **Arquivo HTML único** — funciona com duplo clique, sem servidor
- **Sem bibliotecas externas** — sem React, Chart.js, D3, jQuery, Tailwind
- **Gráficos SVG inline** escritos à mão
- **Funciona em `file://`** — sem `import` de módulos locais, sem `fetch` de JSON local
- **Única exceção externa:** Google Fonts (com fallback `system-ui`)
- **CSS e JS** dentro do próprio arquivo

### Critérios de aceite (29 no total — prompt Parte 5)

**Histórico (6):** persistência após reload, dois dias separados, data local às 22h, sobrevive fechar browser, migração de formato antigo, estado vazio sem erro

**Gráficos (6):** eixos Y separados, filtro atualiza tudo junto, cor segue área (não posição), hover/tooltip 375px, sem NaN com 1 dia, tabela acessível

**Tema (6):** alterna tudo incluindo gráficos, persiste, segue sistema na 1ª abertura, sem piscar, gráficos com cores do tema novo, contraste no claro

**Perfis (8):** B não vê A, A intacto após B, PIN funciona, recarregar mantém sessão, excluir apaga só o dono, XSS com nome, tema não muda ao trocar perfil, (Nível 2) RLS funciona

**Geral (3):** duplo clique funciona, zero erros no console, sem scroll horizontal em 375px

---

## 8. FENÔMENOS CLÍNICOS RELEVANTES PARA O SISTEMA

### Que os instrumentos capturam:
1. **Ruminação Mental** — ASRS hiperatividade + item específico de ruminação no contexto rápido
2. **Rigidez Compensatória** — BRIEF-A flexibilidade (88% baseline) é o indicador mais sensível
3. **Gap Funcional** — diferença entre QI em testes (Digit Span, Stroop) vs impacto funcional (BRIEF-A)
4. **Crash de Ritalina às 18h** — capturado pelo horário do check-in + campo "tomou medicação hoje"
5. **Sobrecarga Sensorial** — energia declarada + estresse Garmin correlacionados

### Baseline crítico para comparação:
```
ASRS Hiperatividade: 97% → meta 9 meses: <60%
BRIEF Flexibilidade: 88% → meta 9 meses: <55%
HRV: 32ms → meta 9 meses: >45ms
Sofrimento: 8/10 → meta 9 meses: <4/10
```

---

## 9. ESTRUTURA RECOMENDADA PARA DESENVOLVIMENTO NO CODE

```
sistema-neuro/
├── index.html           ← sistema_neuro.html (principal, já funciona)
├── protocolo.html       ← protocolo_neuro_v4.html (checkpoints formais)
├── data/
│   ├── baseline.json    ← relatorio_juliana_mar2026.json
│   └── baseline.csv     ← relatorio_juliana_mar2026.csv
├── HANDOFF.md           ← este arquivo
└── README.md            ← instruções de uso
```

### Se migrar para build/servidor no Code:
- Os dados já estão isolados em funções `carregarHistorico()` / `salvarResultado()` (preparado para Nível 2 Supabase)
- Design system inteiramente em variáveis CSS — fácil de extrair para um `tokens.css`
- SVG inline pode ser componentizado sem mudar a lógica de negócio
- Fontes já com fallback — sem dependência crítica de internet

---

## 10. LINKS E REFERÊNCIAS

| Recurso | Local/URL |
|---------|-----------|
| Sistema principal (publicado) | https://claude.ai/artifact/SVmxDy3E43v1DhY3arqNJ8 |
| sistema_neuro.html | `/mnt/user-data/outputs/sistema_neuro.html` |
| protocolo_neuro_v4.html | `/mnt/user-data/outputs/protocolo_neuro_v4.html` |
| relatorio baseline JSON | `/mnt/user-data/outputs/relatorio_juliana_mar2026.json` |
| relatorio baseline CSV | `/mnt/user-data/outputs/relatorio_juliana_mar2026.csv` |
| Backup v4 | `/mnt/user-data/outputs/backup_v4_20260915_162541/` |
| Projeto Claude | `Saúde Mental Retorno ✨` |

---

## 11. NOTAS PARA O CLAUDE CODE

1. **O sistema já funciona** — `sistema_neuro.html` é um arquivo standalone completo. No Code, o foco é refinar e completar os itens da seção 6.

2. **Não reescreva o que funciona** — portal de perfis, check-ins, HUD, tema claro/escuro e persistência estão operacionais.

3. **Priorize os gráficos** — tooltip hover e filtros de período são a maior lacuna funcional visível ao usuário.

4. **Os dados clínicos são reais** — os valores de baseline (97% hiperatividade, 32ms HRV, etc.) são da avaliação real de Juliana em Março/2026. Não altere como referência de comparação.

5. **Tome cuidado com datas** — sempre usar data local (`ymd(new Date())`), nunca `toISOString()` que retorna UTC e quebra após 21h no Brasil (UTC-3).

6. **XSS está tratado** — a função `esc()` já está implementada para nomes de perfil. Manter em qualquer novo ponto que receba input do usuário e coloque em `innerHTML`.

7. **Tema dos gráficos** — ao redesenhar SVG após troca de tema, reler cores via `getComputedStyle(document.documentElement).getPropertyValue('--s1')`, nunca hex fixo no JS.

---

*Documento gerado automaticamente a partir de 5 dias de avaliação neuropsicológica + sessão de desenvolvimento em 15/09/2026.*
*Próxima reavaliação formal: Dezembro 2026 (checkpoint 9 meses).*
