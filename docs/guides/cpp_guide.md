# OTClient C++ Guide (Project-Specific)

## Scope
Este guia define praticas de C++ aplicadas ao OTClient neste repositorio.
Use este arquivo para mudancas em `src/client`, `src/framework` e `src/protobuf`.

## First Principle
Priorize compatibilidade de protocolo e estabilidade de runtime.
Em OTClient, um parser incorreto quebra fluxo inteiro da UI Lua.

## Arquitetura Aplicada ao Projeto
- Entrada de mensagens: `src/client/protocolgameparse.cpp` (`ProtocolGame::parseMessage`).
- Estado do jogo: `src/client/game.cpp`.
- Ponte C++ -> Lua: `g_lua.callGlobalField(...)`.
- Core framework: `src/framework/*` (event loop, rendering, input, net).

## Regras Essenciais para Protocolo
1. Preserve ordem exata de leitura de pacote (`getU8/U16/U32/U64/getString`).
2. Nunca consuma bytes a mais para testar hipotese de payload.
3. Se payload for opcional, condicione por feature/version/flag antes de ler.
4. Ao adicionar callback Lua, envie payload estavel e nomeado de forma consistente.
5. Se parser mudar contrato, documente em `docs/roadmap/decisions.md` quando houver roadmap ativo.

## Padrao de Parse Seguro
Sequencia recomendada para cada opcode:
1. Ler campos obrigatorios.
2. Validar flags/version gates.
3. Ler blocos opcionais apenas quando habilitados.
4. Construir payload com tipos consistentes.
5. Publicar callback Lua no final da funcao.

## C++ -> Lua Callback Contract
- Use callbacks no padrao de dominio (ex.: `onParseCyclopedia...`).
- Evite passar estruturas ambiguas sem nome sem necessidade.
- Mantenha semantica de tipos previsivel:
  - ids/flags pequenos: `uint8_t/uint16_t/uint32_t`
  - valores monetarios/contadores grandes: `uint64_t`
- Se um callback novo for adicionado no parser, alinhar consumo Lua no modulo correspondente.

## Contrato de Evolucao de Payload
Quando precisar mudar payload:
1. Tentar extensao backward compatible antes de quebra direta.
2. Se quebra for inevitavel, alinhar C++ e Lua no mesmo ciclo.
3. Atualizar docs do modulo afetado (`src/client/*.md` ou `modules/*`).
4. Registrar decisao em `docs/roadmap/decisions.md` (quando roadmap ativo).

## Threading e Dispatcher
- Parse de protocolo pode ocorrer fora da UI; atualizacao de estado/efeitos deve respeitar fluxo do projeto.
- Use `g_dispatcher` para trabalho que precisa ocorrer no contexto principal.
- Evite bloquear loop com parse custoso dentro do caminho quente.

## Render/Framework Safety
Para mudancas em `src/framework`:
- Evite alocar em excesso em caminhos por frame.
- Cuidado com codigo executado a cada draw/update.
- Em alteracoes de input, valide impacto em widgets focados e hotkeys.

## Estilo de Mudanca no OTClient
- Mudancas pequenas e localizadas (1-3 arquivos quando possivel).
- Evite refator amplo junto com alteracao de protocolo.
- Sem renomeacoes cosmeticas em arquivos sensiveis de parser.

## Logging e Diagnostico
- Preferir logs objetivos com contexto de pacote/estado.
- Logs temporarios de troubleshoot devem ser removiveis e discretos.
- Evitar flood de log em loops de alta frequencia.

## Review Checklist (C++)
- O parser ainda le todos os campos na ordem correta?
- Existem gates claros para campos opcionais?
- O callback Lua usa nome/assinatura coerentes com o dominio?
- O change set mistura parser + refator visual desnecessario?
- O diff introduz risco de regressao em opcodes vizinhos?

## Teste Minimo Recomendado
- Build local sem novos warnings relevantes.
- Abrir fluxo funcional associado ao opcode alterado.
- Verificar ausencia de erro de parse no log.
- Confirmar callback Lua recebido no modulo consumidor.

## Safety Checklist Antes de Fechar Tarefa C++
- Build local compila sem novos warnings relevantes.
- Parser nao deixou leitura desalinhada.
- Callback Lua correspondente existe e foi conectado.
- Sem regressao obvia em fluxo relacionado.

## Anti-patterns (evitar)
- Ler campo sem validar flag que habilita o bloco.
- Misturar parse, regra de negocio e UI na mesma alteracao.
- Ajustar assinatura de callback Lua sem alinhar modulo Lua consumidor.
- Fazer otimizacao micro sem medir impacto no caminho quente.

## Referencias do Projeto
- `src/client/AGENTS.md`
- `src/framework/AGENTS.md`
- `src/client/network.md`
- `docs/roadmap_codex.md`

## Referencia Externa
- C++ Core Guidelines: https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
