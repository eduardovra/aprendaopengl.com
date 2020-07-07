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
