___

Introdução
---

Já que você veio aqui, provavelmente quer aprender o funcionamento interno da computação gráfica e fazer todas as coisas legais que a galera faz por conta própria. Fazer coisas por si mesmo é extremamente divertido e útil, e te dá a possibilidade de entender de programação gráfica. Contudo, existem algumas coisas que precisam ser levadas em consideração antes de iniciar a sua jornada.

## Pré-requisitos

Já que OpenGL é uma API gráfica, e não uma plataforma por conta própria, ela precisa de uma linguagem para operá-la, e a linguagem escolhida é o <code>C++</code>. Com isso, um certo nível de conhecimento da linguagem de programção <code>C++</code> é necessário para estes capítulos. Contudo, tentarei explicar a maioria dos conceitos utilizados, incluindo tópicos avançados em <code>C++</code> onde necessário, então você não precisa ser um _expert_ em <code>C++</code>, mas deve ser capaz de escrever mais que um simples programa <code>'Hello World'</code>. Se você não possui muita experiência com <code>C++</code>, eu recomendo os tutoriais grátis do site [www.learncpp.com](https://www.learncpp.com).

Além disso, vamos estar utilizando um pouco de matemática (álgebra linear, geometria e trigonometria) pelo caminho e eu tentarei explicar todos os conceitos matemáticos necessários. Contudo, eu não sou um matemático de coração, e apenas das minhas explicações serem fáceis de entender, elas provavelmente estarão incompletas. Então, quando necessário, irei prover apontamentos para bons materiais que explicam o conteúdo de forma mais completa. Não se assuste com o conhecimento de matemática necessário antes de iniciar a sua jornada dentro do OpenGL; praticamente todos os conceitos podem ser compreendidos com conhecimentos básicos de matemática e eu vou tentar manter a matemática no nível mínimo quando possível. Boa parte das funcionalidades nem mesmo exigem que você compreenda todos os cálculos envolvidos contanto que você saiba como utilizá-las.

## Estrutura

AprendaOpenGL está dividido em algumas seções gerais. Cada seção contém diversos capítulos em que cada um explica diferentes conceitos em maiores detalhes. Cada um dos capítulos pode ser encontrado no menu a sua esquerda. Os conceitos são explicados de forma linear (então é recomendado começar do topo para o fundo, a menos que seja orientado de outra forma), onde cada capítulo explica o embasamento teórico e os aspectos práticos.

Para tornar os conceitos mais fáceis de acompanhar, e dá-los alguma estrutura, o livro contém algumas caixas, blocos de código, dicas em cores e referências para funções.

### Caixas

<note>
Caixas <strong>verdes</strong> abordam observações ou recursos/dicas úteis sobre OpenGL ou o assunto em questão.
</note>

<warning>
Caixas <strong>vermelhas</strong> contém avisos ou outras informações que você precisa ter atenção extra.
</warning>

### Código

Você vai encontrar muitos trechos de código pequenos no site, que ficam localizados em caixas preto e cinza.

<pre><code>
// Esta caixa contém código fonte
</code></pre>

Uma vez que estas fornecem apenas trechos de código, quando necessário, irei fornecer um link para o código fonte completo para um determinado assunto.

### Dicas em cores

Algumas palavras são exibidas com uma cor diferente para deixar muito claro que elas carregam um significado especial:

* <def>Definição</def>: palavras em verde são definições. Ex: um aspecto/nome importante de algo que você provavelmente vai ouvir com mais frequência.
* <fun>Estrutura do programa</fun>: palavras em vermelho definem nomes de funções ou nomes de classes.
* <var>Variáveis</var>: palavras em azul especificam variáveis, incluindo todo as constantes do OpenGL.

### Referências a funções do OpenGL

TODO Ver possibilidade

A particularly well appreciated feature of LearnOpenGL is the ability to review most of OpenGL's functions wherever they show up in the content. Whenever a function is found in the content that is documented at the website, the function will show up with a slightly noticeable underline. You can hover the mouse over the function and after a small interval, a pop-up window will show relevant information about this function including a nice overview of what the function actually does. Hover your mouse over glEnable to see it in action.

Now that you got a bit of a feel of the structure of the site, hop over to the Getting Started section to start your journey in OpenGL!
