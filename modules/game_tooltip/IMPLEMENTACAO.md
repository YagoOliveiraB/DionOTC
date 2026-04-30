# Tooltip Custom de Item (Implementacao Completa)

Este documento descreve como implementar o tooltip customizado de item usado no cliente, incluindo:

- Fluxo completo cliente-servidor
- Contrato de comunicacao (extended opcode + JSON)
- O que o servidor precisa enviar
- Passo a passo para integrar em TFS/Canary
- Checklist de testes e troubleshooting

## Visao Geral

O tooltip custom esta no modulo `modules/game_tooltip` e substitui o comportamento de hover do `UIItem` para buscar dados do item no servidor e renderizar um painel custom.

Arquivos principais:

- `modules/game_tooltip/tooltip.otmod`
- `modules/game_tooltip/tooltip.otui`
- `modules/game_tooltip/tooltip.lua`

No cliente, a comunicacao usa `extended opcode 251`.

## Arquitetura e Fluxo

1. Jogador passa mouse sobre um `UIItem`.
2. `UIItem:onHoverChange` (sobrescrito pelo modulo) agenda exibicao com delay curto (`50ms`).
3. O cliente envia `sendExtendedOpcode(251, tostring(clientId))`.
4. Servidor recebe o opcode `251`, le o `clientId`, monta JSON com os dados do item.
5. Servidor responde no mesmo opcode `251`.
6. Cliente valida o payload recebido:
   - JSON valido
   - `clientId` presente
   - `clientId` igual ao item atualmente em hover
7. Cliente renderiza o tooltip custom e inicia follow do cursor.
8. Dados ficam em cache por `30s` por `clientId` para evitar novas requisicoes no mesmo item.

## Comportamento no Cliente (o que ja existe)

O modulo cliente ja faz:

- Registro de handler: `ProtocolGame.registerExtendedOpcode(251, onExtendedOpcode)`
- Hook em `UIItem.onHoverChange`
- Requisicao de dados por `clientId`
- Renderizacao do painel (`name`, `vocation`, `level`, `rare`, atributos)
- Cache com TTL (`TOOLTIP_CACHE_TTL = 30`)
- Follow do cursor (`TOOLTIP_FOLLOW_INTERVAL = 12ms`)
- Fade in/out e posicionamento inteligente na tela
- Restauracao do `UIItem.onHoverChange` original no `terminate()`

## Contrato de Comunicacao (Cliente <-> Servidor)

### Request (cliente -> servidor)

- Opcode: `251`
- Buffer: string contendo o `clientId` numerico do item
- Exemplo de buffer: `"3031"`

### Response (servidor -> cliente)

- Opcode: `251`
- Buffer: JSON (string) com os dados do item

Campos aceitos pelo cliente:

- Obrigatorio:
  - `clientId` (number)
- Opcionais:
  - `name` (string)
  - `vocation` (string)
  - `level` (number)
  - `levelText` (string) - se enviado, tem prioridade sobre `level`
  - `rare` (string)
  - `attack` (number)
  - `defense` (number)
  - `extraDefense` (number) *(atualmente nao exibido no layout atual)*
  - `armor` (number)
  - `speed` (number)
  - `speedAttack` (number)
  - `range` (number)
  - `hitchance` (number)
  - `weight` (number)
  - `containerSize` (number)

Exemplo de payload JSON:

```json
{
  "clientId": 3031,
  "name": "Sword",
  "vocation": "Knight",
  "level": 35,
  "attack": 42,
  "defense": 28,
  "hitchance": 90,
  "weight": 35.0,
  "rare": "Rare"
}
```

## Regras de Validacao no Cliente

O cliente ignora resposta quando:

- JSON invalido
- `clientId` ausente
- `clientId` diferente do item atualmente em hover

Isso evita mostrar tooltip errado quando o jogador move o mouse rapidamente entre itens.

## Requisitos do Servidor

Seu servidor precisa:

1. Suportar extended opcode.
2. Registrar handler para opcode `251`.
3. Ler o `clientId` do buffer recebido.
4. Resolver item por `clientId`.
5. Montar JSON com os campos disponiveis.
6. Enviar resposta no opcode `251`.
7. Garantir encoding UTF-8 no texto enviado.

## Implementacao de Referencia (Servidor Lua)

Exemplo generico (adapte para API do seu servidor):

```lua
local TOOLTIP_OPCODE = 251

local function buildItemInfoByClientId(clientId)
  -- Troque esta funcao pela API real do seu servidor
  local itemType = Game.getItemByClientId and Game.getItemByClientId(clientId)
  if not itemType then
    return nil
  end

  return {
    clientId = clientId,
    name = itemType:getName(),
    level = 1,
    vocation = "Todas",
    attack = itemType:getAttack() or 0,
    defense = itemType:getDefense() or 0,
    armor = itemType:getArmor() or 0,
    weight = itemType:getWeight() or 0
  }
end

local tooltipEvent = CreatureEvent("TooltipExtendedOpcode")

function tooltipEvent.onExtendedOpcode(player, opcode, buffer)
  if opcode ~= TOOLTIP_OPCODE then
    return false
  end

  local clientId = tonumber(buffer)
  if not clientId then
    return true
  end

  local info = buildItemInfoByClientId(clientId)
  if not info then
    return true
  end

  player:sendExtendedOpcode(TOOLTIP_OPCODE, json.encode(info))
  return true
end

tooltipEvent:register()
```

## TFS x Canary

### TFS (geralmente exige ajuste de source)

Em TFS 1.x, muitas distros nao expoem `Game.getItemByClientId` em Lua por padrao.

Quando nao existir, voce precisa:

- adicionar binding na source (C++) para resolver item por `clientId`
- registrar metodo Lua correspondente
- recompilar

Depois disso, o script Lua do opcode funciona normalmente.

### Canary (normalmente mais simples)

Em Canary, pode existir API pronta para resolver item por `clientId`.
Se existir, basta adaptar o script Lua para usar a funcao correta da sua versao.

Se nao existir API, o caminho volta a ser criar binding na source.

## Encoding (UTF-8) - ponto critico

Todo texto no JSON deve chegar em UTF-8. Se vier com encoding incorreto, aparecem caracteres quebrados no tooltip.

Boas praticas:

- Salvar scripts Lua do servidor em UTF-8
- Garantir que strings vindas da source/C++ estejam em UTF-8 antes do `json.encode`
- Testar com nomes contendo acentos (`Nível`, `Épico`, etc.)

Observacao: o cliente atual aplica conversao `utf8ToLatin1` antes de desenhar labels; mesmo assim, a origem correta continua sendo UTF-8 no payload.

## Conflitos e Cuidados de Integracao

- O modulo sobrescreve `UIItem.onHoverChange`; se outro modulo tambem sobrescrever sem encadear o original, pode haver conflito.
- O tooltip custom e global para `UIItem`, nao apenas inventario.
- Evite usar o opcode `251` para outro sistema de extended opcode no mesmo cliente-servidor.
- Se precisar trocar opcode, altere cliente e servidor juntos.

## Performance

Mecanismos atuais:

- Delay curto para evitar trigger em hover rapido (`50ms`)
- Cache por item (`30s`)
- Follow de cursor sem novas requisicoes ao servidor

Recomendacoes de servidor:

- Evitar consultas pesadas por hover
- Usar cache local de metadata de item no proprio servidor
- Retornar payload enxuto (somente campos necessarios)

## Checklist de Implementacao

1. Cliente com modulo `game_tooltip` carregado.
2. Servidor com extended opcode habilitado.
3. Handler de opcode `251` registrado.
4. Conversao `buffer -> clientId` funcionando.
5. Resolucao de item por `clientId` funcionando.
6. Resposta JSON no opcode `251`.
7. Textos em UTF-8.
8. Testes de hover em:
   - inventario
   - container
   - item sem atributos
   - item com acentuacao

## Checklist de Testes Rapidos

- Hover em item conhecido -> tooltip abre com nome correto
- Mover mouse entre itens rapido -> nao exibe tooltip "trocado"
- Hover repetido no mesmo item -> resposta imediata (cache)
- Atributo ausente no servidor -> linha nao deve quebrar layout
- Texto com acento -> exibicao legivel

## Troubleshooting

### Tooltip nao aparece

- Verifique se opcode `251` chega no servidor.
- Verifique se o servidor responde `251`.
- Verifique se resposta e JSON valido.
- Verifique se `clientId` de resposta bate com o item em hover.

### Tooltip aparece vazio

- JSON chegou, mas campos opcionais nao vieram.
- Confirme ao menos `clientId` + `name`.

### Acentos quebrados

- Padronize UTF-8 no servidor (script + origem dos textos).

### Lag ao passar mouse

- Adicione cache no servidor para metadata de itens.
- Reduza campos enviados para o essencial.

## Canary - Script pronto (copiar e colar)

Esta secao ja esta no formato para usuarios copiarem e colarem no servidor Canary.

### Arquivo 1: `data/scripts/custom/item_tooltip_opcode.lua`

```lua
local TOOLTIP_OPCODE = 251

-- Tenta resolver ItemType por clientId usando diferentes assinaturas comuns em Canary forks.
local function getItemTypeByClientId(clientId)
  if Game and type(Game.getItemByClientId) == "function" then
    return Game.getItemByClientId(clientId)
  end

  if ItemType and type(ItemType.getByClientId) == "function" then
    return ItemType.getByClientId(clientId)
  end

  if ItemType and type(ItemType) == "function" then
    local ok, itemType = pcall(ItemType, clientId)
    if ok and itemType and itemType.getName then
      return itemType
    end
  end

  return nil
end

local function safeCallNumber(fnOwner, fnName, defaultValue)
  if not fnOwner then
    return defaultValue
  end

  local fn = fnOwner[fnName]
  if type(fn) ~= "function" then
    return defaultValue
  end

  local ok, value = pcall(fn, fnOwner)
  if not ok or value == nil then
    return defaultValue
  end

  return tonumber(value) or defaultValue
end

local function safeCallString(fnOwner, fnName, defaultValue)
  if not fnOwner then
    return defaultValue
  end

  local fn = fnOwner[fnName]
  if type(fn) ~= "function" then
    return defaultValue
  end

  local ok, value = pcall(fn, fnOwner)
  if not ok or value == nil then
    return defaultValue
  end

  return tostring(value)
end

local function buildItemTooltipData(clientId)
  local itemType = getItemTypeByClientId(clientId)
  if not itemType then
    return nil
  end

  -- Campos padrao esperados pelo cliente (todos opcionais, exceto clientId).
  local payload = {
    clientId = clientId,
    name = safeCallString(itemType, "getName", ""),
    vocation = "Todas",
    level = 1,
    attack = safeCallNumber(itemType, "getAttack", 0),
    defense = safeCallNumber(itemType, "getDefense", 0),
    extraDefense = safeCallNumber(itemType, "getExtraDefense", 0),
    armor = safeCallNumber(itemType, "getArmor", 0),
    range = safeCallNumber(itemType, "getShootRange", 0),
    hitchance = safeCallNumber(itemType, "getHitChance", 0),
    weight = safeCallNumber(itemType, "getWeight", 0),
    containerSize = safeCallNumber(itemType, "getCapacity", 0)
  }

  return payload
end

local tooltipOpcodeEvent = CreatureEvent("TooltipExtendedOpcode")

function tooltipOpcodeEvent.onExtendedOpcode(player, opcode, buffer)
  if opcode ~= TOOLTIP_OPCODE then
    return false
  end

  local clientId = tonumber(buffer)
  if not clientId then
    return true
  end

  local itemData = buildItemTooltipData(clientId)
  if not itemData then
    return true
  end

  player:sendExtendedOpcode(TOOLTIP_OPCODE, json.encode(itemData))
  return true
end

tooltipOpcodeEvent:register()

-- Registra o evento no login para garantir que o player receba onExtendedOpcode.
local tooltipLoginEvent = CreatureEvent("TooltipRegisterOnLogin")

function tooltipLoginEvent.onLogin(player)
  player:registerEvent("TooltipExtendedOpcode")
  return true
end

tooltipLoginEvent:register()
```

### O que esse script ja faz

- Escuta opcode `251`
- Recebe `clientId` enviado pelo cliente
- Resolve `ItemType` por `clientId` (com fallback para APIs comuns de Canary)
- Monta payload JSON no formato esperado pelo cliente
- Envia resposta com `player:sendExtendedOpcode(251, json.encode(payload))`
- Garante registro do evento no login

### Ajuste unico (se necessario)

Se na sua build Canary nenhuma assinatura de `getItemTypeByClientId` funcionar, ajuste **somente** esta funcao:

- `getItemTypeByClientId(clientId)`

Mantendo todo o restante igual.

### Teste rapido no Canary

1. Reinicie o servidor.
2. Logue no jogo.
3. Passe mouse sobre itens no inventario/container.
4. Confirme no cliente se o tooltip abre com nome e atributos.

Se abrir sem atributos, a comunicacao esta correta e basta enriquecer o `buildItemTooltipData`.

