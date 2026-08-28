# Guia de Estudo: HTML, CSS, `div`, `id` e `class` — Análise do site TechBlog

Este guia usa o seu próprio site (`index.html` + `style.css`) como material de estudo. A ideia é: para cada conceito, você vê a **regra geral** e logo em seguida o **trecho real do seu código** que a aplica. Assim você aprende a teoria já vendo ela funcionando.

---

## 1. A base de tudo: o que é uma `<div>`

Uma `<div>` é uma **caixa genérica sem significado próprio**. Ela não é um título, não é um parágrafo, não é uma lista — é só um "container" que existe para você organizar o layout. Pense nela como uma caixa de papelão vazia: o valor dela está no que você põe dentro e em como você a estiliza.

Isso explica por que seu HTML é praticamente uma sequência de `<div>`s aninhadas (uma dentro da outra):

```html
<div id="topo">
  <div class="logo">Tech<span>Blog</span></div>
  <div id="navegacao"> ... </div>
</div>
```

Sem CSS, essas divs não teriam nenhuma aparência especial — seriam blocos empilhados sem cor, sem espaçamento, uma embaixo da outra. É o CSS que transforma essas caixas em navbar, seção hero, cards, sidebar, etc.

**Ponto importante para estudar depois:** existem tags "semânticas" (`<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`) que fazem exatamente o que a `<div>` faz visualmente, mas *comunicam significado* para o navegador, para leitores de tela e para o Google. Seu wireframe já menciona isso ("Tags semânticas: quando parar de usar `<div>` para tudo"). No seu site atual, `#topo` poderia virar `<header>`, `#navegacao` um `<nav>`, `#rodape` um `<footer>` — o visual seria idêntico, mas o código ficaria mais correto semanticamente.

---

## 2. `id` vs `class` — a diferença que organiza o projeto inteiro

Essa é a distinção central do seu wireframe, e seu HTML segue essa regra com bastante disciplina:

| | `id` | `class` |
|---|---|---|
| Quantas vezes pode aparecer na página | **Uma única vez** | Quantas vezes quiser |
| Para que serve | Identificar uma seção **única** do layout | Estilizar **padrões repetíveis** |
| Sintaxe no HTML | `id="topo"` | `class="card-artigo"` |
| Sintaxe no CSS | `#topo { }` | `.card-artigo { }` |
| Especificidade (prioridade) | Alta | Baixa (perde para `id`) |

### Exemplo de `id` no seu código — elementos únicos
```html
<div id="topo">        <!-- só existe UM topo na página -->
<div id="hero">        <!-- só existe UMA seção hero -->
<div id="sidebar">     <!-- só existe UMA sidebar -->
<div id="rodape">      <!-- só existe UM rodapé -->
```
Cada um desses aparece **uma vez só**. Faz sentido dar um `id`, porque você nunca vai precisar repetir essa exata caixa em outro lugar da página.

### Exemplo de `class` no seu código — padrões repetidos
```html
<div class="card-artigo html">   <!-- card 1 -->
<div class="card-artigo css">    <!-- card 2 -->
<div class="card-artigo js">     <!-- card 3 -->
<div class="card-artigo html avancado">  <!-- card 4 -->
```
Aqui você tem **quatro divs diferentes** compartilhando a classe `card-artigo`. Todas herdam o mesmo visual base (borda, cantos arredondados, layout interno) porque no CSS existe uma única regra `.card-artigo { }` que atinge as quatro ao mesmo tempo. Se amanhã você adicionar um quinto artigo, basta copiar a estrutura e ele já nasce estilizado — não precisa escrever CSS novo.

**A regra prática que fica para você:** pergunte-se "isso vai se repetir?". Se sim → `class`. Se é algo que só existe uma vez na página inteira → `id`.

---

## 3. Múltiplas classes no mesmo elemento (composição de estilos)

Este é um dos conceitos mais poderosos do seu CSS, e o card 4 do seu blog é o melhor exemplo:

```html
<div class="card-artigo html avancado">
```

Uma `class` pode conter **vários nomes separados por espaço**. Cada nome é uma classe independente, e o elemento recebe o CSS de todas ao mesmo tempo, como camadas empilhadas:

- `.card-artigo` → dá a estrutura base do card (borda, flex, cantos arredondados)
- `.html` → colore a tag e o card no tom laranja de HTML (ver linhas ~380-404 do seu CSS)
- `.avancado` → adiciona a borda lateral vermelha extra:
  ```css
  .card-artigo.avancado{
      border-left: 4px solid var(--cor-acento);
  }
  ```

Repare que `.card-artigo.avancado` (sem espaço entre os pontos) é diferente de `.card-artigo .avancado` (com espaço). **Sem espaço** = "o elemento precisa ter as duas classes ao mesmo tempo". **Com espaço** = "um elemento com classe `avancado` que esteja dentro de um elemento com classe `card-artigo`" (isso é um seletor descendente, que vem no próximo tópico). No seu CSS, `.card-artigo.avancado` está correto porque `avancado` é uma classe extra no *mesmo* elemento, não um filho.

Essa técnica (classe base + classes modificadoras) é uma metodologia real muito usada na indústria, chamada de **BEM** ou variações dela — você já está aplicando isso intuitivamente.

---

## 4. Seletores descendentes — como o CSS "acha" elementos aninhados

Vários trechos do seu CSS usam dois seletores separados por espaço, o que significa "procure um elemento dentro de outro":

```css
#navegacao a{ ... }
```
Traduzindo: "todo `<a>` que estiver **dentro** de `#navegacao`". Por isso os links do menu ficam com aquele estilo cinza-azulado, mas um `<a>` em outra parte do site não é afetado — só os que moram dentro da navbar.

```css
.widget .lista-categorias li a{ ... }
```
Traduzindo: "todo `<a>`, dentro de um `<li>`, dentro de algo com classe `.lista-categorias`, dentro de algo com classe `.widget`". Cada espaço é um nível de aninhamento. É assim que o CSS consegue estilizar só os links da lista de categorias sem afetar os links da nuvem de tags, mesmo os dois estando dentro de `.widget`.

```css
#rodape-base .redes a{ ... }
```
Mistura um `id` com uma `class` e uma tag — isso é totalmente permitido e muito comum: comece pelo container mais externo (o `id`, que é único) e vá refinando até o elemento que você quer atingir.

---

## 5. Especificidade — por que a ordem e o tipo de seletor importam

Especificidade é a "régua de prioridade" que o navegador usa quando duas regras CSS tentam estilizar o mesmo elemento. Do mais fraco para o mais forte:

1. seletor de tag (`a`, `div`, `h1`) — mais fraco
2. classe (`.card-artigo`) e pseudo-classe (`:hover`)
3. `id` (`#topo`) — mais forte
4. `style="..."` inline no HTML — mais forte ainda (evite usar)

No seu CSS isso aparece em:
```css
a{ text-decoration: none; color: inherit; }        /* regra geral, fraca */
#navegacao a{ color:#aab4c8; ... }                   /* mais específica, vence dentro da navbar */
#navegacao a.btn-assinar{ background-color: ...; }   /* ainda mais específica, vence sobre a anterior */
```
As três regras miram no mesmo `<a>` dentro do botão "Assinar", mas a mais específica (a que combina `id` + tag + `class`) é quem decide a cor final. Entender isso evita a maior frustração de quem está aprendendo CSS: "escrevi a regra e não mudou nada" — geralmente é porque outra regra mais específica está ganhando.

---

## 6. Pseudo-classes e pseudo-elementos

Seu CSS usa bastante os dois, que são fáceis de confundir:

**Pseudo-classe** (`:algo`) — estiliza um elemento em um **estado** específico:
```css
#navegacao a:hover{ color: #fff; background-color: rgba(255,255,255,0.08); }
.campo-email:focus{ border-color: var(--cor-acento); }
.widget .lista-categorias li:last-child{ border-bottom: none; }
.card-artigo:hover .card-corpo h2{ color:var(--cor-acento) }
```
Esse último é interessante: quando você passa o mouse sobre o **card inteiro**, o `h2` **dentro** dele muda de cor — mesmo o hover não sendo diretamente no h2. Isso é um recurso muito usado para criar cards interativos "clicáveis" na aparência.

**Pseudo-elemento** (`::algo`) — cria um pedaço extra de conteúdo que não existe no HTML:
```css
.categoria::before{
    content: '';
    width: 28px;
    height: 2px;
    background-color: var(--cor-acento);
}

.hero-imagem::after{
    content: '</>';
    ...
}
```
Aqui o CSS está literalmente **inserindo** um tracinho decorativo antes do texto "Artigo em destaque" e o símbolo `</>` depois do conteúdo de `.hero-imagem` — sem você precisar escrever nenhuma tag extra no HTML. É uma técnica ótima para decoração pura, que não deveria carregar conteúdo importante (leitores de tela costumam ignorar `::before`/`::after`).

---

## 7. Variáveis CSS (Custom Properties) — o `:root`

```css
:root{
    --cor-fundo:      #f8f7f4;
    --cor-acento:     #e63946;
    --fonte-display:  Georgia, serif;
    --raio:           10px;
    --transicao:      0.22s ease;
}
```

`:root` é basicamente "o documento inteiro". Ao declarar variáveis ali, você cria um **dicionário de valores reutilizáveis** que usa em qualquer lugar com `var(--nome-da-variavel)`:

```css
border-bottom: 3px solid var(--cor-acento);
color: var(--cor-acento);
background-color: var(--cor-acento);
```

A vantagem prática: seu vermelho de destaque (`--cor-acento: #e63946`) é usado em pelo menos 10 lugares diferentes do CSS. Se você decidir trocar a cor da marca, muda **uma linha** no `:root` e o site inteiro se atualiza sozinho. Sem variáveis, você teria que caçar e trocar `#e63946` manualmente em cada regra — e provavelmente esqueceria alguma.

Você também tem variáveis temáticas específicas por categoria: `--cor-html`, `--cor-css`, `--cor-js`, usadas junto com as classes `.html`, `.css`, `.js` dos cards — uma boa prática de "sistema de design" em miniatura.

---

## 8. O reset universal e o `box-sizing`

```css
*, *::before, *::after{
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}
```

O `*` é o "seletor universal" — atinge **todo** elemento da página, incluindo pseudo-elementos. Duas coisas essenciais acontecem aqui:

1. **Zerar margin/padding**: cada navegador tem estilos padrão diferentes para `<h1>`, `<ul>`, `<body>`, etc. Zerar tudo dá um "ponto de partida" limpo e previsível, para você mesmo definir cada espaçamento depois.
2. **`box-sizing: border-box`**: sem isso, quando você define `width: 300px` e depois soma `padding` e `border`, o elemento fica *maior* que 300px (o padding e a borda se somam à largura). Com `border-box`, os 300px já incluem padding e borda — o que é muito mais fácil de prever ao montar layouts.

---

## 9. Flexbox — o motor por trás de quase todo layout do site

`display: flex` é a técnica de layout mais usada no seu CSS. A lógica: você transforma um elemento pai em um "container flexível", e os filhos diretos dele se organizam automaticamente em linha (ou coluna).

### Navbar — itens lado a lado, com espaço entre eles
```css
#topo{
    display: flex;
    justify-content: space-between;  /* logo de um lado, menu do outro */
    align-items: center;             /* tudo alinhado verticalmente ao centro */
}
```

### Hero — texto de um lado, imagem do outro, proporções diferentes
```css
#hero{ display: flex; gap: 3.5rem; align-items: center; }
.hero-texto{ flex: 1; }        /* ocupa todo o espaço "sobrando" */
.hero-imagem{ flex: 0 0 380px; } /* tamanho fixo de 380px, não cresce nem encolhe */
```
`flex: 1` e `flex: 0 0 380px` são a "receita" mais importante do flexbox pra decorar. `flex` é um atalho para três valores: `flex-grow flex-shrink flex-basis`.
- `flex: 1` = "cresça para ocupar o espaço livre"
- `flex: 0 0 380px` = "não cresça, não encolha, comece com 380px fixos"

Esse mesmo padrão se repete em `#wrapper-principal` (`#coluna-artigos{ flex: 1 }` vs `#sidebar{ flex: 0 0 290px }`): a coluna de artigos é elástica e ocupa o espaço restante, a sidebar tem largura fixa de 290px. É assim que se cria um layout de "conteúdo + barra lateral" clássico.

### Coluna (não linha)
```css
#sidebar{
    display: flex;
    flex-direction: column;   /* empilha os widgets verticalmente em vez de lado a lado */
    gap: 1.2rem;
}
```
Por padrão o flex organiza em **linha**. `flex-direction: column` inverte para empilhar de cima para baixo — é assim que os 4 widgets da sua sidebar ficam um embaixo do outro, com `gap` criando o espaçamento entre eles sem precisar de `margin` manual em cada um.

### `flex-wrap` — quando não cabe tudo em uma linha
```css
.numeros-grid{ display: flex; flex-wrap: wrap; }
```
Isso permite que, em telas pequenas, os itens "quebrem linha" em vez de espremer ou vazar da tela — um primeiro passo simples rumo a responsividade.

---

## 10. `position: sticky` — a navbar que "gruda" no topo

```css
#topo{
    position: sticky;
    ...
}
```
`sticky` faz o elemento se comportar normalmente (como se fosse `static`) até que o usuário role a página o suficiente para que ele "bateria no topo" — a partir daí, ele gruda e para de rolar junto com o resto do conteúdo. É por isso que sua barra de navegação fica sempre visível conforme o visitante desce a página. (Vale notar: para funcionar 100%, geralmente também se define `top: 0`, que não aparece no seu CSS atual — pode valer a pena testar se a navbar realmente gruda corretamente ou "escapa" em alguns navegadores.)

---

## 11. Cores dinâmicas com `gradient`, `rgba` e transparência

```css
background: linear-gradient(135deg, #1a2a5e 0%, #0f3460 100%);
color: rgba(255,255,255,0.08);
```
- `linear-gradient(ângulo, cor-inicial, cor-final)` cria uma transição suave entre cores — usado no fundo do hero, dos cards e do "post em destaque".
- `rgba(r, g, b, alfa)` é como `rgb`, mas com um quarto valor (`alfa`) de 0 a 1 controlando a **transparência**. `rgba(255,255,255,0.08)` é branco quase invisível — usado para criar aquele efeito sutil de "leve destaque" no hover da navbar, sem precisar calcular uma cor nova manualmente.

---

## 12. `clamp()` — fonte responsiva em uma linha só

```css
#hero h1{
    font-size: clamp(1.9rem, 3.5vw, 2.9rem);
}
```
`clamp(mínimo, preferido, máximo)` é uma função que diz: "o tamanho ideal é 3.5% da largura da tela (`3.5vw`), mas nunca deixe ficar menor que 1.9rem nem maior que 2.9rem". É uma forma moderna e enxuta de fazer texto responsivo sem escrever várias `@media queries` só para ajustar o tamanho da fonte.

---

## 13. Transições — a diferença entre "mudar" e "mudar suavemente"

```css
transition: all var(--transicao); /* var(--transicao) = 0.22s ease */
```
Sem `transition`, qualquer mudança de estado (como cor no `:hover`) acontece instantaneamente — de um frame para o outro. Com `transition`, o navegador anima a mudança ao longo do tempo definido. É o que dá aquela sensação de "polimento" quando você passa o mouse sobre os links do menu ou os botões do site.

⚠️ **Pequeno bug para você achar e corrigir como exercício**: na linha do `.btn-ler`, seu CSS escreveu `transform: all var(--transicao);` em vez de `transition: all var(--transicao);` — `transform` é outra propriedade (usada para rotacionar/mover elementos), então essa linha não faz nada de transição no botão de "Ler artigo". Um bom exercício de leitura de código é caçar esse tipo de erro de digitação.

---

## 14. IDs dentro de IDs — hierarquia sem conflito

```html
<div id="rodape">
  ...
  <div id="rodape-base"> ... </div>
</div>
```
Como cada `id` é único **na página inteira**, não há problema em ter `#rodape-base` dentro de `#rodape` — são dois identificadores diferentes, cada um aparecendo uma única vez. No CSS, isso permite dois níveis de controle: uma regra geral para tudo dentro de `#rodape`, e uma regra mais específica só para a barrinha final:
```css
#rodape{ background-color: #0e1628; ... }
#rodape-base{ display: flex; justify-content: space-between; ... }
```

---

## 15. Mapa mental resumido do seu HTML

```
<div id="topo">              → só existe 1x → id
  .logo                      → só existe 1x, mas usa class (poderia ser id também — escolha estilística)
  <div id="navegacao">       → só existe 1x → id
    <a>                      → repete várias vezes, sem classe própria (herda de #navegacao a)
    <a class="btn-assinar">  → 1 exceção dentro do grupo → precisa de class para diferenciar

<div id="hero">              → só existe 1x → id
  .hero-texto / .hero-imagem → duas partes fixas, mas usam class (não precisam repetir, então tanto faz — os autores escolheram class por consistência com o resto)

<div id="coluna-artigos">    → só existe 1x → id
  .card-artigo (x4)          → se repete → class
    .card-artigo.html        → variação por categoria → 2ª class
    .card-artigo.avancado    → variação extra → 3ª class

<div id="sidebar">           → só existe 1x → id
  .widget (x4)               → se repete → class
    .widget.newsletter       → variação → 2ª class
    .widget.destaque-lateral → variação → 2ª class

<div id="rodape">            → só existe 1x → id
  .rodape-col (x3)           → se repete → class
  <div id="rodape-base">     → só existe 1x, mesmo dentro de outro id → id
```

Esse padrão — **id para a "casca" estrutural única, class para blocos que se repetem ou variam** — é o esqueleto conceitual do projeto inteiro. Uma vez que você enxerga esse padrão, consegue prever como qualquer outro site profissional vai estruturar seu HTML/CSS, porque é a convenção mais comum do mercado.

---

## 16. Sugestões de exercícios para fixar

1. **Ache o bug do `transform`** (seção 13) e corrija para `transition`.
2. Adicione um 5º card de artigo copiando a estrutura de um existente — repare que ele já nasce estilizado, sem escrever CSS novo.
3. Crie uma nova variável `--cor-sucesso` no `:root` e use-a para estilizar o botão da newsletter de outra cor.
4. Troque as tags genéricas `<div id="topo">`, `<div id="navegacao">` e `<div id="rodape">` pelas semânticas `<header>`, `<nav>` e `<footer>` — o CSS deve continuar funcionando quase sem alterações, já que os seletores são por `id`, não por tag.
5. Use as ferramentas de desenvolvedor do navegador (F12 → aba "Elements"/"Inspector") para clicar em um card e ver, ao vivo, quais regras CSS estão sendo aplicadas e qual delas está "vencendo" por especificidade.
