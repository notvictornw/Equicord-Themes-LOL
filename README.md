# Equicord Themes

Temas para o [Equicord](https://github.com/Equicord). Cada tema mora na sua pasta com CSS, README e licença quando aplicável. Instalação recomendada por **import online**: o `@updateUrl` no CSS deixa o Equicord puxar atualizações sozinho.

## Estado dos temas

| Tema | Estado | O que faz | Plugin |
|---|---|---|---|
| [OperaGX](OperaGX/) | **Ativo — v4.5.4.1** | Fork do OperaGX do HawkRatio: vermelho GX, Chakra Petch nos títulos, scanlines suaves, glow e HUD. Sem salto de layout e com espaçamento nativo. | Nenhum |
| [FolderColumns](FolderColumns/) | **Arquivado / descontinuado (2026)** | Multi-coluna na sidebar do BetterFolders. Sem manutenção; MIT. | BetterFolders (sidebar) |

FolderColumns não é mais recomendado. Para colunas na barra de servidores, usa um plugin dedicado e atualizado. O OperaGX não interfere nisso.

## Instalar

Equicord > Settings > Themes > **Add theme** e cola a URL raw:

- OperaGX: `https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/OperaGX/OperaGX.css`
- FolderColumns (arquivado): `https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/FolderColumns/FolderColumns.css`

Depois ativa o tema. Mudanças no GitHub chegam sozinhas via `@updateUrl`.

Modo manual, sem auto-update: copia o `.css` para `%AppData%\Equicord\themes` e ativa em Settings > Themes.

```powershell
Invoke-WebRequest "https://raw.githubusercontent.com/notvictornw/Equicord-Themes/main/OperaGX/OperaGX.css" `
  -OutFile "$env:APPDATA\Equicord\themes\OperaGX.theme.css"
```

## OperaGX v4.5.4.1

- Base oficial do [HawkRatio](https://github.com/HawkRatio/OperaGXTheme) v1.2 via CDN, dentro de `@layer base` — os overrides do fork continuam mandando.
- **Salto de layout eliminado:** o upstream adiciona `padding-top:8px` + `margin-top:8px` e `margin-left:6px`/`border-top` em cada linha de canal. O fork zera esses valores por prefixo, então a sidebar volta ao tamanho nativo e as mensagens não descem ~23px.
- Fonte `display=optional` no Google Fonts (sem FOUT/reflow na primeira carga) e `content-visibility: visible` nas mensagens — sem reserva invisível de 72px por linha.
- `--categorySize: 12px` para categorias e grupos de membros, e fonte base do corpo voltada ao stack nativo (`gg sans`/`Noto Sans`/Helvetica/Arial). Chakra Petch continua só nos títulos, mantendo a identidade GX sem inflar o layout.
- Tamanhos nativos do Discord restaurados: o tema não força largura dos ícones, gap, raio, padding nem `object-fit` na barra de servidores. Só neutralize `clip-path`, a máscara de fade e o `--cp-size` que o upstream aplica. `box-sizing: border-box` e `margin: 0` absorvem bordas/margens de 1–2px do upstream.
- `zoom: 1` travado em `html`/`body`/`#app-mount` e sem `letter-spacing`/`font-weight` extras em títulos, categorias e mensagens — acaba com o "zoom" visual ao ativar o tema.
- Chamadas em 32px e highlight contido na mensagem.
- Sidebar de canais e DMs redimensionável: controles de resize com `pointer-events`, `touch-action: none`, largura 6px e `cursor: col-resize`.
- Scanlines vermelhas fracas com fade nas bordas; reduced-motion e high-contrast respeitados.
- Sem FolderColumns nativo. Documentação técnica completa em [`docs/THEME_ARCHITECTURE.md`](docs/THEME_ARCHITECTURE.md).

Opções pelo modal **Edit** do Equicord: `gx_glow_intensity_s` (55, 0–100) e `gx_scanline_opacity_s` (4, 0–20). Cores, fontes e raios ficam em `:root` no topo do CSS.

## Requisitos e compatibilidade

- Equicord atual, com suporte a `@import` em CSS e cascade layers.
- Internet para o CDN do HawkRatio e a fonte Chakra Petch. Sem internet, só a camada de overrides aparece.
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
