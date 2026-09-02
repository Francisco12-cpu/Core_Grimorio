# C.O.R.E. — Grimório Digital de Mesa (Core_Grimorio)

Site estático em HTML puro que funciona como um **hub de documentos de RPG** para o mestre
compartilhar com os jogadores: lore, bestiário, fichas, "manuscritos" e relatos de
personagens. Tema visual de grimório/runas, com cadeado rúnico como mecanismo de
**imersão narrativa** (não é segurança real — ver seção "Como funciona a senha").

## Arquitetura

```
index.html      → Hub principal. Lê menu.json e renderiza os cards de "manuscritos".
menu.json       → Fonte de dados do menu (título, descrição, arquivo, senha, aviso,
                   estiloEspecial).
manifest.json   → PWA manifest ("adicionar à tela inicial" no celular).
icons/          → Ícones do manifest (icon-192.png, icon-512.png).
_template.html  → Ponto de partida pra criar um novo documento de conteúdo (viewport,
                   fontes, tema e "modo sonho" já prontos).
bestiario.html  → Documento de conteúdo (bestiário do caçador).
elfos.html      → Documento de conteúdo (compêndio élfico).
relatos.html    → Documento de conteúdo (maior arquivo, 368KB — relatos de Altheris Vonn).
Kael.html       → Documento de conteúdo ("HERMES — A Experiência").
Loki.html       → Documento de conteúdo ("LOKI — A Ilusão").
Jurax.html      → Documento de conteúdo ("Evolução do Contrato").
ficha-core.html → Ferramenta interativa (editor de fichas de personagem), marcada como beta.
README.md       → Instruções de publicação (GitHub Pages) e "add to home screen".
```

Cada página é **autocontida**: HTML + CSS + JS embutidos no próprio arquivo, sem build
step, sem dependências externas além de fontes do Google Fonts (e, em `ficha-core.html`,
Vue 3 + html2canvas via CDN). Isso facilita hospedar em GitHub Pages e é uma escolha
deliberada do projeto — ver "Por que não há tema.css/tema.js compartilhado" abaixo.

### Fluxo do hub (`index.html`)

1. Carrega `core_config` do `localStorage`; se não existir, faz `fetch('menu.json')`.
2. Renderiza um card por item de `menu.itens[]`, cada um com um sigilo SVG decorativo
   gerado proceduralmente a partir do título (runas + hash determinístico). Todo item
   ganha um `item-status` ("Restrito" se tiver `senha`, senão "Disponível").
3. Ao clicar num card:
   - Se o item tem `aviso`, mostra um modal de aviso antes de prosseguir.
   - Se o item tem `senha`, abre o "Cadeado Rúnico" (teclado alfabeto→runa), com um
     aviso na UI de que é um selo narrativo, não uma proteção real.
   - Senão, dispara a transição (poeira dourada explodindo + a página "virando") e
     redireciona pro arquivo. Se o item tem `estiloEspecial: "sonho"`, o arquivo é
     aberto com `?estilo=sonho` na URL.
4. Existe sempre um card fixo extra, **"Manuscrito Mestre"**, protegido pela
   `senhaMestre` (padrão `NAUTILUS`), que abre um **editor do mestre** embutido —
   permite editar título, cores, glitch, senha mestre e os itens do menu (incluindo
   `estiloEspecial`), validar se o `arquivo` de um item realmente existe (fetch HEAD),
   e salvar no `localStorage`, baixar um novo `menu.json` ou copiar o JSON pra área de
   transferência (mais fácil de colar direto pelo GitHub no celular).
5. Um botão fixo no canto superior direito liga/desliga um ambiente sonoro opcional
   (drone + vento sintetizados via Web Audio API, começa mudo por causa das políticas
   de autoplay do navegador).

### "Modo sonho" (`estiloEspecial`)

Qualquer item do `menu.json` pode ter `"estiloEspecial": "sonho"`. O hub anexa
`?estilo=sonho` na URL ao abrir o arquivo, e cada página de conteúdo (inclusive
`_template.html` e `ficha-core.html`) tem um pequeno script que lê esse parâmetro e
aplica um filtro visual (leve blur + saturação + vinheta) via classe `core-modo-sonho`
no `<body>`. Pra marcar um manuscrito futuro como sonho/visão, basta esse campo no JSON
— nenhuma página precisa ser tocada.

### Como funciona a "senha" (importante)

A senha é validada **inteiramente no JavaScript do lado do cliente**, dentro de
`index.html`. As páginas de conteúdo (`Kael.html`, `relatos.html` etc.) **não têm
nenhuma proteção própria** — qualquer pessoa com a URL direta do arquivo acessa o
conteúdo sem digitar senha nenhuma, e a senha correta aparece em texto puro no
`menu.json` e no código-fonte da página se alguém abrir o DevTools.

Isso está OK para o caso de uso atual (mestre repassa a senha verbalmente para
criar suspense/imersão na mesa) — a própria UI do cadeado rúnico já deixa isso
explícito ("Um selo narrativo da mesa — não uma proteção de verdade"). Não confiar
nisso pra conteúdo sensível de verdade (ex.: spoilers que estragariam a campanha se
um jogador curioso abrisse o arquivo direto pela URL); se algum dia isso for
necessário, só um backend validando a senha do lado do servidor resolveria de fato.

## Sistema visual (tema)

- Fontes: `Cinzel` (títulos, rúnico) + `IM Fell English` (corpo de texto), via
  Google Fonts.
- Paleta: fundo quase preto (`#0d0a07`), dourado antigo (`#d4af37`) e vermelho
  sangue (`#8b1a1a`), cards em tom de pergaminho (`#e8d5b5`).
- `index.html`: poeira flutuante, runas decorativas de fundo, glitch periódico nos
  títulos/descrições dos cards, sigilos SVG por manuscrito, transição de "poeira
  dourada explodindo + página virando" ao abrir um manuscrito, ambiente sonoro opcional.
- `bestiario.html`, `elfos.html`, `relatos.html`, `ficha-core.html`: tema "pergaminho
  antigo" mais quieto — sem runas flutuantes, glitch ou som. Decisão de design
  confirmada com o mestre (ver "Por que não há tema.css/tema.js compartilhado").
- `Kael.html`, `Loki.html`, `Jurax.html`: cada um é uma "experiência" interativa
  bespoke (cenas, glitch RGB, áudio sintetizado sob medida) — visualmente aparentadas
  (fonte Cinzel, motivo de runas) mas **não** compartilham código entre si.
- `index.html` e `ficha-core.html` são configuráveis (cor, brilho, senha) via
  editor do mestre; as demais páginas de conteúdo têm o tema fixo no CSS.

### Por que não há tema.css/tema.js compartilhado

Foi cogitado extrair o CSS/JS de poeira/runas/glitch pra arquivos compartilhados.
Investigando o código, porém: `Kael.html`, `Loki.html` e `Jurax.html` não são cópias
umas das outras — cada uma implementa seus efeitos de runa/glitch/áudio com parâmetros
e lógica diferentes (ex.: `#glitch-rgb` tem z-index, timing e estrutura distintos em
cada arquivo), e `bestiario.html`/`elfos.html`/`relatos.html`/`ficha-core.html` nunca
tiveram esses efeitos. Ou seja, não existe duplicação real de código a eliminar sem
achatar o trabalho artesanal já feito em cada página — decisão consciente de manter
cada arquivo autocontido, como já era o padrão do projeto.

## Convenções ao editar

- Não há bundler/framework — edite os `.html` diretamente. Mantenha CSS e JS
  embutidos no próprio arquivo (é o padrão do projeto — ver seção acima).
- Ao criar um novo documento de conteúdo, comece a partir de `_template.html` e
  adicione uma entrada em `menu.json` (`titulo`, `descricao`, `arquivo`, `senha`,
  `aviso`, `estiloEspecial` opcional) — o `index.html` não precisa ser tocado pra isso.
- Sempre inclua `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">`, o favicon rúnico (mesmo `data:` URI usado em
  todas as páginas) e pelo menos um `@media (max-width: 600px)` para manter a
  experiência boa no celular, que é o dispositivo principal dos jogadores. Em campos
  de formulário, use `font-size: 16px` em telas pequenas pra evitar o zoom automático
  do Safari/iOS ao focar o campo.
- Use runas apenas do alfabeto já definido em `alfabetoRunico` (index.html) para
  manter consistência entre o teclado de senha e os textos "glitchados".

---

## Lista de ideias futuras (não implementadas)

- **Ilustrações/artes bespoke por manuscrito** (retrato do personagem, símbolo da
  criatura no bestiário): hoje cada card só tem um sigilo procedural genérico
  (runas + hash do título); arte de verdade por manuscrito ainda é trabalho manual
  em aberto.
- **Página de grimório virando com efeito de flip 3D nas duas metades da página**
  (hoje a transição já faz poeira dourada explodindo + um flip simples no menu; um
  efeito de "livro abrindo" mais elaborado, com as duas metades da tela girando em
  eixos opostos, ficou de fora por complexidade/risco de quebrar em telas pequenas).
- **Categorias no menu**: removido do escopo por decisão do mestre — não usar o
  campo `categoria`.
- **Senha por sessão** (não pedir de novo no mesmo aparelho depois da primeira vez):
  removido do escopo por decisão do mestre — a senha deve continuar sendo pedida
  toda vez.
- **Sigilo real do lado do servidor**: fora de escopo enquanto o site for 100%
  estático — só um backend validando a senha resolveria de verdade.
