# Equicord Themes

Temas para o [Equicord](https://github.com/Equicord). Cada tema mora na sua pasta com CSS, README e licença quando aplicável. Instalação recomendada por **import online**: o `@updateUrl` no CSS deixa o Equicord puxar atualizações sozinho.

## Estado dos temas

| Tema | Estado | O que faz | Plugin |
|---|---|---|---|
| [OperaGX](OperaGX/) | **Ativo — v4.5.6** | Fork do OperaGX do HawkRatio: vermelho GX, Chakra Petch nos títulos, scanlines suaves, glow e HUD. Sem salto de layout, espaçamento nativo, ícones de servidor 100% nativos e controles de chamada arredondados. | Nenhum |
| [FolderColumns](FolderColumns/) | **Arquivado / descontinuado (2026)** | Multi-coluna na sidebar do BetterFolders. Sem manutenção; MIT. | BetterFolders (sidebar) |

FolderColumns não é mais recomendado. Para colunas na barra de servidores, usa um plugin dedicado e atualizado. O OperaGX não interfere nisso.

## Instalar

Equicord > Settings > Themes > **Add theme** e cola a URL raw:

- OperaGX (Equicord): `https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/OperaGX/OperaGX.css`
- OperaGX (BetterDiscord): `https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/OperaGX/OperaGX.theme.css`
- FolderColumns (arquivado): `https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/FolderColumns/FolderColumns.css`

Depois ativa o tema. Mudanças no GitHub chegam sozinhas via `@updateUrl`.

Modo manual, sem auto-update: copia o `.css` para `%AppData%\Equicord\themes` e ativa em Settings > Themes.

```powershell
Invoke-WebRequest "https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/OperaGX/OperaGX.css" `
  -OutFile "$env:APPDATA\Equicord\themes\OperaGX.theme.css"
```

No BetterDiscord, usa `OperaGX/OperaGX.theme.css`: é um único arquivo com meta no início e `@import` remoto do CSS completo, formato exigido pela documentação do BetterDiscord. O `@updateUrl` dele aponta para o próprio wrapper.

O CSS de implementação continua em um único `OperaGX.css` de propósito: os dois loaders consomem uma URL única, e quebrar em vários arquivos exigiria bundle ou `@import`s extras, aumentando latência sem ganho real. A organização é feita por seções comentadas dentro do arquivo.

## OperaGX v4.5.6

- Base oficial do [HawkRatio](https://github.com/HawkRatio/OperaGXTheme) v1.2 via CDN, sem `@layer` — a cascata é a mesma dos outros temas: ordem e especificidade decidem, sem o CSS normal do Discord ganhar das correções do fork.
- **Salto de layout eliminado:** o upstream adiciona `padding-top:8px` + `margin-top:8px` e `margin-left:6px`/`border-top` em cada linha de canal. O fork zera esses valores por prefixo, então a sidebar volta ao tamanho nativo e as mensagens não descem ~23px.
- Títulos usam "OperaGX Chakra", família local com `font-display: optional` — as faces `swap` da base não são carregadas, então não há FOUT/reflow. `content-visibility: visible` nas mensagens evita reserva invisível de 72px por linha.
- `--font-primary`, `--font-display` e `--cf` foram devolvidos ao stack nativo do Discord. O upstream os forçava para Chakra Petch com `!important`, e isso vazava para componentes que não são títulos mesmo com o `body` nativo. `--categorySize: 12px` para categorias e grupos de membros.
- Tamanhos nativos do Discord restaurados: o tema não força largura dos ícones, gap, raio, padding, margin nem `object-fit` na barra de servidores. Só neutralize `clip-path`, a máscara de fade e o `--cp-size` que o upstream aplica. `box-sizing: border-box` absorve bordas do upstream sem mexer na caixa do ícone.
- Sem `zoom` forçado, `letter-spacing` ou `font-weight` extra em títulos e categorias. O texto das mensagens volta ao peso 400 do Discord (a base forçava 500) e o `<strong>` perde o `text-shadow` da base — acaba com o "zoom" visual ao ativar o tema. QuickCSS do usuário pode sobrepor o tema; desative-o ao diagnosticar.
- O `margin-top: calc(1rem - 1px) !important` que o upstream aplica em linhas agrupadas via `style*="margin-top"` é zerado — era a origem do salto vertical ao ativar o tema.
- Chamadas mantêm os 32px nativos do Discord; controles, tiles e avatares são arredondados sem forçar largura/altura. Highlight fica contido na mensagem.
- Sidebar de canais e DMs redimensionável: controles de resize com `pointer-events`, `touch-action: none`, largura 6px e `cursor: col-resize`.
- Scanlines vermelhas fracas com fade nas bordas; reduced-motion e high-contrast respeitados.
- Sem FolderColumns nativo. Documentação técnica completa em [`docs/THEME_ARCHITECTURE.md`](docs/THEME_ARCHITECTURE.md).

Opções pelo modal **Edit** do Equicord: `gx_glow_intensity_s` (55, 0–100) e `gx_scanline_opacity_s` (4, 0–20). Cores, fontes e raios ficam em `:root` no topo do CSS.

Se o layout continuar com cara de zoom, confere duas coisas antes de culpar o tema: o `@version` no topo do CSS precisa ser `4.5.6` (o Equicord pode estar servindo cache antigo — reativa o tema) e o QuickCSS precisa estar desligado. QuickCSS é aplicado depois do tema e sobrepõe as correções.

## Requisitos e compatibilidade

- Equicord ou BetterDiscord com suporte a `@import` em CSS. O wrapper `OperaGX.theme.css` segue o formato de arquivo único da documentação do BetterDiscord.
- Internet para o CDN do HawkRatio e para os arquivos da fonte dos títulos. Sem internet, o CSS do fork ainda entra, mas a base do upstream não carrega e os títulos caem no fallback nativo.
- Funciona com o Discord claro/escuro. Nenhum plugin é obrigatório.

## Licenças

- OperaGX: fork derivado do [HawkRatio/OperaGXTheme](https://github.com/HawkRatio/OperaGXTheme), mantido por NotVictorNW.
- FolderColumns: **MIT**, Copyright (c) 2026 NotVictorNW — ver [`FolderColumns/LICENSE`](FolderColumns/LICENSE). Uso, cópia, correção, melhoria e redistribuição permitidos.

## Contribuir

1. Pasta nova em PascalCase com o `.css` e o `README.md`.
2. `@updateUrl` apontando para a raw URL do arquivo.
3. PR com o que mudou e como validar.

## Autor

NotVictorNW — https://github.com/NotVictorNW
