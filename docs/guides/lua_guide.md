# OTClient Lua Guide (Project-Specific)

## Scope
Guia de Lua aplicado ao OTClient deste repositorio.
Use para `modules/*` e para integracao com callbacks vindos de C++.

## Arquitetura Lua no Projeto
- Base: `modules/corelib`, `modules/gamelib`, `modules/modulelib`.
- Modulos de feature: `modules/game_*`, `modules/client_*`.
- UI principal costuma ser orientada por controllers e widgets do OTClient.

## Regras Essenciais
1. Respeite padrao existente do modulo antes de propor estilo novo.
2. Evite logica nova em escopo global; prefira tabela/controller do modulo.
3. Use guard clauses para tratar estado invalido cedo.
4. Evite hardcode de payload de protocolo; prefira constantes e wrappers.
5. Ao receber callback de parser, normalize dados antes de renderizar UI.

## Padrao de Modulo Recomendado
- Estado local em tabela do modulo (evitar variaveis globais soltas).
- `onInit` para setup minimo.
- `onGameStart`/hooks de jogo para bind de eventos de runtime.
- `terminate` para cleanup total.

## Ciclo de Vida de Modulo
- Inicializacao: `onInit`/`init`.
- Registro de eventos: `registerEvents` no start correto.
- Cleanup: `terminate` com `unregisterEvents`, cancelamento de timers/eventos e destruicao de UI.

## Eventos e Callbacks
- Sempre manter simetria `registerEvents` x `unregisterEvents`.
- Nome de handler deve refletir dominio da feature.
- Nunca assumir que callback sempre vira com payload completo; validar campos.

## Integracao com C++/Protocolo
- Se C++ expor callback novo (`g_lua.callGlobalField`), ligar handler no controller responsavel.
- Se UI fizer request para C++, usar wrappers em `modules/gamelib/game.lua` quando aplicavel.
- Evitar chamar API inexistente de `g_game` no fluxo principal.

## Contrato de Dados na UI
Antes de atualizar tela, garantir:
1. Campos obrigatorios presentes.
2. Tipos esperados coerentes.
3. Fallback para campos opcionais.
4. Render sem crash quando lista vier vazia.

## UI e Estado
- Estado de tela deve ser derivado de dados reais quando houver payload de servidor.
- Remover mock/runtime fake quando feature depender de resposta real do protocolo.
- Tratar estado vazio/erro sem crash e com feedback claro.

## Performance no contexto OTClient
- Evitar loops pesados por frame.
- `scheduleEvent`/`cycleEvent` devem ter cleanup no terminate.
- Evitar recriar widgets desnecessariamente em updates frequentes.
- Evitar refresh completo de lista quando apenas item isolado mudou.

## Observabilidade
- Logs de debug apenas quando necessario e removiveis.
- Mensagens para usuario devem ser curtas e orientadas a acao.
- Nao deixar placeholders de debug no texto final da UI.

## Review Checklist (Lua)
- O modulo encerra sem vazamento de eventos/timers?
- Existe chamada para API de `g_game` que pode nao existir?
- O handler de callback trata payload incompleto sem erro?
- A UI depende de mock quando ja existe payload real?
- O patch alterou estilo local sem necessidade?

## Teste Minimo Recomendado
- Abrir/fechar modulo sem erro Lua.
- Executar acao principal da feature.
- Verificar callback recebido e UI atualizada.
- Testar estado vazio/erro do fluxo.

## Checklist Antes de Fechar Tarefa Lua
- Sem erro Lua em abertura/fechamento do modulo.
- Eventos conectados/desconectados corretamente.
- Fluxo principal funciona com dados reais (quando disponivel).
- Sem placeholders de debug no caminho final de UI.

## Anti-patterns (evitar)
- Chamar `g_game.request*` inexistente.
- Manter dados mockados quando servidor ja fornece payload.
- Acrescentar logs e nao remover/condicionar depois.
- Misturar mudanca funcional com refactor amplo do modulo inteiro.

## Referencias do Projeto
- `modules/AGENTS.md`
- `modules/gamelib/AGENTS.md`
- `modules/modulelib/AGENTS.md`
- `modules/game_interface/AGENTS.md`
- `docs/roadmap_codex.md`

## Referencia Externa
- Lua style reference (geral): https://github.com/luarocks/luarocks/blob/main/docs/style_guide.md
