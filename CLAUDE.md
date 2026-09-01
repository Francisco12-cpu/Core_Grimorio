# C.O.R.E. — Grimório Digital de Mesa (Core_Grimorio)

Site estático em HTML puro que funciona como um **hub de documentos de RPG** para o mestre
compartilhar com os jogadores: lore, bestiário, fichas, "manuscritos" e relatos de
personagens. Tema visual de grimório/runas, com cadeado rúnico como mecanismo de
**imersão narrativa** (não é segurança real — ver seção "Como funciona a senha").

## Arquitetura

```
index.html      → Hub principal. Lê menu.json e renderiza os cards de "manuscritos".
menu.json       → Fonte de dados do menu (título, descrição, arquivo, senha, aviso).
bestiario.html  → Documento de conteúdo (bestiário do caçador).
elfos.html      → Documento de conteúdo (compêndio élfico).
relatos.html    → Documento de conteúdo (maior arquivo, 368KB — relatos de Altheris Vonn).
Kael.html       → Documento de conteúdo ("HERMES — A Experiência").
Loki.html       → Documento de conteúdo ("LOKI — A Ilusão").
Jurax.html      → Documento de conteúdo ("Evolução do Contrato").
ficha-core.html → Ferramenta interativa (editor de fichas de personagem), marcada como beta.
```

Cada página é **autocontida**: HTML + CSS + JS embutidos no próprio arquivo, sem build
step, sem dependências externas além de fontes do Google Fonts. Isso facilita hospedar
em GitHub Pages, mas gera duplicação de código entre arquivos (ver Melhorias).

### Fluxo do hub (`index.html`)

1. Carrega `core_config` do `localStorage`; se não existir, faz `fetch('menu.json')`.
2. Renderiza um card por item de `menu.itens[]`. Todo item ganha um `item-status`
   ("Restrito" se tiver `senha`, senão "Disponível").
3. Ao clicar num card:
   - Se o item tem `aviso`, mostra um modal de aviso antes de prosseguir.
   - Se o item tem `senha`, abre o "Cadeado Rúnico" (teclado alfabeto→runa).
   - Senão, redireciona direto pro arquivo.
4. Existe sempre um card fixo extra, **"Manuscrito Mestre"**, protegido pela
   `senhaMestre` (padrão `NAUTILUS`), que abre um **editor do mestre** embutido —
   permite editar título, cores, glitch, senha mestre e os itens do menu, e
   salvar no `localStorage` ou baixar um novo `menu.json`.

### Como funciona a "senha" (importante)

A senha é validada **inteiramente no JavaScript do lado do cliente**, dentro de
`index.html`. As páginas de conteúdo (`Kael.html`, `relatos.html` etc.) **não têm
nenhuma proteção própria** — qualquer pessoa com a URL direta do arquivo acessa o
conteúdo sem digitar senha nenhuma, e a senha correta aparece em texto puro no
`menu.json` e no código-fonte da página se alguém abrir o DevTools.

Isso está OK para o caso de uso atual (mestre repassa a senha verbalmente para
criar suspense/imersão na mesa), mas **não é segurança real** — é só um "cadeado
narrativo". Vale deixar isso consciente para não confiar nisso para conteúdo
sensível de verdade (ex.: spoilers que estragariam a campanha se um jogador
curioso abrisse o arquivo direto pela URL).

## Sistema visual (tema)

- Fontes: `Cinzel` (títulos, rúnico) + `IM Fell English` (corpo de texto), via
  Google Fonts.
- Paleta: fundo quase preto (`#0d0a07`), dourado antigo (`#d4af37`) e vermelho
  sangue (`#8b1a1a`), cards em tom de pergaminho (`#e8d5b5`).
- Efeitos de imersão reutilizados em várias páginas: poeira flutuante (partículas
  douradas), runas decorativas de fundo, "glitch" periódico trocando letras por
  runas em textos, sons sintetizados via Web Audio API em cliques/navegação,
  animação de entrada com runas.
- `index.html` e `ficha-core.html` são configuráveis (cor, brilho, senha) via
  editor do mestre; as demais páginas de conteúdo têm o tema fixo no CSS.

## Convenções ao editar

- Não há bundler/framework — edite os `.html` diretamente. Mantenha CSS e JS
  embutidos no próprio arquivo (é o padrão do projeto).
- Ao criar um novo documento de conteúdo, adicione uma entrada em `menu.json`
  (`titulo`, `descricao`, `arquivo`, `senha`, `aviso`, `categoria`) — o
  `index.html` não precisa ser tocado para isso.
- Sempre inclua `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">` e pelo menos um `@media (max-width: 600px)` (ou 480px) para manter a
  experiência boa no celular, que é o dispositivo principal dos jogadores.
- Use runas apenas do alfabeto já definido em `alfabetoRunico` (index.html) para
  manter consistência entre o teclado de senha e os textos "glitchados".

---

## Análise do estado atual

**Pontos fortes**
- Identidade visual forte e consistente (fontes, paleta, runas, poeira, glitch)
  em quase todos os arquivos — já entrega a sensação de "grimório antigo".
- Mobile já é levado a sério: viewport travado, media queries, teclado rúnico
  adaptado para telas pequenas.
- O editor do mestre embutido no `index.html` é uma boa ideia: dá pro mestre
  adicionar itens sem editar JSON na mão.
- Tudo funciona sem servidor/backend — fácil publicar em GitHub Pages.

**Problemas / inconsistências encontradas**
1. **Bug de título**: `relatos.html` tem `<title>Bestiário do Caçador</title>`
   (copiado de `bestiario.html` por engano) — a aba do navegador mostra o nome
   errado.
2. **"Senha" não protege nada de verdade** (ver seção acima) — qualquer jogador
   que salve/compartilhe a URL do arquivo ignora o cadeado.
3. **Sem "voltar ao menu"**: as páginas de conteúdo não parecem ter um link de
   volta pro `index.html` — no celular, o jogador tem que usar o botão "voltar"
   do navegador.
4. **Duplicação de CSS/JS** entre `Kael.html`, `Loki.html`, `Jurax.html`,
   `bestiario.html`, `elfos.html` (poeira, runas, glitch, fontes) — qualquer
   ajuste no tema precisa ser replicado manualmente em cada arquivo.
5. **`relatos.html` com 368KB** num único arquivo é pesado pra 3G/4G fraco de
   mesa — vale investigar se dá pra separar imagens/assets ou lazy-load trechos.
6. **Sem link do GitHub Pages confirmado** (não há `.github/workflows` nem
   `CNAME`) — não está claro se o site já está publicado num link único e fixo
   para mandar aos jogadores.
7. **`ficha-core.html`** está marcado como beta ("pode se alterar") e sem
   `@media` própria — funciona por ter layout centralizado estreito, mas não
   foi testado explicitamente pra mobile como as outras páginas.
8. **`config.itens` do editor do mestre só salva no `localStorage` do próprio
   navegador do mestre** ou exige baixar um `menu.json` e subir manualmente pro
   repo — não há como o mestre editar direto do celular e isso já refletir pros
   jogadores sem um passo manual de deploy.

---

## Lista de melhorias e adições sugeridas

### Rápidas (alto impacto, baixo esforço)
- [ ] Corrigir o `<title>` de `relatos.html`.
- [ ] Adicionar um botão/link fixo "◂ Voltar ao Grimório" em todas as páginas de
      conteúdo, apontando pra `index.html`.
- [ ] Publicar via GitHub Pages e fixar a URL única num só lugar (ex.: no topo
      do `index.html` como comentário, ou README) pra sempre mandar o mesmo link
      pros jogadores.
- [ ] Adicionar favicon/ícone consistente com o rúnico já usado em `index.html`
      nas demais páginas (hoje só o hub tem).

### Estrutural (organização e manutenção)
- [ ] Extrair o CSS/JS comum (poeira, runas de fundo, glitch, fontes, cores) pra
      arquivos compartilhados `tema.css` e `tema.js`, incluídos via `<link>`/
      `<script src>` em todas as páginas — elimina duplicação e permite trocar o
      visual do grimório inteiro num lugar só.
  - Compensação: quebra o padrão "arquivo único autocontido" atual, que também
    tem a vantagem de cada página poder ser aberta isoladamente sem depender de
    outro arquivo. Vale decidir isso com o mestre antes de migrar.
- [ ] Padronizar breakpoints (hoje varia entre 480px e 600px) num só valor.
- [ ] Criar um pequeno template-base (`_template.html`) pra novas páginas de
      conteúdo já saírem com viewport, fontes, tema e botão de voltar prontos.

### Imersão / experiência ("sonhos", mais visual)
- [ ] Transições de página tipo "dissolver em poeira dourada" ao entrar/sair de
      um manuscrito (hoje só existe overlay simples com runas).
- [ ] Efeito de "página de grimório virando" ao trocar de manuscrito.
- [ ] Trilha sonora ambiente opcional (loop baixo de vento/fogueira), com botão
      de mute — hoje só há efeitos sonoros pontuais de clique.
- [ ] Ilustrações/artes por manuscrito (retrato do personagem, símbolo da
      criatura no bestiário) — pode ser SVG inline pra não pesar o carregamento.
- [ ] "Modo sonho": um filtro visual (blur leve + saturação diferente) pra
      páginas marcadas como sonho/visão, ativável por uma flag no `menu.json`
      (ex.: `"estiloEspecial": "sonho"`).
- [ ] Categorias reais no menu (`categoria` já existe no JSON mas não é usada
      pra agrupar/filtrar os cards) — permitiria seções tipo "Personagens",
      "Bestiário", "Lore do Mundo".

### Senha / controle de acesso
- [ ] Deixar explícito na UI que a senha é uma "trava narrativa", não uma
      trava de segurança (ex.: texto discreto tipo "protegido por um selo
      rúnico" ao invés de sugerir criptografia).
- [ ] Opção de senha por sessão: depois de digitada uma vez, não pedir de novo
      no mesmo aparelho (guardar flag no `localStorage` por item) — evita o
      jogador ter que perguntar a senha de novo toda vez que reabre o link.
- [ ] Se algum dia precisar de sigilo de verdade (ex.: revelação que muda o
      rumo da campanha), considerar um pequeno backend/Cloudflare Worker que
      sirva o conteúdo só após validar a senha no servidor — os arquivos
      estáticos atuais nunca vão proteger de fato quem souber a URL.

### Mobile / responsividade
- [ ] Testar `ficha-core.html` em telas pequenas de verdade e adicionar
      `@media` se necessário (hoje depende só do `max-width:520px` do
      container).
- [ ] Adicionar "add to home screen" (manifest.json simples) pra virar um
      atalho na tela inicial do celular do jogador, reforçando a sensação de
      "grimório instalado".

### Fluxo do mestre
- [ ] Facilitar publicar as edições do "Manuscrito Mestre" sem precisar baixar
      o JSON e subir manualmente — por exemplo, um botão que já gera o link do
      GitHub pra colar o conteúdo, ou migrar a config pra um serviço simples
      tipo GitHub Gist/Pages Actions.
- [ ] Validar no editor do mestre se `arquivo` de um novo item realmente existe
      antes de salvar, pra evitar cards quebrados no menu.
