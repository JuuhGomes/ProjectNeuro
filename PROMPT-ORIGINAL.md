# PROMPT — Sistema de Testes de QI Diários: histórico por dia + gráficos + layout "Sistema"

> Cole este prompt inteiro na sessão do projeto do teste de QI.

---

## CONTEXTO

Este projeto é um sistema de **testes de QI diários**: a pessoa faz um teste por dia, recebe uma
pontuação e deveria acompanhar a evolução ao longo dos dias.

Tenho outro projeto meu — um game de metas em HTML único ("Sistema de Despertar") — e quero que
este projeto fique **com a mesma cara** dele. O design system completo está na PARTE 3.

Cinco trabalhos, nesta ordem:

1. **Corrigir o bug**: os resultados diários não aparecem separados por dia.
2. **Criar os gráficos**: evolução diária, média e total, com filtros.
3. **Aplicar o layout** da PARTE 3.
4. **Adicionar o botão de modo claro/escuro** (PARTE 4).
5. **Criar o login / perfis**, porque mais pessoas vão usar o sistema (PARTE 5).

## RESTRIÇÃO TÉCNICA — LEIA ANTES DE COMEÇAR

O entregável é **um arquivo `.html` único**, que eu abro com **duplo clique** no Windows e funciona.

- **Nada de Python.** Não quero `python -m http.server`, não quero script `.py`, não quero nada que
  dependa de ter Python instalado. Se a sua solução só roda com um servidor local, ela está errada.
- **Nada de Node, npm, build, bundler, `npm run dev`.** Sem passo de compilação.
- **Nada de biblioteca externa**: sem React, sem Vue, sem Tailwind CDN, sem Chart.js, sem D3,
  sem jQuery. Os gráficos são **SVG inline escrito à mão**.
- Tem que funcionar em **`file://`**. Isso proíbe, na prática:
  - `<script type="module">` com `import` de arquivo local (o navegador bloqueia por CORS);
  - `fetch()` de arquivo `.json` local pelo mesmo motivo.
  Use `<script>` comum e deixe os dados embutidos no próprio HTML / no `localStorage`.
- A **única** coisa externa tolerada é o `<link>` das fontes do Google Fonts — e mesmo assim o
  layout tem que continuar legível se não houver internet (por isso as fontes têm fallback
  `system-ui` na pilha).
- CSS e JS **dentro do próprio arquivo**, em `<style>` e `<script>`. Um arquivo só, como o meu game.

Não faça refatoração geral, não troque de framework. Mexa só no que este prompt pede.

---

# PARTE 1 — CONSERTAR O HISTÓRICO DIÁRIO

## 1.1 O sintoma

Faço o teste hoje, faço amanhã, e a tela **não mostra os dois dias separados**. Ou aparece só o
último resultado, ou tudo vira um número só, ou o resultado de ontem some.

## 1.2 Antes de escrever código: diagnostique

Abra o código atual e verifique, nesta ordem, quais destas causas existem de fato.
**Me diga qual era a causa real** antes de aplicar a correção:

1. **O resultado é guardado numa variável única, não numa lista.**
   Ex.: `localStorage.setItem('resultado', JSON.stringify(r))` — cada teste sobrescreve o anterior.
   É a causa mais comum.

2. **A chave do dia usa `toISOString()`.**
   `new Date().toISOString().slice(0,10)` devolve a data em **UTC**. No Brasil (UTC-3), tudo que é
   feito depois das **21h** é gravado com a data do dia seguinte — testes de dias diferentes se
   misturam na mesma chave e um dia "pula". A chave tem que ser data **local**.

3. **A lista existe, mas a exibição usa índice fixo** (`historico[0]`, `historico.at(-1)`) ou
   `.find()`, então só um resultado aparece.

4. **Grava com data-hora completa** (`Date.now()` ou ISO com hora) e depois compara datas com `===`
   — nunca agrupa nada por dia.

5. **O estado vive só em memória** (variável JS / `useState`) e se perde no reload.

6. **Salva, mas o render do histórico não é chamado** depois de finalizar o teste.

## 1.3 Modelo de dados exigido

Um registro **por dia**, numa lista, com chave de data **local**:

```js
// Data local YYYY-MM-DD. NÃO use toISOString().
const ymd = d => {
  const z = n => String(n).padStart(2, "0");
  return d.getFullYear() + "-" + z(d.getMonth() + 1) + "-" + z(d.getDate());
};
const hoje = () => ymd(new Date());
```

Cada resultado diário (ajuste os nomes aos que o projeto já usa, mas mantenha **todos** os campos
— são o que o histórico e os gráficos precisam):

```js
{
  dia: "2026-09-15",        // chave local YYYY-MM-DD — uma por dia
  em: 1757952000000,        // timestamp do término (para mostrar a hora)
  acertos: 17,
  total: 20,
  percentual: 85,           // acertos/total * 100, arredondado
  qi: 118,                  // QI estimado da rodada
  tempoSeg: 412,            // duração total em segundos
  categorias: {             // desempenho por área — é o que dá valor ao histórico
    logica:   { acertos: 5, total: 5 },
    numerico: { acertos: 4, total: 5 },
    verbal:   { acertos: 4, total: 5 },
    espacial: { acertos: 4, total: 5 }
  },
  respostas: [              // uma entrada por questão, para o detalhe do dia
    { q: 1, categoria: "logica", marcada: "B", correta: "B", ok: true, tempoSeg: 18 }
  ]
}
```

Estado persistido (um único objeto no `localStorage`):

```js
{
  v: 1,
  historico: [ /* registros acima, do mais novo para o mais antigo */ ],
  sequencia: 0,        // dias seguidos com teste feito
  melhorSequencia: 0,
  ultimoDia: null
}
```

`try/catch` em **toda** leitura e escrita (modo anônimo bloqueia e derruba a página):

```js
function ls(k, def){ try{ const v = localStorage.getItem(k); return v ? JSON.parse(v) : def; }catch(e){ return def; } }
function lsSet(k, v){ try{ localStorage.setItem(k, JSON.stringify(v)); return true; }catch(e){ return false; } }
```

Se `lsSet` devolver `false`, avise na tela com um toast (3.6) — nunca falhe em silêncio.

## 1.4 Regras de gravação

- Ao **finalizar** o teste, monte o registro completo e **empurre no início**:
  `estado.historico.unshift(registro)`.
- **Um registro por dia.** Se já existe registro com o mesmo `dia`:
  - substitua, marque `refeito: true` e guarde `tentativas: n` (para exibir "2ª tentativa do dia"), **ou**
  - bloqueie o segundo teste do dia com "Você já fez o teste de hoje. Volte amanhã."
  **Escolha uma das duas e me diga qual escolheu.**
- Nunca apague registro antigo. Limite a 365 dias (`if (h.length > 365) h.length = 365`).
- **Salve imediatamente** ao finalizar e **só então** chame o render do histórico.
- Atualize `sequencia`: registro anterior é de ontem → `sequencia++`; há buraco → volta a 1.
  Atualize `melhorSequencia` junto.
- **Migração**: se existir dado no formato antigo (um resultado solto), converta para um registro
  de histórico ao carregar, em vez de descartar. Não quero perder o que já foi feito.

## 1.5 Aba "Histórico" — o que tem que aparecer

- **Lista de cards, um por dia**, do mais recente para o mais antigo. Cada card: data por extenso
  em pt-BR ("segunda, 15 de setembro"), hora, QI da rodada, acertos/total, percentual, tempo, e uma
  barra fina do percentual (`.qbar` da PARTE 3).
- **Variação** em relação ao dia anterior com teste: `▲ +4` em `--ok`, `▼ -3` em `--danger`,
  `—` em `--muted` no empate.
- **Agrupamento por mês**, com cabeçalho de seção (`.slotlabel`) trazendo a média daquele mês.
- **Clicar no card abre o detalhe do dia**: as questões da rodada (lista `respostas`), acerto/erro
  por questão, e o desempenho por área em barras.
- **Estado vazio** claro (`.empty`): "Nenhum teste registrado ainda. O primeiro resultado aparece
  aqui assim que você terminar o teste de hoje."
- **Exportar**: um botão baixa o histórico em JSON, outro em CSV (uma linha por dia).

---

# PARTE 2 — GRÁFICOS

Tudo em **SVG inline**, sem biblioteca. Três gráficos + uma faixa de números no topo.

## 2.1 Regra que não pode ser quebrada: nunca dois eixos Y

QI diário (escala ~70–140) e **total acumulado** (escala que só cresce) são medidas de grandezas
diferentes. Colocar as duas no mesmo gráfico com dois eixos Y é o erro clássico e deforma a leitura.
São **dois gráficos separados**, empilhados, compartilhando o mesmo eixo X e o mesmo filtro de período.

## 2.2 Faixa de números (antes dos gráficos)

Cinco números não pedem gráfico — pedem destaque. Use os `.chip` ou blocos de número grande em
`--f-d`: **melhor QI**, **média dos últimos 7 dias**, **média geral**, **dias testados**,
**sequência atual**. Cada um com o rótulo miúdo em caixa alta por baixo.

## 2.3 Gráfico 1 — "Evolução diária" (o principal)

- **Linha** do QI por dia, no período filtrado. Série **única** → **sem caixa de legenda**;
  o título do painel já diz o que é.
- Cor da linha: `#2f95db`. Espessura **2px**. Pontos de dado com marcador de **8px** de diâmetro.
- **Linha de média sobreposta**: tracejada 1px, cor `--muted` (`#62809f`), rotulada **direto na ponta
  direita** ("média 112"). É uma linha de referência, não uma segunda série — por isso fica em tom
  neutro, sem entrar em legenda nenhuma.
- **Grade discreta**: só linhas horizontais, `rgba(94,199,255,.08)`, 4 ou 5 no máximo.
  **Sem grade vertical.** Textos dos eixos em `--muted`, `font-variant-numeric:tabular-nums`.
- **Rótulos seletivos**: número visível só no primeiro ponto, no último e no maior.
  **Nunca um número em cima de cada ponto.**
- **Hover obrigatório**: linha vertical de mira (crosshair) acompanhando o cursor + tooltip com
  data, QI, acertos/total e a variação. A área de captura do mouse é uma faixa vertical inteira por
  dia — bem maior que o marcador — para funcionar com o dedo no celular.
- **Poucos dados**: com 1 dia, não desenhe linha — mostre o número grande e "faça mais um dia para
  ver a evolução". Com 2 ou 3, desenhe normalmente.

## 2.4 Gráfico 2 — "Total acumulado"

Gráfico próprio, logo abaixo do primeiro, mesmo eixo X e mesmo filtro.

- **Área/linha** do acumulado de acertos (ou de testes concluídos) ao longo do período.
- Cor `#2f95db` com preenchimento abaixo em `rgba(47,149,219,.18)`.
- Mesmo hover, mesma grade discreta. Rótulo direto só no valor final.
- Alternativa, se para o projeto "total" significar **total de acertos do dia**: aí é **barra** por
  dia, com o topo da barra arredondado em 4px e **2px de respiro** entre barras vizinhas.
  Escolha a leitura que fizer mais sentido e me diga qual usou.

## 2.5 Gráfico 3 — "Por área"

Quatro séries (lógica, numérico, verbal, espacial). Duas opções, **prefira a primeira**:

- **Pequenos múltiplos** (recomendado): quatro mini-gráficos de linha lado a lado, um por área,
  todos na mesma escala Y. Evita o emaranhado de quatro linhas cruzadas e cada área fica legível.
- **Quatro linhas num gráfico só**: aí é **obrigatório** legenda **e** rótulo direto na ponta de
  cada linha — identidade nunca pode depender só da cor.

**Paleta das áreas — use exatamente nesta ordem, sempre a mesma cor para a mesma área.**
São **dois conjuntos**, um por tema: cada um foi validado (banda de luminosidade, chroma,
separação para daltonismo protan/deutan e contraste) contra o fundo do seu próprio tema. Marca de
gráfico não é a mesma coisa que cor de interface, e a versão escura não serve no tema claro.

| Área | Tema escuro (fundo `#0b1526`) | Tema claro (fundo `#ffffff`/`#eef3f9`) |
|---|---|---|
| Lógica | `#2f95db` | `#1f6fa8` |
| Numérico | `#b8851a` | `#8f6512` |
| Verbal | `#8f5cf0` | `#6d3bc4` |
| Espacial | `#1fae79` | `#157f58` |

Declare esses valores como variáveis CSS (`--s1`…`--s4`) dentro de cada tema e leia-os no JS com
`getComputedStyle(document.documentElement).getPropertyValue('--s1')`. Assim **trocar de tema
repinta os gráficos sozinho**, sem duplicar a lógica de desenho.

A cor segue a **área**, nunca a posição no ranking: se um filtro esconder uma área, as outras
**mantêm** suas cores. Não gere cor nova, não cicle a lista.

## 2.6 Filtros

Uma **única linha de controles acima dos gráficos** (não espalhe filtro por gráfico):

- **Período**: `7 dias` · `30 dias` · `90 dias` · `Tudo` — botões `.btn` com o ativo destacado
  (borda `--cyan`), mesmo padrão visual das abas.
- **Área**: `Todas` · Lógica · Numérico · Verbal · Espacial — filtra os três gráficos ao mesmo tempo.
- **Métrica**: `QI` · `% de acerto` · `Tempo` — troca o que o gráfico 1 mostra.
- Os filtros valem para os três gráficos e para a faixa de números do 2.2, tudo junto.
- Estado do filtro guardado, para não resetar a cada visita.

## 2.7 Acessibilidade dos gráficos

- Botão **"Ver tabela"** abaixo do bloco de gráficos, que mostra os mesmos dados em `<table>`
  (dia, QI, acertos, %, tempo). Quem não enxerga o gráfico tem o dado.
- Texto sempre em cor de texto (`--text`, `--text-2`, `--muted`) — **nunca** pinte o rótulo com a
  cor da série; a cor fica no marquinho ao lado.
- Respeite `prefers-reduced-motion` nas animações de entrada das linhas.

## 2.8 Depois de desenhar, olhe

Abra a tela e confira de olho: rótulo colidindo, linha saindo da área, eixo cortado, gráfico
estourando a largura no celular. O código pode estar certo e o desenho errado.

---

# PARTE 3 — LAYOUT (design system do "Sistema de Despertar")

Visual: **HUD de jogo, escuro, azul-ciano, cantos chanfrados**. HTML/CSS puro.
Os blocos abaixo são o CSS real do meu game — copie.

## 3.1 Tokens e base

```css
:root{
  --void:#04070f; --void2:#080e1c; --ink:#0b1526;
  --panel-a:rgba(14,29,54,.94); --panel-b:rgba(7,14,28,.96);
  --line:rgba(94,199,255,.30);
  --cyan:#5ec7ff; --cyan-hi:#a8e6ff; --cyan-dim:#2b6c9e;
  --violet:#a06bff; --gold:#f2c14e; --danger:#ff4d6a; --ok:#46e0a0;
  --text:#d7e7fb; --text-2:#93aed0; --muted:#62809f;
  --cut:14px;
  --f-d:"Chakra Petch",ui-sans-serif,system-ui,sans-serif;  /* títulos e números */
  --f-b:"Rajdhani",ui-sans-serif,system-ui,sans-serif;      /* corpo */
}
*{box-sizing:border-box}
html{background:var(--void)}
body{
  margin:0; background:var(--void); color:var(--text);
  font-family:var(--f-b); font-size:16px; line-height:1.45;
  -webkit-text-size-adjust:100%; min-height:100vh;
}
.app{position:relative; z-index:1; max-width:1180px; margin:0 auto; padding:14px 12px 108px}
[hidden]{display:none !important}
```

> Atenção: os tokens `--cyan/--violet/--gold/--ok` são as cores de **interface** (chips, bordas,
> brilhos). As cores de **série de gráfico** são as do 2.5 — mais escuras de propósito, porque
> marca de gráfico sobre fundo escuro não pode estourar. Não troque uma pela outra.

Fontes, no `<head>`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Chakra+Petch:wght@500;600;700&family=Rajdhani:wght@400;500;600;700&display=swap">
```

Fundo em duas camadas — brilhos radiais + grade técnica esmaecida nas bordas:

```css
body::before{
  content:""; position:fixed; inset:0; pointer-events:none; z-index:0;
  background:
    radial-gradient(1100px 620px at 12% -8%, rgba(38,120,200,.20), transparent 62%),
    radial-gradient(900px 560px at 92% 4%, rgba(140,80,255,.14), transparent 60%),
    radial-gradient(800px 800px at 50% 115%, rgba(20,80,150,.16), transparent 62%),
    linear-gradient(180deg,#04070f,#04070f);
}
body::after{
  content:""; position:fixed; inset:0; pointer-events:none; z-index:0; opacity:.35;
  background-image:linear-gradient(rgba(94,199,255,.055) 1px, transparent 1px),
    linear-gradient(90deg, rgba(94,199,255,.055) 1px, transparent 1px);
  background-size:46px 46px;
  -webkit-mask-image:radial-gradient(closest-side at 50% 30%, #000 40%, transparent 100%);
  mask-image:radial-gradient(closest-side at 50% 30%, #000 40%, transparent 100%);
}
```

## 3.2 Painel — use para TODO bloco de conteúdo

Estrutura sempre:
`<section class="win"><div class="wb"><h2 class="wt">TÍTULO</h2> …conteúdo… </div></section>`

```css
.win{
  background:linear-gradient(150deg, rgba(120,212,255,.78), rgba(44,96,160,.30) 46%, rgba(150,110,255,.44));
  padding:1px;
  clip-path:polygon(var(--cut) 0, 100% 0, 100% calc(100% - var(--cut)), calc(100% - var(--cut)) 100%, 0 100%, 0 var(--cut));
  box-shadow:0 10px 34px rgba(0,0,0,.55), 0 0 30px rgba(50,140,230,.10);
}
.win>.wb{
  background:linear-gradient(168deg, var(--panel-a), var(--panel-b));
  clip-path:polygon(var(--cut) 0, 100% 0, 100% calc(100% - var(--cut)), calc(100% - var(--cut)) 100%, 0 100%, 0 var(--cut));
  padding:14px 15px;
}
.wt{
  display:flex; align-items:center; gap:10px; margin:0 0 12px;
  font-family:var(--f-d); font-weight:700; font-size:.78rem; letter-spacing:.20em;
  text-transform:uppercase; color:var(--cyan-hi);
}
.wt::before{content:""; width:8px; height:8px; background:var(--cyan); transform:rotate(45deg); box-shadow:0 0 10px var(--cyan); flex:none}
.wt::after{content:""; flex:1; height:1px; background:linear-gradient(90deg, var(--line), transparent)}
.wt .cnt{font-size:.72rem; letter-spacing:.1em; color:var(--muted)}
.slotlabel{font-family:var(--f-d); font-size:.68rem; letter-spacing:.2em; color:var(--muted); text-transform:uppercase; margin:14px 0 7px; display:flex; align-items:center; gap:9px}
.slotlabel::after{content:""; flex:1; height:1px; background:rgba(94,199,255,.14)}
.empty{padding:18px 10px; text-align:center; color:var(--muted); font-size:.88rem; letter-spacing:.04em}
```

O **fundo do gráfico** é o `.wb` — foi contra ele (`#0b1526`) que as cores de série do 2.5 foram
validadas. Não coloque gráfico sobre fundo mais claro sem revalidar.

## 3.3 HUD do topo

No game é nível/XP/ouro. Aqui: **QI atual** no lugar do nível, **barra até o próximo marco** no
lugar do XP, chips com média 7 dias / sequência / dias testados. O losango da esquerda
(`.rankmark`) mostra a faixa ("A", "B", "MÉDIO", "ALTO" — escolha e mantenha).

```css
.hud{margin-bottom:14px}
.hud .wb{padding:13px 15px}
.hud-top{display:flex; align-items:center; gap:13px}
.rankmark{
  flex:none; width:54px; height:54px; display:grid; place-items:center;
  font-family:var(--f-d); font-weight:700; font-size:1.45rem; line-height:1;
  border:1px solid currentColor; color:var(--cyan);
  clip-path:polygon(50% 0,100% 26%,100% 74%,50% 100%,0 74%,0 26%);
  background:rgba(94,199,255,.08); text-shadow:0 0 14px currentColor;
}
.rankmark small{display:block; font-size:.42rem; letter-spacing:.18em; opacity:.75; margin-top:2px}
.who{min-width:0; flex:1}
.who h1{margin:0; font-family:var(--f-d); font-size:1.22rem; font-weight:700; letter-spacing:.04em; white-space:nowrap; overflow:hidden; text-overflow:ellipsis}
.who .sub{font-size:.82rem; color:var(--text-2); letter-spacing:.06em; text-transform:uppercase}
.lvbox{text-align:right; flex:none}
.lvbox .n{font-family:var(--f-d); font-size:2rem; font-weight:700; line-height:.95; color:#fff; text-shadow:0 0 18px rgba(94,199,255,.75)}
.lvbox .l{font-size:.6rem; letter-spacing:.26em; color:var(--muted); text-transform:uppercase}
.xpline{display:flex; justify-content:space-between; font-size:.72rem; letter-spacing:.12em; color:var(--text-2); margin:11px 0 5px; text-transform:uppercase}
.bar{height:7px; background:rgba(94,199,255,.11); border:1px solid rgba(94,199,255,.18); overflow:hidden}
.bar i{display:block; height:100%; background:linear-gradient(90deg,#2e8fd6,var(--cyan),var(--cyan-hi)); box-shadow:0 0 12px rgba(94,199,255,.85); transition:width .5s cubic-bezier(.2,.8,.3,1)}
.chips{display:flex; flex-wrap:wrap; gap:7px; margin-top:11px}
.chip{
  display:inline-flex; align-items:center; gap:6px; padding:3px 10px;
  border:1px solid rgba(94,199,255,.22); background:rgba(94,199,255,.06);
  font-size:.76rem; letter-spacing:.07em; color:var(--text-2);
  clip-path:polygon(6px 0,100% 0,100% calc(100% - 6px),calc(100% - 6px) 100%,0 100%,0 6px);
}
.chip b{color:var(--text); font-weight:600; font-variant-numeric:tabular-nums}
.chip.gold{border-color:rgba(242,193,78,.35); color:var(--gold); background:rgba(242,193,78,.07)}
.chip.fire{border-color:rgba(255,120,60,.35); color:#ff9a5c; background:rgba(255,120,60,.07)}
.chip.vi{border-color:rgba(160,107,255,.4); color:#c7a5ff; background:rgba(160,107,255,.09)}
```

## 3.4 Abas — barra inferior no celular, linha de botões no desktop

Abas: **Hoje · Histórico · Evolução · Mais**.

```css
.tabs{
  position:fixed; left:0; right:0; bottom:0; z-index:40;
  display:grid; grid-auto-flow:column; grid-auto-columns:1fr; gap:1px;
  background:rgba(94,199,255,.16); border-top:1px solid var(--line); backdrop-filter:blur(10px);
}
.tabs button{
  appearance:none; border:0; cursor:pointer; padding:9px 2px calc(9px + env(safe-area-inset-bottom));
  background:rgba(6,12,24,.94); color:var(--muted);
  font-family:var(--f-d); font-size:.62rem; font-weight:600; letter-spacing:.13em; text-transform:uppercase;
  display:flex; flex-direction:column; align-items:center; gap:5px;
}
.tabs button i{width:14px; height:14px; border:1.5px solid currentColor; display:block; transform:rotate(45deg); transition:all .2s}
.tabs button[aria-selected="true"]{color:var(--cyan-hi); background:rgba(16,40,72,.96)}
.tabs button[aria-selected="true"] i{background:var(--cyan); border-color:var(--cyan); box-shadow:0 0 12px var(--cyan)}
@media(min-width:960px){
  .tabs{position:static; grid-auto-columns:auto; justify-content:start; gap:6px; background:none; border:0; backdrop-filter:none; margin-bottom:12px}
  .tabs button{flex-direction:row; padding:8px 16px; font-size:.7rem; background:rgba(10,22,42,.7); border:1px solid rgba(94,199,255,.16);
    clip-path:polygon(8px 0,100% 0,100% calc(100% - 8px),calc(100% - 8px) 100%,0 100%,0 8px)}
  .tabs button[aria-selected="true"]{border-color:var(--cyan)}
}
.pane{display:none}
.pane.on{display:block}
.pane>*+*{margin-top:14px}
.grid{display:grid; grid-template-columns:1fr; gap:14px; align-items:start}
@media(min-width:960px){
  .grid{grid-template-columns:352px minmax(0,1fr)}
  .app{padding:20px 18px 40px}
}
```

Use `role="tablist"` / `role="tab"` / `aria-selected` de verdade.

## 3.5 Card do dia e botões

No game é a missão (`.q`). Aqui é **o resultado de um dia**. `.done` (verde) para dia acima da
média, `.late` (vermelho) para dia abaixo.

```css
.q{padding:11px 12px; border:1px solid rgba(94,199,255,.15); background:linear-gradient(160deg,rgba(20,40,72,.5),rgba(9,18,34,.5));
  clip-path:polygon(10px 0,100% 0,100% calc(100% - 10px),calc(100% - 10px) 100%,0 100%,0 10px)}
.q+.q{margin-top:9px}
.q.done{border-color:rgba(70,224,160,.35); background:linear-gradient(160deg,rgba(12,48,40,.45),rgba(7,20,20,.5))}
.q.late{border-color:rgba(255,77,106,.4)}
.qhead{display:flex; align-items:flex-start; gap:11px}
.qmain{flex:1; min-width:0}
.qname{font-family:var(--f-d); font-size:1rem; font-weight:600; letter-spacing:.02em; word-break:break-word}
.qmeta{display:flex; flex-wrap:wrap; gap:9px; font-size:.76rem; color:var(--muted); letter-spacing:.04em; margin-top:1px; font-variant-numeric:tabular-nums}
.qmeta .s{color:var(--cyan)} .qmeta .x{color:var(--cyan-hi)} .qmeta .g{color:var(--gold)} .qmeta .d{color:var(--danger)}
.qbar{height:6px; background:rgba(94,199,255,.10); margin-top:9px; overflow:hidden}
.qbar i{display:block; height:100%; background:linear-gradient(90deg,var(--cyan-dim),var(--cyan)); transition:width .35s}
.q.done .qbar i{background:linear-gradient(90deg,#1f8a63,var(--ok))}
.qact{display:flex; flex-wrap:wrap; gap:6px; margin-top:9px}

.btn{appearance:none; cursor:pointer; font-family:var(--f-d); font-size:.72rem; font-weight:600;
  letter-spacing:.1em; text-transform:uppercase; padding:6px 12px;
  background:rgba(94,199,255,.08); border:1px solid rgba(94,199,255,.3); color:var(--cyan-hi);
  clip-path:polygon(6px 0,100% 0,100% calc(100% - 6px),calc(100% - 6px) 100%,0 100%,0 6px); transition:background .15s}
.btn:hover{background:rgba(94,199,255,.22)}
.btn.ghost{background:transparent; border-color:rgba(94,199,255,.16); color:var(--muted)}
.btn.vi{border-color:rgba(160,107,255,.45); color:#c9abff; background:rgba(160,107,255,.1)}
.btn.gold{border-color:rgba(242,193,78,.45); color:var(--gold); background:rgba(242,193,78,.09)}
.btn.warn{border-color:rgba(255,77,106,.4); color:#ff8ea0; background:rgba(255,77,106,.08)}
.btn:disabled{opacity:.35; cursor:not-allowed}
.btn.wide{width:100%; justify-content:center; text-align:center; padding:10px}
```

Formulários e diálogos (o teste em si e o detalhe do dia):

```css
label.f{display:block; margin-bottom:10px}
label.f>span{display:block; font-size:.66rem; letter-spacing:.16em; text-transform:uppercase; color:var(--muted); margin-bottom:4px}
input,select,textarea{width:100%; font-family:var(--f-b); font-size:1rem; color:var(--text);
  background:rgba(4,10,22,.85); border:1px solid rgba(94,199,255,.25); padding:8px 10px; border-radius:0}
input:focus,select:focus,textarea:focus,button:focus-visible{outline:2px solid var(--cyan); outline-offset:1px}
.row2{display:grid; grid-template-columns:1fr 1fr; gap:10px}
.row3{display:grid; grid-template-columns:1fr 1fr 1fr; gap:10px}
dialog{border:0; padding:0; background:transparent; color:var(--text); width:min(480px,94vw); max-height:90vh}
dialog::backdrop{background:rgba(2,5,12,.82)}
dialog .wb{max-height:86vh; overflow:auto}
.dlg-act{display:flex; gap:8px; margin-top:14px}
.dlg-act .btn{flex:1; text-align:center}
```

## 3.6 Avisos e recorde

```css
#toasts{position:fixed; top:10px; right:10px; left:10px; z-index:60; display:flex; flex-direction:column; gap:8px; pointer-events:none; align-items:flex-end}
.toast{pointer-events:none; max-width:330px; width:100%;
  background:linear-gradient(150deg,rgba(120,212,255,.7),rgba(60,120,200,.3)); padding:1px;
  clip-path:polygon(10px 0,100% 0,100% calc(100% - 10px),calc(100% - 10px) 100%,0 100%,0 10px); animation:slide .3s ease-out}
.toast>div{background:linear-gradient(168deg,rgba(12,28,52,.97),rgba(6,12,26,.98)); padding:9px 12px;
  clip-path:polygon(10px 0,100% 0,100% calc(100% - 10px),calc(100% - 10px) 100%,0 100%,0 10px)}
.toast b{display:block; font-family:var(--f-d); font-size:.62rem; letter-spacing:.2em; color:var(--cyan-hi); text-transform:uppercase; margin-bottom:2px}
.toast p{margin:0; font-size:.88rem; color:var(--text)}
.toast.bad{background:linear-gradient(150deg,rgba(255,77,106,.75),rgba(120,20,40,.4))}
.toast.bad b{color:#ff8ea0}
.toast.gold{background:linear-gradient(150deg,rgba(242,193,78,.75),rgba(120,80,20,.4))}
.toast.gold b{color:var(--gold)}
@keyframes slide{from{opacity:0; transform:translateX(26px)}to{opacity:1; transform:none}}
@media (prefers-reduced-motion: reduce){*{animation-duration:.01ms !important; transition-duration:.01ms !important}}
```

**Recorde pessoal**: quando o QI do dia supera o melhor já registrado, mostre uma tela cheia rápida
no estilo "LEVEL UP" do game — texto enorme em `--f-d`, brilho ciano, fecha ao tocar — e dispare um
toast `gold`.

## 3.7 Regras de estilo inegociáveis

- **Números sempre** em `--f-d` com `font-variant-numeric:tabular-nums`. Coluna de números tem que
  ficar alinhada entre os cards do histórico.
- **Zero `border-radius`** na interface. O corte é feito com `clip-path` — é a assinatura do visual.
  (A única exceção é a ponta de barra de gráfico, 4px, conforme 2.4.)
- **Verde (`--ok`)** = acerto/melhora. **Vermelho (`--danger`)** = erro/piora. **Dourado (`--gold`)**
  = recorde. **Roxo (`--violet`)** = ação secundária. Não invente cor fora disso.
- **Título de painel** sempre em caixa alta com `letter-spacing:.20em` (`.wt`).
- **Responsivo de verdade**: tem que funcionar em **375px**. Uma coluna no celular, duas (`.grid`)
  a partir de 960px. **Nada pode causar rolagem horizontal** — gráfico incluído.
- **Textos em pt-BR**, tom direto e seco, como no game ("O Sistema observa. Só o que é feito conta.").
  Datas com `toLocaleDateString('pt-BR', …)`.
- **Acessibilidade**: `aria-selected` nas abas, foco visível (já no CSS), e a animação de recorde
  respeitando `prefers-reduced-motion`.

---

# PARTE 4 — BOTÃO DE MODO CLARO / ESCURO

Quero o botãozinho de alternância no estilo pílula: **lua** no modo escuro, **sol** no modo claro,
com a bolinha deslizando de um lado para o outro.

## 4.1 Onde fica

No canto superior direito do HUD (3.3), alinhado com o número grande do QI. No celular, ele fica
na mesma linha do título — nunca dentro do menu, tem que estar sempre a um toque.

## 4.2 Como o tema é decidido (nesta ordem)

1. Se a pessoa **já escolheu** antes, vale a escolha dela — salva no `localStorage` (`qi.tema`).
2. Se nunca escolheu, segue a **preferência do sistema**: `matchMedia("(prefers-color-scheme: light)")`.
3. Se nada disso responder, o padrão é **escuro** (é a identidade do projeto).

Enquanto a pessoa não tiver escolhido manualmente, mudar o tema do Windows/celular deve mudar o app
junto (escute o evento `change` do `matchMedia`). Depois que ela escolher, a escolha dela manda e o
sistema para de interferir.

**Sem piscada de tema errado**: aplique o atributo do tema num `<script>` curto **dentro do
`<head>`**, antes do `<body>` renderizar. Se o app abrir escuro e "pular" para claro meio segundo
depois, está errado.

```html
<script>
  try{
    var t = JSON.parse(localStorage.getItem("qi.tema") || "null");
    if(!t) t = matchMedia("(prefers-color-scheme: light)").matches ? "claro" : "escuro";
    document.documentElement.setAttribute("data-theme", t);
  }catch(e){ document.documentElement.setAttribute("data-theme","escuro"); }
</script>
```

## 4.3 Como os temas são escritos

O tema escuro é o que já está no `:root` da 3.1 — **não mexa nele**. O tema claro é um bloco que
**só redefine os tokens**, em `:root[data-theme="claro"]`. Nenhum componente (`.win`, `.chip`,
`.btn`, `.q`…) pode ganhar regra própria por tema: se um componente precisar de `[data-theme]` no
seletor, é sinal de que faltou um token.

```css
:root[data-theme="claro"]{
  --void:#e7eef6; --void2:#eef3f9; --ink:#ffffff;
  --panel-a:rgba(255,255,255,.97); --panel-b:rgba(240,245,251,.98);
  --line:rgba(31,111,168,.30);
  --cyan:#1f6fa8; --cyan-hi:#13486f; --cyan-dim:#8fc3e6;
  --violet:#6d3bc4; --gold:#8f6512; --danger:#c22b46; --ok:#157f58;
  --text:#14212f; --text-2:#3d5570; --muted:#61758c;
  /* cores de série dos gráficos (2.5) */
  --s1:#1f6fa8; --s2:#8f6512; --s3:#6d3bc4; --s4:#157f58;
}
:root{ /* no tema escuro, as séries são as outras */
  --s1:#2f95db; --s2:#b8851a; --s3:#8f5cf0; --s4:#1fae79;
}
```

Três ajustes de fundo que o tema claro precisa, porque brilho de neon sobre papel fica sujo:

```css
:root[data-theme="claro"] body::before{
  background:
    radial-gradient(1100px 620px at 12% -8%, rgba(31,111,168,.10), transparent 62%),
    radial-gradient(900px 560px at 92% 4%, rgba(109,59,196,.07), transparent 60%),
    linear-gradient(180deg,#eef3f9,#e7eef6);
}
:root[data-theme="claro"] body::after{ opacity:.5;
  background-image:linear-gradient(rgba(31,111,168,.06) 1px, transparent 1px),
    linear-gradient(90deg, rgba(31,111,168,.06) 1px, transparent 1px);
}
:root[data-theme="claro"] .win{
  background:linear-gradient(150deg, rgba(31,111,168,.45), rgba(31,111,168,.14) 46%, rgba(109,59,196,.28));
  box-shadow:0 8px 24px rgba(20,40,70,.12);
}
```

No tema claro, **tire os `text-shadow` de brilho** (`.lvbox .n`, `.crest h1`, `.rankmark`) — glow
sobre fundo claro vira borrão. Use `:root[data-theme="claro"] .lvbox .n{text-shadow:none}`.

## 4.4 O botão

Este é o **único** lugar do projeto onde `border-radius` é permitido — a pílula é a forma do
controle. Todo o resto continua chanfrado (3.7).

```css
.tema{
  appearance:none; cursor:pointer; flex:none;
  width:56px; height:30px; padding:3px; border-radius:999px;
  border:1px solid var(--line); background:#0b1526;
  display:flex; align-items:center; position:relative;
  transition:background .25s, border-color .25s;
}
.tema .knob{
  width:22px; height:22px; border-radius:50%; background:#fff;
  box-shadow:0 2px 6px rgba(0,0,0,.45);
  transform:translateX(26px); transition:transform .25s cubic-bezier(.2,.8,.3,1);
}
.tema .ico{
  position:absolute; top:50%; transform:translateY(-50%);
  width:16px; height:16px; display:grid; place-items:center;
  transition:opacity .2s;
}
.tema .lua{left:7px;  color:#cfe6ff; opacity:1}
.tema .sol{right:7px; color:#fff;    opacity:0}
.tema:focus-visible{outline:2px solid var(--cyan); outline-offset:2px}

/* estado claro */
:root[data-theme="claro"] .tema{background:#f28c28; border-color:rgba(0,0,0,.18)}
:root[data-theme="claro"] .tema .knob{transform:translateX(0)}
:root[data-theme="claro"] .tema .lua{opacity:0}
:root[data-theme="claro"] .tema .sol{opacity:1}
```

Marcação — `role="switch"` de verdade, não uma `<div>` clicável:

```html
<button class="tema" id="btnTema" role="switch" aria-checked="true" aria-label="Alternar modo claro e escuro" title="Alternar tema">
  <span class="ico lua" aria-hidden="true"><!-- svg da lua --></span>
  <span class="ico sol" aria-hidden="true"><!-- svg do sol --></span>
  <span class="knob"></span>
</button>
```

Os dois ícones são **SVG inline** desenhados à mão (lua crescente e sol com raios), `currentColor`,
`stroke-width:2`. Nada de emoji, nada de fonte de ícone, nada de imagem externa.

## 4.5 A troca

```js
const TEMA_KEY = "qi.tema";
function aplicarTema(t, salvar){
  document.documentElement.setAttribute("data-theme", t);
  const b = document.querySelector("#btnTema");
  if (b) b.setAttribute("aria-checked", String(t === "escuro"));
  const meta = document.querySelector('meta[name="theme-color"]');
  if (meta) meta.setAttribute("content", t === "escuro" ? "#04070f" : "#eef3f9");
  if (salvar) lsSet(TEMA_KEY, t);
  redesenharGraficos();   // relê --s1..--s4 e as cores de eixo/grade
}
```

O ponto crítico: **`redesenharGraficos()` tem que reler as variáveis CSS**, não usar hex fixo no JS.

```js
const cor = n => getComputedStyle(document.documentElement).getPropertyValue(n).trim();
// uso: cor("--s1"), cor("--muted"), cor("--text-2")
```

Se as cores estiverem escritas direto no código do SVG, trocar de tema deixa o gráfico com a cor do
tema anterior — é o erro mais comum aqui. A cor da grade, do texto dos eixos, do tooltip e da linha
de média vêm todas de token.

## 4.6 Regras do tema claro

- **O contraste é responsabilidade sua nos dois temas.** Texto secundário em `--muted` sobre
  `--panel-a` claro tem que continuar legível — se ficar lavado, escureça o token, não o componente.
- **Verde/vermelho/dourado mudam de tom entre os temas**, mas nunca de significado.
- A tela de recorde ("LEVEL UP") no tema claro perde o fundo preto translúcido: use um véu claro
  (`rgba(240,245,251,.92)`) com o texto em `--cyan-hi`.
- Os dois temas passam pelos mesmos 14 critérios de aceite. Teste tudo **nos dois**.

---

# PARTE 5 — LOGIN / MÚLTIPLAS PESSOAS

Mais gente além de mim vai usar o sistema. Cada pessoa precisa do **seu próprio histórico**,
sem misturar com o das outras.

## 5.1 A verdade sobre "login" num HTML solto — leia antes de codar

Um arquivo `.html` que roda no navegador **não tem como guardar segredo**. Se os dados ficam no
`localStorage`, qualquer pessoa com acesso ao aparelho abre o DevTools (F12) e lê tudo — inclusive
o PIN, inclusive o histórico dos outros perfis. Isso **não é** contornável com criptografia no
front, com PIN "hasheado", nem escondendo o código: a chave estaria no mesmo arquivo.

Então:

- **Não escreva um sistema de senha caseiro** e **não me diga que está seguro.** Nada de inventar
  hash próprio, nada de `btoa()` fingindo criptografia.
- Nos textos da tela, seja honesto, no mesmo tom do meu game: *"o PIN só serve para ninguém abrir o
  seu por engano — não é senha de banco."*
- Se em algum momento entrar dado sensível de verdade, o Nível 2 deixa de ser opcional.

São dois níveis possíveis. **Implemente o Nível 1 sempre.** O Nível 2 só se eu confirmar.

## 5.2 NÍVEL 1 — Perfis locais com PIN (faça este)

É o modelo do meu game "Sistema de Despertar": um portal de entrada antes do app.

**Serve para**: várias pessoas usando **o mesmo aparelho** (mesmo computador, mesmo celular).
**Não serve para**: cada um no seu aparelho — o histórico não viaja junto, fica preso ao navegador
daquela máquina. Se for esse o caso, é Nível 2.

### Portal de entrada

Tela inicial, antes do app, no estilo do meu portal:

- **Lista de perfis** já criados no aparelho, cada um num card clicável com a inicial num losango,
  o nome, e o resumo ("QI médio 112 · 23 dias · última vez ontem").
- **"+ Nova pessoa"** para criar perfil: nome (até 24 caracteres) e **PIN de 4 dígitos opcional**
  (vazio = entra direto).
- Se o perfil tem PIN, pedir o PIN. Erro mostra mensagem clara, não trava, não conta tentativas.
- **"Trocar de pessoa"** dentro do app, na aba Mais, volta ao portal.
- Botões de **exportar** e **importar** backup em `.json`, e **"colar código"** para levar o
  progresso para outro aparelho manualmente.

### Dados

```js
const K_USERS = "qi.users";              // [{id, nome, pin, qiMedio, dias, visto}]
const K_SESS  = "qi.sessao";             // id de quem está logado agora
const K_SAVE  = id => "qi.save." + id;   // o estado da PARTE 1, um por pessoa
```

- **Cada pessoa tem o seu `historico` completo**, salvo na chave dela. Nada é compartilhado.
- Ao entrar, carregue `K_SAVE(id)`; ao salvar, grave só nessa chave. Toda função que hoje mexe no
  estado tem que passar a mexer no estado **da pessoa logada**.
- `visto: Date.now()` a cada login, para ordenar a lista do portal pelo mais recente.
- **Excluir perfil** pede confirmação digitando o nome, e apaga só a chave daquela pessoa.
- O **tema** (PARTE 4) é preferência **do aparelho**, não da pessoa — fica numa chave global e não
  muda quando troca de perfil.

### Cuidado que sempre escapa

Nome de perfil vai para dentro de `innerHTML` em vários lugares. **Escape sempre**, como no meu game:

```js
const esc = s => String(s).replace(/[&<>"]/g, c => ({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;"}[c]));
```

Sem isso, alguém cadastra um nome com `<img onerror=...>` e executa script no seu app.

## 5.3 NÍVEL 2 — Login de verdade (só se eu confirmar)

Necessário **se as pessoas forem usar aparelhos diferentes** e o histórico tiver que acompanhar
cada uma. Aí precisa de servidor — e eu já uso **Supabase** no meu outro projeto, pelo REST puro,
com `fetch`, sem biblioteca nenhuma. Siga o mesmo padrão.

**Não reaproveite o projeto Supabase do meu game.** Crie um projeto (ou ao menos tabelas) só para
o sistema de QI, e deixe URL e chave no topo do arquivo, como no meu game:

```js
const NUVEM = { url: "https://SEU-PROJETO.supabase.co", key: "SUA_CHAVE_PUBLISHABLE" };
```

### O que de fato protege os dados

A chave publishable **fica visível no arquivo** — e tudo bem, ela é feita para isso. Quem protege
o dado é o **RLS (Row Level Security)** no banco, amarrado ao usuário autenticado. Sem RLS, a chave
visível vira acesso livre ao histórico de todo mundo. **Isto é o item mais importante desta parte.**

```sql
create table public.resultados (
  id         uuid primary key default gen_random_uuid(),
  user_id    uuid not null references auth.users(id) on delete cascade,
  dia        date not null,
  qi         int,
  acertos    int,
  total      int,
  percentual int,
  tempo_seg  int,
  categorias jsonb,
  respostas  jsonb,
  criado_em  timestamptz default now(),
  unique (user_id, dia)          -- um resultado por pessoa por dia, garantido no banco
);

alter table public.resultados enable row level security;

create policy "le so o proprio"      on public.resultados for select
  using (auth.uid() = user_id);
create policy "insere so o proprio"  on public.resultados for insert
  with check (auth.uid() = user_id);
create policy "atualiza so o proprio" on public.resultados for update
  using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

Confira, depois de criar: **logado como A, não consigo ler nenhuma linha de B.** Teste isso de
verdade, com duas contas.

### Autenticação, sem biblioteca

E-mail + senha, pelos endpoints REST:

- Cadastro: `POST {url}/auth/v1/signup` — corpo `{email, password}`, header `apikey`.
- Entrar: `POST {url}/auth/v1/token?grant_type=password` — devolve `access_token` e `refresh_token`.
- Renovar: `POST {url}/auth/v1/token?grant_type=refresh_token` quando o token expirar (~1h).
  Renove **antes** de expirar ou trate o 401 renovando e repetindo a chamada uma vez.
- Nas chamadas de dado: headers `apikey: <chave>` **e** `Authorization: Bearer <access_token>`.
- Guarde **só os tokens** no `localStorage`. **Nunca guarde a senha**, nem em memória além do
  instante do envio.
- "Esqueci a senha": `POST {url}/auth/v1/recover`. Não invente fluxo próprio.
- Se o painel do Supabase estiver exigindo **confirmação de e-mail**, ou desligue essa opção, ou
  trate o estado "confirme seu e-mail para entrar" na tela. Não deixe a pessoa num limbo sem aviso.

### Sincronização

- **Local primeiro**: salve no `localStorage` e mostre o resultado na hora; mande para a nuvem
  depois. Internet caindo não pode fazer a pessoa perder o teste que acabou de fazer.
- Enviar com `POST /rest/v1/resultados` e header `Prefer: resolution=merge-duplicates`, aproveitando
  o `unique (user_id, dia)` — refazer o dia atualiza a linha em vez de duplicar.
- Ao entrar, baixe o histórico (`GET /rest/v1/resultados?order=dia.desc`) e funda com o local.
  Em conflito no mesmo dia, **o mais recente por `criado_em` vence**.
- Toda falha de rede é silenciosa no console, mas **visível na tela**: um chip "offline — salvo
  neste aparelho", igual ao `cloudInfo` do meu game.

### Hospedagem

Com Nível 2, o arquivo precisa sair do `file://` e ir para uma URL — **GitHub Pages** resolve, é de
graça, e eu já uso isso. O `file://` manda `Origin: null` e alguns endpoints recusam.
**Neste caso, e só neste caso, a regra do duplo clique da restrição técnica fica suspensa** — mas
continua valendo tudo o mais: um arquivo, sem build, sem Python, sem biblioteca.

## 5.4 Antes de implementar, me pergunte

**As pessoas vão usar o mesmo aparelho ou cada uma no seu?**

- Mesmo aparelho → Nível 1, e acabou.
- Aparelhos diferentes → Nível 1 **+** Nível 2.

Se eu não responder, faça o **Nível 1** e deixe o código organizado de um jeito que o Nível 2 possa
entrar depois sem reescrever tudo: toda leitura e gravação do histórico passando por duas funções
(`carregarHistorico()` / `salvarResultado()`), nunca com `localStorage` espalhado pelo código.

---

# CRITÉRIOS DE ACEITE — verifique um por um antes de dizer que terminou

**Histórico**

1. Faço um teste, recarrego a página: o resultado continua lá.
2. Simulo dois dias diferentes (pode editar o `dia` de um registro pelo console): aparecem **dois
   cards separados**, com datas distintas, do mais novo para o mais antigo.
3. Um teste finalizado **às 22h** fica gravado com a data de **hoje**, não a de amanhã.
   *(Teste isso explicitamente — é o bug do `toISOString`.)*
4. O histórico sobrevive a fechar e reabrir o navegador.
5. Dado antigo, no formato anterior, continua aparecendo depois da migração.
6. Com zero testes, o histórico mostra o estado vazio, sem erro no console.

**Gráficos**

7. QI diário e total acumulado estão em **gráficos separados** — não existe segundo eixo Y em lugar
   nenhum.
8. Trocar o filtro de período atualiza os três gráficos **e** a faixa de números, de uma vez.
9. Esconder uma área pelo filtro **não** muda a cor das áreas que continuam visíveis.
10. O hover mostra tooltip com o dia certo, e funciona com o dedo numa tela de 375px.
11. Com um único dia de histórico, nenhum gráfico quebra nem mostra NaN.
12. O botão "Ver tabela" mostra os mesmos dados do gráfico.

**Tema claro/escuro**

13. O botão alterna o app inteiro — HUD, painéis, cards, diálogos, toasts e **os gráficos**.
14. Troquei o tema, recarreguei a página: continua no tema que escolhi.
15. Abrindo pela primeira vez (`localStorage` limpo) num Windows configurado em modo claro, o app
    abre **claro**; em modo escuro, abre **escuro**.
16. **Não pisca**: o app não abre num tema e troca para o outro depois de carregar.
17. Depois de trocar o tema, os gráficos estão com as cores do tema **novo** — nenhuma linha,
    grade, rótulo ou tooltip ficou com a cor do tema anterior.
18. No tema claro, nenhum texto fica lavado a ponto de não dar para ler.

**Login / várias pessoas**

19. Crio a pessoa A, faço um teste; crio a pessoa B: B começa **do zero**, sem ver nada de A.
20. Volto para A: o histórico de A está intacto, com os dias certos.
21. Perfil com PIN pede o PIN; PIN errado não entra e mostra mensagem clara; perfil sem PIN entra direto.
22. Recarrego a página: continuo logado na mesma pessoa (ou volto ao portal — decida e me diga qual,
    mas seja consistente).
23. Excluir um perfil apaga **só** os dados dele.
24. Cadastro um perfil com o nome `<img src=x onerror=alert(1)>`: aparece como texto na tela,
    **nada executa**.
25. Trocar de pessoa **não** troca o tema (o tema é do aparelho).
26. *(Só no Nível 2)* Logado como A, nenhuma chamada devolve linha de B — testado com duas contas.

**Geral**

27. **O arquivo `.html` abre com duplo clique e funciona** — sem servidor, sem Python, sem npm.
    *(Teste de verdade: feche o editor, dê duplo clique no arquivo, use o app do começo ao fim.
    Exceção única: se eu tiver pedido o Nível 2 do login.)*
28. Nenhum erro no console em nenhum dos fluxos acima, **nos dois temas**.
29. Em 375px de largura, nenhuma tela rola na horizontal, **nos dois temas**.

---

# ORDEM DE TRABALHO

1. Me pergunte a dúvida do **5.4** (mesmo aparelho ou aparelhos diferentes). Enquanto eu não
   respondo, siga — é a única coisa que pode mudar de escopo depois.
2. Diagnostique o bug e **me diga qual era a causa real** (1.2).
3. Corrija o modelo de dados e a gravação (1.3, 1.4), com migração do que já existe, **já isolando**
   tudo em `carregarHistorico()` / `salvarResultado()` (5.4).
4. Construa a aba **Histórico** (1.5).
5. Construa os **gráficos e filtros** (PARTE 2), já lendo as cores de token.
6. Aplique o **layout** (PARTE 3), tela por tela.
7. Adicione o **botão de tema** (PARTE 4) e revise as duas telas lado a lado.
8. Faça o **portal de perfis** (5.2) e mova o histórico para a chave por pessoa.
9. Rode os **29 critérios de aceite** e me diga o resultado de cada um.
10. No fim: o que mudou, quais arquivos, e o que ficou de fora — se ficou.
