# SLO — Correções e Melhorias · Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Seis melhorias no `slo.html`: remover timer duplo, aceitar número no campo descanso, tornar o card de hoje clicável, adicionar exercício equivalente, refatorar tela de progresso com gráficos SVG e filtros de período, e extrair função de som do timer.

**Architecture:** Single-file app (`slo.html`). Todas as mudanças são neste arquivo. Ordem das tasks: fixes simples → exercício equivalente → progresso (maior mudança).

**Tech Stack:** HTML + CSS inline + JS vanilla + SVG gerado por JS. Sem build step. Deploy via `git push` (GitHub Action copia para `index.html` e deploya no Netlify).

---

## Arquivos modificados

- `slo.html` — único arquivo da aplicação

---

## Task 1: Extrair `playTimerSound()`

**Files:**
- Modify: `slo.html` (~linha 1244–1246 dentro de `startInlineTimer`, e logo após `getAudioCtx`)

- [ ] **Passo 1: Localizar o bloco de beeps dentro de `startInlineTimer`**

  Buscar no arquivo o trecho exato (está dentro do `if(_inlineRem<=0)` de `startInlineTimer`):
  ```js
  const ac=getAudioCtx();
  if(ac){try{[0,300,600].forEach(d=>{const o=ac.createOscillator(),g=ac.createGain();o.connect(g);g.connect(ac.destination);o.frequency.value=880;g.gain.value=0.35;const t=ac.currentTime+d/1000;o.start(t);o.stop(t+0.18);});}catch(e){}}
  ```

- [ ] **Passo 2: Adicionar função `playTimerSound` logo após `getAudioCtx`**

  Inserir após o fechamento da função `getAudioCtx` (após a linha com `return _audioCtx;`):
  ```js
  function playTimerSound(){
    const ac=getAudioCtx();
    if(ac){try{[0,300,600].forEach(d=>{const o=ac.createOscillator(),g=ac.createGain();o.connect(g);g.connect(ac.destination);o.frequency.value=880;g.gain.value=0.35;const t=ac.currentTime+d/1000;o.start(t);o.stop(t+0.18);});}catch(e){}}
  }
  ```

- [ ] **Passo 3: Substituir o bloco de beeps em `startInlineTimer` pela chamada**

  Substituir o trecho do Passo 1 por:
  ```js
  playTimerSound();
  ```

- [ ] **Passo 4: Verificar no browser**

  Abrir `slo.html`, abrir o registro de um exercício, marcar uma série como concluída e aguardar o timer zerar. Os três beeps devem soar normalmente.

- [ ] **Passo 5: Commit**
  ```bash
  git add slo.html
  git commit -m "refactor: extrair playTimerSound() para preparar áudios SLO"
  ```

---

## Task 2: Remover timer entre exercícios

**Files:**
- Modify: `slo.html` (CSS ~linhas 252–257, HTML ~linhas 538–545, JS ~linhas 1278–1338, chamada em saveLog ~linha 1419)

- [ ] **Passo 1: Remover CSS do rest-timer**

  Localizar e remover as 6 linhas de CSS que começam com `.rest-timer`:
  ```css
  .rest-timer{position:fixed;...}
  .rest-timer.hidden{...}
  .rest-timer.done{...}
  .rest-timer-label{...}
  .rest-timer-count{...}
  .rest-timer-close{...}
  ```

- [ ] **Passo 2: Remover HTML do `#rest-timer`**

  Localizar e remover o bloco (está entre `<!-- Rest Timer -->` e seu `</div>` de fechamento):
  ```html
  <!-- Rest Timer -->
  <div id="rest-timer" class="rest-timer hidden" onclick="cancelTimer()">
    <span class="rest-timer-label">Descanso · toque p/ cancelar</span>
    <button class="rest-timer-close" onclick="event.stopPropagation();adjTimer(-15)" style="width:auto;padding:0 8px;border-radius:6px;font-size:11px;font-weight:700">−15s</button>
    <button class="rest-timer-close" onclick="event.stopPropagation();adjTimer(15)" style="width:auto;padding:0 8px;border-radius:6px;font-size:11px;font-weight:700">+15s</button>
    <span class="rest-timer-count" id="rest-timer-count">90s</span>
    <button class="rest-timer-close" onclick="cancelTimer()">✕</button>
  </div>
  ```

- [ ] **Passo 3: Remover as funções `startRestTimer`, `cancelTimer`, `adjTimer`**

  Localizar e remover as três funções JS (do `function startRestTimer(secs){` até o fechamento de `adjTimer`).

- [ ] **Passo 4: Remover chamada em `saveLog`**

  Em `saveLog`, localizar e remover a linha:
  ```js
  startRestTimer(_restSecs);
  ```

- [ ] **Passo 5: Verificar no browser**

  Abrir `slo.html`, registrar um exercício e confirmar que nenhum timer flutuante aparece no rodapé. O timer inline (dentro do formulário) deve continuar funcionando ao marcar séries.

- [ ] **Passo 6: Commit**
  ```bash
  git add slo.html
  git commit -m "fix: remover timer entre exercícios (duplicate com timer de série)"
  ```

---

## Task 3: Campo descanso aceitar número puro

**Files:**
- Modify: `slo.html` (função `parseRestSecs` ~linha 1255, placeholder do input `#ex-rest` ~linha 457)

- [ ] **Passo 1: Atualizar `parseRestSecs`**

  Localizar a função atual:
  ```js
  function parseRestSecs(str){
    if(!str)return 90;
    const m=str.match(/(\d+)\s*(min|m|s)/i);
    if(!m)return 90;
    return m[2][0].toLowerCase()==='m'?parseInt(m[1])*60:parseInt(m[1]);
  }
  ```

  Substituir por:
  ```js
  function parseRestSecs(str){
    if(!str)return 90;
    const m=str.match(/(\d+)\s*(min|m|s)?/i);
    if(!m)return 90;
    if(!m[2])return parseInt(m[1]);
    return m[2][0].toLowerCase()==='m'?parseInt(m[1])*60:parseInt(m[1]);
  }
  ```

- [ ] **Passo 2: Atualizar placeholder do input**

  Localizar:
  ```html
  <div class="fg"><label class="lbl">Descanso</label><input id="ex-rest" class="inp" placeholder="90s"></div>
  ```

  Substituir por:
  ```html
  <div class="fg"><label class="lbl">Descanso</label><input id="ex-rest" class="inp" placeholder="90 ou 90s"></div>
  ```

- [ ] **Passo 3: Verificar no browser**

  Editar um exercício, digitar `90` no campo descanso (sem "s"), salvar e abrir o registro. O timer deve iniciar com 90 segundos.

- [ ] **Passo 4: Commit**
  ```bash
  git add slo.html
  git commit -m "fix: campo descanso aceita número puro (ex: 90 além de 90s)"
  ```

---

## Task 4: Treino de hoje clicável

**Files:**
- Modify: `slo.html` (função `renderTodayWidget` ~linha 850)

- [ ] **Passo 1: Adicionar onclick ao today-card**

  Em `renderTodayWidget`, localizar o HTML do card quando há treino (o `div.today-card` que mostra nome e foco):
  ```js
  w.innerHTML=`<div class="today-card">
    <div class="today-eyebrow">Hoje · ${todayName}</div>
    <div class="today-name">${esc(todayWk.name)}</div>
    <div class="today-focus">${esc(todayWk.focus||'')}</div>
    <span class="today-chip">${exCount} exercícios</span>
  </div>`;
  ```

  Substituir por (adiciona `onclick`, `cursor:pointer` e `title`):
  ```js
  w.innerHTML=`<div class="today-card" onclick="goTo('workouts')" style="cursor:pointer" title="Ir para o treino de hoje">
    <div class="today-eyebrow">Hoje · ${todayName}</div>
    <div class="today-name">${esc(todayWk.name)}</div>
    <div class="today-focus">${esc(todayWk.focus||'')}</div>
    <span class="today-chip">${exCount} exercícios →</span>
  </div>`;
  ```

  Nota: a seta `→` no chip indica interatividade. `goTo('workouts')` já chama `renderWorkouts()`, que auto-abre o card do dia atual (linha ~999 do código).

- [ ] **Passo 2: Verificar no browser**

  Na tela Início, tocar no card do treino de hoje. Deve navegar para Treinos com o card do dia já expandido.

- [ ] **Passo 3: Commit**
  ```bash
  git add slo.html
  git commit -m "feat: card do treino de hoje clicável na tela inicial"
  ```

---

## Task 5: Exercício equivalente — formulário e modelo de dados

**Files:**
- Modify: `slo.html` (HTML do formulário ~linha 457, funções `openAddEx`, `openEditEx`, `saveExercise` ~linhas 1098–1165)

- [ ] **Passo 1: Adicionar campo no formulário de exercício**

  Localizar no HTML do sheet `#sh-exercise`:
  ```html
  <div class="fg"><label class="lbl">Descanso</label><input id="ex-rest" class="inp" placeholder="90 ou 90s"></div>
  <div class="fg"><label class="lbl">Observações</label><textarea id="ex-notes" class="inp" placeholder="Técnica, variações, dicas…"></textarea></div>
  ```

  Substituir por:
  ```html
  <div class="fg"><label class="lbl">Descanso</label><input id="ex-rest" class="inp" placeholder="90 ou 90s"></div>
  <div class="fg"><label class="lbl">Exercício equivalente</label><input id="ex-equivalent" class="inp" placeholder="Ex: Supino na Máquina"></div>
  <div class="fg"><label class="lbl">Observações</label><textarea id="ex-notes" class="inp" placeholder="Técnica, variações, dicas…"></textarea></div>
  ```

- [ ] **Passo 2: Limpar campo em `openAddEx`**

  Dentro de `openAddEx`, após a linha `document.getElementById('ex-notes').value='';`, adicionar:
  ```js
  document.getElementById('ex-equivalent').value='';
  ```

- [ ] **Passo 3: Carregar valor em `openEditEx`**

  Dentro de `openEditEx`, após a linha `document.getElementById('ex-notes').value=ex.notes||'';`, adicionar:
  ```js
  document.getElementById('ex-equivalent').value=ex.equivalent_name||'';
  ```

- [ ] **Passo 4: Salvar em `saveExercise`**

  Dentro de `saveExercise`, no objeto `data`, adicionar o campo `equivalent_name` após `notes`:
  ```js
  const data={
    workout_id:wid,
    name:document.getElementById('ex-name').value,
    sets:document.getElementById('ex-sets').value,
    reps:document.getElementById('ex-reps').value,
    rest_time:document.getElementById('ex-rest').value,
    notes:document.getElementById('ex-notes').value,
    equivalent_name:document.getElementById('ex-equivalent').value,
  };
  ```

- [ ] **Passo 5: Verificar no browser**

  Editar um exercício, preencher o campo "Exercício equivalente" com um nome, salvar. Reabrir o formulário de edição e confirmar que o campo está preenchido.

- [ ] **Passo 6: Commit**
  ```bash
  git add slo.html
  git commit -m "feat: campo exercício equivalente no formulário de edição"
  ```

---

## Task 6: Exercício equivalente — ícone de troca no card

**Files:**
- Modify: `slo.html` (função `renderWkCard` ~linha 1022, nova função `swapEquivalent`)

- [ ] **Passo 1: Adicionar ícone ⇄ no card do exercício**

  Em `renderWkCard`, dentro do map de exercícios, localizar:
  ```js
  return`<div class="ex-item" onclick="openLog('${ex.id}')" style="cursor:pointer">
    <div>
      <div class="ex-name">${esc(ex.name)}</div>
  ```

  Substituir por (adiciona o ícone de swap antes do nome, visível apenas quando há equivalente):
  ```js
  return`<div class="ex-item" onclick="openLog('${ex.id}')" style="cursor:pointer">
    <div>
      <div class="ex-name" style="display:flex;align-items:center;gap:6px">
        ${esc(ex.name)}
        ${ex.equivalent_name?`<button class="ibtn" title="Trocar com ${esc(ex.equivalent_name)}" onclick="event.stopPropagation();swapEquivalent('${ex.id}')" style="width:22px;height:22px;flex-shrink:0;color:var(--ink-3)"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M7 16V4m0 0L3 8m4-4l4 4"/><path d="M17 8v12m0 0l4-4m-4 4l-4-4"/></svg></button>`:''}
      </div>
  ```

- [ ] **Passo 2: Adicionar função `swapEquivalent`**

  Inserir após `deleteEx`:
  ```js
  function swapEquivalent(id){
    const ex=DB.getExercises().find(e=>e.id===id);
    if(!ex||!ex.equivalent_name)return;
    DB.updateExercise(id,{name:ex.equivalent_name,equivalent_name:ex.name});
    renderWorkouts();
    toast('Exercício trocado: '+esc(ex.equivalent_name),'ok');
  }
  ```

- [ ] **Passo 3: Verificar no browser**

  1. Editar um exercício e preencher "Exercício equivalente" com "Supino na Máquina"
  2. Fechar o formulário — o ícone ⇄ deve aparecer ao lado do nome
  3. Tocar no ícone — o nome do exercício e o equivalente devem trocar
  4. Tocar novamente — volta ao estado original

- [ ] **Passo 4: Commit**
  ```bash
  git add slo.html
  git commit -m "feat: ícone de troca de exercício equivalente no card do treino"
  ```

---

## Task 7: Progresso — funções auxiliares SVG e filtro de período

**Files:**
- Modify: `slo.html` (adicionar funções antes de `renderProgress` ~linha 1431)

- [ ] **Passo 1: Adicionar variáveis de estado do progresso**

  Localizar o comentário `// ═══ PROGRESSO` e inserir antes dele:
  ```js
  let _progressPeriod='month', _progressTab='muscle';
  ```

- [ ] **Passo 2: Adicionar `filterLogsByPeriod`**

  Logo após as variáveis de estado:
  ```js
  function filterLogsByPeriod(logs,period){
    const now=new Date();
    const cutoff=new Date(now);
    if(period==='week')cutoff.setDate(now.getDate()-7);
    else if(period==='month')cutoff.setMonth(now.getMonth()-1);
    else if(period==='3months')cutoff.setMonth(now.getMonth()-3);
    else if(period==='year')cutoff.setFullYear(now.getFullYear()-1);
    return logs.filter(l=>new Date(l.date)>=cutoff);
  }
  ```

- [ ] **Passo 3: Adicionar `drawSparkline`**

  ```js
  function drawSparkline(values){
    const w=280,h=48,pad=6;
    if(!values.length)return`<svg viewBox="0 0 ${w} ${h}" style="width:100%;height:${h}px"><line x1="${pad}" y1="${h/2}" x2="${w-pad}" y2="${h/2}" stroke="var(--line)" stroke-width="1.5" stroke-dasharray="4 3"/></svg>`;
    if(values.length===1)return`<svg viewBox="0 0 ${w} ${h}" style="width:100%;height:${h}px"><circle cx="${w/2}" cy="${h/2}" r="4" fill="var(--amber)"/></svg>`;
    const mn=Math.min(...values),mx=Math.max(...values),range=mx-mn||1;
    const pts=values.map((v,i)=>{
      const x=pad+(i/(values.length-1))*(w-2*pad);
      const y=h-pad-((v-mn)/range)*(h-2*pad);
      return[+x.toFixed(1),+y.toFixed(1)];
    });
    const line=pts.map((p,i)=>(i?'L':'M')+p[0]+' '+p[1]).join(' ');
    const dots=pts.map(p=>`<circle cx="${p[0]}" cy="${p[1]}" r="3" fill="var(--amber)"/>`).join('');
    return`<svg viewBox="0 0 ${w} ${h}" style="width:100%;height:${h}px;overflow:visible"><path d="${line}" fill="none" stroke="var(--amber)" stroke-width="2" stroke-linejoin="round" stroke-linecap="round"/>${dots}</svg>`;
  }
  ```

- [ ] **Passo 4: Adicionar `drawBars`**

  ```js
  function drawBars(values){
    const w=280,h=64,pad=4,gap=2;
    const mx=Math.max(...values,1);
    const n=values.length||1;
    const bw=Math.max(2,(w-pad*2-(n-1)*gap)/n);
    const bars=values.map((v,i)=>{
      const x=pad+i*(bw+gap);
      const bh=Math.max(v>0?3:1,((v/mx)*(h-pad*2)));
      const y=h-pad-bh;
      return`<rect x="${x.toFixed(1)}" y="${y.toFixed(1)}" width="${bw.toFixed(1)}" height="${bh.toFixed(1)}" rx="2" fill="var(--amber)" opacity="${v===0?'0.2':'1'}"/>`;
    }).join('');
    return`<svg viewBox="0 0 ${w} ${h}" style="width:100%;height:${h}px">${bars}</svg>`;
  }
  ```

- [ ] **Passo 5: Verificar no browser**

  Abrir `slo.html` e confirmar que não há erros no console. A tela de Progresso ainda deve funcionar igual (ainda não alteramos `renderProgress`).

- [ ] **Passo 6: Commit**
  ```bash
  git add slo.html
  git commit -m "feat: funções auxiliares drawSparkline, drawBars e filterLogsByPeriod"
  ```

---

## Task 8: Progresso — nova `renderProgress` com abas e filtros

**Files:**
- Modify: `slo.html` (substituir função `renderProgress` inteira ~linhas 1431–1470, adicionar CSS para tabs/pills)

- [ ] **Passo 1: Adicionar CSS para abas e pills do progresso**

  Localizar no bloco de CSS a linha com `.prog-card{` e adicionar antes dela:
  ```css
  .prog-tabs{display:flex;border-bottom:2px solid var(--line);margin-bottom:16px}
  .prog-tab{background:none;border:none;padding:10px 16px;font-size:14px;font-weight:600;color:var(--ink-4);cursor:pointer;border-bottom:2px solid transparent;margin-bottom:-2px;transition:color .15s,border-color .15s}
  .prog-tab.on{color:var(--amber-dark);border-bottom-color:var(--amber)}
  .prog-period{display:flex;gap:6px;flex-wrap:wrap;margin-bottom:16px}
  .prog-period .mode-pill{font-size:12px;padding:5px 12px}
  .prog-section.hidden{display:none}
  ```

- [ ] **Passo 2: Substituir `renderProgress` inteira**

  Localizar a função completa (de `function renderProgress(){` até o `}` final) e substituir por:

  ```js
  function renderProgress(){
    const container=document.getElementById('progress-content');
    const allLogs=DB.getLogs();
    const filtered=filterLogsByPeriod(allLogs,_progressPeriod);

    const periodHtml=`<div class="prog-period">${[['week','Semana'],['month','Mês'],['3months','3 Meses'],['year','Ano']].map(([k,l])=>`<button class="mode-pill${_progressPeriod===k?' on':''}" onclick="_progressPeriod='${k}';renderProgress()">${l}</button>`).join('')}</div>`;

    const tabsHtml=`<div class="prog-tabs"><button class="prog-tab${_progressTab==='muscle'?' on':''}" onclick="_progressTab='muscle';renderProgress()">Musculação</button><button class="prog-tab${_progressTab==='activity'?' on':''}" onclick="_progressTab='activity';renderProgress()">Atividades</button></div>`;

    container.innerHTML=periodHtml+tabsHtml+'<div id="prog-body"></div>';

    if(_progressTab==='muscle')renderProgressMuscle(filtered);
    else renderProgressActivity(filtered);
  }

  function renderProgressMuscle(logs){
    const body=document.getElementById('prog-body');
    const exercises=DB.getExercises();
    const byEx={};
    logs.filter(l=>!String(l.exercise_id).startsWith('activity_')).forEach(l=>{
      if(!byEx[l.exercise_id])byEx[l.exercise_id]=[];
      byEx[l.exercise_id].push(l);
    });
    const cards=Object.entries(byEx).map(([exId,exLogs])=>{
      const ex=exercises.find(e=>e.id===exId);
      const name=ex?ex.name:'Exercício';
      exLogs.sort((a,b)=>a.date-b.date);
      const first=exLogs[0],last=exLogs[exLogs.length-1];
      const gain=last.weight-first.weight;
      const pct=first.weight>0?((gain/first.weight)*100).toFixed(1):null;
      const lastDate=new Date(last.date).toLocaleDateString('pt-BR');
      const sparkValues=exLogs.map(l=>l.weight);
      return`<div class="prog-card">
        <div class="prog-name">${esc(name)}</div>
        <div class="prog-grid">
          <div class="prog-stat"><div class="prog-stat-lbl">Inicial</div><div class="prog-stat-val">${first.weight}kg</div></div>
          <div class="prog-stat"><div class="prog-stat-lbl">Atual</div><div class="prog-stat-val">${last.weight}kg</div></div>
          <div class="prog-stat"><div class="prog-stat-lbl">Ganho</div><div class="prog-stat-val ${gain>0?'up':''}">${gain>=0?'+':''}${gain.toFixed(1)}kg</div></div>
          <div class="prog-stat"><div class="prog-stat-lbl">%</div><div class="prog-stat-val ${gain>0?'up':'neutral'}">${pct!==null?(gain>=0?'+':'')+pct+'%':'—'}</div></div>
        </div>
        <div style="margin:10px 0 4px">${drawSparkline(sparkValues)}</div>
        <div class="prog-sessions">${exLogs.length} sessão${exLogs.length!==1?'ões':''} registrada${exLogs.length!==1?'s':''}</div>
        <div class="prog-last">Último registro: ${lastDate}</div>
      </div>`;
    }).join('');
    body.innerHTML=cards||`<div class="empty"><div class="empty-icon">📈</div><div class="empty-title">Sem registros no período</div></div>`;
  }

  function renderProgressActivity(logs){
    const body=document.getElementById('prog-body');
    const actLogs=logs.filter(l=>String(l.exercise_id).startsWith('activity_'));
    if(!actLogs.length){
      body.innerHTML=`<div class="empty"><div class="empty-icon">🏃</div><div class="empty-title">Sem atividades no período</div></div>`;
      return;
    }
    const byType={};
    actLogs.forEach(l=>{
      const t=l.exercise_id.replace('activity_','');
      if(!byType[t])byType[t]=[];
      byType[t].push(l);
    });
    const ACT_ICONS={caminhada:'🚶',natacao:'🏊',pedal:'🚴',corrida:'🏃',outro:'⚡'};
    const ACT_NAMES={caminhada:'Caminhada',natacao:'Natação',pedal:'Pedal',corrida:'Corrida',outro:'Outro'};
    const cards=Object.entries(byType).map(([type,tLogs])=>{
      tLogs.sort((a,b)=>a.date-b.date);
      const icon=ACT_ICONS[type]||'⚡';
      const name=ACT_NAMES[type]||type;
      const distLogs=['caminhada','pedal','corrida'];
      const useDistance=distLogs.includes(type);
      const values=tLogs.map(l=>{
        if(useDistance&&l.activity_data&&l.activity_data.fields&&l.activity_data.fields.distance)
          return parseFloat(l.activity_data.fields.distance)||0;
        return parseFloat(l.reps_completed)||0;
      });
      const metricLabel=useDistance?'km':'min';
      const best=Math.max(...values);
      const last=values[values.length-1];
      const lastDate=new Date(tLogs[tLogs.length-1].date).toLocaleDateString('pt-BR');
      return`<div class="prog-card">
        <div class="prog-name">${icon} ${esc(name)}</div>
        <div class="prog-grid">
          <div class="prog-stat"><div class="prog-stat-lbl">Sessões</div><div class="prog-stat-val">${tLogs.length}</div></div>
          <div class="prog-stat"><div class="prog-stat-lbl">Melhor</div><div class="prog-stat-val">${best>0?best+metricLabel:'—'}</div></div>
          <div class="prog-stat"><div class="prog-stat-lbl">Último</div><div class="prog-stat-val">${last>0?last+metricLabel:'—'}</div></div>
          <div class="prog-stat"><div class="prog-stat-lbl">Data</div><div class="prog-stat-val" style="font-size:11px">${lastDate}</div></div>
        </div>
        <div style="margin:10px 0 4px">${drawBars(values)}</div>
        <div class="prog-last">Última atividade: ${lastDate}</div>
      </div>`;
    }).join('');
    body.innerHTML=cards;
  }
  ```

- [ ] **Passo 3: Verificar no browser**

  1. Abrir tela Progresso
  2. Confirmar que as abas Musculação / Atividades aparecem
  3. Confirmar que os filtros de período (Semana / Mês / 3 Meses / Ano) funcionam
  4. Selecionar um período sem treinos — deve mostrar "Sem registros no período" ou barras zeradas
  5. Se houver registros de musculação, confirmar que o sparkline aparece no card
  6. Se houver registros de atividade, confirmar que as barras aparecem

- [ ] **Passo 4: Commit**
  ```bash
  git add slo.html
  git commit -m "feat: tela de progresso com abas, filtros de período e gráficos SVG"
  ```

---

## Task 9: Deploy e verificação final

- [ ] **Passo 1: Push para o repositório**
  ```bash
  git push
  ```

- [ ] **Passo 2: Aguardar deploy**

  O GitHub Action copia `slo.html → index.html` e deploya no Netlify automaticamente. Verificar em https://slo-treinos.netlify.app após 1–2 minutos.

- [ ] **Passo 3: Verificar no celular**

  Checklist final no dispositivo móvel:
  - [ ] Timer inline funciona ao marcar série; timer flutuante não aparece mais ao salvar
  - [ ] Campo descanso com `90` (sem "s") funciona no timer
  - [ ] Card do treino de hoje leva para aba Treinos com o card expandido
  - [ ] Ícone ⇄ aparece em exercícios com equivalente cadastrado e faz a troca
  - [ ] Tela Progresso mostra abas e filtros de período

---

## Notas para o implementador

- `slo.html` tem ~1863 linhas. Use busca por string exata (Ctrl+F / grep) para localizar trechos — não confie cegamente nos números de linha, pois cada task anterior altera o arquivo.
- Após cada task, abrir o arquivo diretamente no browser (não precisa de servidor) e testar o fluxo específico antes de commitar.
- Os logs de atividades têm `exercise_id` que começa com `'activity_'`. Logs de musculação têm `exercise_id` que é um UUID. Esse é o discriminador usado em `renderProgressMuscle` e `renderProgressActivity`.
