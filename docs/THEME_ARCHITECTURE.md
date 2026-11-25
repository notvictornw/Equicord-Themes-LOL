# OperaGX (NotVictorNW Fork) — arquitetura e guia de desenvolvimento

Este documento explica **como o tema funciona**, **por que cada decisão existe**, **onde mexer** e **o que ainda pode ser melhorado**. Ele é voltado a devs que precisam manter ou forkar o tema sem quebrar a identidade visual nem reintroduzir saltos de layout.

Público assumido: uma pessoa usando Equicord (ou outro mod compatível com temas CSS no estilo Vencord) com o tema aplicado na inicialização. Não há build, pacote ou servidor — o "deploy" é um arquivo CSS servido por `raw.githubusercontent.com`. Não existe documentação oficial independente do BetterCord; o contrato de tema é o mesmo CSS/Vencord e o mesmo `@import`.

---

## 1. Visão geral

O tema é uma **camada de personalização** sobre o tema base `HawkRatio/OperaGXTheme` v1.2. Ele não reimplementa o Discord: importa a base por CDN e sobrescreve o que o base quebra ou deixa feio.

Identidade preservada:

- vermelho GX (`250,30,70` / `255,55,109`),
- Chakra Petch **apenas em títulos/destaques**,
- scanlines vermelhas fracas com fade nas bordas,
- glow, HUD e acentos de borda,
- nenhum FolderColumns nativo (colunas são responsabilidade de plugins).

O tema **não define tamanho de ícone de servidor, gap, raio forçado nem `object-fit`**. Esses valores são nativos do Discord. O fork só neutraliza o que o upstream adiciona e quebra o layout.

---

## 2. Estrutura do repositório

| Caminho | Papel |
| --- | --- |
| `OperaGX/OperaGX.css` | Tema ativo. Todo o comportamento vive aqui. |
| `OperaGX/README.md` | Documentação curta para usuário final: instalação, opções, créditos. |
| `FolderColumns/FolderColumns.css` | Tema antigo, **arquivado e descontinuado**. MIT. |
| `FolderColumns/README.md` | Aviso de arquivamento e licença. |
| `FolderColumns/LICENSE` | MIT, Copyright (c) 2026 NotVictorNW. |
| `README.md` (raiz) | Status dos temas, instalação rápida, requisitos, licenças. |
| `docs/THEME_ARCHITECTURE.md` | Este arquivo. |

Não existe `package.json`, pipeline de build nem suite de testes automatizados. A verificação é feita com uma página HTML local e leitura de `getComputedStyle()` no navegador.

---

## 3. Como o tema é carregado

O Equicord cria um `<style id="vencord-themes">` e coloca links de tema como:

```css
@import url("https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/OperaGX/OperaGX.css");
```

Ou seja: **o tema já entra dentro de um `@import`**. Isso vem do loader: `Equicord/src/api/Themes.ts` monta `themesStyle.textContent = Array.from(links).map(link => \`@import url("${link.trim()}")\`).join("\n")` em `<style id="vencord-themes">`, com suporte a `@light`/`@dark` e cache-busting em `vencord:///themes/<id>?v=<Date.now()>`. `@import` aninhado é válido, mas a ordem é rígida.

Regras MDN aplicáveis (confirmadas em `https://developer.mozilla.org/en-US/docs/Web/CSS/@import`):

- `@import` deve vir antes de todas as regras, exceto `@charset` e `@layer statement`;
- `@import url(...) layer(base)` coloca o stylesheet importado dentro da camada `base`;
- as declarações importadas cascateiam **como se estivessem escritas no ponto do import**.

Por isso o topo do arquivo é, nesta ordem:

```css
@layer base, opengx, overrides, states, a11y;

@import url(https://hawkratio.github.io/OperaGXTheme/release/main.css) layer(base);

@font-face { font-family: "OperaGX Chakra"; /* ... */ font-display: optional; }
```

> **Erro histórico:** a base já foi importada sem `layer(base)`. CSS fora de camada sempre vence qualquer `@layer`. Isso fazia os overrides do fork perderem e o wallpaper base64 do upstream cobrir o app. Corrigido em v4.1.1.

---

## 4. Sistema de camadas

```css
@layer base, opengx, overrides, states, a11y;
```

A ordem é de menor para maior prioridade. Regra geral: **o que corrige o upstream vai em `base`**; o que é identidade do fork vai nas camadas seguintes.

| Camada | Conteúdo | Exemplo |
| --- | --- | --- |
| `base` | Neutralização do upstream: raios forçados a `0px !important`, `clip-path`, margens, máscaras, variáveis de fonte globais e `box-sizing`. | `--radius-md: 12px !important` em `.theme-dark`; `--font-primary` nativo; `margin-top:0` em linhas de canal. |
| `opengx` | Tokens e elementos próprios do fork. | scanline, glow, keyframes, variáveis de cor. |
| `overrides` | Visual do fork sobre componentes reais. | botões, avatares, popout, menus, mensagens, HUD. |
| `states` | Otimizações e estado de scroll. | `contain: layout style` em listas; `content-visibility: visible` em mensagens. |
| `a11y` | `prefers-reduced-motion`, `prefers-contrast: more`, foco. | desliga scanlines e animações. |

### Regra de ouro para `!important`

Só use `!important` na camada `base`, e somente para empatar uma declaração `!important` do upstream **na mesma camada**. Um `!important` em camada posterior **não vence** um `!important` em camada anterior. É por isso que os raios foram restaurados dentro de `@layer base`, e não em `overrides`.

---

## 5. Base upstream e deriva de hash

A base é servida por:

```
https://hawkratio.github.io/OperaGXTheme/release/main.css
```

É a mesma versão publicada em `HawkRatio/OperaGXTheme/release/main.css` (~390 KB, v1.2). O CDN antigo `l-ratio.github.io` devolve 404.

O upstream usa classes com hash curta que muda conforme o Discord atualiza:

- `.guilds_pog62O`, `.wrapper_5dqHTG`, `.tree_5dqHTG`
- `.guildList_c8c448`, `.card_e90879`, `.loaded_e90879`
- `.messageListItem_SY_P2T`, `.messageContent_2i8Bfm`
- `.containerDefault-3GGEv_`, `.name-IbjUBS`

**Nunca confie apenas no sufixo.** O v4.5.2 trocou seletores travados em hash por prefixo:

```css
nav[class*="guilds_"] [class*="card_"]
nav[class*="guilds_"] [class*="tree_"]
#app-mount :where([class*="containerDefault"], [class*="membersGroup"])
```

A validação usa sufixos diferentes dos reais (`guilds_ZZZZ`, `card_AAAA`, `tree_XXXX`) para provar que a regra não depende do hash.

Sempre que o upstream atualizar, rode o diff de `main.css`. A maioria dos updates é só troca de hash; se aparecer regra nova com `!important`, ela precisa de neutralização em `base`.

---

## 6. Variáveis

### 6.1 Opções expostas na UI do Equicord (`@var range`)

| Nome | Default | Faixa | Efeito |
| --- | --- | --- | --- |
| `gx_glow_intensity_s` | `55` | 0–100 | Multiplica a intensidade de glow/sombra. `0` desliga. |
| `gx_scanline_opacity_s` | `4` | 0–20 | Opacidade das scanlines vermelhas. `0` desliga. |

O sufixo `_s` gera a variável runtime `_v` usada no CSS (`--gx-glow-int`, `--gx-scanline-opacity`).

### 6.2 Tokens principais em `:root`

| Variável | Valor | Por quê |
| --- | --- | --- |
| `--accent-color-1` | `250,30,70` | Vermelho GX. Base de bordas, glow e highlight. |
| `--accent-color-2` | `255,55,109` | Versão clara para texto sobre fundo escuro (contraste). |
| `--gx-red`, `--gx-red-2` | `255,34,34`, `255,66,66` | Acento de chamada/estado. |
| `--fullwhite-font`, `--normal-font`, `--muted-font` | `220,222,225`, `188,190,194`, `108,112,120` | Escala de texto. |
| `--customFont` | `"OperaGX Chakra"` | Família local com `font-display: optional`, usada **só em títulos**, nunca no `body`. O nome próprio impede o uso das faces `swap` do upstream. |
| `--font-primary`, `--font-display`, `--cf` | stack nativa (`gg sans`/`Noto Sans`/Helvetica/Arial) | Redefinidas em `.theme-*` dentro de `base` com `!important`: o upstream as força para `var(--cf)` e isso vazava para componentes que não são títulos. |
| `--codeBlocksCustomFont`, `--cbcf` | stack nativa de monospace | Blocos de código não usam Chakra Petch. |
| `--border-brightness` | `0.35` | upstream consome essa var para bordas; com `0` elas ficavam invisíveis. |
| `--radius-sm/md/lg` | `8/12/16px` | Restauradas em `base` porque o upstream força `0px !important`. |
| `--categorySize` | `12px` | upstream usa `var(--categorySize, 15px)` em categorias/grupos. |
| `--gx-highlight-color` | `250,30,70` | Cor do highlight de mensagem. |
| `--gx-highlight-width` | `2px` | Espessura da borda interna do highlight. |
| `--gx-scanline-scale` | `3px` | Período da repeating-linear-gradient. |
| `--guild-icon-hover-radius`, `--guild-icon-selected-radius` | `16px` | upstream consome, mas não define. |
| `--cp-size` | `0px` (em `nav[class*="guilds_"] *`) | upstream usa em `clip-path: polygon()`; zerar evita canto cortado no ícone. |
| `--background` | `rgb(var(--darker-background-color))` | **Não** aponta para `--opera-gx-bg-1`. O JPEG base64 do upstream cobria a tela. |

Variáveis que existiam para FolderColumns (`--guildsize`, `--guildgap`, `--server-listing-width`) foram removidas em v4.2 e não devem voltar.

---

## 7. Estratégia de seletores

1. **Prefixo antes de hash:** `[class*="button_"]`, `nav[class*="guilds_"]`, `[class*="messageListItem_"]`.
2. **Escopo quando o prefixo é genérico:** `nav[class*="guilds_"] img` em vez de `img` solto.
3. **`:where()` para zero especificidade** em grupos de neutralização amplos.
4. **Evite `[class^=...]`:** o prefixo muda (`message-` virou `messageListItem_`). Use `*=`.
5. **Nunca delete 99 tokens "ausentes" do upstream sem checar:** muitos são elementos reais que a base não estiliza (status, toast, note, switch).

Se precisar adicionar um seletor novo, confirme o nome no DOM real do Discord antes de escrever a regra.

---

## 8. Conflitos conhecidos do upstream e como o fork lida com eles

| Upstream | Sintoma | Neutralização no fork |
| --- | --- | --- |
| `--radius-*: 0px !important` em `.theme-dark/.theme-light` | tudo quadrado | redeclaração em `@layer base` com `!important` e mesma especificidade |
| `border-radius:0 !important` em popout/menu/tooltip | cantos mortos | `border-radius:12px !important` em `base` |
| `clip-path: polygon(var(--cp-size)...)` em botões/cards | cantos cortados | `clip-path:none !important` + `--cp-size:0px` |
| `.tree_5dqHTG` com `mask-size:100% 16px` | ícones de servidor com fade em cima/baixo | `mask:none`/`mask-image:none`/`mask-size:auto` |
| `.card_e90879 { margin:1px }` | ícone 72px vira 70px | `margin:0` no card/loaded |
| `padding-top:8px` + `margin-top:8px` em `.containerDefault_*` | linha de canal duplicava altura | `padding-top:0`, `margin-top:0`, `margin-left:0`, `border-top:0` em `base` |
| `body { font-family: var(--cf) !important }` → Chakra Petch global | reflow e títulos mais altos | `body` com stack nativo em `base` |
| `--font-display`/`--font-primary`/`--cf` = `var(--cf) !important` | componentes comuns herdam Chakra Petch mesmo com `body` nativo | redeclaração em `base` com a stack nativa e `!important`; `--customFont` continua só nos títulos |
| `@font-face` "Chakra Petch" com `font-display:swap` no upstream | FOUT troca métricas depois do primeiro paint | títulos usam "OperaGX Chakra" com `display:optional`; nenhuma face `swap` é solicitada |
| `#app-mount .messageContent_* { font-weight:500 }` e `.markup_* strong { text-shadow:... }` no upstream | texto mais espesso, leitura com cara de zoom | `#app-mount [class*="messageContent_"] { font-weight:400 }` e `[class*="markup_"] strong { text-shadow:none }` em `base` |
| `background: ..., var(--background)` com JPEG base64 | tela coberta | `--background` cor sólida |
| `overflow:hidden` em menu/submenu | submenu inalcançável | `overflow:visible !important` |
| `border-radius:0 !important` em scrollbar | barra quadrada | raio `999px` em `base` |
| highlight de mensagem com `rgb(var(--accent), 0.2)` fora do balão | seleção "vazada" | neutralize tint e pinte o `[class*="messageContent_"]` internamente |
| `.avatar_* { width/height:32px !important }` em call | avatar quadrado/pequeno | `border-radius:50%`, `overflow:hidden`, `clip-path:none` |

---

## 9. Performance e estabilidade

Decisões tomadas para evitar CLS (Cumulative Layout Shift) e jank:

- **Família de título própria com `font-display:optional`.** O fork não importa o Google Fonts e nunca usa a família "Chakra Petch" do upstream (`swap`). "OperaGX Chakra" tem seis `@font-face` locais (400/600/700, latim + latim-ext) com `optional`: se a fonte não estiver pronta no primeiro paint, o fallback é usado sem troca de métricas. Verificado no harness: `0` faces `swap` carregadas.
- **Sem `content-visibility:auto` em mensagens.** O valor reservado era `contain-intrinsic-size: auto 72px` — sintaxe de **um valor**, aplicada aos dois eixos. Mensagens fora da tela reservavam 72×72px e causavam salto ao rolar. Agora `content-visibility:visible`.
- **`contain: layout style` restrito** a `[class*="members_"]` e `[class*="scroller_"]`. Nunca `paint`: ele corta popout/tooltip.
- **`box-sizing:border-box`** em nav e sidebar para absorver bordas de 1–2px do upstream sem mudar largura.
- **QuickCSS do usuário fica fora de camadas** e, por isso, vence qualquer `@layer` do tema. Se o layout "parecer zoom" só com o tema ligado, desative QuickCSS primeiro; a FAQ oficial do Equicord recomenda exatamente isso antes de culpar o tema.
- **Scanlines**: `z-index:1`, `pointer-events:none`, `mix-blend-mode:normal`, opacidade ~0.024 e máscara com fade de 80px no topo/base. Nunca use `z-index:9999`: isso cobria modais e composer.
- **Animações** só com `transform`/`opacity`/`filter`. `@keyframes gx-glitch` foi removido porque dependia de `data-text` que o Discord não fornece.

---

## 10. Acessibilidade

- `@media (prefers-reduced-motion: reduce)` desliga animações e esconde `html #app-mount::after` (scanlines).
- `@media (prefers-contrast: more)` clareia `--normal-font`/`--muted-font` e `--link-text`.
- Texto de categoria usa `--accent-color-2` em vez de `--accent-color-1` com opacity — contraste maior.
- Overlay de scanline é puramente decorativo e não intercepta ponteiro.
- Foco visível é preservado; não sobrescreva `:focus-visible` sem alternativa.

---

## 11. Como verificar alterações

1. Crie uma página HTML temporária na raiz do repo com `<style id="vencord-themes">@import url("OperaGX/OperaGX.css");</style>`.
2. Use classes com **sufixos diferentes dos reais** para provar resiliência a hash; para o chat, os sufixos atuais observados são `.content_z_gUBg`, `.messageListItem_SY_P2T` e `.messageContent_2i8Bfm`. `.content_4TfzT5` é o divisor de "novo dia" (`width:20%`), não o container do chat.
3. Abra a página no navegador.
4. Meça `getBoundingClientRect()` e `getComputedStyle()` **antes e depois** de injetar o import.
5. Compare: altura da linha de mensagem, posição do composer, largura da sidebar/barra, `contentVisibility` (`visible`) e `containIntrinsicSize` (`none`). Confira também `--font-primary`/`--cf` (stack nativa), `--customFont` (`"OperaGX Chakra"`), `fontFamily` de título/corpo, `fontWeight` de `.messageContent_*` (`400`) e `document.fonts`: `0` faces com `display:"swap"` e `status:"loaded"`.
6. Confira o console do navegador em nível de erro.
7. Apague a página temporária e feche a aba antes de commitar.
8. Rode `git diff --check` e contagem de chaves (`{` vs `}`).

Verificação v4.5.4.2: harness com o import do tema em `<style id="vencord-themes">`, comparando `HEAD` (v4.5.4.1) contra a versão atual. Geometria idêntica nos dois: título `y=82.4375`, `h=42`; mensagem `y=145.875`, `h=18`; wrapper da mensagem `y=145.875`, `h=18`; nav/sidebar/grupo inalterados. Diferença funcional: `--font-primary`, `--font-display` e `--cf` passaram de `"Chakra Petch"` para a stack nativa; `--customFont` passou para `"OperaGX Chakra"`; `0` faces `swap` carregadas contra `2` antes; `zoom` calculado continua `1` sem declaração no tema. Medição anterior (v4.5.4.1): mensagem 20px→20px, composer y 104→105, sidebar 240→240, mount 841→841.

Última verificação (v4.5.4.3): mesmo harness, agora comparando `HEAD` (v4.5.4.2) contra a versão atual, com `<strong>` dentro de `.messageContent_*`. `fontWeight` da mensagem caiu de `500` para `400`; `<strong>` permanece `600` com `textShadow: none`. Geometria idêntica: título `y=82.4375`, `h=42`; mensagem e wrapper `y=145.875`, `h=18`; nav `x=8`, `y=8`, `w=1264`, `h=18`. `document.fonts`: `22` faces Chakra no total, `16` `swap`/`0` carregadas, `6` `optional`/`1` carregada nos dois estados — a correção não trouxe nenhuma face nova.

---

## 12. Como estender com segurança

### Faça

- Adicione novas opções como `@var range ... _s` no header e use `--nome_v` no CSS.
- Coloque correção de upstream em `@layer base`.
- Coloque identidade visual em `opengx`/`overrides`.
- Documente qualquer `!important` novo com comentário dizendo qual regra do upstream ele vence.
- Teste com sufixo de classe diferente.

### Não faça

- Não importe a base fora de `@layer base`.
- Não force `width`/`height` em ícone de servidor, nem `object-fit`.
- Não use `contain: paint` em container que tenha popout.
- Não use `z-index` alto na scanline.
- Não restaure `--background: var(--opera-gx-bg-1)` sem antes neutralizar o `::before`/`::after` do `#app-mount`.
- Não reative FolderColumns nativo. Use plugin.

Checklist antes de commit:

```
[ ] @version atualizado
[ ] mudança documentada no header do CSS
[ ] README do tema e README raiz atualizados
[ ] página temporária removida
[ ] braces balanceados
[ ] git diff --check limpo
[ ] commit + push
```

---

## 13. O que pode ser adicionado ou melhorado

Ideias ordenadas por valor, sem obrigatoriedade:

1. **Teste no Discord real automatizado** — hoje toda verificação é sintética. Um screenshot + DOM dump do cliente real eliminaria a última milha de incerteza sobre hash e layout.
2. **Métricas de fallback da "OperaGX Chakra"** — `@font-face` com `size-adjust`, `ascent-override`, `descent-override` e `line-gap-override` permitiria `display:swap` sem CLS e exibir Chakra já na primeira visita; hoje `optional` prefere estabilidade e usa fallback até a fonte estar em cache.
3. **Sincronização upstream automatizada** — script que baixa `main.css`, faz diff e avisa quando surgir `!important` novo.
4. **Opção para ligar/desligar glow, scanline e highlight** — hoje só intensidade; um `@var boolean` por efeito daria controle fino.
5. **Polimento da call UI** — upstream quase não estiliza chamadas; há espaço para participantes em grid, falante ativo com ring e botões com estado de mute/vídeo.
6. **Limpeza de seletores mortos** — 99 de 158 tokens `[class*=]` não aparecem no `main.css`. Muitos são reais no Discord, mas uma auditoria com DOM dump permitiria remover os realmente inexistentes.
7. **Aviso de versão no tema** — `@invite` e `@updateUrl` já existem; um badge no README com última sincronização ajudaria usuários a saberem se estão com cache antigo.
8. **Tradução dos READMEs** — hoje tudo em português. Uma versão em inglês ampliaria o uso do fork.

---

## 14. Histórico resumido

| Versão | Mudança |
| --- | --- |
| 4.0 | Reescrita completa: layers, tokens, scanlines, glow, HUD. |
| 4.1 | Base sincronizada com HawkRatio v1.2, CDN trocado de L-Ratio (404) para HawkRatio. |
| 4.1.1 | Import para `layer(base)`; overrides voltam a vencer. |
| 4.1.2 | Wallpaper base64 neutralizado; scanlines em `::after`. |
| 4.2 | FolderColumns nativo removido. |
| 4.3 | Raios/bordas restaurados, clip/contain corrigidos, contraste e hovers. |
| 4.4 | Servidores, call, highlight e scanlines refinados. |
| 4.5 | Densidade compacta. |
| 4.5.1 | Barra 72px e ícones sem fade/clip. |
| 4.5.2 | Seletores por prefixo, imunes a hash. |
| 4.5.3 | Tamanhos nativos devolvidos ao Discord. |
| 4.5.4 | Spacing do upstream zerado; fonte base nativa. |
| 4.5.4.1 | `display=optional`, sem reserva 72px, `box-sizing` em nav/sidebar. |
| 4.5.4.2 | Variáveis de fonte globais neutralizadas, família de título própria (`"OperaGX Chakra"`) sem faces `swap`, densidade nativa preservada e `zoom` forçado removido. |
| 4.5.4.3 | Peso 400 no texto das mensagens e `text-shadow` removido do `<strong>`, seguindo o padrão do tema Ultra (BetterDiscord). |

---

## 15. Créditos e licença

- Tema original: [HawkRatio/OperaGXTheme](https://github.com/HawkRatio/OperaGXTheme).
- Fork e manutenção: NotVictorNW.
- Suporte: `https://discord.gg/RrXeKaVZsm`.
- `FolderColumns` é MIT (arquivado). O OperaGX fork segue a licença do upstream quando aplicável; consulte o README raiz para o estado atual de cada tema.
