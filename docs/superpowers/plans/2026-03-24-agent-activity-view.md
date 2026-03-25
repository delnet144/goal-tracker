# Agent Activity View on Goal Focus — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** When a user clicks a goal card in the 3D space, show a right-sidebar panel listing tasks, agent names, and live-streaming terminal output — while also displaying agent sprites near their assigned task dots in the 3D scene.

**Architecture:** All changes are confined to a single `index.html` file. New elements layered in order: (1) mock data extended with agent fields, (2) CSS for panel + terminal, (3) a `#detail-panel` DOM node, (4) Three.js agent sprites built during the existing GOALS loop, (5) animation loop extended to pulse agents and track connector lines, (6) open/close panel logic with an interval registry for clean teardown, (7) a `streamTerminal()` function driving independent per-agent output.

**Tech Stack:** Vanilla JS ES modules, Three.js 0.165 (already loaded via importmap), CSS custom properties, `setInterval`-based terminal streaming, CSS2DRenderer for goal cards (unchanged).

---

## ⚠️ No Automated Tests

This is a single-file HTML prototype with no build step or test framework. Every "verify" step is a **browser check**: open `index.html` directly in Chrome/Firefox (no server needed — all assets from CDN). After each task, refresh and confirm the described behavior visually.

---

## File Structure

Single file only:

| Section | Lines (approx) | What changes |
|---------|---------------|--------------|
| `:root` CSS variables | ~13–35 | Add `--terminal-bg` token |
| CSS component rules | ~37–142 | Add panel, terminal, agent chip, task-row styles |
| HTML body | ~144–174 | Add `#detail-panel` div |
| `GOALS` data | ~193–237 | Add `agent` field to 3 tasks |
| JS: sprite factory area | ~260–375 | Add `makeAgentSprite()`, `allAgents[]` array |
| JS: GOALS build loop | ~375–453 | Extend inner `tasks.forEach` for agent sprites + connectors |
| JS: `setFadeTargets` | ~498–513 | Add agent fade targets |
| JS: `focusGoal` / `resetView` | ~515–535 | Wire panel open/close, clear timers |
| JS: new functions | after `resetView` | `activeTimers`, `clearAllTimers`, `openPanel`, `closePanel`, `buildTaskRow`, `streamTerminal` |
| JS: `animate()` loop | ~590–638 | Add agent pulse + connector endpoint tracking |

---

## Task 1: Extend Mock Data with Agent Fields

**Files:**
- Modify: `index.html` — `GOALS` array (approx lines 193–237)

> Change `t3` (Billing) status from `'todo'` → `'in-progress'` so an agent can work on it.
> Add `agent` object to three tasks: `t2`, `t3` (g1 — demonstrates parallel agents), and `t10` (g3).

- [ ] **Step 1: Replace the g1 tasks array**

Find this block (inside the `g1` goal object):
```js
tasks: [
  { id: 't1',  label: 'Auth flow',      status: 'done' },
  { id: 't2',  label: 'Dashboard UI',   status: 'in-progress' },
  { id: 't3',  label: 'Billing',        status: 'todo' },
  { id: 't4',  label: 'Onboarding',     status: 'todo' },
  { id: 't5',  label: 'Deploy to prod', status: 'todo' },
]
```

Replace with:
```js
tasks: [
  { id: 't1',  label: 'Auth flow',      status: 'done' },
  { id: 't2',  label: 'Dashboard UI',   status: 'in-progress', agent: {
    id: 'a1', name: 'Coder-α', color: '#7fbbb3',
    commands: [
      '$ git checkout -b feature/dashboard-v2',
      '$ npm run dev -- --port 3001',
      '  generating component tree...',
      '$ writing DashboardLayout.tsx',
      '$ writing StatsCard.tsx',
      '$ writing ActivityFeed.tsx',
      '$ npx vitest run src/dashboard',
      '  ✓ DashboardLayout renders',
      '  ✓ StatsCard shows metric',
      '$ git add -p',
      '$ git commit -m "feat: dashboard components"',
      '  [feature/dashboard-v2 3a8f2c1]',
    ]
  }},
  { id: 't3',  label: 'Billing',        status: 'in-progress', agent: {
    id: 'a2', name: 'Deployer-β', color: '#d699b6',
    commands: [
      '$ stripe listen --forward-to localhost:3000/webhook',
      '  Ready! Forwarding from api.stripe.com',
      '$ writing billing/webhook.ts',
      '$ writing billing/stripe-client.ts',
      '  creating payment intent...',
      '  → POST /webhook 200',
      '  ✓ payment intent created',
      '  ✓ webhook signature verified',
      '$ git commit -m "feat: stripe billing integration"',
      '  [main 7b4e9d2]',
    ]
  }},
  { id: 't4',  label: 'Onboarding',     status: 'todo' },
  { id: 't5',  label: 'Deploy to prod', status: 'todo' },
]
```

- [ ] **Step 2: Replace the g3 tasks array**

Find this block (inside the `g3` goal object):
```js
tasks: [
  { id: 't9',  label: 'Chapters 1–5',  status: 'done' },
  { id: 't10', label: 'Chapters 6–10', status: 'in-progress' },
  { id: 't11', label: 'CLI project',   status: 'todo' },
  { id: 't12', label: 'Async chapter', status: 'todo' },
]
```

Replace with:
```js
tasks: [
  { id: 't9',  label: 'Chapters 1–5',  status: 'done' },
  { id: 't10', label: 'Chapters 6–10', status: 'in-progress', agent: {
    id: 'a3', name: 'Scout-γ', color: '#dbbc7f',
    commands: [
      '$ rustup update stable',
      '  stable-aarch64 updated',
      '$ cargo new chapter6-exercises --lib',
      '$ writing src/ownership.rs',
      '$ cargo test -- --nocapture',
      '  ✓ test_borrow_rules ... ok',
      '  ✓ test_lifetime_elision ... ok',
      '$ writing src/enums_patterns.rs',
      '$ cargo clippy',
      '  Finished — no warnings',
      '$ cargo doc --open',
    ]
  }},
  { id: 't11', label: 'CLI project',   status: 'todo' },
  { id: 't12', label: 'Async chapter', status: 'todo' },
]
```

- [ ] **Step 3: Browser verify**

Open `index.html` in browser. Expected:
- No console errors
- g1 (Ship MVP) now shows 2 orange dots instead of 1 (t2 was already orange; t3 was gray, now orange)
- HUD "done" count unchanged (t3 was todo→in-progress, not done)
- Hovering the 5th dot on Ship MVP shows "Billing — in progress" tooltip

- [ ] **Step 4: Commit**
```bash
git add index.html
git commit -m "feat: extend mock data with agent assignments on 3 tasks"
```

---

## Task 2: Add CSS — Terminal Token, Panel, Terminal Output, Agent Styles

**Files:**
- Modify: `index.html` — `<style>` block (approx lines 10–142)

All additions go **inside** the existing `<style>` tag. Add them just before the closing `</style>`.

- [ ] **Step 1: Add `--terminal-bg` to `:root`**

Find the `:root` block which ends with:
```css
    --s-cancelled:   #cf3c45;
  }
```

Replace with:
```css
    --s-cancelled:   #cf3c45;
    --terminal-bg:   #1e2328;
  }
```

- [ ] **Step 2: Add detail panel CSS**

Find `</style>` and insert the following block immediately before it:

```css
  /* ─── Detail Panel (right sidebar) ─────────────────── */
  #detail-panel {
    position: fixed;
    top: 0; right: 0;
    width: 35%;
    min-width: 320px;
    max-width: 520px;
    height: 100vh;
    background: rgba(45, 53, 59, 0.94);
    border-left: 1px solid rgba(127, 187, 179, 0.22);
    backdrop-filter: blur(20px) saturate(1.4);
    -webkit-backdrop-filter: blur(20px) saturate(1.4);
    padding: 28px 22px 32px;
    overflow-y: auto;
    z-index: 150;
    pointer-events: all;
    transform: translateX(102%);
    transition: transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
    outline: none;
  }
  #detail-panel.open { transform: translateX(0); }

  .panel-header {
    display: flex; align-items: center; justify-content: space-between;
    margin-bottom: 6px;
  }
  .panel-title {
    font-size: 15px; font-weight: 700;
    color: var(--fg); letter-spacing: 0.02em;
  }
  .panel-desc {
    font-size: 10.5px; color: var(--fg-dim);
    line-height: 1.5; margin-bottom: 18px;
  }

  /* ─── Task rows in panel ─────────────────────────────── */
  .panel-task-row {
    border-left: 3px solid transparent;
    padding: 10px 0 10px 14px;
    border-bottom: 1px solid rgba(127, 187, 179, 0.07);
  }
  .panel-task-row:last-child { border-bottom: none; }

  .task-row-header {
    display: flex; align-items: center; gap: 8px; margin-bottom: 4px;
  }
  .task-status-dot {
    width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0;
  }
  .task-row-label {
    font-size: 12px; font-weight: 500; color: var(--fg);
    flex: 1; letter-spacing: 0.02em;
  }

  /* ─── Agent name chip ────────────────────────────────── */
  .agent-name-chip {
    display: inline-flex; align-items: center; gap: 5px;
    font-size: 9px; font-weight: 700; letter-spacing: 0.12em;
    text-transform: uppercase;
    padding: 2px 8px; border-radius: 20px;
    border: 1px solid; flex-shrink: 0;
    /* color and border-color set inline per agent */
  }
  .agent-name-chip::before {
    content: ''; display: block;
    width: 5px; height: 5px; border-radius: 50%;
    background: currentColor; animation: agentPulse 1.4s ease-in-out infinite;
  }
  @keyframes agentPulse {
    0%, 100% { opacity: 1; transform: scale(1); }
    50%       { opacity: 0.4; transform: scale(0.7); }
  }

  /* ─── Terminal output ────────────────────────────────── */
  .terminal-output {
    background: var(--terminal-bg);
    border-radius: 6px;
    padding: 10px 12px;
    font-size: 10px;
    line-height: 1.7;
    font-family: var(--font);
    color: var(--fg);
    max-height: 170px;
    overflow-y: auto;
    margin-top: 8px;
    scrollbar-width: thin;
    scrollbar-color: var(--bg3) transparent;
  }
  .terminal-output::-webkit-scrollbar { width: 4px; }
  .terminal-output::-webkit-scrollbar-thumb {
    background: var(--bg3); border-radius: 2px;
  }
  .terminal-line { white-space: pre-wrap; word-break: break-all; }
  .terminal-line.cmd { color: var(--accent); }
  .terminal-line.ok  { color: var(--green); }

  .terminal-cursor {
    display: inline-block;
    width: 6px; height: 11px;
    background: var(--fg);
    border-radius: 1px;
    vertical-align: text-bottom;
    margin-left: 2px;
    animation: blink 1s step-end infinite;
  }
  @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
```

- [ ] **Step 3: Browser verify**

Refresh. Expect no visual change yet (panel hidden off-screen, styles not applied to any elements). No console errors.

---

## Task 3: Add `#detail-panel` DOM Element

**Files:**
- Modify: `index.html` — HTML body, after `#legend` div (approx line 174)

- [ ] **Step 1: Add the panel element**

Find:
```html
</div>

<script type="importmap">
```

(The closing `</div>` of `#legend`.) Replace with:

```html
</div>

<div id="detail-panel" tabindex="-1" role="complementary" aria-label="Goal detail view"></div>

<script type="importmap">
```

- [ ] **Step 2: Add stopPropagation on panel (decision 1A)**

This is done in JS, not HTML. We'll add it in Task 6 when we initialise the panel. No action here yet.

- [ ] **Step 3: Browser verify**

Open DevTools → Elements. Confirm `#detail-panel` exists in the DOM, is `position: fixed; right: 0`, and is translated off-screen. No console errors.

---

## Task 4: Agent Sprite Factory + allAgents Array + Build During GOALS Loop

**Files:**
- Modify: `index.html` — JS section

- [ ] **Step 1: Add `makeAgentSprite()` factory function**

Find this comment and code block:
```js
// ─────────────────────────────────────────────────────────────────
//  RENDERER + SCENE
// ─────────────────────────────────────────────────────────────────
```

Insert the following **before** that comment:

```js
// ─────────────────────────────────────────────────────────────────
//  AGENT SPRITE FACTORY — canvas-drawn diamond, distinct from ring dots
// ─────────────────────────────────────────────────────────────────
function makeAgentSprite(hexColor, size = 96) {
  const canvas = document.createElement('canvas');
  canvas.width = size; canvas.height = size;
  const ctx = canvas.getContext('2d');
  const cx = size / 2, cy = size / 2;

  // soft radial glow behind diamond
  const grad = ctx.createRadialGradient(cx, cy, 2, cx, cy, size * 0.48);
  grad.addColorStop(0, hexColor + 'bb');
  grad.addColorStop(0.5, hexColor + '44');
  grad.addColorStop(1, hexColor + '00');
  ctx.fillStyle = grad;
  ctx.beginPath(); ctx.arc(cx, cy, size * 0.48, 0, Math.PI * 2); ctx.fill();

  // diamond (rotated square)
  const d = size * 0.2;
  ctx.save();
  ctx.translate(cx, cy);
  ctx.rotate(Math.PI / 4);
  ctx.shadowColor = hexColor;
  ctx.shadowBlur = 10;
  ctx.fillStyle = hexColor;
  ctx.fillRect(-d, -d, d * 2, d * 2);
  // brighter inner highlight
  ctx.globalAlpha = 0.5;
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(-d * 0.35, -d * 0.35, d * 0.7, d * 0.7);
  ctx.restore();

  return new THREE.SpriteMaterial({
    map: new THREE.CanvasTexture(canvas),
    transparent: true,
    depthWrite: false,
    opacity: 1.0,
  });
}

```

- [ ] **Step 2: Declare `allAgents` tracking array**

Find:
```js
const allDots = [];     // { sprite, goalId, task, phaseX/Y/Z, basePos, targetOpacity, currentOpacity }
const allLines = [];    // { line, goalId, targetOpacity, currentOpacity }
const goalCards = {};   // goalId → DOM element
const goalPositions = {};
```

Replace with:
```js
const allDots = [];     // { sprite, goalId, task, phaseX/Y/Z, basePos, targetOpacity, currentOpacity }
const allLines = [];    // { line, goalId, targetOpacity, currentOpacity }
const allAgents = [];   // { sprite, goalId, agentData, taskDotRef, basePos, phase, targetOpacity, currentOpacity, connectorLine }
const goalCards = {};   // goalId → DOM element
const goalPositions = {};
```

- [ ] **Step 3: Build agent sprites inside the existing `tasks.forEach` loop**

Find the end of the inner `tasks.forEach` loop body. Look for the block that ends with `allLines.push(...)`:

```js
    allLines.push({
      line, goalId: goal.id,
      targetOpacity: 0.18,
      currentOpacity: 0.18,
      baseOpacity: 0.18,
    });
  });
});
```

Replace with:

```js
    allLines.push({
      line, goalId: goal.id,
      targetOpacity: 0.18,
      currentOpacity: 0.18,
      baseOpacity: 0.18,
    });

    // ── Agent sprite (if this task has an agent) ──
    if (task.agent) {
      const agentAngle = angle + Math.PI * 0.45;
      const agentRadius = radius + 0.55;
      const agentBasePos = new THREE.Vector3(
        gp.x + Math.cos(agentAngle) * agentRadius,
        gp.y + yOff + 0.32,
        gp.z + Math.sin(agentAngle) * agentRadius,
      );

      const agentMat = makeAgentSprite(task.agent.color);
      const agentSprite = new THREE.Sprite(agentMat);
      agentSprite.scale.setScalar(0.3);
      agentSprite.position.copy(agentBasePos);
      scene.add(agentSprite);

      // thin connector: agent → task dot
      const connGeo = new THREE.BufferGeometry().setFromPoints([
        agentBasePos.clone(), basePos.clone()
      ]);
      const connMat = new THREE.LineBasicMaterial({
        color: new THREE.Color(task.agent.color),
        transparent: true,
        opacity: 0.35,
        depthWrite: false,
      });
      const connLine = new THREE.Line(connGeo, connMat);
      scene.add(connLine);

      const dotRef = allDots[allDots.length - 1]; // the dot we just pushed
      allAgents.push({
        sprite: agentSprite,
        goalId: goal.id,
        agentData: task.agent,
        taskDotRef: dotRef,
        basePos: agentBasePos.clone(),
        phase: Math.random() * Math.PI * 2,
        targetOpacity: 1.0,
        currentOpacity: 1.0,
        connectorLine: connLine,
      });
    }
  });
});
```

- [ ] **Step 4: Browser verify**

Refresh. Expected:
- 3 small diamond shapes visible in the scene (near Ship MVP's Dashboard UI and Billing dots, and near Learn Rust's Chapters 6-10 dot)
- Each diamond is colored: teal (#7fbbb3), purple (#d699b6), yellow (#dbbc7f)
- Thin connector lines from each diamond to its task dot
- All existing interactions still work (orbit, tooltips, card hover)
- No console errors

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "feat: add agent diamond sprites and connector lines to 3D scene"
```

---

## Task 5: Extend Animate Loop — Agent Pulse + Connector Tracking

**Files:**
- Modify: `index.html` — `animate()` function (approx lines 592–638)

- [ ] **Step 1: Add agent animation block to `animate()`**

Find the line in `animate()` that reads:
```js
  // Starfield drift
  stars.rotation.y = t * 0.005;
```

Insert the following block **before** "Starfield drift":

```js
  // Agent pulse + connector endpoint tracking
  allAgents.forEach(a => {
    const pulse = Math.sin(t * 2.1 + a.phase) * 0.04;
    a.sprite.scale.setScalar(0.28 + pulse);

    // Smooth opacity
    a.currentOpacity += (a.targetOpacity - a.currentOpacity) * 0.07;
    a.sprite.material.opacity = a.currentOpacity;

    // Update connector: start = agent (static pos), end = task dot (wobbling)
    const cp = a.connectorLine.geometry.attributes.position;
    cp.setXYZ(0, a.sprite.position.x, a.sprite.position.y, a.sprite.position.z);
    if (a.taskDotRef) {
      cp.setXYZ(1, a.taskDotRef.sprite.position.x,
                   a.taskDotRef.sprite.position.y,
                   a.taskDotRef.sprite.position.z);
    }
    cp.needsUpdate = true;
    a.connectorLine.material.opacity = a.currentOpacity * 0.38;
  });

```

- [ ] **Step 2: Update `setFadeTargets` to include agents**

Find:
```js
function setFadeTargets(focusId) {
  allDots.forEach(d => {
    d.targetOpacity = (!focusId || d.goalId === focusId) ? 1.0 : 0.05;
  });
  allLines.forEach(l => {
    l.targetOpacity = (!focusId || l.goalId === focusId) ? l.baseOpacity : 0.0;
  });
```

Replace with:
```js
function setFadeTargets(focusId) {
  allDots.forEach(d => {
    d.targetOpacity = (!focusId || d.goalId === focusId) ? 1.0 : 0.05;
  });
  allLines.forEach(l => {
    l.targetOpacity = (!focusId || l.goalId === focusId) ? l.baseOpacity : 0.0;
  });
  allAgents.forEach(a => {
    a.targetOpacity = (!focusId || a.goalId === focusId) ? 1.0 : 0.05;
  });
```

- [ ] **Step 3: Browser verify**

Refresh. Expected:
- Agent diamonds pulse (scale up/down) independently of the task dot wobble
- When clicking a goal (e.g. Ship MVP): its 2 agent diamonds stay bright, all other agents and task dots from other goals fade to near-invisible
- Connector lines fade with agents
- Pressing Escape restores full visibility

- [ ] **Step 4: Commit**
```bash
git add index.html
git commit -m "feat: animate agent pulse and fade targets in render loop"
```

---

## Task 6: Timer Registry, Panel Open/Close, Focus Management, Wire Into Focus/Reset

**Files:**
- Modify: `index.html` — JS section after `resetView` function

- [ ] **Step 1: Add timer registry, `clearAllTimers`, and one-time panel click guard**

Find:
```js
renderer.domElement.addEventListener('click', () => { if (focusedGoalId) resetView(); });
window.addEventListener('keydown', e => { if (e.key === 'Escape') resetView(); });
```

Insert the following block **before** those two lines:

```js
// ─────────────────────────────────────────────────────────────────
//  TIMER REGISTRY — prevents ghost intervals on rapid focus changes
// ─────────────────────────────────────────────────────────────────
const activeTimers = [];

function clearAllTimers() {
  activeTimers.forEach(id => { clearTimeout(id); clearInterval(id); });
  activeTimers.length = 0;
}

// Stop panel clicks from propagating to canvas dismiss handler (decision 1A)
// Added once here (not inside openPanel) to avoid accumulating duplicate listeners.
document.getElementById('detail-panel').addEventListener('click', e => e.stopPropagation());

```

- [ ] **Step 2: Add `buildTaskRow` helper**

Directly after the `clearAllTimers` function, add:

```js
// ─────────────────────────────────────────────────────────────────
//  PANEL CONTENT BUILDERS
// ─────────────────────────────────────────────────────────────────
function buildTaskRow(task) {
  const dotColor = STATUS_COLOR[task.status] || '#9da9a0';
  const hasAgent = !!task.agent;
  const borderStyle = hasAgent ? `border-left-color: ${task.agent.color}` : '';
  const agentChip = hasAgent
    ? `<span class="agent-name-chip"
         style="color:${task.agent.color}; border-color:${task.agent.color}; background:${task.agent.color}18"
       >${task.agent.name}</span>`
    : '';
  const terminal = hasAgent
    ? `<div class="terminal-output" id="terminal-${task.agent.id}" aria-live="polite" aria-label="${task.agent.name} terminal output"></div>`
    : '';

  return `
    <div class="panel-task-row${hasAgent ? ' agent-active' : ''}" style="${borderStyle}">
      <div class="task-row-header">
        <div class="task-status-dot" style="background:${dotColor}"></div>
        <div class="task-row-label">${task.label}</div>
        ${agentChip}
      </div>
      ${terminal}
    </div>`;
}

```

- [ ] **Step 3: Add `streamTerminal` function**

After `buildTaskRow`, add:

```js
// ─────────────────────────────────────────────────────────────────
//  TERMINAL STREAMING
// ─────────────────────────────────────────────────────────────────
function streamTerminal(agent, rate) {
  const container = document.getElementById(`terminal-${agent.id}`);
  if (!container) return null;

  let lineIndex = 0;

  function appendNextLine() {
    // loop commands when exhausted
    if (lineIndex >= agent.commands.length) {
      lineIndex = 0;
      container.innerHTML = '';
    }

    const text = agent.commands[lineIndex++];

    // remove old cursor
    container.querySelector('.terminal-cursor')?.remove();

    const el = document.createElement('div');
    el.className = 'terminal-line';
    if (text.trimStart().startsWith('$')) el.className += ' cmd';
    else if (text.includes('✓'))          el.className += ' ok';
    el.textContent = text;
    container.appendChild(el);

    // blinking cursor at end
    const cursor = document.createElement('span');
    cursor.className = 'terminal-cursor';
    container.appendChild(cursor);

    // auto-scroll to bottom
    container.scrollTop = container.scrollHeight;
  }

  appendNextLine(); // show first line immediately on open
  return setInterval(appendNextLine, rate);
}

```

- [ ] **Step 4: Add `openPanel` and `closePanel`**

After `streamTerminal`, add:

```js
// ─────────────────────────────────────────────────────────────────
//  DETAIL PANEL OPEN / CLOSE
// ─────────────────────────────────────────────────────────────────
function openPanel(goalId) {
  const goal = GOALS.find(g => g.id === goalId);
  if (!goal) return;

  clearAllTimers();

  const panel = document.getElementById('detail-panel');

  panel.innerHTML = `
    <div class="panel-header">
      <div class="panel-title">${goal.title}</div>
      <div class="card-badge badge-${goal.status}">${goal.status.replace('-', '\u2009')}</div>
    </div>
    <div class="panel-desc">${goal.description}</div>
    <div class="panel-task-list">
      ${goal.tasks.map(t => buildTaskRow(t)).join('')}
    </div>
  `;

  panel.classList.add('open');

  // Focus management (a11y — decision 4A)
  panel.focus();

  // Start streaming — stagger rates so agents feel independent (decision 5A)
  const agentTasks = goal.tasks.filter(t => t.agent);
  agentTasks.forEach((task, i) => {
    const rate = 110 + i * 35; // 110ms, 145ms, 180ms …
    const intervalId = streamTerminal(task.agent, rate);
    if (intervalId) activeTimers.push(intervalId);
  });
}

function closePanel() {
  clearAllTimers();
  const panel = document.getElementById('detail-panel');
  panel.classList.remove('open');
  // Clear content after slide-out so terminals don't flash on reopen
  setTimeout(() => { if (!panel.classList.contains('open')) panel.innerHTML = ''; }, 380);
}

```

- [ ] **Step 5: Wire `openPanel` into `focusGoal` and `closePanel` into `resetView`**

Find:
```js
function focusGoal(id, gp) {
  if (focusedGoalId === id) { resetView(); return; }
  focusedGoalId = id;

  document.querySelectorAll('.goal-card').forEach(c => c.classList.remove('focused'));
  document.querySelector(`[data-goal-id="${id}"]`)?.classList.add('focused');

  const camOffset = new THREE.Vector3(gp.x, gp.y + 0.5, gp.z + 3.8);
  startTween(camOffset, gp.clone());
  setFadeTargets(id);
  document.getElementById('back-hint').classList.add('visible');
}

function resetView() {
  focusedGoalId = null;
  document.querySelectorAll('.goal-card').forEach(c => c.classList.remove('focused', 'dimmed'));
  startTween(DEFAULT_CAM, DEFAULT_TARGET);
  setFadeTargets(null);
  document.getElementById('back-hint').classList.remove('visible');
  controls.autoRotate = true;
}
```

Replace with:
```js
function focusGoal(id, gp) {
  if (focusedGoalId === id) { resetView(); return; }
  focusedGoalId = id;

  document.querySelectorAll('.goal-card').forEach(c => c.classList.remove('focused'));
  document.querySelector(`[data-goal-id="${id}"]`)?.classList.add('focused');

  const camOffset = new THREE.Vector3(gp.x, gp.y + 0.5, gp.z + 3.8);
  startTween(camOffset, gp.clone());
  setFadeTargets(id);
  document.getElementById('back-hint').classList.add('visible');
  openPanel(id);
}

function resetView() {
  focusedGoalId = null;
  document.querySelectorAll('.goal-card').forEach(c => c.classList.remove('focused', 'dimmed'));
  startTween(DEFAULT_CAM, DEFAULT_TARGET);
  setFadeTargets(null);
  document.getElementById('back-hint').classList.remove('visible');
  controls.autoRotate = true;
  closePanel();
}
```

- [ ] **Step 6: Browser verify**

Refresh. Expected:
- Click "Ship MVP": right panel slides in (~350ms ease). Shows 5 task rows. t2 (Dashboard UI) has teal accent border + "Coder-α" chip. t3 (Billing) has purple border + "Deployer-β" chip. Terminal areas visible under each agent row.
- Click "Learn Rust": panel slides in with Scout-γ on Chapters 6-10.
- Click "Health Ritual" or "Deep Work Habit": panel slides in, no terminal areas (no agents).
- Press Escape: panel slides out. Click background: same.
- Click the same goal card again (re-focus): panel closes (resetView called, then nothing reopens).
- No console errors.

- [ ] **Step 7: Commit**
```bash
git add index.html
git commit -m "feat: detail panel open/close with timer registry and focus management"
```

---

## Task 7: Verify Terminal Streaming Behavior

This task is browser-only verification. No new code. Confirms the streaming added in Task 6's `streamTerminal` works correctly.

- [ ] **Step 1: Verify independent streaming rates**

Click "Ship MVP". Watch both terminal areas. Expected:
- Lines appear at different rates (~110ms vs ~145ms)
- Blinking cursor advances with each new line
- Outputs feel visually independent (different content, different rhythm)

- [ ] **Step 2: Verify auto-scroll**

Wait for each terminal to fill past 8 lines (commands loop). Expected:
- Older lines scroll off the top
- Latest line always visible
- No overflow visible outside the terminal box

- [ ] **Step 3: Verify loop / reset**

After ~90 seconds (commands exhaust and restart). Expected:
- Terminal clears and replays from first command
- No stack overflow, no memory leak (Chrome DevTools Memory tab: heap stable)

- [ ] **Step 4: Verify ghost interval cleanup**

Rapidly click between 3 goals:
1. Click Ship MVP → terminal starts
2. Immediately click Learn Rust → Ship MVP terminal stops, Learn Rust terminal starts
3. Press Escape → all streaming stops

Expected: only the most recently opened goal's terminals are streaming. No duplicate intervals.

- [ ] **Step 5: Verify `aria-live` region**

Open screen reader or check DevTools Accessibility panel. The `#detail-panel` has `role="complementary"` and `aria-label`. Each terminal div has `aria-live="polite"`. This satisfies decision 4A accessibility requirement.

---

## Task 8: Full Regression Check

No code changes. Verify the 7 success criteria from the spec.

- [ ] **SC1:** Open `index.html`. 3D scene renders (goal cards, task dots, starfield). No regressions.

- [ ] **SC2:** Click Ship MVP. Camera tweens. Panel appears. Task list shows agent names. Terminal begins streaming.

- [ ] **SC3:** Ship MVP shows 2 agent terminals streaming independently (Coder-α and Deployer-β at different rates). Both diamond sprites visible near their respective task dots.

- [ ] **SC4:** Press Escape. Panel disappears. Camera returns to overview. Terminal streaming stops.

- [ ] **SC5:** Orbit and zoom work in both focused and unfocused states. Verify by dragging and scrolling during focused state.

- [ ] **SC6:** Visual consistency: panel uses same Everforest palette (`--bg`, `--fg`, `--accent`, `--terminal-bg`), JetBrains Mono typography, frosted glass treatment matching existing goal cards.

- [ ] **SC7:** Hover tooltips still work on task dots in both focused and unfocused states.

- [ ] **Final commit:**
```bash
git add index.html
git commit -m "feat: AI agent activity view — panel, terminals, 3D sprites complete"
```

---

## Quick Reference: Decisions Baked In

| Decision | Where implemented |
|----------|-------------------|
| 1A — `stopPropagation()` on panel | `openPanel()` — Task 6 Step 4 |
| 2A — Right sidebar ~35% width | CSS `#detail-panel` — Task 2 Step 2 |
| 3A — `--terminal-bg: #1e2328` | `:root` CSS token — Task 2 Step 1 |
| 4A — `tabindex` + `focus()` on open | `panel.focus()` in `openPanel()` — Task 6 Step 4 |
| 5A — `activeTimers` registry | `clearAllTimers()` in `resetView()` — Task 6 Step 1 |
| 6A — Agent card with color-accent border + name chip | `.panel-task-row` + `.agent-name-chip` CSS + `buildTaskRow()` — Tasks 2 & 6 |
