# Galotito — Linguagem Visual (Design Tokens)

Documentação da identidade visual do app **Galotito** (sorteio de times). Tema **escuro**,
compacto, mobile-first. Extraído de `index.html`.

> **Nota:** isto é uma documentação de tokens/estilo — **não** é uma biblioteca de componentes
> compilada, então **não é sincronizável** pelo `claude.ai/design` (`/design-sync` precisa de
> componentes num `dist/` ou Storybook). Serve como referência de marca e base para, no futuro,
> extrair componentes de verdade.

Arquivos:
- [`tokens.css`](tokens.css) — variáveis CSS prontas pra `var(--token)`.
- [`tokens.json`](tokens.json) — mesma coisa em formato estruturado (design tokens).

## Paleta

### Superfícies
| Token | Hex | Uso |
|-------|-----|-----|
| `--color-bg` | `#1a1a2e` | Fundo da página e inputs |
| `--color-surface` | `#16213e` | Cards, linhas de jogador, painéis |
| `--color-surface-deep` | `#0f1a30` | Painel de edição de notas |

### Marca / ações
| Token | Hex | Uso |
|-------|-----|-----|
| `--color-primary` | `#e94560` | Vermelho da marca — destaque, remover, alerta, foco |
| `--color-action` | `#0f3460` | Azul — botões de ação (sortear, gerar, registrar) |
| `--color-gold` | `#f5c518` | Estrelas de nota, tag "convidado", pontos |
| `--color-success` | `#2e7d32` | Confirmado (check preenchido) |
| `--color-success-bright` | `#4caf50` | Vitórias, toggle ativo |
| `--color-info` | `#4fc3f7` | Tag "extra" (avulso na partida) |
| `--color-link` | `#8ab4f8` | Botão "usar esta base" |

### Times
| Token | Hex |
|-------|-----|
| `--color-team-preto-bg` / `-border` | `#111` / `#444` |
| `--color-team-vermelho-bg` / `-border` | `#3a0a0a` / `#e94560` |

### Texto (forte → fraco)
`--color-text` `#eee` · `--color-text-2` `#ccc` · `--color-text-muted` `#aaa` ·
`--color-text-subtle` `#888` · `--color-text-faint` `#666` · `--color-text-on-color` `#fff`

## Tipografia

- **Família:** `'Segoe UI', sans-serif`
- **Escala (rem):** hero `1.8` · score `1.4` · xl `1.2` · lg `1.15` · md `1.1` · base `1` · sm `0.9` · xs `0.8` · 2xs `0.7` · 3xs `0.65`
- **Pesos:** normal `400`, bold `600`, heavy `700`

## Raios & espaçamento

- **Raios:** card `12px` · base (botão/input) `8px` · sm `6px` · tag `5px` · xs `4px`
- **Espaçamento:** escala de `2` a `24px`. Padding de card `16px`, padding da página `20px`, separação entre seções `24px`.
- **Container:** largura máx. `800px`, centralizado.
- **Foco:** `box-shadow: 0 0 0 2px var(--color-primary)`.

## Vocabulário de componentes

### Botões
| Papel | Cor | Exemplo |
|-------|-----|---------|
| Primário | `--color-primary` (fundo cheio) | "Adicionar" |
| Ação | `--color-action` (fundo cheio) | "Sortear time base", "Registrar partida" |
| Perigo/link | `--color-primary` (contorno, fundo transparente) | "Limpar convidados", "Limpar histórico" |
| Info/link | `--color-link` (contorno) | "Usar esta base" |
| Ícone | transparente, cor semântica | ✓ toggle, ✎ editar, × remover |
| Alerta pulsante | `--color-primary` + `pulse 1.2s infinite` | "Sortear novamente (obrigatório)" |

### Tags / badges (`font-size: 0.65–0.7rem`, contorno 1px, radius 5px)
| Tag | Cor | Significado |
|-----|-----|-------------|
| **conv.** / **Convidado** | `--color-gold` | Jogador de fora |
| **extra** | `--color-info` | Regular avulso (voltou de lesão, sem time base) — jogando como convidado |
| **falta** | `--color-primary` | Está na base mas não confirmou presença |
| **Lesionado** | `--color-primary` | Jogador desativado |

### Padrões
- **Card:** `background: --color-surface; border-radius: 12px; padding: 16px`.
- **Estrelas de nota:** inativa `#444`, ativa/hover `--color-gold` (`★`, 1–5 por categoria).
- **Check de confirmação:** caixa 22px, borda `#555`; confirmado = fundo `--color-success` com ✓ branco; borda esquerda verde no item confirmado.
- **Tabela de classificação:** cabeçalho `--color-text-subtle`; V verde, D vermelha, Pts dourado; 1º lugar destacado em dourado.

## Como usar

```html
<link rel="stylesheet" href="design/tokens.css">
<style>
  .meu-card {
    background: var(--color-surface);
    border-radius: var(--radius-card);
    padding: var(--space-16);
    color: var(--color-text);
    font-family: var(--font-family);
  }
  .meu-botao-acao {
    background: var(--color-action);
    color: var(--color-text-on-color);
    border-radius: var(--radius);
    padding: var(--space-14) var(--space-20);
  }
</style>
```
