# CLAUDE.md — Galotito (Sorteio de Times)

App de página única para sortear e gerenciar times de futebol/pelada, com notas por
jogador, confirmação por dia, convidados, placar por partida, classificação e histórico.

- **URL em produção:** https://cagartner.github.io/galotito/
- **Repositório:** https://github.com/cagartner/galotito (conta pessoal `cagartner`)
- **UI toda em português** (pt-BR). Os termos do domínio são em português e devem ser mantidos
  (Time Base, Times da Partida, Convidado, "extra", "falta", etc.).

## Stack e execução

- **Um único arquivo:** `index.html` (HTML + CSS + JS, tudo inline). Não há build, bundler nem dependências instaladas.
- **Vue 3** carregado por CDN (`unpkg.com/vue@3`), Options via `createApp({ setup() {...} })` com Composition API (`ref`, `reactive`, `computed`, `watch`). **Precisa de internet** para o CDN.
- **Persistência:** `localStorage` (por navegador/aparelho — não sincroniza entre dispositivos; use Exportar/Importar).
- **Rodar localmente:** servir por HTTP (o CDN e o `localStorage` não funcionam bem via `file://` no preview).
  ```bash
  python3 -m http.server 8765
  # abrir http://localhost:8765/index.html
  ```
- **Deploy:** GitHub Pages (branch `main`, raiz). Publicar é só commitar e dar push:
  ```bash
  git add -A && git commit -m "..." && git push
  ```
  O Pages reconstrói em ~1 min. Verificar build: `gh api repos/cagartner/galotito/pages/builds/latest --jq '.status'`.
  Atenção: há duas contas gh (`cagartner-id` e `cagartner`); usar a pessoal `cagartner` (`gh auth switch --user cagartner`).

## Modelo de dados (localStorage)

| Chave | Conteúdo |
|-------|----------|
| `sorteio_players` | Array de jogadores (ver abaixo) — **fonte de verdade** de base/confirmação/convidado |
| `sorteio_blocks` | Array de pares `[nomeA, nomeB]` que não podem jogar juntos |
| `sorteio_history` | Histórico de **sorteios de base** `{date, preto:[nomes], vermelho:[nomes]}` |
| `sorteio_matches` | Histórico de **partidas** `{drawId, date, preto:[nomes], vermelho:[nomes], scorePreto, scoreVermelho, winner}` |
| `sorteio_last_draw` | Times da partida atuais `{id, preto:[obj], vermelho:[obj]}` (`id` = id da sessão de base) |

### Objeto do jogador
```js
{
  name: 'Carlos',
  active: true,      // false = lesionado/desativado (fora de tudo)
  confirmed: false,  // presença HOJE (opt-in; zera a cada "Novo dia")
  guest: false,      // true = convidado de fora (nunca entra no sorteio da base)
  base: 'preto'|'vermelho'|null,  // time base do jogador (definido pelo Sorteio 1)
  ratings: { geral, fisico, finalizacao, visao, fominha }  // 1..5 cada
}
```
- `migratePlayer()` preenche defaults ausentes ao carregar (compatível com backups antigos; `base` novo → `null`).
- **Nota média** = média das 5 categorias (`avgRating`). Usada para balancear os times.

## Conceito central: DOIS sorteios

O sistema tem **dois níveis** distintos (é a nuance mais importante do projeto):

### 1) Time Base (`sortBaseTeams`)
- Pool = **todos os regulares ativos** (`baseRegulars` = `active && !guest`), **independente de confirmação**.
- Divide em `Base Preto` / `Base Vermelho`, gravando `base` em cada jogador (persistente).
- **Persiste até um time vencer 2×** na rodada (regra abaixo). Não muda a cada dia.
- Convidados **nunca** entram aqui.
- Algoritmo:
  1. Ordena por nota desc; **snake draft** (serpentina) para balancear.
  2. Roda até **300 tentativas** embaralhando jogadores de nota parecida (diferença ≤ 0.5).
  3. Respeita bloqueios (`checkBlocks`). Se nenhuma combinação válida, cai num fallback que corrige bloqueios por troca.
  4. **Custo** de cada combinação = `diff + 0.25 * changeRatio`, onde:
     - `diff` = |soma de notas A − soma de notas B| (**equilíbrio é prioridade**).
     - `changeRatio` = sobreposição com o sorteio de base anterior (`overlapCount`), 0 = mudou tudo, 1 = igual. O peso 0.25 faz, **entre opções equilibradas**, escolher a que mais troca jogadores de time.
  5. Cor (preto/vermelho) sorteada aleatoriamente; grava `base` nos jogadores sorteados e **limpa `base` (=null) de quem ficou de fora** (lesionados/convidados). Isso evita "base fantasma" ao reativar alguém.
  6. Registra em `sorteio_history` e chama `generateMatchTeams()`.

### 2) Times da Partida (`generateMatchTeams`)
- É quem **realmente joga hoje**. Deriva da base + confirmação + convidados:
  - Membros da base **confirmados** de cada lado (`base === 'preto'/'vermelho'` e `confirmed`).
  - **+ Convidados confirmados** (`guest && confirmed`).
  - **+ "Avulsos" (floaters):** regulares **confirmados sem base** (`confirmed && base == null`) — ver abaixo.
- Convidados e avulsos são **distribuídos automaticamente** para equilibrar: para cada um (maior nota primeiro), joga no time com **menos jogadores**; empatou no tamanho, no de **menor nota somada**. Com total ímpar, o split fica o mais próximo possível (ex.: 4×3).
- Resultado vai para `teamPreto`/`teamVermelho` (refs concretos → permitem troca manual por clique) e é salvo em `sorteio_last_draw`.

## Convidados x "Avulsos" (floaters)

- **Convidado** (`guest: true`): jogador de fora. Some da base, entra só na partida. Tag amarela **conv.**
- **Avulso/floater:** um **regular** que estava **lesionado na hora do sorteio da base** (ficou com `base = null`). Ao voltar a ficar disponível (reativar + confirmar), a base já está fechada, então ele entra na partida **tratado como convidado** (distribuído automaticamente). Tag azul **extra**. No próximo sorteio de base ele volta a ter time base normalmente.
- Ambos aparecem em "Fora do time base" na seção 1 enquanto não tiverem base.

## Regra das 2 vitórias (obrigar novo sorteio de base)

- `currentDrawId` = id da **sessão** de base (setado no sorteio de base e em "Novo dia"). Partidas registradas herdam esse id em `drawId`.
- `sessionRecord` = contagem de vitórias/empates da sessão atual (só partidas com `drawId === currentDrawId`). **Empate NÃO conta como vitória** (mas é registrado).
- `mustResort` = `true` quando `preto >= 2` **ou** `vermelho >= 2` vitórias na sessão.
- Quando `mustResort`: o painel de placar **bloqueia** o registro e exige "Sortear time base novamente"; o botão do topo fica vermelho pulsante. `registerMatch` também recusa. Re-sortear a base gera novo `currentDrawId` → zera a rodada e libera.

## Confirmação / Novo dia

- Confirmação é **opt-in**: `confirmed` começa `false`; jogador clica no check à esquerda do nome. Convidados nascem confirmados (foram adicionados porque vão jogar).
- **Novo dia** (`newDay`): desmarca todos, limpa os times da partida atuais, começa **nova sessão** (novo `currentDrawId`), **mantém a base e os convidados** na lista.
- **Confirmar todos**, **Limpar convidados** (remove todos os `guest`).

## Placar e Classificação

- `registerMatch`: lê o placar, cria um registro em `sorteio_matches` com `winner` (`preto`/`vermelho`/`empate`) e snapshot dos nomes dos dois times.
- `standings` (computed): agrega TODAS as partidas por nome → `J, V, E, D`, `pontos = 3*V + E`, `aprov% = pontos/(3*J)`. Ordena por pontos, depois aproveitamento, vitórias, nome. Cobre convidados também (usa os nomes dos snapshots).

## Reverter base pelo histórico (`restoreBase`)

- Cada item de "Histórico de Sorteios (Time Base)" tem **"Usar esta base"**: reaplica aquela divisão (seta `base` nos jogadores que ainda existem/ativos, `null` no resto), gera nova sessão (`currentDrawId`) e regenera os times da partida. Não cria novo item de histórico.
- Cada item também tem **× (excluir)** individual (`removeHistory`), além de "Limpar histórico".

## Import/Export

- **Exportar** baixa um JSON com `players`, `blocks`, `history`, `matches` e `lastDraw` (com `id`).
- Como base/confirmação/convidado/notas ficam **dentro de cada `player`**, o backup carrega **todo o estado**. `standings` e times da partida se reconstroem a partir disso.
- **Importar** substitui tudo (com confirmação) e roda `migratePlayer` (backups antigos sem `base` viram `null`).
- **Sempre que adicionar um campo novo em qualquer entidade, conferir se export/import e `migratePlayer` cobrem.**

## Convenções de UI / cores

- Fundo `#1a1a2e`, cards `#16213e`, destaque `#e94560` (vermelho), azul `#0f3460`, dourado `#f5c518`.
- Tags: **conv.** (amarelo) = convidado; **extra** (azul) = avulso; **falta** (vermelho) = na base mas não confirmado.
- Times: `team-preto` (preto) e `team-vermelho` (vermelho).

## Notas para desenvolvimento / testes

- Ao alterar, **verificar no navegador** servindo por HTTP (ver acima). Erros de template do Vue aparecem no console.
- **`window.confirm`/`alert` são auto-dispensados** no preview automatizado (retornam sem interação). Para testar ações com `confirm()` (Novo dia, Importar, Usar esta base, limpezas), sobrescrever no console: `window.confirm = () => true`.
- Fluxo de teste típico: semear `localStorage` via console → recarregar → interagir → conferir. Limpar com:
  ```js
  ['sorteio_players','sorteio_history','sorteio_matches','sorteio_last_draw','sorteio_blocks'].forEach(k=>localStorage.removeItem(k))
  ```
- Todo o estado exposto ao template está no `return` do `setup()`. Ao adicionar função/computed novo, **lembrar de expô-lo lá**.
