# OTClient Git Guide (Project-Specific)

## Scope
Padrao de commits e fluxo Git para este repositorio com uso de Codex.

## Commit Prefix
- `fix:` correcao de bug.
- `feat:` nova funcionalidade.
- `imp:` melhoria/refactor sem nova feature.
- `docs:` documentacao versionada.
- `codex:` ajustes de artefatos operacionais do Codex quando aplicavel.

## Regras Essenciais
1. Commits atomicos.
2. Evitar commit misto (codigo + docs + config) quando der para separar.
3. Mensagens curtas e objetivas.
4. Para `fix`, validar com usuario se o bug foi resolvido antes do commit.
5. Se houver regra mais restritiva no contexto atual (ex.: AGENTS/roadmap), seguir a mais restritiva.

## Fluxo Recomendado por Tipo de Mudanca
- `fix`: reproduzir -> corrigir -> validar com usuario -> commitar.
- `feat`: quebrar em passos pequenos -> validar fluxo principal -> commitar por etapa.
- `imp`: foco em legibilidade/performance sem alterar contrato funcional.
- `docs`: manter alinhado ao estado real do codigo e ferramentas.

## Branch Hygiene
- Trabalhar na branch da feature.
- Evitar reescrever historico compartilhado sem alinhamento previo.
- Nao usar comandos destrutivos sem necessidade explicita.

## Staging Checklist
- `git status` limpo do que nao pertence ao escopo da task.
- Conferir diff por arquivo antes de commitar.
- Garantir que arquivos locais/temporarios estao ignorados.
- Revisar se commit inclui apenas mudancas intencionais.

## Review Checklist Antes de Commit
- O prefixo escolhido corresponde ao tipo real da mudanca?
- O commit esta pequeno o suficiente para rollback facil?
- Existe mistura desnecessaria de codigo e documentacao?
- Existe dependencia nao documentada entre commits?

## Regras para Wiki/Bundle Codex
- `docs/roadmap/` e artefatos locais devem seguir estrategia de ignore do projeto.
- Bundle deve ser gerado por `tools/export_wiki_bundle.py`.
- Nao commitar zip de export quando padrao do projeto for manter artefato fora do repo.
- Se links de release mudarem, atualizar `docs/README.md`.

## Quando Separar Commits
Separar commits quando:
- ha mudanca de contrato tecnico e documentacao associada;
- ha alteracao em parser C++ e consumo Lua em etapas distintas;
- ha ajustes de tooling e ajustes de feature no mesmo ciclo.

## Recommended Commit Templates
- `feat: add codex wiki bundle export flow`
- `imp: align roadmap bootstrap with project guides`
- `fix: correct house auction callback payload mapping`
- `docs: document codex release links and setup`

## Referencias do Projeto
- `AGENTS.global.md`
- `docs/roadmap_codex.md`
- `docs/README.md`
