# OperaGX (NotVictorNW Fork)

Fork do OperaGX do [HawkRatio](https://github.com/HawkRatio/OperaGXTheme) (antigo L-Ratio) com otimizações de performance, acessibilidade e ~30 melhorias de UX. Mantém a identidade GX do fork: vermelho, Chakra Petch, scanlines, glow e HUD. Sem FolderColumns nativo — colunas ficam com plugins dedicados.

## Pré-requisitos

- Equicord instalado — https://github.com/Equicord
- **Internet** — o tema carrega a base do HawkRatio via CDN e os arquivos da fonte dos títulos. Sem internet, o CSS do fork ainda entra, mas a base do upstream não carrega e os títulos usam fallback nativo.

## Sincronização v4.1

- v4.5.6: base sem `@layer` — a cascata volta a ser a dos outros temas (ordem + especificidade), o que impede o CSS normal do Discord de vencer as correções no toggle. Controles de chamada (`actionButtons_*`/`lookFilled.colorBrand`), tiles e avatares agora são mirados pelas classes reais e ficam arredondados sem mexer nos 32px nativos. O `margin-top` forçado pelo upstream em linhas agrupadas foi zerado: medição no harness caiu de `15px` para `0px`, sem mudar altura de mensagem/título.
- v4.5.5: nenhuma geometria de ícone de servidor é forçada — o fork deixou de impor raio, margin, padding, width e hover nos cards/imagens e só desliga o `clip-path` do upstream. Medição no harness: card e ícone passam a herdar `border-radius`/`margin`/`padding` nativos em vez dos `12px`/`0px` injetados. Entrada `OperaGX.theme.css` adicionada para BetterDiscord (meta + `@import` remoto, formato da doc oficial).
- v4.5.4.3: texto das mensagens com `font-weight: 400` (a base forçava 500) e `<strong>` sem `text-shadow` — mesma ideia do Ultra (BetterDiscord), sem mexer em tamanhos nativos. Geometria medida idêntica.
- v4.5.4.2: `--font-primary`, `--font-display` e `--cf` devolvidos ao stack nativo do Discord — o upstream os forçava para Chakra Petch com `!important` e isso vazava para componentes comuns mesmo com o `body` nativo. Títulos agora usam "OperaGX Chakra" (`display:optional`), família local que evita as faces `swap` da base: nenhuma face `swap` é carregada e a geometria medida continua idêntica. `zoom` forçado removido. Documentação técnica em [`docs/THEME_ARCHITECTURE.md`](../docs/THEME_ARCHITECTURE.md).
- v4.5.4.1: fonte com `display=optional` (sem FOUT/reflow), `content-visibility: visible` nas mensagens (sem reserva 72px), `box-sizing: border-box`/`margin: 0` absorvendo bordas e margens do upstream.
- v4.5.4: salto de layout eliminado — o `padding-top:8px` + `margin-top:8px` e o `margin-left:6px`/`border-top` que o upstream injeta em cada linha de canal são zerados por prefixo. `--categorySize: 12px`, corpo com fonte nativa (`gg sans`/`Noto Sans`/Helvetica/Arial) e Chakra Petch mantida só nos títulos. Resizer com `pointer-events`/`touch-action: none`.
- v4.5.3: tamanhos nativos do Discord restaurados na barra de servidores — o tema só retira `clip-path`, a máscara de fade e o `--cp-size`, sem forçar tamanho/gap/raio/padding. Sem `letter-spacing`/`font-weight` extras: some o "zoom" visual ao ativar o tema. Resizer da sidebar com `pointer-events`, `touch-action: none`, 6px e `cursor: col-resize`.
- v4.5: densidade compacta — servidores em 48px com gap 4px usando as classes reais do upstream, chamadas em 32px, `contain-intrinsic-size` de 72px e reduced-motion alinhado. Scanlines continuam suaves e a sidebar continua redimensionável.
- Base agora importada do CDN oficial `https://hawkratio.github.io/OperaGXTheme/release/main.css` (v1.2), sem camadas. O link antigo do L-Ratio devolvia 404.
- v4.1.2: `--background` não aponta mais para `--opera-gx-bg-1` (o JPEG do upstream) e as scanlines saíram de `#app-mount::before` para `::after`, que o upstream não usa. A interface deixa de ser coberta pelo wallpaper mesmo que o loader atrase o import.
- v4.1.1: a base do upstream entrava como CSS sem camada e cobria os overrides do fork — inclusive o `background` do `::before` do `#app-mount`, que tapava a interface com a imagem de fundo em `z-index: 9999`. Isso motivou o uso de layers na época; em v4.5.6 as layers foram removidas porque o CSS normal unlayered do Discord voltou a ganhar das correções no toggle.
- v4.2: FolderColumns nativo removido; o tema não depende mais de BetterFolders para nada.
- v4.4: ícones de servidor em 52px com imagem inteira e raio 16px, participantes de call circulares em 36px, botões de câmera/transmissão/config arredondados, highlight de mensagem dentro do balão e scanlines vermelhas suaves com fade nas bordas. A sidebar de canais volta a ser redimensionável pelo Discord.
- v4.3: raios e bordas restaurados (o upstream forçava `--radius-*: 0 !important`), `clip-path` e `contain: paint` fora de botões/canais/overlays, menus com submenus visíveis, hovers sem salto, contraste de labels/subtexto corrigido e barras de progresso persistentes.
- Seletores da base acompanham a renomeação de classes do Discord de 07–08/11/2026; os seletores do fork usam `[class*=]` resistente a hash.
- Correções do upstream que passam a valer: lista de servidores atrás do perfil, DMs atravessando a barra e borda do embed de mensagem de voz.
- A paleta vermelha do fork continua aplicada. Se preferires o visual burgundy do upstream v1.2, troca em `:root`: `--accent-color-1: 222,67,100`, `--accent-color-2: 222,67,100`, `--border-brightness: 1`, `--gx-red: 181,32,63`, `--gx-red-2: 130,27,49` e `--customFont: unset`. Para voltar a ter o wallpaper JPEG, troca também `--background` para `var(--opera-gx-bg-1)`.

## O que mudou face ao original

**Performance**
- Animar só `transform` / `opacity` / `filter` (GPU); nada de `box-shadow` ou `width` animados.
- Sem `backdrop-filter`. `will-change` só no hover.
- `content-visibility: visible` nas mensagens — sem reserva `contain-intrinsic-size` de 72px que causava salto ao rolar. `contain: layout style` só em listas, nunca `paint`.

**Acessibilidade**
- Cascata sem camadas, igual à dos outros temas — `!important` fica só onde o upstream também usa (`--radius-*`, fontes, `clip-path`, `margin-top`) e a ordem do import + especificidade resolvem o resto.
- `prefers-reduced-motion` e `prefers-contrast: more` tratados.

**UX**
- Status indicator com ring, account panel redesenhado, popout/modal de perfil estilo GX, raios consistentes em menus/tooltips/scrollbars, replies com accent, mentions pill animada, unread com glow, call ring, toasts com slide. Hover mantém a forma e não empurra layout; sidebar de canais flexível, servidores com imagem completa e chamada com avatares circulares.

Se o layout continuar igual depois de atualizar, confirma o `@version: 4.5.6` no CSS carregado e desativa o QuickCSS. Reativa o tema para romper cache; QuickCSS é aplicado depois e vence as correções do fork.

## Instalação

Recomendado — import online (auto-update): em Equicord > Settings > Themes cola
`https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/OperaGX/OperaGX.css`

Ou copia o `OperaGX.css` para `%AppData%\Equicord\themes` e ativa em Settings > Themes (sem auto-update).

No BetterDiscord, instala `OperaGX.theme.css` — o único arquivo necessário é o wrapper com meta no início e `@import` do CSS completo, como manda a documentação oficial. O auto-update vem do `@updateUrl` dele.

Se quiseres colunas na barra de servidores, instala um plugin dedicado — o tema não interfere mais nisso.

## Configuração

Pelo modal Edit:

| Variável | Descrição | Default | Min | Max |
|---|---|---|---|---|
| `gx_glow_intensity_s` | Intensidade do glow (0=desligado, 100=máximo) | 55 | 0 | 100 |
| `gx_scanline_opacity_s` | Opacidade das scanlines vermelhas (0=desligado, 20=máximo) | 4 | 0 | 20 |

Cores, fontes, transparências e raios ficam em `:root` no topo do CSS — edita diretamente se quiseres mudar o vermelho GX ou a fonte.

## Suporte

Discord: [RrXeKaVZsm](https://discord.gg/RrXeKaVZsm)

## Créditos

Original (upstream ativo): HawkRatio — https://github.com/HawkRatio/OperaGXTheme
Fork: NotVictorNW
