# WYD Automation Platform

## Visão Geral

Plataforma SaaS de automação para WYD Global. Toda regra de negócio fica no backend. O cliente local (Agent) atua apenas como observador e executor — nunca toma decisões.

---

## Arquitetura

```
┌─────────────────────────────────────────────────────┐
│                   Desktop App (Electron)             │
│  Login │ Dashboard │ Config (endereços Cheat Engine) │
│                                                      │
│  spawna Agent Python como processo filho             │
│  lê stdout do Agent (JSON logs) e exibe na UI        │
└───────────────┬──────────────────┬───────────────────┘
                │ HTTP             │ IPC (child_process)
                ▼                  ▼
┌───────────────────┐   ┌──────────────────────────────┐
│  Backend (Hono)   │   │  Agent (Python)              │
│  PostgreSQL       │◄──│  ReadProcessMemory (X, Y)    │
│  Drizzle ORM      │   │  Detecção de morte (delta≥50)│
│  Regras de negócio│──►│  PostMessage (KEY/CLICK/WAIT)│
└───────────────────┘   └──────────────────────────────┘
```

---

## Componentes

### 1. Agent (Python)

**Localização:** `agent/`

**Responsabilidades:**
- Lê coordenadas X e Y via `ReadProcessMemory` em endereços fornecidos pelo usuário
- Detecta morte por variação abrupta de coordenadas (delta ≥ 50 unidades)
- Executa ações no jogo via `PostMessage` (sem precisar de foco na janela)
- Reporta estado ao backend a cada tick
- Emite logs em JSON no stdout para o Electron consumir

**Execução:**
```bash
python main.py --pid 1234 --character NomeDoChar
```

**Variáveis de ambiente (`.env`):**
```
BACKEND_URL=http://localhost:3000
LICENSE_TOKEN=dev-token-123
TICK_INTERVAL_MS=1000
LOG_LEVEL=DEBUG
```

**Estrutura:**
```
agent/
├── main.py              # loop principal
├── logger.py            # formatter JSON para stdout
├── config.py            # lê .env
├── requirements.txt
├── setup.bat            # cria venv e instala deps
├── game/
│   ├── process.py       # OpenProcess + ReadProcessMemory
│   ├── reader.py        # lê X/Y, detecta morte
│   └── input.py         # PostMessage executor
├── api/
│   └── client.py        # fetch_config() + tick()
└── models/
    ├── agent_config.py  # addrX, addrY
    └── objective.py     # Objective + ObjectiveType
```

**Fluxo do loop:**
```
ReadProcessMemory(addrX, addrY)
    → GameState { x, y, isDead }
    → POST /agent/tick
    → Objective { type, payload }
    → execute_macro(steps)
    → PostMessage → WYD.exe
```

**Detecção de morte:**
```python
delta = abs(new_x - prev_x) + abs(new_y - prev_y)
is_dead = delta >= 50
```

**Tipos de ação suportados (V1):**
- `KEY` — pressionar tecla via `WM_KEYDOWN` / `WM_KEYUP`
- `CLICK` — clique do mouse via `WM_LBUTTONDOWN` / `WM_LBUTTONUP`
- `WAIT` — aguardar N milissegundos

---

### 2. Backend (Hono + PostgreSQL + Drizzle)

**Localização:** `backend/`

**Variáveis de ambiente (`.env`):**
```
DATABASE_URL=postgresql://wyd:wyd@localhost:5432/wyd_automation
PORT=3000
```

**Endpoints:**

| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/auth/validate` | Valida token + personagem (usado no login do Desktop) |
| `GET`  | `/agent/config` | Retorna addrX, addrY, characterName para o Agent |
| `POST` | `/agent/tick` | Recebe GameState, retorna Objective (V1: sempre IDLE) |
| `POST` | `/agent/error` | Recebe erros do Agent para telemetria |
| `PUT`  | `/characters/:name/memory-config` | Salva addrX e addrY vindos do Cheat Engine |

**Auth:** `Authorization: Bearer <token>` + `X-Character: <nome>` em todos os endpoints do Agent.

**Schema do banco:**
```
licenses
  id, token, expires_at, max_characters, active, created_at

characters
  id, license_id, nickname, active, addr_x, addr_y, created_at

agent_logs
  id, character_id, x, y, is_dead, created_at
```

**Planos:**
- Starter: 1 personagem
- Premium: 3 personagens

**Estrutura:**
```
backend/
├── src/
│   ├── index.ts              # entry point
│   ├── db/
│   │   ├── index.ts          # conexão Drizzle
│   │   └── schema.ts         # tabelas
│   ├── middleware/
│   │   └── auth.ts           # valida licença + personagem
│   ├── routes/
│   │   ├── auth.ts
│   │   ├── agent.ts
│   │   └── characters.ts
│   └── seed.ts               # dados de teste
├── drizzle.config.ts
└── package.json
```

**Scripts:**
```bash
npm run dev        # servidor em modo watch
npm run db:push    # aplica schema no banco
npm run db:seed    # cria licença e personagem de teste
```

**Dados de teste (seed):**
- Token: `dev-token-123`
- Personagem: `TestChar`

---

### 3. Desktop App (Electron + React + TypeScript)

**Localização:** `desktop/`

**Variáveis de ambiente (`.env`):**
```
VITE_BACKEND_URL=http://localhost:3000
```

**Páginas:**

| Página | Função |
|--------|--------|
| Login | Token de licença + nome do personagem |
| Dashboard | Seleciona instância WYD, start/stop Agent, visualiza logs em tempo real |
| Config | Cola endereços X e Y encontrados no Cheat Engine |

**Fluxo do Agent no Electron:**
```typescript
// main process spawna o Agent
const agent = spawn("python", ["main.py", "--pid", pid, "--character", name], {
  env: { LICENSE_TOKEN, BACKEND_URL }
})

// lê JSON do stdout e envia para o renderer via IPC
agent.stdout.on("data", (chunk) => {
  mainWindow.webContents.send("agent:log", JSON.parse(chunk))
})
```

**IPC exposto via `window.api`:**
```typescript
window.api.listWydInstances()     // lista processos WYD.exe
window.api.startAgent(options)    // spawna o Agent
window.api.stopAgent()            // mata o processo
window.api.onAgentLog(cb)         // recebe logs em tempo real
window.api.onAgentStarted(cb)
window.api.onAgentStopped(cb)
```

**Estrutura:**
```
desktop/
├── src/
│   ├── main/
│   │   ├── index.ts      # cria janela Electron
│   │   ├── agent.ts      # spawn/stop/gerencia Agent
│   │   └── ipc.ts        # handlers ipcMain
│   ├── preload/
│   │   └── index.ts      # contextBridge → window.api
│   └── renderer/src/
│       ├── App.tsx
│       └── pages/
│           ├── Login.tsx
│           ├── Dashboard.tsx
│           └── Config.tsx
└── package.json
```

---

## Setup Local

### Pré-requisitos
- Node.js 20+
- Python 3.11+
- Docker Desktop

### Primeira vez

```bash
# 1. Banco de dados
docker compose up -d

# 2. Dependências Node
npm install && npm run install:all

# 3. Schema + seed
npm run db:push && npm run db:seed

# 4. Dependências Python
cd agent && setup.bat
```

### Rodando

```bash
# Terminal 1 — banco (se não estiver rodando)
docker compose up -d

# Terminal 2 — backend + desktop
npm run dev
```

---

## Decisões Técnicas

| Decisão | Escolha | Motivo |
|---------|---------|--------|
| Leitura de estado do jogo | `ReadProcessMemory` | Sem injeção, baixo footprint, WYD é antigo |
| Execução de ações | `PostMessage` | Funciona sem foco na janela, suporta múltiplas instâncias |
| Detecção de morte | Delta de coordenadas ≥ 50 | Simples, sem dependências extras |
| Endereços de memória | Dinâmicos, via Desktop App | Usuário cola do Cheat Engine; endereços mudam a cada sessão |
| Logs do Agent | JSON no stdout | Electron lê via `child_process` e exibe na UI |
| Frontend | Electron (era Tauri na spec) | Mudança de decisão durante o desenvolvimento |
| Deploy | Railway / Fly.io | Simples, PostgreSQL incluso |

---

## Roadmap

### V1 (atual)
- [x] Agent: ReadProcessMemory + PostMessage + detecção de morte
- [x] Backend: licenciamento, config de memória, tick (IDLE)
- [x] Desktop: login, dashboard com logs, configuração de endereços
- [ ] Endereços de memória validados com Cheat Engine no WYD Global
- [ ] Macro "Voltar para spot"

### V2
- [ ] Editor de rotas (waypoints)
- [ ] Perfis de automação
- [ ] Objetivos configuráveis no backend

### V3
- [ ] Painel administrativo
- [ ] Marketplace de rotas
- [ ] Estatísticas e dashboard
