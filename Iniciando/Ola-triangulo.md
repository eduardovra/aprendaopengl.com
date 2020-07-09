---
layout: page
title: Olá Triângulo
nav_order: 6
parent: Iniciando
---

No OpenGL, tudo está no espaço 3D, mas a tela ou a janela é uma matriz de pixels 2D, portanto grande parte do trabalho do OpenGL é sobre a transformação de todas as coordenadas 3D em pixels 2D que cabem na tela. O processo de transformação de coordenadas 3D em pixels 2D é gerenciado pelo <def>pipeline gráfico</def> do OpenGL. O pipeline gráfico pode ser dividido em duas grandes partes: a primeira transforma suas coordenadas 3D em coordenadas 2D e a segunda parte transforma as coordenadas 2D em pixels coloridos reais. Neste capítulo, discutiremos brevemente o pipeline gráfico e como podemos usá-lo em nosso benefício para criar pixels sofisticados.

O pipeline gráfico toma como entrada um conjunto de coordenadas 3D e as transforma em pixels 2D coloridos na tela. O pipeline gráfico pode ser dividido em várias etapas, nas quais cada etapa requer a saída da etapa anterior como entrada. Todas essas etapas são altamente especializadas (possuem uma função específica) e podem ser facilmente executadas em paralelo. Devido à sua natureza paralela, as placas gráficas atuais possuem milhares de pequenos núcleos de processamento para processar rapidamente seus dados no pipeline gráfico. Os núcleos de processamento executam pequenos programas na GPU para cada etapa do pipeline. Esses pequenos programas são chamados de <def>shaders</def>.

Alguns desses shaders são configuráveis ​​pelo desenvolvedor, o que nos permite escrever nossos próprios shaders para substituir os shaders padrão existentes. Isso nos dá um controle muito mais refinado sobre partes específicas do pipeline e, como elas rodam na GPU, elas também podem economizar um tempo valioso da CPU. Os shaders são escritos em uma linguagem de programação chamada <def>OpenGL Shading Language (GLSL)</def>, a qual será apronfundada no próximo capítulo.

Abaixo, você encontrará uma representação abstrata de todos os estágios do pipeline gráfico. Observe que as seções azuis representam seções nas quais podemos injetar nossos próprios shaders.

![O pipeline gráfico do OpenGL com os estágios de shading](/assets/images/pipeline.png){: .clean}

Como você pode ver, o pipeline gráfico contém um grande número de seções, cada uma lidando com uma parte específica da conversão de seus dados de vértice em um pixel totalmente renderizado. Explicaremos brevemente cada parte do pipeline de uma maneira simplificada para fornecer uma boa visão geral de como o pipeline opera.

Como entrada para o pipeline gráfico, passamos por uma lista de três coordenadas 3D que devem formar um triângulo em uma matriz aqui chamada <code>Vertex Data</code>; esses dados de vértice são uma coleção de vértices. Um <def>vertex (vértice)</def> é uma coleção de dados por coordenada 3D. Os dados desse vértice são representados usando <def>vertex attributes</def> que podem conter quaisquer dados que desejamos, mas, para simplificar, vamos supor que cada vértice consiste apenas em uma posição 3D e em algum valor de cor.

<note>
Para que o OpenGL saiba o que fazer com sua coleção de coordenadas e valores de cores, o OpenGL exige que você indique (<i>hint</i>) que tipo de renderização você deseja formar com os dados. Queremos que os dados sejam renderizados como uma coleção de pontos, uma coleção de triângulos ou talvez apenas uma linha longa? Estes hints são chamadas de <def>primitivas</def> e são dadas ao OpenGL ao chamar qualquer um dos comandos de desenho. Alguns exemplos de hints são <var>GL_POINTS</var>, <var>GL_TRIANGLES</var> e <var>GL_LINE_STRIP</var>.
</note>

A primeira parte do pipeline é o <def>vertex shader</def>, que recebe como entrada um único vértice. O principal objetivo do vertex shader é transformar as coordenadas 3D em coordenadas 3D diferentes (mais sobre isso mais tarde) e o vertex shader nos permite fazer algum processamento básico nos atributos de vértice.

O estágio de <def>primitive assembly</def> recebe como entrada todos os vértices (ou vértice se o modo <var>GL_POINTS</var> for escolhido) do vertex shader que formam uma primitiva e reúne todos os pontos na forma primitiva fornecida; neste caso, um triângulo.

A saída do estágio de primitive assembly é passada para o <def>geometry shader</def>. O geometry shader usa como entrada uma coleção de vértices que formam uma primitiva e tem a capacidade de gerar outras formas emitindo novos vértices para formar uma nova (ou outra) primitiva (s). Nesse caso de exemplo, ele gera um segundo triângulo a partir da forma especificada.

A saída do geometry shader é então passada para o <def>rasterization stage (estágio de rasterização)</def>, onde mapeia a (s) primitiva (s) resultante (s) para os pixels correspondentes na tela final, resultando em fragmentos para o fragment shader usar. Antes da execução dos fragment shaders, o <def>clipping (recorte)</def> é realizado. O clipping descarta todos os fragmentos que estão fora da sua visão, aumentando o desempenho.

<note>
Um fragmento no OpenGL corresponde a todos os dados necessários para o OpenGL renderizar um único pixel.
</note>

O principal objetivo do <def>fragment shader</def> é calcular a cor final de um pixel e, geralmente, é o estágio em que ocorrem todos os efeitos avançados do OpenGL. Normalmente, o fragment shader contém dados sobre a cena 3D que podem ser usados ​​para calcular a cor final do pixel (como luzes, sombras, cor da luz e assim por diante).

Após todos os valores de cores correspondentes terem sido determinados, o objeto final passará por mais um estágio que chamamos de <def>alpha test</def> e estágio de <def>blending</def>. Esse estágio verifica o valor de profundidade (e estêncil) correspondente (veremos mais a respeito adiante) do fragmento e os utiliza para verificar se o fragmento resultante está na frente ou atrás de outros objetos e deve ser descartado de acordo. O estágio também verifica os valores <def>alpha</def> (valores alfa definem a opacidade de um objeto) e combina os objetos de acordo. Portanto, mesmo que uma cor de saída de pixel seja calculada no fragment shader, a cor final do pixel ainda poderá ser algo completamente diferente ao renderizar vários triângulos.

Como você pode ver, o pipeline gráfico é bastante complexo e contém muitas partes configuráveis. No entanto, para quase todos os casos, precisamos trabalhar apenas com o vextex shader e o fragment shader. O geometry shader é opcional e geralmente é deixado no shader padrão. Há também o estágio de tessellation e o estágio de transform feedback loop que não descrevemos aqui, mas isso é algo para mais tarde.

No OpenGL moderno, é **obrigatório** definir pelo menos um vertex e um fragment shader (não há shaders de vértice / fragmento padrão na GPU). Por esse motivo, muitas vezes é bastante difícil começar a aprender o OpenGL moderno, pois é necessário muito conhecimento antes de poder renderizar seu primeiro triângulo. No entanto, depois de finalmente renderizar seu triângulo no final deste capítulo, você saberá muito mais sobre programação gráfica.

## Vértices de entrada

Para começar a desenhar algo, precisamos primeiro fornecer ao OpenGL alguns dados de vértices de entrada. O OpenGL é uma biblioteca de gráficos 3D, portanto, todas as coordenadas que especificamos no OpenGL estão em 3D (coordenadas <code>x</code>, <code>y</code> e <code>z</code>). O OpenGL não transforma simplesmente todas as suas coordenadas 3D em pixels 2D na tela; Ele processa apenas coordenadas 3D quando elas estão em um intervalo específico entre <code>-1,0</code> e <code>1,0</code> em todos os 3 eixos (<code>x</code>, <code>y</code> e <code>z</code>). Todas as coordenadas dentro desse intervalo, que é chamado <def>normalized device coordinates</def>, serão visíveis na tela (e todas as coordenadas fora desta região não serão).

Como queremos renderizar um único triângulo, devemos especificar um total de três vértices, com cada vértice tendo uma posição 3D. Nós os definimos em normalized device coordinates (a região visível do OpenGL) em um array de <code>floats</code>:

```cpp
float vertices[] = {
    -0,5f, -0,5f, 0,0f,
     0,5f, -0,5f, 0,0f,
     0,0f,  0,5f, 0,0f
};
```

Como o OpenGL trabalha no espaço 3D, renderizamos um triângulo 2D com cada vértice tendo uma coordenada <code>z</code> de <code>0,0</code>. Dessa forma, a profundidade do triângulo permanece a mesma, fazendo com que pareça 2D.

<note>
<strong>Normalized Device Coordinates (NDC)</strong>
<br/>
<p>
Depois que suas coordenadas de vértice forem processadas no vertex shader, elas deverão estar em <def>coordenadas de dispositivo normalizadas</def>, um espaço pequeno onde os valores de <code>x</code>, <code>y</code> e <code>z</code> variam de <code>-1,0</code> a <code>1,0</code>. Quaisquer coordenadas que estiverem fora desse intervalo serão descartadas / cortadas e não serão visíveis na tela. Abaixo, você pode ver o triângulo especificado nas coordenadas normalizadas do dispositivo (ignorando o eixo <code>z</code>):
</p>

<img src="/assets/images/ndc.png" alt="Coordenadas de dispositivos normalizados 2D, como mostrado em um gráfico" class="clean">

<p>
Diferentemente das coordenadas comuns da tela, os pontos positivos do eixo y na direção superior e as coordenadas (<code>0,0</code>) estão no centro do gráfico, em vez de no canto superior esquerdo. Eventualmente, você deseja que todas as coordenadas (transformadas) estejam nesse espaço de coordenadas, caso contrário elas não serão visíveis.
</p>

<p>
Suas coordenadas NDC serão transformadas em <def>screen-space coordinates</def> por meio da <def>viewport transform</def> usando os dados que você forneceu com o <function>glViewport</function>. As coordenadas resultantes do espaço na tela são transformadas em fragmentos como entradas para o seu fragment shader.
</p>
</note>

Com os dados de vértice definidos, gostaríamos de enviá-los como entrada para o primeiro processo do pipeline gráfico: o vertex shader. Isso é feito alocando memória na GPU onde armazenamos os dados do vértice, configuramos como o OpenGL deve interpretar a memória e especificamos como enviar os dados para a placa gráfica. O vertex shader processa todos os vértices que solicitamos a partir de sua memória.

Gerenciamos essa memória por meio dos chamados <def>vertex buffer objects</def> (<def>VBO</def>) que podem armazenar um grande número de vértices na memória da GPU. A vantagem de usar esses objetos de buffer é que podemos enviar grandes lotes de dados de uma só vez para a placa gráfica e mantê-los lá se houver memória suficiente, sem ter que enviar os dados um vértice por vez. O envio de dados para a placa gráfica da CPU é relativamente lento, portanto, sempre que pudermos, tentaremos enviar o máximo de dados possível de uma só vez. Uma vez que os dados estão na memória da placa gráfica, o vertex shader tem acesso quase instantâneo aos vértices, tornando-o extremamente rápido

Um vertex buffer object é nossa primeira ocorrência de um objeto OpenGL, como discutimos no capítulo OpenGL. Assim como qualquer objeto no OpenGL, esse buffer possui um ID exclusivo correspondente a esse buffer. Podemos gerar um ID de buffer usamos a função <function>glGenBuffers</function>:

```cpp
unsigned int VBO;
glGenBuffers(1, &VBO);
```

O OpenGL tem muitos tipos de objetos de buffer e o tipo de buffer de um objeto de buffer de vértice é <var>GL_ARRAY_BUFFER</var>. O OpenGL nos permite fazer bind em vários buffers de uma só vez, desde que eles tenham um tipo de buffer diferente. Podemos fazer o bind do buffer recém-criado ao target <var>GL_ARRAY_BUFFER</var> com a função <function>glBindBuffer</function>:

```cpp
glBindBuffer (GL_ARRAY_BUFFER, VBO);
```

A partir desse ponto, todas as chamadas de buffer que fizermos (no alvo <var>GL_ARRAY_BUFFER</var>) serão usadas para configurar o buffer atualmente vinculado, que é o <var>VBO</var>. Em seguida, podemos fazer uma chamada para a função <function>glBufferData</function> que copia os dados de vértice definidos anteriormente para memória do buffer:

```cpp
glBufferData (GL_ARRAY_BUFFER, sizeof (vértices), vértices, GL_STATIC_DRAW);
```

<function>glBufferData</function> é uma função especificamente direcionada para copiar dados definidos pelo usuário parar o buffer atualmente vinculado. Seu primeiro argumento é o tipo de buffer no qual queremos copiar dados: o vertex buffer object atualmente vinculado ao destino <var>GL_ARRAY_BUFFER</var>. O segundo argumento especifica o tamanho dos dados (em bytes) que queremos passar para o buffer; basta um <code>sizeof</code> dos dados do vértice. O terceiro parâmetro são os dados efetivamente que queremos enviar.

O quarto parâmetro especifica como queremos que a placa gráfica gerencie os dados fornecidos. Existem três formas:

* <var>GL_STREAM_DRAW</var>: os dados são configurados apenas uma vez e utilizados pela GPU algumas poucas vezes.
* <var>GL_STATIC_DRAW</var>: os dados são configurados apenas uma vez e usados ​​várias vezes.
* <var>GL_DYNAMIC_DRAW</var>: os dados são alterados com frequência e usados ​​muitas vezes.

Os dados de posição do triângulo não são alterados, são muito utilizados e permanecem os mesmos para todas as chamadas de renderização, portanto, seu tipo de uso deve ser <var>GL_STATIC_DRAW</var>. Se, por exemplo, tivéssemos um buffer contendo dados com probabilidade de alteração frequente, o tipo de uso <var>GL_DYNAMIC_DRAW</var> garantiria que a placa gráfica coloque os dados na memória, permitindo gravações mais rápidas.

A partir de agora, armazenamos os dados dos vértices na memória na placa gráfica, gerenciados por um objeto de buffer de vértice chamado <var>VBO</var>. Em seguida, queremos criar um vertex shader e um fragment shader que realmente processe esses dados, então vamos começar a construí-los.
