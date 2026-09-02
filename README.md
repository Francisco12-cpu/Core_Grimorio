# C.O.R.E. — Grimório Digital de Mesa

Hub de documentos de RPG (lore, bestiário, fichas, manuscritos e relatos de
personagens) para o mestre compartilhar com os jogadores. Ver `CLAUDE.md` para
a documentação técnica completa do projeto.

## Como publicar (GitHub Pages)

O site é 100% estático (HTML puro), então dá pra publicar de graça pelo
GitHub Pages:

1. Neste repositório, vá em **Settings → Pages**.
2. Em **Source**, escolha a branch `main` (ou a branch que você usa como
   "oficial") e a pasta `/ (root)`.
3. Salve. Em alguns minutos o GitHub mostra a URL fixa do site, algo como:
   `https://<seu-usuario>.github.io/Core_Grimorio/`
4. Esse é o link único que você manda pros jogadores — ele sempre aponta pra
   última versão publicada na branch escolhida.

Depois de configurado uma vez, toda vez que você mergear mudanças nessa
branch (incluindo edições feitas pelo Editor do Mestre e reenviadas como
`menu.json`), o site atualiza sozinho em poucos minutos — não precisa
reconfigurar nada.

## Instalar como app no celular

O site tem um `manifest.json`, então os jogadores podem "instalar" o
grimório na tela inicial do celular (Chrome/Android: menu → *Adicionar à
tela inicial*; Safari/iOS: compartilhar → *Adicionar à Tela de Início*).
