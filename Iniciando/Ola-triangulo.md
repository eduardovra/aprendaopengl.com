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

## Vertex shader

O vertex shader é um dos shaders programáveis por pessoas como nós. O OpenGL moderno exige que, pelo menos, configuremos um vertex shader e um fragment shader, para que possamos fazer alguma renderização. Então vamos introduzir brevemente shaders e configurar dois shaders muito simples para desenhar nosso primeiro triângulo. No próximo capítulo, discutiremos os shaders com mais detalhes.

A primeira coisa que precisamos fazer é escrever o vertex shader na linguagem GLSL (OpenGL Shading Language) e compilar esse shader para que possamos usá-lo em nossa aplicação. Abaixo, você encontrará o código-fonte de um vertex shader muito básico em GLSL:

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

Como você pode ver, a linguagem GLSL é semelhante ao C. Cada shader começa com uma declaração de sua versão. Desde o OpenGL 3.3 (e superiores), os números de versão do GLSL correspondem à versão do OpenGL (a versão 420 da GLSL corresponde à versão 4.2 do OpenGL, por exemplo). Também mencionamos explicitamente que estamos usando a funcionalidade de core profile.

Em seguida, declaramos todos os atributos de vértice de entrada no vertex shader com a palavra-chave <code>in</code>. No momento, apenas nos preocupamos com dados de posição, portanto, precisamos apenas de um único vertex attribute. A GLSL tem um tipo de dados vector que contém 1 a 4 floats com base em seu dígito de sufíxo. Como cada vértice tem uma coordenada 3D, criamos uma variável de entrada <code>vec3</code> com o nome <var>aPos</var>. Também definimos especificamente o local da variável de entrada usando <code>layout (location = 0)</code>. Mais tarde, você verá por que precisamos desse local.

<note>
<strong>Vetor</strong>
<br/>
Em programação gráfica, usamos o conceito matemático de um vetor com bastante frequência, pois ele representa claramente as posições / direções em qualquer espaço e possui propriedades matemáticas úteis. Um vetor na GLSL tem um tamanho máximo de 4 e cada um de seus valores pode ser recuperado via <code>vec.x</code>, <code>vec.y</code>, <code>vec.z</code> e <code>vec.w</code> respectivamente, onde cada um deles representa uma coordenada no espaço. Observe que o componente <code>vec.w</code> não é usado como uma posição no espaço (estamos lidando com 3D, não com 4D), mas é usado para algo chamado <def>perspective division</def>. Discutiremos vetores com muito mais profundidade em um capítulo posterior.
</note>

Para definir a saída do vertex shader, precisamos atribuir os dados de posição à variável <var>gl_Position</var> predefinida, que é uma vec4 por traz dos panos. No final da função <fun>main</fun>, o que quer que tenhamos definido como <var>gl_Position</var> será usado como saída do vertex shader. Como nossa entrada é um vetor de tamanho 3, temos que convertê-lo em um vetor de tamanho 4. Podemos fazer isso inserindo os valores <code>vec3</code> dentro do construtor de <code>vec4</code> e configurando seu componente <code>w</code> como <code>1.0f</code> (será explicado o porque em um capítulo posterior).

O atual vertex shader é provavelmente o vertex shader mais simples que podemos imaginar, porque não processamos os dados de entrada e simplesmente o encaminhamos para a saída do shader. Em aplicações reais, os dados de entrada geralmente ainda não estão nas coordenadas normalizadas do dispositivo, portanto, primeiro precisamos transformar os dados de entrada em coordenadas que se enquadram na região visível do OpenGL.

## Compilando um shader

Por hora, nós pegaremos o código-fonte para o vertex shader e o armazenamos em uma string const C na parte superior do código fonte:

```cpp
const char *vertexShaderSource = "#version 330 core\n"
    "layout (location = 0) in vec3 aPos;\n"
    "void main()\n"
    "{\n"
    "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
    "}\0";
```

Para que o OpenGL use o shader, é necessário compilá-lo dinamicamente em tempo de execução a partir do código-fonte. A primeira coisa que precisamos fazer é criar um objeto shader, novamente referenciado por um ID. Portanto, armazenamos o vertex shader como um <code>unsigned int</code> e criamos o shader com <function>glCreateShader</function>:

```cpp
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
```

Fornecemos o tipo de shader que queremos criar como argumento para <function>glCreateShader</function>. Como estamos criando um vertex shader, passamos <var>GL_VERTEX_SHADER</var>.

Em seguida, anexamos o código-fonte do shader ao objeto shader e compilamos:

```cpp
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

A função <function>glShaderSource</function> recebe o shader object para compilar como seu primeiro argumento. O segundo argumento especifica quantas strings estamos passando como código-fonte, que é apenas uma. O terceiro parâmetro é o código fonte real do vertex shader e podemos deixar o quarto parâmetro como <var>NULL</var>.

<note markdown="1">
É uma boa prática verificar se a compilação foi bem-sucedida após a chamada ao <function>glCompileShader</function> e, caso não, quais erros foram encontrados para que você possa corrigi-los. A verificação de erros em tempo de compilação é realizada da seguinte maneira:
<br/>

<code><pre>
int  success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
</pre></code>

<br/>
Primeiro, definimos um integer para indicar sucesso e um contêiner de armazenamento para as mensagens de erro (se houverem). Em seguida, verificamos se a compilação foi bem-sucedida com a função <function>glGetShaderiv</function>. Se a compilação falhar, devemos obter a mensagem de erro com <function>glGetShaderInfoLog</function> e imprimi-la.
<br/>

<code><pre>
if(!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
</pre></code>
</note>

Se nenhum erro foi detectado durante a compilação do vertex shader, ele estará compilado.

## Fragment shader

O fragment shader é o segundo e último shader que vamos criar para renderizar um triângulo. O fragment shader tem tudo a ver com o cálculo da saída de cores dos seus pixels. Para simplificar, o fragment shader sempre exibirá uma cor laranja.

<note>
As cores na computação gráfica são representadas como uma matriz de 4 valores: o componente vermelho, verde, azul e alfa (opacidade), comumente abreviado para RGBA. Ao definir uma cor no OpenGL ou GLSL, definimos a força de cada componente para um valor entre <code>0.0</code> e <code>1.0</code>. Se, por exemplo, definirmos vermelho em <code>1.0</code> e verde em <code>1.0</code>, obteremos uma mistura das duas cores resultando na cor amarela. Dados esses três componentes de cores, podemos gerar mais de 16 milhões de cores diferentes!
</note>

```glsl
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
} 
```

O fragment shader requer apenas uma variável de saída e esse é um vetor de tamanho 4 que define a saída de cor final que devemos calcular. Podemos declarar valores de saída com a palavra-chave <code>out</code>, que chamamos aqui prontamente de <var>FragColor</var>. Em seguida, simplesmente atribuímos um <code>vec4</code> à saída de cor como uma cor laranja com um valor alfa de <code>1.0</code> (<code>1.0</code> sendo completamente opaco).

O processo para compilar um fragment shader é semelhante ao vertex shader, embora desta vez utilizemos a constante <var>GL_FRAGMENT_SHADER</var> como o tipo de shader:

```glsl
unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);
```

Agora, os dois shaders estão compilados e a única coisa a fazer é vincular os dois objetos de shader a um <def>shader program</def> que podemos usar para renderizar. Verifique também se há erros de compilação aqui!

### Shader program

Um objeto shader program é a versão linkada final de vários shaders combinados. Para usar os shaders compilados recentemente, precisamos fazer o <def>link</def> deles a um object shader program e, em seguida, ativar esse shader program ao renderizar objetos. Os shaders do shader program ativado serão usados quando efetuarmos chamadas de renderização.

Ao linkar os shaders a um programa, ele vincula as saídas de cada shader às entradas do próximo shader. Também é aqui que você obterá erros de linking se suas saídas e entradas não corresponderem.

Criar um program object é fácil:

```cpp
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
```

A função <function>glCreateProgram</function> cria um programa e retorna um ID de referência para o program object recém-criado. Agora precisamos anexar os shaders compilados anteriormente ao objeto do programa e depois linka-los usando <function>glLinkProgram</function>:

```cpp
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);
```

O código deve ser bastante autoexplicativo, anexamos os shaders ao programa e os vinculamos via <function>glLinkProgram</function>.

<note>
Assim como a compilação do shader, também podemos verificar se o link de um shader program falhou e recuperar o log correspondente. No entanto, em vez de usar <function>glGetShaderiv</function> e <function>glGetShaderInfoLog</function>, agora usamos:
<br/>

<code><pre>
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
if(!success) {
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
    ...
}
</pre></code>
</note>

O resultado é um program object que podemos ativar chamando <function>glUseProgram</function> com o objeto de programa recém-criado como argumento:

```cpp
glUseProgram(shaderProgram);
```

Cada chamada de shader ou de renderização após o <function>glUseProgram</function> usará esse program object (e, portanto, os shaders).

Ah, sim, e não se esqueça de excluir os shader objects assim que os linkarmos ao program object; não precisamos mais deles:

```cpp
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```

No momento, enviamos os dados de vértice de entrada para a GPU e instruímos a GPU como deve processar os dados de vértice dentro de um vertex shader e um fragment shader. Estamos quase lá. O OpenGL ainda não sabe como deve interpretar os dados de vértice na memória e como deve conectar os dados de vértice aos attributes do vertex shader. Seremos legais e diremos ao OpenGL como fazer isso.

## Linking dos Vertex Attributes

O vertex shader nos permite especificar qualquer entrada que desejamos na forma de vertex attributes e, embora isso permita uma grande flexibilidade, significa que temos que especificar manualmente qual parte de nossos dados de entrada vai para qual atributo de vértice no shader de vértice. Isso significa que precisamos especificar como o OpenGL deve interpretar os dados do vértice antes da renderização.

Nossos dados de buffer de vértice são formatados da seguinte maneira:

![Setup do ponteiro de vertex attributesno VBO do OpenGL](/assets/images/vertex_attribute_pointer.png){: .clean}

* Os dados da posição são armazenados como valores de ponto flutuante de 32 bits (4 bytes).
* Cada posição é composta por 3 desses valores.
* Não há espaço (ou outros valores) entre cada conjunto de 3 valores. Os valores estão <def>tightly packed</def> no array.
* O primeiro valor nos dados está no início do buffer.

Com esse conhecimento, podemos dizer ao OpenGL como ele deve interpretar os dados do vértice (por vertex attribute) usando <function>glVertexAttribPointer</function>:

```cpp
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0); 
```

A função <function>glVertexAttribPointer</function> possui alguns parâmetros, portanto, vamos examiná-los com cuidado:

* O primeiro parâmetro especifica qual vertex attribute queremos configurar. Lembre-se de que especificamos a <var>position</var> do vertex attribute np vertex shader com o <code>layout (localização = 0)</code>. Isso define o local do atributo de vértice como <code>0</code> e, como queremos passar dados para esse vertex attribute, passamos <code>0</code>.
* O próximo argumento especifica o tamanho do atributo de vértice. O vertex attribute é um <code>vec3</code>, portanto é composto por <code>3</code> valores.
* O terceiro argumento especifica o tipo de dados que é <var>GL_FLOAT</var> (um <code>vec*</code> na GLSL consiste em valores float).
* O próximo argumento especifica se queremos que os dados sejam normalizados. Se estivermos inserindo tipos de dados inteiros (int, byte) e definimos como <var>GL_TRUE</var>, os dados inteiros são normalizados para <code>0</code> (ou <code>-1</code> para dados com sinal) e <code>1</code> quando convertidos para float. Isso não é relevante para nós, portanto deixaremos em <var>GL_FALSE</var>.
* O quinto argumento é conhecido como <def>stride</def> e nos diz o espaço entre atributos consecutivos do vértice. Como o próximo conjunto de dados de posição está localizado exatamente 3 vezes o tamanho de um <code>float</code>, especificamos esse valor como o stride. Observe que, como sabemos que a matriz está tightly packed (não há espaço entre o próximo valor do vertex attribute), também poderíamos ter especificado o stride como <code>0</code> para permitir que o OpenGL determine o stride (isso só funciona quando os valores estão bem compactados). Sempre que temos mais vertex attributes, precisamos definir cuidadosamente o espaçamento entre cada atributo de vértice, mas veremos mais exemplos disso posteriormente.
* O último parâmetro é do tipo <code>void*</code> e, portanto, requer essa cast estranho. Esse é o <def>offset</def> de onde os dados de posição começam no buffer. Como os dados da posição estão no início do array de dados, esse valor é <code>0</code>. Vamos explorar esse parâmetro com mais detalhes posteriormente.

<note>
Cada vertex attribute obtém seus dados da memória gerenciada por um VBO, e o que determina qual VBO é utilizado (você pode ter vários VBOs) é o VBO corrente em bind ao chamar a função <function>glVertexAttribPointer</function> passando <var>GL_ARRAY_BUFFER</var> como parâmetro. Como o VBO definido anteriormente ainda está em bind antes de chamarmos a função <function>glVertexAttribPointer</function>, o vertex attribute <code>0</code> agora ficou corretamente associado aos seus dados de vértices.
</note>

Agora que especificamos como o OpenGL deve interpretar os dados do vértice, também devemos habilitar o vertex attribute com a função <function>glEnableVertexAttribArray</function>, fornecendo a localização do vertex attribute como argumento; vertex attributes são desativados por padrão. A partir desse ponto, configuramos tudo: inicializamos os dados do vértice em um buffer usando um vertex buffer object, configuramos um vertex shader e um fragment shader e dissemos ao OpenGL como vincular os dados do vértice aos atributos de vértice do vertex shader. Desenhar um objeto no OpenGL agora ficaria assim:

```cpp
// 0. copy our vertices array in a buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 1. then set the vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);  
// 2. use our shader program when we want to render an object
glUseProgram(shaderProgram);
// 3. now draw the object 
someOpenGLFunctionThatDrawsOurTriangle();
```

Temos que repetir esse processo toda vez que queremos desenhar um objeto. Pode não parecer muito, mas imagine se tivermos mais de 5 vertex attributes e talvez centenas de objetos diferentes (o que não é incomum). Fazer o bind de buffer objects e configurar todos os vertex attributes para cada um desses objetos rapidamente se torna um processo complicado. E se houvesse alguma maneira de armazenar todas essas configurações de estado em um objeto e simplesmente fazendo o bind deste objeto restaurar seu estado ?
