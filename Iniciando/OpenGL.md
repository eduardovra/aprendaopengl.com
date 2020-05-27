___
layout: page
title: OpenGL
---

Antes de iniciar a nossa jornada devemos primeiramente definir o que exatamente é o OpenGL. O OpenGL é considerado principalmente uma API (<def>Application Programming Interface</def>) que nos prove uma grande quantidade de funções que nós podemos usar para manipular gráficos e imagens. Contudo, o OpenGL, por si próprio não é uma API, mas meramente uma especificação, desenvolvida e mantida pelo [Grupo Khronos](http://www.khronos.org/).

![Imagem da logo do OpenGL](/assets/images/opengl.jpg){: .right}

A especificação do OpenGL define o que exatamente deve ser a saída/resultado de cada função e como ela deve executar. Fica a cargo dos desenvolvedores que estão implementado a especificação fornecer uma solução para como essa função deve operar. Uma vez que a especificação do OpenGL não provê os detalhes de implementação, as versões efetivamente implementadas são autorizadas a terem diferentes implementações, contanto que os seus resultados estejam de acordo com a especificação (e apresentam portanto o mesmo para o usuário).

As pessoas realmente desenvolvendo bibliotecas OpenGL são os fabricantes de placas de vídeo. Cada placa de vídeo que você compra suporta versões específicas do OpenGL, que são versões especialmente desenvolvidas para aquela série de placas. Ao usar um sistema da _Apple_, a biblioteca OpenGL é mantida pela própria _Apple_ e no Linux existe uma combinação de fornecedores de versões de placa e adaptações de _hobistas_ destas bibliotecas. Isso também significa que quando o OpenGL apresenta um comportamento estranho que não deveria, provavelmente é culpa dos fabricantes de placas (ou quem quer que tenha desenvolvido a biblioteca).

<note>
Uma vez que a maioria das implementações são criadas pelos fabricantes de placas. Quando existe um bug na implementação, ele é geralmente resolvido ao atualizar o driver da interface de vídeo; estes drivers incluem a versão mais recente do OpenGL que a sua placa suporta. Esta é uma das razões porque é recomendado atualizar regularmente seus drivers de vídeo.
</note>

O grupo Khronos hospeda publicamente todas as especificações para as versões do OpenGL. O leitor interessado, pode encontrar a especificação da versão 3.3 do OpenGL (que é a que irem utilizar) [aqui](https://www.opengl.org/registry/doc/glspec33.core.20100311.withchanges.pdf), que é uma boa leitura se você quer mergulhar nos detalhes do OpenGL (observe como eles descrevem apenas os resultados e não implementações). A especificação também prove uma excelente referência para descobrir o funcionamento exato das suas funções.

## Core-profile vs Immediate mode

No passado, usar o OpenGL significava desenvolver no <def>immediate mode</def> (frequentemente referido como o <def>pipeline de função fixa</def>), que era um método de fácil utilização para desenhar gráficos. A maior parte da funcionalidade do OpenGL estava oculta dentro da biblioteca e os desenvolvedores não tinham muito controle sobre como o OpenGL fazia os cálculos. Os desenvolvedores eventualmente ficaram afoitos por mais flexibilidade, e ao longo do tempo as especificações ficaram mais flexíveis como resultado; os desenvolvedores ganharam mais controle sobre seus gráficos. O immediate mode é realmente fácil de usar e entender, mas ele também é extremamente ineficiente. Por esta razão, a especificação começou a obsoletar as funcionalidades do immediate mode a partir da versão 3.2 e começou a engajar os desenvolvedores a desenvolver no modo <def>core-profile</def> do OpenGL, que é uma divisão da especificação do OpenGL que removeu todas as funcionalidades obsoletas antigas.

Ao utilizar o modo core-profile do OpenGL, ele nos força a utilizar práticas modernas. Caso tentarmos utilizar uma das funções obsoletadas, o OpenGL lança um erro e para de desenhar. A vantagem de aprender a abordagem moderna, é que ela é muito flexível e eficiente. Contudo, é também mais difícil de aprender. O immediate mode abstraia muitas das operações que o OpenGL executava, e enquanto era de fácil aprendizado, era difícil saber como o OpenGL estava realmente operando. A abordagem moderna força o desenvolvedor a realmente entender o OpenGL e a programação de gráficos, e ainda que seja um pouco difícil, permite muito mais flexibilidade, mais eficiência, e mais importante: um entendimento muito melhor de programação gráfica.

Esta também é uma razão porque este livro é direcionado ao modo core-profile do OpenGL na versão 3.3. Embora seja mais difícil, o esforço claramente vale a pena.

Atualmente, versões mais recentes do OpenGL estão disponíveis para escolha (no momento da escrita 4.5), então você deve se perguntar: por que eu vou querer aprender sobre a versão 3.3 do OpenGL quando a 4.5 já está disponível ? A resposta para esta questão é relativamente simples. Todas as versões futuras do OpenGL, a partir da 3.3, adicionam recursos úteis extras sem mudar o mecanismo central do OpenGL; as versões mais atuais apenas introduzem formas levemente mais eficientes ou mais úteis de completar as mesmas tarefas. O resultado é que todos os conceitos e técnicas permanecem as mesmas através das versões modernas do OpenGL, sendo assim, é perfeitamente válido aprender a versão 3.3. Quando você estiver pronto e/ou mais experiente, você poderá usar facilmente funcionalidades de versões mais novas do OpenGL.

<warning>
Ao utilizar funcionalidades da versão do OpenGL, apenas as placas de video mais modernas serão capazes de executar sua aplicação. Frequentemente, este é o motivo porque a maioria dos desenvolvedores focam em versões menores do OpenGL, habilitando as funcionalidades de versões mais recentes apenas opcionalmente.
</warning>

Em alguns capítulos você irá encontrar funcionalidades mais modernas que são devidamente sinalizadas como tais.

## Extensões

Uma das grandes funcionalidades do OpenGL é seu suporte a extensões. Sempre que um fabricante cria uma nova técnica ou uma grande otimização para renderização, esta é disponibilizada com frequência através de uma <def>extensão</def> nos drivers. Se o hardware que a aplicação roda suportar esta extensão, o desenvolvedor por utilizá-la para proporcional gráficos mais avançados ou mais eficientes. Desta forma, um desenvolvedor gráfico pode utilizar estas novas técnicas de renderização sem a necessidade de aguardar o OpenGL incluí-la nas versões futuras, simplismente verificando se a extensão é suportada pela placa de video. Com frequência, quando uma extensão se torna popular ou é muito útil, ela eventualmente se torna parte de versões futuras do OpenGL.

O desenvolvedor precisa consultar se alguma das extensões está disponível antes de utilizá-las (or usar uma biblioteca de extensões do OpenGL). Isto permite ao desenvolvedor realizar algumas coisas melhor, ou de forma mais eficiente, baseado no fato da extensão estar disponível:

```markdown
if(GL_ARB_extension_name)
{
    // Do cool new and modern stuff supported by hardware
}
else
{
    // Extension not supported: do it the old way
}
```
Com a versão 3.3 do OpenGL nós raramente precisaremos de alguma extensão para maior parte das técnicas, contudo, caso necessário, instruções adequadas serão fornecidas.

## Máquina de estados

O OpenGL é por si só uma grande máquina de estados: uma coleção de variáveis que definem como o OpenGL deve operar naquele momento. O estado do OpenGL é comumente referido como o <def>contexto</def> do OpenGL. No uso do OpenGL, nós frequentemente alteramos seu estado definindo algumas opções, manipulando alguns _buffers_ e então renderizando usando o contexto corrente.

Sempre que nós dizemos ao OpenGL que queremos desenhar linhas ao invés de triângulos por exemplo, nós alteramos o estado do OpenGL ao alterar alguma variável de contexto que define como o OpenGL deve desenhar. Assim que o contexto é alterado, ao dizer ao OpenGL que ele deve desenhar linhas, os próximos comandos de desenho vão desenhar linhas ao invés de triângulos.

Quando trabalhando com OpenGL, nós vamos passar por diversas funções de <def>alteração de estado</def>, que mudam o contexto, e várias funções de <def>utilização de estado</def>, que realizam operações baseadas no estado corrente do OpenGL. Contanto que você tenha em mente que o OpenGL é basicamente uma grande máquina de estados, a maioria das funcionalidades farão mais sentido.

## Objetos

As bibliotecas de OpenGL são escritas em C e permitem muitas derivações em outras linguagens, mas no seu núcleo, ela permanece como uma biblioteca C. Uma vez que muitas das construções da linguagem C não se traduzem bem para outras linguagens de nível mais alto, o OpenGL foi desenvolvido com diversas abstrações em mente. Uma destas abstrações são os <def>objetos</def> no OpenGL.

Um <def>objeto</def> no OpenGL é uma coleção de opções, que representa um subconjunto do estado do OpenGL. Por exemplo, nós podemos ter um objeto que representa as configurações da janela de desenho (_drawing window_); então nós podemos definir o seu tamanho, quantas cores ela suporta e assim por diante. Pode-se visualizar um objeto como uma _struct_ da linguagem C:

```cpp
struct object_name {
    float  option1;
    int    option2;
    char[] name;
};
```

Sempre que queremos usar objetos, geralmente se parece com isso (visualizando o contexto do OpenGL como uma grande estrutura):

```cpp
// The State of OpenGL
struct OpenGL_Context {
  	...
  	object_name* object_Window_Target;
  	...  	
};
```

```cpp
// create object
unsigned int objectId = 0;
glGenObject(1, &objectId);
// bind/assign object to context
glBindObject(GL_WINDOW_TARGET, objectId);
// set options of object currently bound to GL_WINDOW_TARGET
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH,  800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// set context target back to default
glBindObject(GL_WINDOW_TARGET, 0);
```

Este pequeno trecho de código é um fluxo de trabalho que você verá frequentemente ao trabalhar com o OpenGL. Primeiro, criamos um objeto e armazenamos uma referência a ele como um ID (os dados do objeto real são armazenados internamente). Em seguida, vinculamos o objeto (usando seu ID) ao local de destino no contexto (no exemplo, o local do objeto destino é definido como <var>GL_WINDOW_TARGET</var>). Em seguida, definimos as opções da janela e, finalmente, desassociamos o objeto, definindo o ID do objeto atual do destino da janela como 0. As opções que definimos são armazenadas no objeto referenciado por <var>objectId</var> e restauradas assim que o objeto for vinculado novamente a <var>GL_WINDOW_TARGET</var>.

<warning>
As amostras de código vistas até o momento são apenas aproximações de como o OpenGL realmente opera; ao longo do livre você verá vários exemplos reais.
</warning>

O melhor de usar esses objetos é que podemos definir mais de um objeto em nosso programa, definir suas opções e sempre que iniciamos uma operação que usa o estado do OpenGL, vinculamos o objeto às nossas configurações preferidas. Por exemplo, existem objetos que atuam como um contêiner para dados de modelo 3D (uma casa ou um personagem) e sempre que queremos desenhar um deles, vinculamos o objeto que contém os dados do modelo que queremos desenhar (primeiro criamos e configuramos as opções para esses objetos). Ter vários objetos nos permite especificar muitos modelos e sempre que queremos desenhar um modelo específico, simplesmente vinculamos (_bind_) o objeto correspondente antes de desenhar sem precisar definir todas as suas opções novamente.

## Vamos começar

Agora você aprendeu um pouco sobre a especificação e a biblioteca OpenGL, aprendeu mais ou menos como o OpenGL opera por baixo dos panos e viu alguns truques personalizados que o OpenGL usa. Não se preocupe se você não conseguiu entender tudo; ao longo do livro, percorreremos cada etapa e você verá exemplos suficientes para realmente entender o OpenGL.

## Recursos adicionais

* [opengl.org](https://www.opengl.org/): site oficial do OpenGL.
* [OpenGL registry](https://www.opengl.org/registry/): hospeda as especificações e extensões do OpenGL para todas as versões.
