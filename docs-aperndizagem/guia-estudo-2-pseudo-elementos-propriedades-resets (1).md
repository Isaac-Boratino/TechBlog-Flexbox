# Guia de Estudo 2: Pseudo-elementos, Propriedades, Valores e Resets — TechBlog

Continuação do primeiro guia. Aqui a lupa vai para quatro coisas específicas: o comentário "IMPORTAÇÃO DE FONTES" no topo do seu CSS, o que são pseudo-elementos de verdade (não só `::before`/`::after`), a anatomia de "propriedade + valor" e por que o reset existe.

---

## 1. O comentário "IMPORTAÇÃO DE FONTES" — o que é e por que está vazio

No topo do seu `style.css`:

```css
/* IMPORTAÇÃO DE FONTES */

/* VARIÁVEIS GLOBAIS*/
:root{
```

Esse comentário é um **placeholder**: um lembrete deixado por quem escreveu o CSS, indicando "aqui é onde as fontes customizadas deveriam ser importadas" — mas atualmente não há nenhum código embaixo dele. Isso não é um erro, é simplesmente uma seção ainda não usada.

### O que seria "importar uma fonte", na prática?

Por padrão, o navegador só tem acesso às fontes já instaladas no computador do visitante (Arial, Georgia, Times New Roman, etc). Se você quer uma fonte "de designer" (tipo Poppins, Inter, Montserrat, Playfair Display), ela **não existe** na máquina do usuário — você precisa trazê-la pela internet. Existem três formas de fazer isso:

**Opção 1 — `<link>` no HTML** (a mais comum com Google Fonts):
```html
<head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap" rel="stylesheet">
</head>
```

**Opção 2 — `@import` no topo do CSS** (é isso que o comentário provavelmente estava reservando o lugar):
```css
/* IMPORTAÇÃO DE FONTES */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap');
```
`@import` precisa ficar **antes de qualquer outra regra** no arquivo CSS — por isso o comentário está logo na primeira linha, acima até das variáveis. Isso é uma pista de que o autor planejava usar `@import` ali.

**Opção 3 — `@font-face`** (quando você tem o arquivo da fonte, ex: `.woff2`, baixado no seu projeto, sem depender do Google):
```css
@font-face {
  font-family: 'MinhaFonte';
  src: url('fontes/minhafonte.woff2') format('woff2');
}
```

### Por que seu site funciona sem isso

Olhando as variáveis de fonte que você realmente usa:
```css
--fonte-display: Georgia, serif;
--fonte-corpo:   system-ui, sans-serif;
--fonte-mono:    'Corier New', monospace;
```
Todas são **fontes do sistema** — já vêm instaladas no computador (Georgia, uma serifada clássica; `system-ui`, que pega automaticamente a fonte padrão do sistema operacional do usuário; e uma tentativa de "Courier New", monoespaçada). Nenhuma delas precisa de importação externa, então o comentário ficou como uma seção "reservada para o futuro", caso você queira trocar por fontes mais estilizadas depois.

⚠️ **Pequeno erro de digitação para você notar**: `'Corier New'` deveria ser `'Courier New'` (faltou o "u"). Como o nome está errado, o navegador não reconhece essa fonte e cai direto para o fallback genérico `monospace` — o efeito visual quase não muda (porque monospace já é parecido), mas tecnicamente a fonte pretendida nunca é aplicada. Isso é um ótimo exemplo de por que sempre vale a pena conferir o nome exato da fonte entre aspas.

**Exercício:** tente importar a fonte "Poppins" do Google Fonts usando `@import` no lugar do comentário vazio, e troque `--fonte-corpo` para usá-la. Veja o site inteiro mudar de "cara" trocando uma linha só.

---

## 2. Pseudo-elementos — o quadro completo

No guia anterior vimos `::before` e `::after` na prática. Agora vamos entender a família toda e a lógica por trás dela.

### O que é, tecnicamente

Um pseudo-elemento **não seleciona algo que já existe no HTML** — ele cria (ou mira em) uma "parte" do elemento que normalmente não é endereçável diretamente. A sintaxe usa **dois dois-pontos** (`::`), para diferenciar de pseudo-*classes* (um dois-pontos, `:hover`, `:focus`), embora navegadores antigos aceitassem um só para os dois — hoje o padrão moderno é sempre `::`.

### `::before` e `::after` — os mais usados

```css
.categoria::before{
    content: '';
    width: 28px;
    height: 2px;
    background-color: var(--cor-acento);
}
```
Isso insere um elemento "fantasma" **imediatamente antes** do conteúdo real de `.categoria` (o texto "Artigo em destaque"). Repare que `content: ''` está vazio — nesse caso, o pseudo-elemento não mostra texto nenhum, só existe para ser estilizado como uma barrinha decorativa (`width: 28px; height: 2px`). Sem a propriedade `content`, o `::before` **nem aparece** — ela é obrigatória para pseudo-elementos gerados.

```css
.hero-imagem::after{
    content: '</>';
    font-family: var(--fonte-mono);
    font-size: 4.5rem;
    color: rgba(230,57,70,0.18);
}
```
Aqui o `content` **tem** texto: literalmente os caracteres `</>`. Isso prova que `content` pode ser usado tanto para decoração pura (caso anterior) quanto para inserir conteúdo textual/simbólico direto do CSS — sem precisar escrever esse símbolo no HTML.

**Regra mental simples:** `::before` = "cole isso um passo antes do conteúdo do elemento". `::after` = "cole isso um passo depois". Os dois são "filhos virtuais" do elemento — por isso, para aparecerem, o elemento pai geralmente precisa de `display` compatível (block, inline-block, flex) e às vezes `position: relative` no pai + `position: absolute` no pseudo-elemento, como acontece em `.hero-imagem` (que tem `position: relative`) e `.hero-imagem::after` (que tem `position: absolute`) — isso ancora o `</>` dentro dos limites da caixa `.hero-imagem`, em vez de vazar pela página.

### Outros pseudo-elementos que existem (não estão no seu CSS, mas valem conhecer)

| Pseudo-elemento | O que faz | Exemplo de uso |
|---|---|---|
| `::first-letter` | Estiliza só a primeira letra de um bloco de texto | Efeito de "letra capitular" em artigos |
| `::first-line` | Estiliza só a primeira linha de um parágrafo | Destacar a abertura de um texto |
| `::selection` | Estiliza o texto quando o usuário **seleciona** (arrasta o mouse) | Trocar a cor de fundo da seleção da marca-texto |
| `::placeholder` | Estiliza o texto de exemplo dentro de um `<input>` | Aplicável direto no seu `.campo-email`, que hoje usa a cor padrão do navegador para "seu@email.com" |
| `::marker` | Estiliza o marcador de itens de lista (bolinha/número) | Customizar bullets de `<li>` |

**Exercício:** adicione `.campo-email::placeholder{ color: var(--cor-texto-suave); }` no seu CSS e veja o texto de exemplo do campo de e-mail mudar de cor.

---

## 3. Pseudo-classes — o outro lado da moeda dos pseudo-elementos

Antes de seguir para propriedades e valores, vale fechar o par que ficou pela metade no guia anterior: se pseudo-*elementos* (`::`) criam ou miram em **partes** de um elemento, pseudo-*classes* (um só `:`) miram no elemento inteiro, mas apenas quando ele está em um **estado**, **posição** ou **condição** específica. Elas não criam nada novo no DOM — só ligam/desligam um estilo dependendo da situação.

### `:hover` — quando o mouse está em cima

```css
#navegacao a:hover{
    color: #fff;
    background-color: rgba(255,255,255,0.08);
}

.card-artigo:hover .card-corpo h2{
    color:var(--cor-acento)
}

.tag-item:hover{
    background-color: var(--cor-topo);
    color: #fff;
    border-color: var(--cor-topo);
}
```
`:hover` aplica o estilo só enquanto o cursor do mouse está posicionado sobre o elemento. Repare de novo no caso `.card-artigo:hover .card-corpo h2`: o `:hover` está grudado em `.card-artigo` (sem espaço), mas o efeito aparece no `h2` **descendente** dele (com espaço depois) — ou seja, "quando o card inteiro estiver em hover, mude a cor do h2 que mora dentro dele". Isso é diferente de `.card-corpo h2:hover`, que exigiria o mouse exatamente em cima do próprio texto do h2.

### `:focus` — quando um campo está "selecionado" para digitação

```css
.campo-email:focus{
    border-color: var(--cor-acento);
}
```
`:focus` ativa quando o elemento recebe foco — geralmente por clique ou por navegação via `Tab` no teclado. É essencial em formulários: sem um estilo de `:focus` visível, uma pessoa navegando só pelo teclado não consegue saber em qual campo está digitando. Note que `:hover` precisa do mouse continuar em cima; `:focus` permanece ativo mesmo depois que o mouse sai, até o usuário clicar em outro lugar ou apertar Tab novamente.

### `:last-child` — mirando pela posição entre os "irmãos"

```css
.widget .lista-categorias li:last-child{
    border-bottom: none;
}
```
Essa é uma pseudo-classe **estrutural**: ela olha para a posição do elemento dentro do seu grupo de irmãos, sem precisar de nenhuma classe extra no HTML. `:last-child` seleciona o último `<li>` de cada `<ul class="lista-categorias">`. O efeito prático: todos os outros itens têm uma linha divisória embaixo (`border-bottom`, definida em outra regra), mas o último item não — porque senão sobraria uma linha "solta" encostada na borda de baixo do widget, sem nenhum outro item depois dela.

Outras pseudo-classes estruturais da mesma família, que não estão no seu CSS mas seguem a mesma lógica, valem conhecer:
| Pseudo-classe | Seleciona |
|---|---|
| `:first-child` | O primeiro elemento entre os irmãos |
| `:nth-child(2)` | O elemento na posição exata que você escolher (2º, 3º...) |
| `:nth-child(odd)` / `:nth-child(even)` | Elementos em posições ímpares/pares — ótimo para "zebrar" linhas de tabela |
| `:only-child` | Um elemento que é filho único (sem irmãos) |

**Exercício:** troque `:last-child` por `:first-child` no seu CSS e veja a linha divisória "migrar" para o topo da lista em vez do final — isso ajuda a visualizar concretamente o que "seleção por posição" quer dizer.

### Diferença resumida: pseudo-classe vs pseudo-elemento

| | Pseudo-classe (`:`) | Pseudo-elemento (`::`) |
|---|---|---|
| O que faz | Estiliza o elemento **inteiro** sob uma condição (estado, posição) | Estiliza/cria uma **parte** específica do elemento |
| Precisa de interação do usuário? | Às vezes (`:hover`, `:focus`) | Nunca — são estáticos, aparecem sempre que a regra bate |
| Cria conteúdo novo? | Nunca | Pode, via `content` (`::before`/`::after`) |
| Exemplos no seu CSS | `:hover`, `:focus`, `:last-child` | `::before`, `::after` |

Um jeito simples de lembrar: pseudo-classe pergunta "**quando/onde** esse elemento se encontra?" (passando o mouse, focado, sendo o último). Pseudo-elemento pergunta "**qual pedacinho** desse elemento eu quero pegar?" (o antes, o depois, a primeira letra).

---

## 4. Anatomia de uma regra CSS: seletor, propriedade e valor

Antes de falar de "propriedades e valores" separadamente, vale fixar a estrutura completa de uma regra CSS, porque tudo no seu arquivo segue esse molde:

```css
seletor {
    propriedade: valor;
    propriedade: valor;
}
```

Exemplo real do seu código:
```css
.btn-newsletter{
    background-color: var(--cor-topo);   /* propriedade: background-color | valor: var(--cor-topo) */
    color: #fff;                          /* propriedade: color | valor: #fff */
    border: none;                         /* propriedade: border | valor: none */
    padding: 0.68rem;                     /* propriedade: padding | valor: 0.68rem */
    border-radius: 6px;                   /* propriedade: border-radius | valor: 6px */
    cursor: pointer;                      /* propriedade: cursor | valor: pointer */
}
```

- **Seletor** (`.btn-newsletter`) → "quem" vai receber o estilo.
- **Propriedade** (`background-color`, `padding`, `cursor`...) → "o quê" você está controlando. É sempre um nome fixo, definido pela especificação CSS — você não inventa nomes novos.
- **Valor** (`var(--cor-topo)`, `0.68rem`, `pointer`...) → "quanto" ou "de que jeito". Cada propriedade aceita um conjunto específico de tipos de valor (cores, medidas, palavras-chave, funções).
- Cada par `propriedade: valor;` é uma **declaração**, e sempre termina com ponto e vírgula.

### Categorias de propriedades presentes no seu CSS

**Cor** — aceitam hex (`#e63946`), `rgba()`, palavras-chave (`transparent`, `inherit`) ou variáveis:
```css
color: #ffffff;
background-color: rgba(255,255,255,0.08);
```

**Medida/dimensão** — aceitam unidades diferentes, cada uma com um comportamento:
```css
width: 28px;        /* pixel: valor fixo, não escala com nada */
font-size: 1.35rem; /* rem: relativo ao font-size do <html> (definido como 16px) */
padding: 0 5vw;      /* vw: relativo à largura da viewport (tela) — escala com o tamanho da janela */
line-height: 1.65;   /* número puro (sem unidade): multiplica o font-size do próprio elemento */
```

**Palavra-chave** — valores de texto fixo, sem unidade:
```css
display: flex;
position: sticky;
cursor: pointer;
text-transform: uppercase;
```

**Função** — valores que "calculam" algo, sempre com parênteses:
```css
background: linear-gradient(135deg, #1a2a5e 0%, #0f3460 100%);
font-size: clamp(1.9rem, 3.5vw, 2.9rem);
color: var(--cor-acento);
color: rgba(255,255,255,0.08);
```

**Atalho (shorthand)** — uma propriedade que na verdade define várias de uma vez:
```css
padding: 0.45rem 0.9rem;
/* equivale a escrever separadamente: */
/* padding-top: 0.45rem; padding-right: 0.9rem; padding-bottom: 0.45rem; padding-left: 0.9rem; */

border: 1px solid rgba(255,255,255,0.06);
/* equivale a: */
/* border-width: 1px; border-style: solid; border-color: rgba(255,255,255,0.06); */

border-bottom: 3px solid var(--cor-acento);
/* atalho ainda mais específico: só a borda de baixo, com largura+estilo+cor juntos */
```
Entender que `padding`/`margin`/`border` são atalhos explica por que às vezes você vê 1 valor, às vezes 2, às vezes 4 — o CSS interpreta a quantidade de valores de forma diferente (1 valor = todos os lados; 2 valores = vertical/horizontal; 4 valores = topo/direita/baixo/esquerda, sentido horário).

---

## 5. Unidades de medida — o que cada uma significa e quando usar

Você já viu de relance `px`, `rem` e `vw` na seção anterior. Aqui vale abrir cada unidade usada no seu CSS, porque escolher a unidade certa é uma das decisões que mais separam CSS "amador" de CSS bem pensado.

### `px` (pixel) — valor absoluto, fixo

```css
height: 62px;
border-radius: 10px;
width: 28px;
```
Um pixel é uma unidade **fixa**: `10px` sempre mede 10px, não importa o tamanho da tela, o zoom do navegador ou a fonte configurada pelo usuário (na prática, isso é uma simplificação — zoom do navegador afeta px também — mas comparado às unidades relativas abaixo, `px` é a mais "rígida"). No seu CSS, `px` aparece em coisas que **não fazem sentido escalar**: a espessura de uma borda (`border-radius`, `border: 1px`), o tamanho de um avatar circular (`width: 28px; height: 28px`), a altura fixa da navbar (`height: 62px`). São elementos onde você quer controle exato, não proporcional.

### `rem` — relativo ao `<html>`, a unidade "mestra" de texto

```css
html{ font-size: 16px; }
...
font-size: 1.35rem;   /* = 1.35 × 16px = 21.6px */
padding: 0.68rem;     /* = 0.68 × 16px = 10.88px */
```
`rem` significa *root em* — "em relativo à raiz". `1rem` sempre equivale ao `font-size` definido no `<html>`, que seu reset fixa em `16px`. A vantagem gigante do `rem`: se um usuário aumentar o tamanho de fonte padrão do navegador (configuração de acessibilidade, comum em quem tem dificuldade de leitura), **todo o site escala proporcionalmente junto** — títulos, espaçamentos, botões — porque tudo está amarrado a essa única referência. É por isso que praticamente todo `font-size`, `padding` e `margin` do seu CSS usa `rem` em vez de `px`: o projeto inteiro "respira" junto se a base mudar.

### `em` — relativo ao próprio elemento (mais "local" que `rem`)

```css
letter-spacing: 0.02em;
letter-spacing: 0.2rem;   /* obs: aqui seu CSS mistura rem, veja nota abaixo */
letter-spacing: 0.14em;
```
`em` é parecido com `rem`, mas relativo ao `font-size` **do próprio elemento** (não da raiz `<html>`). Se um elemento tem `font-size: 20px`, então `1em` ali vale 20px — diferente de outro elemento com `font-size: 14px`, onde `1em` vale 14px. Isso é mais usado para espaçamentos que devem **acompanhar o tamanho daquele texto específico**, como `letter-spacing` (espaçamento entre letras): faz sentido que o espaçamento entre letras cresça junto se o texto for maior, e isso é exatamente o que `em` proporciona.

📌 Nota de leitura atenta: seu CSS usa `em` na maioria dos `letter-spacing`, mas em pelo menos uma linha (`.categoria{ letter-spacing: 0.2rem; }`) usa `rem` no lugar de `em`. Não é um "erro" que quebra o site, mas é uma inconsistência — `rem` ali vai se basear no tamanho da fonte raiz (16px) em vez do tamanho da própria `.categoria`, o que foge um pouco da intenção normal de `letter-spacing`. Bom exemplo de como vale sempre reler o que você escreveu.

### `vw` (viewport width) — relativo à largura da tela

```css
padding: 0 5vw;
padding: 4.5rem 5vw;
```
`1vw` = 1% da largura da **janela do navegador** (não da página, da tela visível). `5vw` significa "5% da largura da tela, seja ela qual for". Isso faz o espaçamento lateral do site (`#topo`, `#hero`, `#faixa-numeros`, `#rodape` todos usam `5vw` nas laterais) **crescer ou encolher junto com o tamanho da janela** — em uma tela grande, a margem lateral fica maior; em uma tela estreita, menor. É uma forma simples e eficaz de criar espaçamento que reage ao tamanho da tela sem precisar escrever `@media queries` só para isso.

### `%` (porcentagem) — relativo ao elemento pai

```css
max-width: 100%;
width: 100%;
border-radius: 50%;
```
Porcentagem sempre se refere a alguma medida do **elemento pai** (ou, em alguns casos específicos, à própria caixa do elemento). `width: 100%` no `.campo-email` significa "ocupe 100% da largura de quem for o pai dele" (no caso, o `.widget-corpo`) — se a sidebar mudar de tamanho, o campo de e-mail acompanha automaticamente. Já `border-radius: 50%` é um caso especial e muito usado: aplicado em um elemento quadrado (mesma largura e altura, como o `.avatar` de 28×28px), 50% de raio em cada canto faz os quatro cantos se encontrarem exatamente no meio, formando um **círculo perfeito** — é assim que os avatares circulares dos autores são feitos, sem nenhuma imagem redonda de verdade.

### `s` (segundos) — unidade de tempo, não de espaço

```css
--transicao: 0.22s ease;
transition: all var(--transicao);
```
Diferente de todas as anteriores, `s` mede **tempo**, usada em `transition` e `animation`. `0.22s` = 0.22 segundos = 220 milissegundos, o tempo que a mudança de cor/posição/tamanho leva para acontecer suavemente (também existe `ms` para escrever direto em milissegundos, ex: `220ms` — seu CSS optou por segundos). Depois do tempo, no seu caso vem a palavra `ease`, que é a "curva de aceleração" da animação (começa mais rápido, desacelera no final) — outras curvas comuns são `linear` (velocidade constante), `ease-in` (começa devagar), `ease-out` (termina devagar).

### Valores sem unidade — quando "número puro" é intencional

```css
line-height: 1.65;
font-weight: 700;
flex: 1;
z-index: 10;
```
Algumas propriedades **não usam unidade nenhuma**, e isso é proposital, não esquecimento:
- `line-height: 1.65` sem unidade significa "1.65 vezes o `font-size` do próprio elemento" — é a forma recomendada de definir altura de linha, porque escala automaticamente se o `font-size` mudar (escrever `line-height: 26px` fixo quebraria em telas ou fontes diferentes).
- `font-weight: 700` é uma escala numérica padronizada de peso de fonte (100 = mais fino, 900 = mais grosso; 400 = normal, 700 = negrito) — não é medida de espaço, é um índice de uma tabela de pesos.
- `flex: 1` já vimos no guia 1 — é uma proporção relativa entre os irmãos flex, não um tamanho absoluto.

### Tabela-resumo das unidades do seu projeto

| Unidade | Relativa a | Usada no seu CSS para | Exemplo |
|---|---|---|---|
| `px` | Nada (fixa) | Bordas, ícones pequenos, alturas fixas | `border-radius: 10px` |
| `rem` | `font-size` do `<html>` | Fontes, paddings, margins — a maioria das medidas | `font-size: 1.08rem` |
| `em` | `font-size` do próprio elemento | Espaçamento entre letras | `letter-spacing: 0.12em` |
| `vw` | Largura da janela do navegador | Espaçamento lateral que reage ao tamanho da tela | `padding: 0 5vw` |
| `%` | Elemento pai (ou a própria caixa, no caso de `border-radius`) | Larguras fluidas, círculos perfeitos | `border-radius: 50%` |
| `s` | Tempo (não é espaço) | Duração de transições/animações | `0.22s ease` |
| *(sem unidade)* | Depende da propriedade | Altura de linha, peso de fonte, proporção flex | `line-height: 1.65` |

**Exercício:** troque, só como teste, o `padding: 0 5vw;` de `#topo` para `padding: 0 40px;` (um valor fixo em `px`) e redimensione a janela do navegador (ou abra no celular). Compare com o comportamento original em `vw` — você vai ver visualmente a diferença entre uma unidade que reage ao tamanho da tela e uma que não reage.

---

## 6. O Reset — por que "zerar tudo" antes de começar

```css
*, *::before, *::after{
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

html{
    scroll-behavior: smooth;
    font-size: 16px;
}

body{
    font-family: var(--fonte-corpo);
    background-color: var(--cor-fundo);
    color: var(--cor-texto);
    line-height: 1.65;
}

a{
    text-decoration: none;
    color: inherit;
}

img{
    display: block;
    max-width: 100%;
}

ul{
    list-style: none;
}
```

### O problema que o reset resolve

Cada navegador (Chrome, Firefox, Safari) vem com uma **folha de estilo padrão própria** (chamada *user agent stylesheet*), que aplica estilos automáticos mesmo sem você escrever CSS nenhum. Por exemplo, sem reset: `<ul>` já vem com bolinhas e recuo, `<a>` já vem azul e sublinhado, `<h1>` já vem com margem grande em cima e embaixo, `<body>` já vem com uma margem de 8px na borda da tela. O problema é que **cada navegador define esses valores um pouco diferente**, então o mesmo HTML pode parecer sutilmente diferente no Chrome vs no Firefox.

O reset existe para **anular esses padrões inconsistentes** e te dar um ponto de partida neutro e previsível — depois disso, todo espaçamento e cor que aparece no site foi *você* quem decidiu, não o navegador.

### Explicando cada linha do seu reset

```css
*, *::before, *::after{ box-sizing: border-box; margin: 0; padding: 0; }
```
- `*` = seletor universal, pega **literalmente qualquer elemento**.
- `*::before, *::after` estão incluídos porque pseudo-elementos também podem ter margin/padding/box-sizing próprios — sem incluir eles aqui, seus `::before`/`::after` ficariam de fora do reset.
- `margin: 0; padding: 0;` — zera os espaçamentos padrão de tudo (títulos, parágrafos, listas, botões).
- `box-sizing: border-box` — já explicado no guia 1: garante que `width`/`height` já incluam padding e borda, evitando que elementos "estourem" o tamanho esperado.

```css
html{ scroll-behavior: smooth; font-size: 16px; }
```
- `scroll-behavior: smooth` — quando algum link interno da página leva a uma âncora (`<a href="#secao">`), a rolagem acontece de forma suave e animada, em vez de "pular" instantaneamente.
- `font-size: 16px` — define explicitamente o tamanho base da fonte no `<html>`. Isso é importante porque **toda unidade `rem` no CSS é relativa a esse valor**: `1rem` = 16px, `1.35rem` = 21.6px, e assim por diante. Fixar isso no reset garante que seus `rem`s tenham uma base confiável e conhecida.

```css
body{ font-family: var(--fonte-corpo); background-color: var(--cor-fundo); color: var(--cor-texto); line-height: 1.65; }
```
Define os estilos "herdáveis" de base — fonte, cor de fundo geral, cor de texto padrão e altura de linha — direto no `<body>`. Como `font-family` e `color` são propriedades que **herdam** por padrão para os filhos, você só precisa declarar isso uma vez aqui, e todo elemento dentro do `<body>` já nasce com essa fonte e cor, a menos que uma regra mais específica sobrescreva (como acontece com `#hero h1`, que redefine a cor para branco).

```css
a{ text-decoration: none; color: inherit; }
```
- `text-decoration: none` — remove o sublinhado padrão dos links.
- `color: inherit` — em vez de usar o azul/roxo padrão do navegador para links, o `<a>` **herda a cor do elemento pai**. É por isso que um link dentro de `#navegacao` fica cinza-azulado (herdando o esquema daquela área) e um link no rodapé fica com outro tom — o valor `inherit` faz o link "se camuflar" no contexto onde está, e cada seção depois define explicitamente a cor que quer via seletor descendente.

```css
img{ display: block; max-width: 100%; }
```
- `display: block` — por padrão, `<img>` é `inline`, o que causa um espacinho fantasma embaixo da imagem (por causa de como texto inline lida com a "linha de base" das letras). Virar `block` remove esse espaço extra indesejado.
- `max-width: 100%` — impede que uma imagem maior que o container "vaze" para fora dele; ela nunca ultrapassa a largura do pai, mesmo que a imagem original seja enorme. (Seu site atual não usa `<img>` de verdade — usa emojis como espaços reservados — mas essa regra já deixa o projeto preparado para quando imagens reais entrarem.)

```css
ul{ list-style: none; }
```
Remove as bolinhas/marcadores padrão de listas (`<ul>`). É por isso que `.lista-categorias` e `.nuvem-tags` não mostram nenhum símbolo de lista — o visual de "linhas" ou "chips" que você vê é 100% construído por CSS (bordas, padding, flex), não por marcadores de lista.

### Por que isso é chamado de "reset" e não de "estilo normal"

Tecnicamente, o que você tem aqui é um **reset simplificado, feito à mão**. Existem também bibliotecas prontas e mais completas para isso — as mais conhecidas são **Normalize.css** (que *ajusta* inconsistências entre navegadores, em vez de zerar tudo) e o **Reset CSS de Eric Meyer** (que zera quase tudo, de forma mais agressiva, parecido com o seu). O seu reset é uma versão enxuta e objetiva: só zera o que realmente atrapalharia esse projeto específico (margin, padding, list-style, decoração de link, box-sizing) — não é menos válido por ser mais curto, é só uma escolha de escopo.

---

## 7. Resumo rápido para revisão

- **Comentário "IMPORTAÇÃO DE FONTES"**: é um espaço reservado para `@import`/`@font-face`, hoje vazio porque o site só usa fontes de sistema (Georgia, system-ui, e uma "Courier New" com erro de digitação no nome).
- **Pseudo-elementos**: sintaxe `::`, criam/miram partes "virtuais" do elemento. `::before`/`::after` sempre precisam de `content` para aparecer. Existem outros (`::selection`, `::placeholder`, `::first-letter`, `::marker`) que o projeto ainda não usa, mas que se encaixam no mesmo padrão.
- **Pseudo-classes**: sintaxe `:` (um só dois-pontos), estilizam o elemento inteiro sob uma condição — estado (`:hover`, `:focus`) ou posição entre irmãos (`:last-child`, `:nth-child`). Não criam conteúdo novo, só ligam/desligam estilo conforme a situação.
- **Unidades de medida**: `px` (fixa), `rem` (relativa à fonte raiz — a mais usada no projeto), `em` (relativa à fonte do próprio elemento), `vw` (relativa à largura da tela), `%` (relativa ao pai), `s` (tempo, em transições) e valores sem unidade (`line-height`, `flex`) que são proporções, não medidas.
- **Propriedade e valor**: propriedade = "o quê" estilizar (nome fixo da linguagem CSS); valor = "quanto/como" (cor, medida, palavra-chave ou função). Propriedades como `padding`, `border` e `background` são atalhos (*shorthand*) para várias propriedades de uma vez.
- **Reset**: zera as inconsistências padrão de cada navegador (margens, listas, links, imagens) para te dar um ponto de partida limpo e 100% controlado por você.

## 8. Exercícios para fixar

1. Corrija `'Corier New'` para `'Courier New'` no `:root` e observe (ou não) diferença visual — depois pesquise por que ela quase não muda mesmo corrigido (dica: tem a ver com o fallback `monospace`).
2. Adicione `::placeholder` estilizado no `.campo-email`.
3. Adicione `::selection` no `body` para trocar a cor de fundo quando o usuário seleciona texto do site.
4. Escreva, sem consultar o código, o que `border: 1px solid rgba(255,255,255,0.06);` expande para em três propriedades separadas — depois confira comparando com a seção 3 deste guia.
5. Remova temporariamente o bloco de reset (`*, *::before, *::after{...}`) do seu CSS e recarregue a página no navegador — observe visualmente tudo que "volta" ao padrão feio do navegador (espaçamentos, bolinhas de lista, sublinhado de link) para sentir na prática por que o reset existe.
