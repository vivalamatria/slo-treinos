# SLO — Contexto do Projeto

## O que é
App web mobile-first de gerenciamento de treinos de musculação. Conceito: mistura de Tamagochi com app fitness. Fase atual: validação prática antes do redesign de interface.

**URL em produção:** https://slo-treinos.netlify.app  
**Netlify site ID:** `85cd5608-c200-4c1c-af73-fe2495de95c6`

---

## Arquitetura

**Single-file app** — toda a aplicação está em `slo.html` (HTML + CSS inline + JS inline).  
`index.html` é uma cópia sincronizada via `cp slo.html index.html` antes de cada deploy (Netlify exige `index.html` como entry point).

**Sem build step, sem dependências externas** além de Google Fonts (Inter).

### Persistência
localStorage puro. Chaves:
```
slo_mesocycles   → mesociclos
slo_workouts     → treinos
slo_exercises    → exercícios
slo_workout_logs → registros de execução
```
Objeto `DB` centraliza todos os acessos. Função `uid()` gera UUIDs.

### Estrutura do JS (ordem no arquivo)
1. `MESO_A` / `MESO_B` — seed data dos mesociclos prontos
2. `K`, `get()`, `set()`, `uid()`, `esc()` — storage e utils
3. `DB` — CRUD para cada entidade
4. `toast()` — notificações
5. `goTo()` — navegação entre seções
6. `openSheet()` / `closeSheet()` — modais bottom-sheet
7. `renderHome()` + `renderCalWidget()` + `renderTodayWidget()` — tela Início
8. `renderWorkouts()` + `renderWkCard()` + `toggleWk()` — tela Treinos
9. `openLog()` + `saveLog()` + steppers (`_w`, `_reps`, `_repMode`) — registro de execução
10. `renderProgress()` — tela Progresso
11. `renderProfile()` — tela Perfil
12. `DOMContentLoaded` — init

---

## Design System

```css
--amber: #dd942d        /* laranja principal */
--amber-dark: #b87824   /* laranja escuro */
--amber-light: #fdf0dc  /* laranja claro */
--ink: #0F172A          /* texto principal */
--surface: #F8FAFC      /* fundo */
--white: #FFFFFF
--red: #EF4444
--green: #10B981
```

Tipografia: Inter (300–900). Mobile-first. Bottom-sheet modals. Bottom nav.

---

## Funcionalidades implementadas

- Mesociclos: CRUD + ativar/concluir + importar Mesociclo A (35 exercícios) e B (26 exercícios)
- Treinos: CRUD por dia da semana + foco muscular, agrupados por mesociclo
- Exercícios: CRUD com séries/reps/descanso/notas
- Registro de execução: stepper de carga (−2.5/−0.5/+0.5/+2.5) + stepper de reps por série com modos Iguais/↑Prog./↓Reg.
- Check button: branco com borda âmbar (não registrado hoje) → âmbar sólido (registrado hoje)
- Limpar registros: apaga logs do treino sem excluir exercícios
- Tela Início: calendário do mês (dias treinados em âmbar claro, hoje em âmbar sólido) + treino de hoje + "Ver treinos da semana"
- Treinos: auto-expande o card do dia atual no mesociclo ativo
- Estado dos cards (aberto/fechado) preservado após re-render via `openCards` Set
- Progresso: evolução de carga por exercício (inicial, atual, ganho, %)
- Perfil: estatísticas gerais + streak + apagar dados

---

## Convenções importantes

- **Nunca rotar o logo** — o logo `slo_laranja@2x.png` não tem transform
- **Datas**: salvar como `new Date(dateString + 'T12:00:00').getTime()` para evitar bug de fuso horário (UTC-3 Brasil)
- **Comparar datas de log**: usar `.toDateString()` em ambos os lados
- **XSS**: sempre usar `esc()` ao inserir strings do usuário no HTML
- **Deploy**: `cp slo.html index.html` → gerar token Netlify MCP → `npx @netlify/mcp@latest --site-id 85cd5608...`

---

## Workflow de deploy

```bash
# 1. Editar slo.html
# 2. Commitar e empurrar — o GitHub Action faz o resto
git add slo.html
git commit -m "descrição da mudança"
git push
```

O GitHub Action (.github/workflows/deploy.yml) copia slo.html → index.html e deploya no Netlify automaticamente.
Repositório: https://github.com/vivalamatria/slo-treinos

---

## Roadmap (próximas melhorias)

### UX / Funcional
- [ ] Timer de descanso entre séries
- [ ] Editar registros existentes (hoje só salva novo)
- [ ] Filtrar progresso por mesociclo ativo
- [ ] Swipe para deletar exercício/treino

### Gamificação (Fase 3)
- [ ] Sistema de streaks mais elaborado
- [ ] Badges por marcos (primeira semana, 30 dias, PR quebrado)
- [ ] Volume semanal (séries × reps × carga)

### Técnico
- [ ] PWA completo (service worker + cache offline)
- [ ] Exportar/importar dados (JSON backup)
- [ ] Múltiplos usuários / sync na nuvem

### Redesign de interface (após validação)
- Design mais refinado baseado no feedback de uso real
- Manter minimalismo suíço como base
