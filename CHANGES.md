# Alterações da Sessão

## agent/game/input.py

- Implementado **hold click real**: `press_hold` traz o WYD para primeiro plano via `AttachThreadInput + SetForegroundWindow`, posiciona o cursor e envia `LEFTDOWN` sem soltar
- `maintain_hold` reenvia `LEFTDOWN` a cada tick; re-foca o WYD se outra janela tiver roubado o foco entre ticks
- `release_hold` envia `LEFTUP`, restaura cursor e janela anterior ao foco
- Adicionados campos de instância `_prev_fg` e `_orig_cursor` para preservar estado entre press/maintain/release
- `_client_to_screen` agora usa `self._hwnd` (janela principal do WYD) em vez de `self._render_hwnd` — consistente com o sistema de captura que usa `MainWindowHandle` no PowerShell

## backend/src/routes/agent.ts

- **Destino final checado somente após todos os waypoints concluídos** (estava sendo checado antes, o que causava parada prematura se o personagem passasse perto do destino durante a rota)
- Adicionada guarda `!dirs?.xPlus` — retorna IDLE se os cliques direcionais não estiverem configurados no banco (evita crash)
- Logging mais limpo nos `console.log` de navegação

## desktop/src/main/ipc.ts

- Registrado atalho global **F11** via `globalShortcut.register("F11", ...)` que chama `stopAgent()` e emite `agent:stopped` para o renderer — funciona mesmo com o WYD em primeiro plano

## agent/injector/ (criado mas não utilizado em produção)

Tentativa de background input via DLL injection — abordagem abandonada pelo usuário:

- `wyd_input.c` — DLL 32-bit que hookeia `GetCursorPos` e `GetAsyncKeyState` em user32.dll via inline hook x86 (5-byte JMP), com servidor de named pipe `\\.\pipe\wyd_input_{PID}`
- `__init__.py` — módulo Python para injetar a DLL via `CreateRemoteThread + LoadLibraryA` e enviar comandos de clique pelo pipe
- `build.bat` — script de compilação com MinGW32 (MSYS2)
- `wyd_input.dll` — DLL compilada (32-bit, x86)

> Windows Defender sinalizou o `.pyc` do injector como vírus (comportamento esperado para código de DLL injection). A abordagem foi descartada em favor de foco com janela ativa.

## Decisão de Arquitetura

Background input (clique sem janela em foco) foi descartado após explorar todas as alternativas viáveis:

| Abordagem | Resultado |
|---|---|
| PostMessage / SendMessage (main hwnd) | WYD ignora — usa DirectInput/Raw Input |
| PostMessage / SendMessage (Static child hwnd) | Idem |
| AttachThreadInput + SendInput | SendInput requer janela em foco |
| BlockInput + SetForegroundWindow + SendInput | Não funcionou |
| DLL injection (hook GetCursorPos + GetAsyncKeyState) | Não foi testado em produção — abandonado pelo usuário |

**Solução adotada:** uma conta por vez, com foco ativo na janela do WYD durante a navegação (hold click com SetForegroundWindow).
