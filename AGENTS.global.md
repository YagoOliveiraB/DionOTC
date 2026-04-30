---
alwaysApply: true
---

# OTClient Agent Global Guide

## Scope
Padrao global local para operacao de agentes neste workspace.
Este guia integra 2 frentes: Wiki de conhecimento (`wiki.md`) e roadmap de execucao (`docs/roadmap/`).

## Source Priority
1. `/home/jp/.codex/AGENTS.md` (regras globais do ambiente).
2. `AGENTS.global.md` (este arquivo).
3. `AGENTS.md` mais proximo da pasta alvo.
4. `.md` complementares da mesma pasta (ex.: `src/client/world.md`).
5. `docs/roadmap/*.md` quando houver execucao por milestones/tasks.

## Operating Modes

### 1) Wiki / Discovery Mode
Use quando o foco for mapear arquitetura, criar/expandir documentacao e enriquecer contexto para Codex.

- Comecar por `wiki.md` para localizar os guias.
- Ler `AGENTS.md` de raiz de area (`src`, `modules`, `data`, `tools`) antes de subpastas.
- Evitar duplicacao: AGENTS deve servir como mapa + regras; detalhes tecnicos ficam nos `.md` complementares.
- Ao criar/alterar guia relevante, atualizar o indice em `wiki.md`.

### 2) Project Delivery Mode (Roadmap Bootstrap)
Use quando o foco for entrega de feature/projeto com milestones e tarefas.

- Politica de bootstrap: `docs/roadmap_codex.md`.
- Guia operacional do assunto ativo: `docs/roadmap/agents.md`.
- Se `docs/roadmap/` ainda nao existir no ambiente do usuario:
  1. iniciar pelo `docs/roadmap_codex.md`;
  2. solicitar ao Codex o bootstrap dos arquivos canonicos em `docs/roadmap/`;
  3. depois seguir o loop operacional por `status.md` + `tasks.md`.
- Loop obrigatorio:
  1. Ler `docs/roadmap/status.md`.
  2. Executar `NEXT_TASK` definida em `docs/roadmap/tasks.md`.
  3. Validar escopo minimo (build/runtime/log/teste manual).
  4. Atualizar `status.md` e sincronizar `tasks.md`, `milestones.md`, `decisions.md` se houver mudanca de plano.

## Canonical Roadmap Files
Quando houver roadmap local, manter:
- `docs/roadmap/agents.md`
- `docs/roadmap/vision.md`
- `docs/roadmap/roadmap.md`
- `docs/roadmap/milestones.md`
- `docs/roadmap/tasks.md`
- `docs/roadmap/status.md`
- `docs/roadmap/decisions.md`
- `docs/roadmap/architecture.md`

## Where to Start by Area
- `src/AGENTS.md`: core C++, protocolo e arquitetura do cliente.
- `modules/AGENTS.md`: modulos Lua e UI.
- `data/AGENTS.md`: OTUI, layouts e propriedades de widgets.
- `tools/AGENTS.md`: build, CLI, encrypt, launcher e scripts.

## Documentation Growth Rules
- Escrever em PT-BR com termos tecnicos em ingles quando necessario.
- Frases curtas, listas objetivas e escopo claro no inicio.
- Quebrar guias grandes por subpasta e manter o guia pai como indice.
- Reaproveitar contexto existente antes de criar novo arquivo.
- Usar `wiki.md` como indice oficial da documentacao local.

## Compatibility Note
- `agent.md` e `agents.global` sao legados neste repositorio.
- O arquivo ativo para contexto global local e `AGENTS.global.md`.
