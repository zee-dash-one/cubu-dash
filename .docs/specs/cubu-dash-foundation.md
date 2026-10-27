> **Status (27/10/2026, BRT): aprovada; decisões finais.** Plano do M0 aprovado pelo Hil em 27/10 19:45 BRT (commit direto na main, sem PR). Hil aceitou as recomendações em 12/10 ("Aceito tuas recomendações, pode começar o M0").
> **Decidido:** D1 (a) sem backend: guest local + conta do portal (Poki/CrazyGames) atrás do `AuthPort` (Poki proíbe tela obrigatória de login e login por e-mail/Google/Facebook) · D3 (a) Studio in-game só em builds dev, feature tag `studio` · D5 (a) JSON em `res://content/` · D6 (a) cosmético comprado vence pack sazonal · D7 (a) 4 mundos × 4 fases · história em inglês · **transição = 0,35 s** (constante única configurável; alinhar `RULES.md` e o comentário de 400 ms) · **testes = GUT** (confirmado 27/10 19:36 BRT; se o GUT não rodar no Godot 4.7, o agente local pode usar outro framework que funcione e registra a escolha na tarefa).
> **Hipóteses:** D2 (a) ranking só local · D4 (d) cards de texto + ilustração antes da fase · D8 fora do M0.
> **Regra 3.9 (27/10):** nada de agentes de nuvem; todo trabalho de código é local. Onde este texto diz "agentes de nuvem", leia "agente local".

# Cubu Dash (Godot) — Spec de Fundação (v0 · para aprovação)

_09/10/2026 (BRT). Escopo: tudo o que o jogo precisa **antes** do refinamento do gameloop. **Nenhum código, PR ou tarefa no ClickUp foi criado.**_
_Fontes: `hilderney/cubu-dash-godot@main` (lido via API do GitHub), `hilderney/cubu-dash` (só referência), `/workspace/cubu-dash-plano.md` e `/workspace/cubu-dash-backlog.md`._
_Convenção: **hipótese** = suposição minha, a confirmar. Identificadores, código e textos de UI ficam em **inglês** (regra do `RULES.md`)._

---

## 1. Estado atual (resumo)

**Base técnica:** Godot 4.7, GL Compatibility, viewport 1280×720 (`canvas_items`). A cena principal é `scenes/app/MainMenu.tscn`.

| Área | O que existe | Arquivo(s) | Reaproveitar? |
|---|---|---|---|
| Autoloads | Na ordem: `EventBus`, `AudioManager`, `GameManager`, `PlayerStats`, `SaveLoad`, `InputManager` | `project.godot`, `autoload/*.gd` | Sim |
| Navegação | `GameManager.change_scene(path)` = `change_scene_to_file` + `EventBus.scene_changed`. Paths em constantes `SCENE_*`. Sem pilha de "voltar" nem parâmetros: o estado passa por variáveis globais (`current_world`, `current_phase`, `is_dev_lab_run`, `pending_screen_title`) | `autoload/GameManager.gd` | Encapsular num Router |
| Flag de dev | `const IS_DEV_BUILD := true` (troca manual) mostra o Debugger no menu | `GameManager.gd`, `main_menu.gd` | Trocar por feature tag de export |
| EventBus | `player_dashed`, `player_dash_ended`, `player_jumped`, `player_died`, `coin_collected(amount)`, `scene_changed(path)` | `autoload/EventBus.gd` | Ampliar |
| Save | Port + Adapter + Factory: `SavePort` → `FileSaveAdapter` (`user://save.json`) ← `SaveFactory.make_default()`. A fachada `SaveLoad` guarda um único `Dictionary` e **reescreve o arquivo a cada `set_value`**. Só a chave `"input"` é usada | `save/*`, `autoload/SaveLoad.gd` | Sim + schema versionado |
| Input | Port + Adapter + Factory completo: 4 esquemas exclusivos (keyboard, mix, gamepad, touch), remap, overlay de touch e persistência | `input/*`, `autoload/InputManager.gd`, `scenes/app/ControlSettings.tscn` | Sim, sem mudanças |
| Áudio e stats | `AudioManager.gd` e `PlayerStats.gd` são **stubs vazios e idênticos** (mesmo blob) | `autoload/` | Substituir |
| Menu | Start, Sound, Graphic, Controls, Debugger. Sound e Graphic abrem `UnderConstruction` | `scenes/app/main_menu.gd` | Virar Tela Inicial |
| Seleção | Anel tipo relógio Mundo → Fase, com spin/hold/fila e transições `zoom_in/out` e `dive_in/out`, polidos. **Os mundos estão hardcoded em `_build_world_data()`:** 4 mundos (NEON CUBES, PIXEL PEAKS, VOLT VALLEY, CANDY CORE) × 4 fases "PHASE w-p". Lock fixo: só o mundo 1 e a fase 1 abrem. Ao confirmar, grava `GameManager.current_world/phase` e abre `Game.tscn` | `scenes/select/level_select.gd`, `carousel_item.gd` | Sim. A fonte de dados passa a ser o ContentRepository |
| Run | `game.gd` rola o `World` com o player travado em X. **Ignora `current_world/phase`.** Fora do lab, o chão é um retângulo fixo de 4000 px, sem kill zone e sem linha de chegada (**hipótese:** a run normal não termina). Morte no lab recarrega a cena; fora do lab, volta ao menu. Não há tela de resultado | `scenes/run/game.gd`, `Game.tscn` | Refatorar para `RunRequest` |
| Lab dev | `DevLabCatalog.build(world)`: chão com buracos, kill zone e 8 marcadores (A JUMP…H CLEAR), só visuais | `stages/dev/phase_00_lab.gd`, `scenes/run/kill_zone.gd` | Vira um preset do Sandbox |
| Tuning | `DevTuningPanel` edita em runtime os `@export` do Player e do Game | `scenes/hud/dev_tuning_panel.gd` | Base do Sandbox |
| Player | Visual = `ColorRect` (sem sprite), pulo duplo + dash, squash/stretch | `scenes/run/player.gd` | Visual passa a usar "slots" de tema |
| Pausa | Resume / Main Menu | `scenes/run/pause_menu.gd` | Sim |
| Conteúdo | `stages/` só tem o lab. `audio/` vazio. `assets/` só tem a fonte Press Start 2P | — | — |

**Observações:**
- `RULES.md` define transições de **800 ms**, mas o código usa `TRANSITION_MS = 0.35`, e um comentário diz "400ms". Confirmar o valor correto.
- `RULES.md` cita `.docs/idtcs-pattern.md`, `.docs/reactive-sounds.md` e `.docs/hitstop.md`, mas `.docs/` está no `.gitignore`. **Agentes de nuvem não vão ver esses specs.**

**Contratos do web (`hilderney/cubu-dash`) para portar como referência:**
- `src/shared/types.ts`:
  - `UserSession {userId, displayName, isGuest}`
  - `AuthRepository` (`signInGuest`, `signInEmail`, `signUp`, `signOut`). O e-mail é stub e delega para guest.
  - `LeaderboardEntry {name, timeMs, date}` + períodos `weekly|monthly|allTime`
  - `ShopItem {id, name, priceSoft}`, `PurchaseResult {ok, reason}`
  - `ProgressRow {levelId, completed, bestTime, stars}`, `DailyStreakState`
  - `LevelDefinition {…, enabled, themeId, screens, problems, sprites, props, audio}`
  - `ThemeDefinition {palette, bpm, musicOffsetMs, music}`, `AssetRef {kind, key, uri}`
  - `LevelCatalogRepository` (`getAll`/`upsert`/`remove` = CRUD)
- `src/adapters/persistence/indexeddb.ts`: stores `profile`, `settings`, `progress`, `currency`, `inventory {ownedItemIds, equippedSkinId, equippedTrailId}`, `leaderboard`, `levelCatalog`, `dailyStreak`, `session`, `worldSelect`.
- `src/contracts/`: `IAchievementsProvider.unlock(id)`, `IPlatformSDK` (`init`, `gameplayStart/Stop`, `commercialBreak`, `happyTime`), `IAdsProvider`.
- `src/assets/levels/seed.json`: 3 temas (Nebulosa 128, Cristal 140, Vazio 152 BPM), 4 galáxias, 3 sistemas e **3 levels**, em formato `screens`/`problems` por beat.

---

## 2. Arquitetura proposta

### 2.1 Princípios
1. **Conteúdo orientado a dados:** mundos, fases, histórias, conquistas, cosméticos, packs e configurações de loop são **dados**, não código. O código só lê através de um Port.
2. **Port + Adapter + Factory** (`RULES.md`) para tudo que pode trocar de backend: conteúdo, auth, ranking, loja e temas remotos.
3. **Separação clara:**
   - **Telas do jogador**, que vão para produção.
   - **Studio** (ferramentas de admin/dev), que **nunca** entra no build de release enquanto não houver papel de admin validado no servidor.
4. **Uma única cena de run** (`Game.tscn`) para Lab, Sandbox e Gameloop Oficial. O que muda é o `RunRequest`.

### 2.2 Modelo de conteúdo (classes `Resource` tipadas + JSON canônico)

As classes `class_name … extends Resource` têm `from_dict()` / `to_dict()` / `validate()`. O formato canônico em disco é **JSON em `res://content/`**:
- dá diff legível nos PRs;
- é fácil para os agentes escreverem;
- tem paridade com o `seed.json` do web e com um backend futuro.

`.tres` fica como alternativa (ver decisão D5).

| Tipo | Campos principais |
|---|---|
| `WorldDef` | `id`, `name`, `order`, `enabled`, `theme_id`, `color`, `unlock_rule {type: always\|clear_phase\|collectibles, ref, amount}`, `phase_ids[]` |
| `PhaseDef` | `id`, `world_id`, `order`, `enabled`, `name`, `bpm`, `chart_ref`, `duration_target_ms`, `collectibles_total` (3), `unlock_rule`, `story {intro_beat_id?, outro_beat_id?}`, `ranking_enabled`, `loop_config_id` |
| `StoryBeat` | `id`, `phase_id`, `trigger: on_enter\|on_first_clear\|on_clear`, `format: cards\|dialogue\|cutscene`, `pages[{speaker?, portrait?, background?, text_key}]`, `skippable`, `play_once` |
| `AchievementDef` | `id`, `name_key`, `desc_key`, `icon`, `hidden`, `enabled`, `metric` (nome de evento, ex. `phase_cleared`, `collectible_picked`, `death`), `filter {world_id?, phase_id?}`, `target` (int), `reward_soft`, `platform_ids {steam?, play?}` |
| `CosmeticItem` | `id`, `type: skin\|trail\|…`, `name_key`, `price_soft`, `rarity`, `enabled`, `available {from?, until?}`, `acquire: shop\|achievement\|default`, `slot_overrides {slot: res_path}` |
| `ThemePack` | `id`, `name`, `enabled`, `priority`, `schedule {mode: always\|manual\|date_range, start "MM-DD", end "MM-DD"}`, `slot_overrides {slot: res_path}` |
| `GameLoopConfig` | `id`, `version`, `locked`, valores de feel (`base_scroll_speed`, `dash_scroll_multiplier`, `jump_force`, `gravity`, `max_jumps`, `dash_duration`, `coyote_ms`…) e `rules {fall_kills, retry_ms_target, …}` |

**Ports e serviços:**
- **`ContentPort`:** `get_worlds()`, `get_world(id)`, `get_phase(id)`, `get_story_beat(id)`, `list(kind)`, `upsert(kind, def)`, `remove(kind, id)`, `validate_all() -> Array[String]`.
- **Adapters de conteúdo:**
  - `BundledJsonContent`: lê `res://`, só leitura (release).
  - `StudioFileContent`: escreve em `res://content/` quando roda pelo editor e em `user://content_override/` quando exportado, com Export/Import JSON.
  - `RemoteContent`: futuro (live-ops via backend).
- **`ContentRepository`:** autoload-fachada que cacheia, resolve referências e roda `validate_all()` no boot (dev).
- **`PlayerState`:** fachada sobre o `SavePort` existente. Schema versionado: `{version, profile, progress{phase_id:{cleared, best_ms, collectibles[], clears}}, story{seen[]}, wallet{soft}, inventory{owned[], equipped{skin, trail}}, achievements{id:{progress, unlocked_at}}, leaderboard_local{phase_id:[…]}, settings, input}`.
  - Migra o save atual, que só tem `input`.
  - Salva com debounce, e não a cada `set_value`.

### 2.3 Roteamento (`Router`)

Novo autoload `Router` com:
- tabela `route_id → scene_path`;
- `go(route, params := {})`, `back()` com pilha;
- `current_params()`;
- guards (`requires_admin`, `requires_feature("studio")`);
- emissão de `EventBus.scene_changed` (o `InputManager` já escuta).

`GameManager.change_scene` / `go_*` viram wrappers finos (compatibilidade). O `GameManager` fica só com estado de run.

**Rotas:**
- **Jogador:** `title`, `login`, `select`, `story`, `run`, `results`, `ranking`, `shop`, `achievements`, `settings_*`.
- **Studio:** `studio`, `studio/narrative`, `studio/achievements`, `studio/shop`, `studio/themes`, `sandbox`.
- `official_loop` = `run` com `RunRequest.mode = official`.

**Contratos da run:**
- `RunRequest {phase_id, mode: official|sandbox|lab, loop_config_id, modifiers{}}`
- `RunResult {phase_id, mode, outcome: cleared|died|quit, time_ms, collectibles[], coins, new_best}`

**Sinais novos no EventBus:** `run_started(req)`, `run_finished(result)`, `collectible_picked(phase_id, index)`, `achievement_unlocked(id)`, `currency_changed(balance)`, `theme_changed(pack_id)`, `session_changed(session)`, `content_changed(kind)`.

### 2.4 Admin: Studio in-game × plugin de editor

| Opção | Prós | Contras |
|---|---|---|
| **A. Studio in-game** (rotas `studio/*`, só com feature tag `studio` + papel admin) | Roda em qualquer build dev (PC, web, celular). Testa o conteúdo no runtime real (preview de tema e de história). Telas Control simples, boas para agentes de nuvem. Vira live-ops quando houver backend | Mais UI para construir. Precisa de adapter de escrita. Risco de vazar para o release (mitigado pelo filtro de export) |
| **B. Plugin de editor** (`addons/cubu_studio`, `@tool` + docks) | Zero risco no release. Usa o Inspector (CRUD de `.tres` quase de graça) | Só funciona no editor desktop. Não testa no celular nem no web. `@tool` é mais frágil. Não evolui para live-ops |
| C. Ambos | Máxima flexibilidade | Custo dobrado |

**Recomendação: A.**
- Studio in-game, com o código em `res://studio/`, **excluído dos presets de release** (filtro de export) e gated por `OS.has_feature("studio") and Session.is_admin()`.
- O botão Debugger atual vira a entrada do Studio.
- O plugin de editor fica só como upgrade opcional, se a edição de charts pedir isso.

### 2.5 ThemePack (troca sazonal de sprites e áudio)

**Slots semânticos.** Os nós visuais não apontam para arquivos, e sim para chaves:
- `player.body`, `player.trail`
- `world.ground`, `world.bg_far`, `world.hazard.spike`
- `ui.title_bg`
- `sfx.jump`, `music.world.<id>`

Um componente `ThemeSlot`, nos `Sprite2D` / `TextureRect` / `AudioStreamPlayer`, pede `ThemeService.resolve(slot)` e reage a `theme_changed`.

**Ordem de resolução (hipótese, ver D6):** cosmético equipado (só slots `player.*`) → pack sazonal ativo de maior `priority` → tema do mundo (`WorldDef.theme_id`) → pack `default`.

**Agenda:**
- `schedule.date_range` é avaliado na data local no boot e a cada volta à Tela Inicial.
- O Studio tem "simular data" para preview.
- `manual` = liga e desliga no Studio.

**Tamanho no web (Poki < 8 MB):**
- packs grandes podem virar `.pck` separado carregado sob demanda (`ProjectSettings.load_resource_pack`);
- o pack `default` vai embutido.

**Fallback:** um slot sem asset cai no `ColorRect`/cor atual. Isso permite construir todo o sistema antes de existir arte.

---

## 3. Auth (login e validação de usuário)

**Restrições reais das plataformas:**
- **Poki:** jogar sem conta é obrigatório.
  - Existe um módulo **User Accounts** opcional (`PokiSDK.login()` só após ação do usuário, `getUser()`, `getToken()` = JWT verificável no backend).
  - **É proibido** ter login próprio por e-mail, Google ou Facebook.
  - Sistema de conta próprio só com username (com filtro de palavrões) + senha, sem dados pessoais, e com aprovação de CSP/privacidade.
  - Fontes: developers.poki.com/guide/accounts e /external-resources-policy.
- **CrazyGames:** módulo `user`:
  - `isUserAccountAvailable`, `getUser()`, `showAuthPrompt()`, `getUserToken()` (JWT de 1 h, verificar no servidor com a chave pública);
  - há suporte a Godot 4 (`CrazyGames.User.get_user_async()`).
  - Fonte: docs.crazygames.com/sdk/user.
- **Android e Steam:** o login vem do backend escolhido (ou Play Games / Steam ID).

**Contrato `AuthPort`:**
- `current_session() -> Session`
- `sign_in_guest(display_name)`
- `sign_in(provider, payload)`
- `sign_out()`
- `get_token() -> String` (para o backend)
- `is_admin() -> bool`

`Session {user_id, display_name, is_guest, provider: local|poki|crazygames|backend, roles[], linked_ids{}}`.

**Adapters:**
1. `LocalGuestAuth` (**agora**): gera um UUID no `PlayerState.profile` e salva o nome.
2. `DevAdminAuth` (**agora**): só existe com a feature `studio`. Dá o papel `admin` ao perfil local (PIN opcional; **não é segurança**, é só para evitar clique acidental).
3. `PokiAuth` / `CrazyGamesAuth` (com o `PlatformPort`, via JavaScriptBridge): lê o usuário do portal e vincula `linked_ids`.
4. `BackendAuth` (decisão D1): Supabase, Firebase, PlayFab ou Nakama.

**"Validação de usuário" na prática:**
- **No cliente:** nome com 3–16 caracteres `[A-Za-z0-9_.]` e filtro de palavrões (exigência da Poki para username).
- **Identidade:** só é validada no **servidor** (verificar o JWT do portal ou o token do backend). Nada do cliente é confiável para ranking ou papel de admin.
- Por isso, **admin em produção só existe depois de haver backend com roles**. Até lá, o Studio só existe em builds dev.

**Opções de backend** (nenhuma pesquisada a fundo; preço e maturidade dos SDKs Godot são **hipótese**):

| Backend | Para quê serve bem | Custo / risco |
|---|---|---|
| Supabase | Postgres + Auth + RLS. Ranking via SQL. REST pelo `HTTPRequest` | Tier grátis. SDK Godot comunitário |
| Firebase | Auth + Firestore. Integra com Android | SDK Godot comunitário. Regras de segurança custam esforço |
| PlayFab | Feito para jogos: leaderboards, economia, title data (conteúdo remoto) | Lock-in na Microsoft |
| Nakama | Servidor de jogo open-source, client Godot oficial | Self-host = infraestrutura para manter |

**Caminho que não bloqueia nada:** `LocalGuestAuth` + `DevAdminAuth` agora. Todo o resto do jogo depende só do `AuthPort`. O backend entra depois, como um adapter novo.

---

## 4. Telas e sistemas

> Formato: **Propósito · Dados · Fluxos · Depende de · Aceite**. Telas do jogador (P) e do Studio (S).

### P1. Tela Inicial (Title)
- **Propósito:** porta de entrada. Evolui o `MainMenu.tscn` (mantém a animação de intro e os cubos flutuantes).
- **Dados:** `Session`, `wallet.soft`, pack sazonal ativo (fundo `ui.title_bg`).
- **Fluxos:** Play → Seleção · Shop · Ranking · Achievements · Settings (Sound, Graphic, Controls) · Profile · **Studio** (só dev+admin).
- **Depende de:** Router, Session, ThemeService.
- **Aceite:**
  - funciona sem login (guest automático);
  - o botão Studio não aparece em preset de release;
  - o tema sazonal muda o fundo sem reiniciar.

### P2. Login / Perfil
- **Propósito:** criar e editar o perfil guest; mais tarde, vincular a conta do portal/backend.
- **Dados:** `Session`, `profile {id, display_name, created_at}`.
- **Fluxos:** 1º boot → nome opcional (botão "skip" gera "Cubu-XXXX") · editar nome · "Link account" (só aparece se houver adapter online) · sign out.
- **Depende de:** `AuthPort`, `PlayerState`.
- **Aceite:**
  - nunca bloqueia o jogo (regra da Poki);
  - nome validado (tamanho, charset, filtro);
  - persiste entre sessões.

### P3. Seleção de Mundos e Fases
- **Propósito:** o anel atual, agora alimentado por dados.
- **Dados:** `WorldDef` / `PhaseDef` (`enabled`, `order`, `unlock_rule`) + `progress` (cleared, best_ms, collectibles x/3).
- **Fluxos:**
  - lista só os itens `enabled`, ordenados por `order`;
  - lock calculado pela `unlock_rule` + progresso;
  - confirmar fase → story `on_enter` (se houver) → run.
- **Depende de:** ContentRepository, PlayerState, Router.
- **Aceite:**
  - `_build_world_data()` removido, com o mesmo visual e o mesmo comportamento do anel (regras do `RULES.md` intactas);
  - desligar um mundo no Studio faz ele sumir do anel;
  - os badges de progresso aparecem.

### P4. Apresentação de História (Story Player)
- **Propósito:** mostrar o trecho da história ligado à fase.
- **Dados:** `StoryBeat` (`format`, `pages`, `trigger`), `story.seen[]`.
- **Fluxos:**
  - clear → `on_first_clear` (ou `on_clear`) → Story Player → Results / próxima fase;
  - `skippable`;
  - "rever história" disponível depois.
- **Depende de:** ContentRepository, PlayerState, RunResult.
- **Aceite:**
  - o formato `cards` funciona (texto + fundo + speaker);
  - `play_once` é respeitado;
  - o input segue as `InputActions`;
  - som reativo marcado com `TODO: AUDIO`.

### P5. Game Over / Resultado
- **Propósito:** fechar a run (morte ou clear) e permitir retry imediato.
- **Dados:** `RunResult`, `best_ms`, moedas ganhas, conquistas desbloqueadas na run.
- **Fluxos:**
  - **Morte:** Retry (mesma fase, meta < 200 ms, sem voltar ao menu) · Select · Title.
  - **Clear:** Story (se houver) → Next phase · Retry · Ranking.
  - Ponto de inserção futuro para rewarded e intersticial (`AdsPort`, do plano).
- **Depende de:** RunRequest/RunResult, PlayerState, Achievements.
- **Aceite:**
  - a morte deixa de mandar para o menu (substitui o comportamento atual de `game.gd`);
  - o novo recorde é destacado;
  - os dados de resultado ficam salvos.

### P6. Ranking
- **Propósito:** melhores tempos por fase.
- **Dados:** `LeaderboardEntry {name, time_ms, date}` (porte do web) + período `weekly|monthly|all_time`.
- **Fluxos:** escolher mundo/fase → top N + "você" · aba local × online (online só existe com um adapter online).
- **Depende de:** `LeaderboardPort` (`LocalLeaderboard` agora), RunResult, Session.
- **Aceite:**
  - funciona 100% offline;
  - fases com `ranking_enabled=false` não aparecem;
  - a troca para online é só um adapter novo.

### P7. Loja de cosméticos + Inventário
- **Propósito:** gastar moeda soft em skins e trails (sem pay-to-win, conforme o plano) e equipar.
- **Dados:** `CosmeticItem`, `wallet.soft`, `inventory {owned[], equipped{skin, trail}}`.
- **Fluxos:** catálogo (só `enabled` e dentro de `available`) → preview → comprar (`StorePort.purchase` → `PurchaseResult {ok, reason}`) → equipar.
- **Depende de:** `StorePort` / `SoftCurrencyStore`, ThemeService (slots `player.*`), PlayerState.
- **Aceite:**
  - saldo insuficiente → nega com feedback;
  - o item equipado aparece na run;
  - nada na loja altera física ou gameplay.

### P8. Conquistas (visão do jogador)
- **Propósito:** listar conquistas (bloqueadas e liberadas) e o progresso.
- **Dados:** `AchievementDef` + `achievements{progress, unlocked_at}`.
- **Fluxos:**
  - o `AchievementService` escuta o EventBus (`run_finished`, `collectible_picked`, `player_died`…), incrementa `metric` com `filter` e desbloqueia em `target`;
  - toast in-run;
  - paga `reward_soft`.
- **Depende de:** EventBus, PlayerState, ContentRepository.
- **Aceite:**
  - conquistas `hidden` mostram "???";
  - desbloquear é idempotente;
  - o adapter de plataforma (Steam etc.) fica atrás de `AchievementsPort.unlock(id)` (contrato do web).

### S0. Studio (hub)
- **Propósito:** entrada única das ferramentas de admin.
- **Dados:** `Session.roles`, feature `studio`.
- **Fluxos:** hub → Narrativa · Conquistas · Loja · Temas · Sandbox · "Validate content" · Export/Import JSON.
- **Componentes:** widgets genéricos `CrudList` + `CrudForm`, gerados a partir do schema da classe para reaproveitar nas 4 telas de CRUD.
- **Aceite:**
  - inacessível sem feature + admin;
  - salvar roda `validate()` e bloqueia referências quebradas (ex.: fase apontando para um beat inexistente).

### S1. Gestão Narrativa (mundos, fases e história)
- **Propósito:** controlar estrutura, ordem e história.
- **Dados:** `WorldDef`, `PhaseDef`, `StoryBeat`.
- **Fluxos:**
  - CRUD de mundos e fases;
  - **habilitar / desabilitar**;
  - **reordenar** (sobe/desce ou arrastar), que define a sequência do anel e da progressão;
  - definir `unlock_rule`;
  - vincular beats `intro/outro` por fase;
  - CRUD de `StoryBeat` (páginas);
  - **preview** da história e "jogar esta fase".
  - Visão em linha do tempo por mundo: Fase 1 → beat → Fase 2 → beat…
- **Depende de:** S0, P3, P4.
- **Aceite:**
  - mudanças refletem na Seleção sem reiniciar (`content_changed`);
  - não é possível desabilitar a única fase de entrada sem um aviso;
  - os dados são exportados em JSON diffável.

### S2. CRUD de Conquistas
- **Dados:** `AchievementDef`.
- **Fluxos:** criar/editar/duplicar/desativar · escolher `metric` numa lista fechada de eventos conhecidos · `filter` por mundo/fase · testar ("simular N eventos").
- **Depende de:** S0, P8.
- **Aceite:**
  - `metric` inválida é impossível de salvar;
  - desativar não apaga o progresso do jogador.

### S3. CRUD de Itens da Loja
- **Dados:** `CosmeticItem`.
- **Fluxos:** criar/editar · preço · janela `available` · mapear `slot_overrides` (picker de recurso) · preview no Cubu · desativar.
- **Depende de:** S0, P7, ThemeService.
- **Aceite:**
  - item sem asset válido não fica `enabled`;
  - mudar o preço não afeta quem já comprou.

### S4. Gerenciador de Pacotes Temáticos
- **Dados:** `ThemePack` + registro de slots.
- **Fluxos:**
  - criar pack;
  - preencher os overrides slot a slot (com indicação de quais slots faltam, que usam fallback);
  - `schedule` (always/manual/datas);
  - `priority`;
  - **simular data** e preview em Title, Seleção e Run.
- **Depende de:** S0, ThemeService.
- **Aceite:**
  - trocar o pack ativo atualiza sprites e áudio sem reiniciar;
  - duas janelas sobrepostas são resolvidas por `priority`;
  - remover um pack volta para `default`.

### S5. Sandbox (testar gameloops e desafios)
- **Propósito:** laboratório para iterar o feel sem mexer no oficial.
- **Dados:** `GameLoopConfig` (cópia editável), presets salvos, `ChallengeDef` (**hipótese:** `{id, chart_ref|lab, modifiers, goal}`).
- **Fluxos:**
  - escolher fase / chart / Lab (o atual `DevLabCatalog`) + config;
  - editar ao vivo (evolução do `DevTuningPanel`);
  - modifiers (invencível, slow-mo, mostrar hitbox, pular para trecho);
  - overlay de métricas (mortes por trecho, tempo);
  - salvar preset;
  - **"Promote to official"** gera um JSON de config nova (vira PR, nunca escrita silenciosa).
- **Depende de:** S0, `GameLoopConfig`, RunRequest.
- **Aceite:**
  - nada do Sandbox altera o save do jogador nem o ranking;
  - reset volta para o oficial em 1 clique.

### P9. Gameloop Oficial
- **Propósito:** a única run de produção, onde acontece o refinamento final.
- **Dados:** `GameLoopConfig` com `id=official`, `locked=true` e `version`.
- **Fluxos:** Seleção → (story intro) → run oficial → Results → (story outro) → próxima fase. Ranking, conquistas e moedas só contam aqui.
- **Depende de:** P3–P8, `GameLoopConfig`.
- **Aceite:**
  - a run carrega a config oficial por id (sem `@export` soltos como fonte da verdade);
  - a versão da config vai no `RunResult` (os rankings ficam comparáveis);
  - o fluxo ponta a ponta funciona sem nenhuma tela de Studio.

---

## 5. Sequência de implementação (PRs para agentes de nuvem; merge do Hilderney)

> Cada PR é pequeno, tem teste mínimo quando há lógica (GUT ou gdUnit, item 11.1 do backlog) e respeita o `RULES.md` (inglês, `TODO: AUDIO`, IDTCS). ⛓ = depende de.

**M0 · Fundamentos** (sem mudança visual relevante)

| PR | Entrega | ⛓ |
|---|---|---|
| 0.1 | Autoload `Router` (rotas, params, pilha de back, guards). `GameManager.go_*` delega para ele | — |
| 0.2 | Classes de conteúdo (`WorldDef`, `PhaseDef`, `StoryBeat`, `AchievementDef`, `CosmeticItem`, `ThemePack`, `GameLoopConfig`) + `from_dict` / `to_dict` / `validate` + escolha de GUT/gdUnit + testes | — |
| 0.3 | `ContentPort` + `BundledJsonContent` + factory + `res://content/*.json` com **exatamente** os 4×4 atuais. A `LevelSelect` passa a ler do repositório | 0.2 |
| 0.4 | Schema de save v1 + migração (mantém `input`) + `PlayerState` + save com debounce | — |
| 0.5 | `RunRequest` / `RunResult` + sinais novos no EventBus. `game.gd` lê o request. A run normal ganha kill zone + linha de chegada mínimas (gera outcome) | 0.1 |
| 0.6 | `AuthPort` + `LocalGuestAuth` + `DevAdminAuth` + `Session`. `IS_DEV_BUILD` vira a feature tag `studio` + presets de export dev/release com exclusão de `res://studio/` | 0.4 |
| 0.7 | `ThemeService` + registro de slots + componente `ThemeSlot` + pack `default`. O Player usa o slot `player.body` (fallback para `ColorRect`) | 0.3 |

**M1 · Telas do jogador**

| PR | Entrega | ⛓ |
|---|---|---|
| 1.1 | Tela Inicial (P1) | 0.1, 0.6, 0.7 |
| 1.2 | Login/Perfil (P2) | 0.6 |
| 1.3 | Seleção orientada a dados (P3): `enabled`, `order`, `unlock_rule`, badges | 0.3, 0.4 |
| 1.4 | Results/Game Over (P5) + retry rápido | 0.5 |
| 1.5 | Story Player formato `cards` (P4) + gancho pós-clear | 0.3, 0.4, 1.4 |
| 1.6 | `AchievementService` + tela (P8) | 0.4, 0.5 |
| 1.7 | `StorePort` + `SoftCurrencyStore` + Loja/Inventário (P7) + moeda por coletável | 0.4, 0.7 |
| 1.8 | `LeaderboardPort` + `LocalLeaderboard` + Ranking (P6) | 0.5 |

**M2 · Studio (admin)**

| PR | Entrega | ⛓ |
|---|---|---|
| 2.1 | Hub do Studio (S0) + `CrudList` / `CrudForm` + `StudioFileContent` + Export/Import + Validate | 0.3, 0.6 |
| 2.2 | Gestão Narrativa (S1) | 2.1, 1.3, 1.5 |
| 2.3 | CRUD de Conquistas (S2) | 2.1, 1.6 |
| 2.4 | CRUD de Itens da Loja (S3) | 2.1, 1.7 |
| 2.5 | Gerenciador de Pacotes Temáticos (S4) + simular data | 2.1, 0.7 |

**M3 · Sandbox + Gameloop Oficial**

| PR | Entrega | ⛓ |
|---|---|---|
| 3.1 | Extrair os `@export` do Player e do Game para `GameLoopConfig`, com `official.json` v1 (valores atuais) | 0.2, 0.5 |
| 3.2 | Sandbox (S5): config editável, modifiers, presets, Lab como preset, overlay de métricas | 3.1, 2.1 |
| 3.3 | "Promote to official" (gera arquivo de config nova para PR) + versão no `RunResult` | 3.2 |
| 3.4 | Gameloop Oficial (P9): fluxo ponta a ponta Seleção → história → run → resultado → história → próxima | 3.1, 1.3–1.5 |

**Paralelismo:**
- M0.1, M0.2 e M0.4 podem sair juntos.
- Dentro de M1 e M2, as telas são independentes depois das dependências.

**Encaixe com o backlog existente:**
- O spike Web e o áudio (itens 1–2) podem rodar **em paralelo** a M0.
- O vertical slice (item 3: portar os charts do `seed.json`) deve vir **depois de 0.3 e 3.1**, para não ser hardcoded de novo.
- Os itens 3.3 / 3.4 / 3.5 / 3.7 do backlog ficam cobertos por M0.5, 1.4, 1.3 e 3.2.

---

## 6. Decisões abertas (Hilderney)

1. **Backend de login e ranking:**
   - (a) nenhum por enquanto: guest local + conta do portal (Poki/CrazyGames);
   - (b) Supabase;
   - (c) Firebase;
   - (d) PlayFab;
   - (e) Nakama.
   - _Recomendo (a) agora e (b) ou (d) quando o ranking online for aprovado._
2. **Ranking online:**
   - (a) só local no 1º lançamento;
   - (b) online desde o início (exige backend + anticheat básico + política de privacidade/CSP da Poki).
3. **Onde fica o admin:**
   - (a) Studio in-game, só em builds dev (**recomendado**);
   - (b) plugin de editor;
   - (c) ambos;
   - (d) Studio também em produção para admin remoto (só com backend + roles).
4. **Formato da história:**
   - (a) cards de texto + ilustração;
   - (b) diálogo com retratos e speaker;
   - (c) cutscene animada;
   - (d) começar com (a) e evoluir.
   - **Momento:** antes da fase, após o 1º clear ou ambos?
   - **Idioma:** o `RULES.md` exige UI em inglês. A história fica em inglês com chaves de tradução para pt-BR?
5. **Formato canônico do conteúdo:**
   - (a) JSON em `res://content/` (**recomendado**: diff, agentes, backend);
   - (b) `.tres` (Inspector grátis, pior para diff e para o Studio in-game).
6. **Prioridade de visual:**
   - (a) cosmético comprado sempre vence o pack sazonal no Cubu (**recomendado**);
   - (b) o pack sazonal sobrescreve tudo durante o evento.
   - **Ativação dos packs:** por data automática, manual ou ambos?
7. **Mapa de mundos:**
   - (a) manter os 4 mundos do Godot (NEON CUBES, PIXEL PEAKS, VOLT VALLEY, CANDY CORE) × 4 fases;
   - (b) migrar para os temas do web (Nebulosa 128, Cristal 140, Vazio 152). O `seed.json` tem só 3 levels.
   - **Fases por mundo:** 4 ou outro número?
8. **Conquistas e economia:**
   - conquistas dão moeda soft (`reward_soft`)?
   - integram com plataforma (Steam/Play Games) já no M1 ou depois?
   - e o streak/desafio diário do web entra no M1 ou fica para a fase de monetização?

---
_Pendência de repo: `.docs/` (IDTCS, reactive sounds, hitstop) está no `.gitignore`. Para os agentes de nuvem seguirem esses padrões, os arquivos precisam ir para o repo (ex.: `docs/`) ou ser colados no prompt de cada PR. Confirmar também a duração das transições (800 ms no `RULES.md` × 0,35 s no código)._
