# SLO — Correções e Melhorias · Design Spec
**Data:** 2026-05-22

---

## Escopo

Seis itens agrupados em correções simples e funcionalidades novas, todos dentro de `slo.html`.

---

## 1. Remover timer entre exercícios

**Problema:** ao salvar um registro, `saveLog` dispara `startRestTimer(_restSecs)` (linha ~1419). O timer inline da última série já está rodando — os dois sobem simultaneamente.

**Solução:**
- Remover a chamada `startRestTimer(_restSecs)` de `saveLog`
- Remover o elemento HTML `#rest-timer` e seu CSS (`.rest-timer`, `.rest-timer-label`, `.rest-timer-count`, `.rest-timer-close`)
- Remover as funções `startRestTimer`, `cancelTimer`, `adjTimer`
- O timer inline (`startInlineTimer`, `#log-inline-timer`) permanece intacto

---

## 2. Campo descanso aceitar número puro

**Problema:** `parseRestSecs` usa regex `/(\d+)\s*(min|m|s)/i` — não aceita `"90"` sem unidade.

**Solução:**
- Ajustar `parseRestSecs` para, se nenhuma unidade for encontrada mas houver dígitos, tratar como segundos
- Atualizar placeholder do input `#ex-rest` de `"90s"` para `"90 ou 90s"`
- Regras finais: `"90"` → 90s · `"90s"` → 90s · `"1m"` → 60s · `""` → 90s (default)

---

## 3. Progresso com gráfico

### Estrutura

A tela `#sec-progress` ganha duas abas: **Musculação** e **Atividades**.

### Aba Musculação

- Mantém os cards atuais (inicial, atual, ganho, %)
- Substitui a barra de progresso plana por um **sparkline SVG** de linha mostrando a evolução de carga sessão a sessão
- SVG: largura 100%, altura 48px, linha âmbar (`--amber`), pontos circulares em cada sessão

### Aba Atividades

Agrupa por tipo de atividade (`caminhada`, `natacao`, `pedal`, etc.).

Para cada tipo com ao menos um registro:
- **Gráfico de barras SVG** por sessão — métrica principal: distância (km) para caminhada/pedal, duração (min) para natação e demais
- Altura do gráfico: 64px, barras âmbar
- Cards de resumo: total de sessões · melhor sessão · última sessão

### Filtros de período

Ambas as abas têm pills de filtro: **Semana · Mês · 3 Meses · Ano**.

- Períodos sem nenhum registro: gráfico exibe barras zeradas + stats em `0`
- Filtro padrão ao entrar na tela: **Mês**

### Implementação

SVG gerado por JS puro — sem biblioteca externa. Função `drawSparkline(data, svgEl)` para musculação, `drawBars(data, svgEl)` para atividades. Ambas recebem array de valores numéricos e escalam internamente (min→max ou 0→max).

---

## 4. Treino de hoje clicável

**Problema:** card `today-card` na tela Início é visual only.

**Solução:**
- Adicionar `onclick` ao `div.today-card` que chama `goTo('workouts')` e, em seguida, força `openCards.add(todayWk.id)` + re-render para expandir o card do dia
- Cursor pointer + leve feedback visual (opacity on active state via CSS)
- Apenas quando há treino no dia (estado "Dia de descanso" não é clicável)

---

## 5. Exercício equivalente

### Modelo de dados

Adicionar campo `equivalent_name` (string) ao objeto exercício no localStorage.

### Cadastro

Em Editar Exercício (`#sh-exercise`):
- Nova linha abaixo do campo Nome: label `"Equivalente"` + input texto `#ex-equivalent`
- Salvo como `equivalent_name` em `saveExercise`
- Carregado em `openEditEx`

### Troca

- O ícone ⇄ (SVG inline ou unicode `⇄`) aparece ao lado do nome do exercício no card do treino **somente quando `equivalent_name` está preenchido**
- `onclick` chama `swapEquivalent(exId)`:
  - Lê o exercício do DB
  - `newName = ex.equivalent_name`, `newEquivalent = ex.name`
  - Salva `{name: newName, equivalent_name: newEquivalent}` via `DB.updateExercise`
  - Re-render do card
- Tocar novamente reverte (troca bilateral simétrica)
- Sets, reps, rest_time e notes não mudam

---

## 6. Som do timer — arquitetura para áudios SLO

**Agora:** mantém os três beeps via AudioContext exatamente como estão.

**Quando os áudios estiverem prontos:**
- Array `SLO_SOUNDS = ['sounds/bora-treinar.mp3', 'sounds/cuida.mp3', 'proximo.mp3']`
- Função `playTimerSound()` tenta `new Audio(SLO_SOUNDS[random]).play()`, com fallback para os beeps se o arquivo falhar
- Chamada em `startInlineTimer` no lugar dos beeps atuais
- Pasta `sounds/` na raiz do projeto (servida pelo Netlify)

**Nesta entrega:** apenas estruturar o código com a função `playTimerSound()` já extraída, chamando os beeps atuais. Substituição dos arquivos é passo futuro sem alteração de código.

---

## Fora do escopo desta entrega

- Filtros de período com comparação entre períodos (ex: "vs semana anterior")
- Interface visual para upload de áudios pelo aluno
- Sugestão automática de exercícios equivalentes por grupo muscular
- Sets/reps diferentes para o exercício equivalente
