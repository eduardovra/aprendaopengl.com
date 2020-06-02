---
layout: page
title: Criando uma janela
nav_order: 4
parent: Iniciando
---

A primeira coisa que precisamos fazer antes de começar a criar gráficos incríveis é criar um contexto OpenGL e uma janela da aplicação para desenhar. Contudo, estas operações são específicas de cada sistema operacional, e o OpenGL propositalmente tenta se abstrair destas operações. Isso significa que, precisamos criar uma janela, definir um contexto, e gerenciar as entradas do usuário por nós mesmos.

Por sorte, existem diversas bibliotecas por aí que fornecem a funcionalidade que estamos procurando, e algumas focadas especificamente no OpenGL. Estas bibliotecas nos poupam do trabalho de ter que lidar com os detalhes de cada sistema operacional, e nos fornecem uma janela e um contexto do OpenGL pronto para renderização. Algumas das bibliotecas mais populares são GLUT, SDL, SFML e GLFW. No AprendaOpenGL nós vamos utilizar a GLFW. Fique a vontade para utilizar qualquer uma das outras bibliotecas, o _setup_ da maioria é similar ao da GLFW.

## GLFW

A GLFW é uma biblioteca escrita em C, focada especificamente no OpenGL. Ela nos provê com as necessidades básicas para renderizar coisas na tela. Ela nos permite criar um contexto OpenGL, definir os parâmetros da janela, e lidar com as entradas de usuário, o que é mais do suficiente para os nossos propósitos.

O foco deste, e do próximo capítulo, é deixar a GLFW rodando e garantir que ela crie adequadamente um contexto do OpenGL, e exiba uma janela simples pra que possamos brincar. Este capítulo usa uma abordagem passo-a-passo para obter, fazer o _build_ e _linkar_ a biblioteca GLFW. Nós vamos usar a IDE Microsoft Visual Studio 2019 (note que o processo é o mesmo nas versões mais recentes do visual studio). Caso você não esteja usando o Visual Studio (ou usando uma versão antiga), não se preocupe, o processo vai ser similar na maioria das IDEs.

## Compilando a GLFW

A GLFW pode ser obtida a partir da página de [downloads](http://www.glfw.org/download.html) no site do projeto. Ela já possui binários pré-compilados e os arquivos de cabeçalho (_headers_) para o Visual Studio desde a versão 2012 até a 2019. Mas para fazer o serviço completo nós vamos compilar a GLFW nós mesmos a partir do código fonte. O objetivo disso é lhe dar uma noção do processo de compilação de bibliotecas _open-source_ por conta própria, já que nem toda biblioteca disponibiliza os binários pré-compilados. Então, vamos fazer o download do _Source Package_ (pacote com código fonte).

<warning>
Nós vamos compilar todas as bibliotecas como binários 64-bit, então confirme que você pegou os binários de 64-bit caso decida utilizar os pré-compilados.
</warning>

Quando você tiver feito o download do pacote com o código fonte, faça extração do arquivo e abra o seu conteúdo. Nós estamos interessados apenas em alguns poucos itens:

* A biblioteca resultado da compilação.
* O diretório de **include**.

Compilar a biblioteca a partir do código fonte garante que o resultado é perfeitamente adaptado para a sua CPU e o seu sistema operacional, um luxo que binários pré-compilados nem sempre provêem (as vezes, as bibliotecas pré-compiladas nem vão estar disponíveis para o seu sistema). O problema em disponibilizar o código fonte para o mundo inteiro, contudo, é que nem todos usam a mesma IDE ou sistema de _build_ para desenvolver suas aplicações. O que significa que os arquivos do projeto/solução podem não ser compatíveis com o _setup_ de outras pessoas. Então as pessoas precisam configurar o seu próprio projeto ou solução com os arquivos .c/.cpp e .h/.hpp, o que é trabalhoso. Exatamente por estas razões existe uma ferramenta chamada CMake.

### CMake

O CMake é uma ferramenta que pode gerar os arquivos de projeto/solução de escolha do usuário (ex. Visual Studio, Code::Blocks, Eclipse) a partir de uma coleção de arquivos de código fonte, usando scripts do CMake pré-definidos. Isso nos permite gerar um arquivo de projeto para o Visual Studio 2019, a partir do _source package_ da biblioteca GLFW, o que vai nos permitir então compilar a biblioteca. Primeiramente nós precisamos baixar e instalar o CMake, o que pode ser feito na seção de [downloads](http://www.cmake.org/cmake/resources/software.html) do projeto.

Quando o CMake estiver instalado, você pode optar por executá-lo a partir da linha de comandos ou através do GUI (interface gráfica). Já que não estamos tentando complicar demais as coisas vamos usar a interface gráfica. O CMake exige um diretório de origem com o código fonte e um diretório de destino para os binários. Para o diretório do código fonte, nós vamos escolher a pasta raiz que contém o código fonte da biblioteca GLFW. Para o diretório de build nós vamos criar uma nova pasta _build_ e então selecioná-la.

![Imagem com a logo do CMake](/assets/images/cmake.png)

Quando os diretórios _destination_ (destino) e _source_ (origem) estiverem definidos, clique no botão <code>Configure</code> para o que <code>CMake</code> possa ler os parâmetros necessários e o código fonte. Então nós precisamos escolher o _generator_ para o projeto, e uma vez que estamos utilizando o Visual Studio 2019, nós vamos escolher a opção <code>Visual Studio 16</code> (o Visual Studio 2019 também é conhecido como Visual Studio 16). O Cmake então vai exibir as opções possíveis para configurar a compilação da biblioteca. Podemos deixar as opções padrão e clicar em <code>Configure</code> novamente para salvar as configurações. Agora que as configurações estão definidas, nós clicamos em <code>Generate</code> e os arquivos resultantes do projeto serão gerados na sua pasta <code>build</code>.

### Compilação

No diretório <code>build</code> agora existe um arquivo chamado <code>GLFW.sln</code> e nós podemos abri-lo com o Visual Studio 2019. Considerando que o arquivo de projeto gerado pelo CMake já possui os parâmetros de configuração, nós precisamos apenas fazer o _build_ da solução. O CMake já deve ter configurado a solução para compilar em modo 64-bit; agora pressione _build solution_. Isso vai gerar uma biblioteca compilada com nome <code>glfw3.lib</code>, que pode ser encontrada no diretório <code>build/src/Debug</code>.

Quando gerarmos a biblioteca precisamos nos certificar que a IDE saiba onde encontrá-la, assim como os arquivos de _include_ para o programa OpenGL. 

* Nós localizamos os diretórios <code>/lib</code> and <code>/include</code> da IDE/compilador e adicionamos o conteúdo do diretório de <code>include</code> da lib GLFW ao diretório <code>/include</code> da IDE. Da mesma forma, adicionamos o arquivo <code>glfw3.lib</code> da biblioteca ao diretório <code>/lib</code> da IDE. Isto funciona, porém não é a abordagem recomendada. É difícil mantem o controle dos arquivos da sua biblioteca e também dos arquivos de cabeçalho, e uma nova instação da sua IDE ou compilador resultaria em ter que repetir todo este processo novamente.
* Outra abordagem (mais recomandada) é criar um conjunto de diretórios em um local de sua escolha que contenha todos os arquivos de cabeçalho e os binários das bibliotecas de terceiros para o qual você pode fazer referência pela IDE/compilador. Você pode, por exemplo, criar uma pasta única que contenha uma subpasta <code>Libs</code> e outra <code>Include</code> onde armazenamos todas as nossas bibliotecas e os arquivos de cabeçalho, respectivamente, para utilização nos projetos OpenGL. Agora, com todas as bibliotecas de terceiros organizadas em um único local (que pode ser compartilhado entre múltiplos computadores). O requisito é, no entanto, que cada vez que for criado um projeto novo, devemos indicar na IDE onde encontrar estes diretórios.

Depois que os arquivos necessários estão armazenados no local de sua escolha, podemos começar a criar o nosso primeiro projeto OpenGL GLFW.

## Nosso primeiro projeto

Primeiramente, vamos abrir o Visual Studio e criar um projeto novo. Escolha o C++ caso mais de uma opção seja apresentada e selecione _Empty Project_ (não esqueça de dar ao projeto um nome adequado). Considerando que nós faremos tudo no modo 64-bit e o padrão do projeto é 32-bit, precisaremos alterar o menu dropdown no topo da tela, próximo ao seletor Debug, de x86 para x64:

![Imagem mostrando como alterar de x86 para x64](/assets/images/x64.png)

Agora que isso está feito, temos um workspace para criar nossa primeira aplicação OpenGL!

## Linking

Para que o projeto possa utilizar a biblioteca GLFW, precisamos fazer o <def>link</def> dela com o nosso projeto. Isso pode ser feito especificando que queremos usar a <code>glfw3.lib</code> na seção de _linker settings_. Para isso precisamos adicionar este diretório ao projeto primeiramente.

Nós podemos dizer a IDE para levar este diretório em conta quando ele precisar procurar pelas bibliotecas ou arquivos de include. Clique com o botão direito no nome do projeto no _solution explorer_ e então vá até <code>VC++ Directories</code> como visto na imagem abaixo:

![Imagem da configuração de diretórios no Visual Studio VC++](/assets/images/vc_directories.png)

A partir deste ponto você pode adicionar os seus próprios diretórios para permitir que o projeto saiba onde procurar. Isso pode ser feito inserindo manualmente o diretório no campo de texto ou clicando na _location string_ apropriada e selecionando a opção <code>&lt;Edit..&gt;</code>. Faça isso para os diretórios da biblioteca <code>Library Directories</code> e os diretórios de include <code>Include Directories</code>:

![Imagem da configuração dos diretórios de include no Visual Studio](/assets/images/include_directories.png)

Aqui você pode adicionar quantos diretórios você quiser, e depois disso a IDE também vai procurar estes diretórios ao buscar os arquivos de bibliotecas e arquivos de include. Assim que a nossa pasta de <code>Include</code> da GLFW for incluída, você vai ser capaz de localizar todos os arquivos de cabeçalho fazendo include <code>&lt;GLFW/..&gt;</code>. O mesmo se aplica ao diretório das bibliotecas.

Agora que o VS é capaz de encontrar todos os arquivos necessários nós podemos _linkar_ a GLFW ao projeto através da tab <code>Linker</code> e <code>Input</code>.

![Imagem da configuração de link do Visual Studio](/assets/images/linker_input.png)

Para finalmente _linkar_ a biblioteca você precisa especificar o nome dela ao _linker_. Considerando que o nome da biblioteca é <code>glfw3.lib</code>, nós adicionamos isso ao campo <code>Additional Dependencies</code> (manualmente ou usando a opção <code>&lt;Edit..&gt;</code>) e a partir deste ponto a GLFW vai estar _linkada_ quando compilarmos. Além da GLFW, nós vamos precisar também fazer o _link_ com a biblioteca OpenGL, mas isso pode variar de acordo com o sistema operacional:

### Biblioteca OpenGL no Windows

Se você estiver no Windows, a biblioteca OpenGL <code>opengl32.lib</code> vem com o SDK da Microsoft, que é instalado por padrão quando você instala o Visual Studio. Já que este capítulo usa o compilador do VS e estamos no windows, então adicionamos o arquivo <code>opengl32.lib</code> no _linker settings_. Observe que o equivalente em 64-bit da biblioteca OpenGL é chamada opengl32.lib, da mesma forma que o equivalente em 32-bit, sendo uma nomeclatura muito ruim.

### Biblioteca OpenGL no Linux

Em sistemas Linux você deve _linkar_ a biblioteca <code>libGL.so</code> adicionando <code>-lGL</code> nas configurações do seu linker. Caso você não consiga localizar a biblioteca, provavelmente você precisa instalar o pacote Mesa ou os pacotes dev da NVidia ou AMD.

Agora que você adicionou as bibliotecas GLFW e do OpenGL as opções do linker, você pode incluir os arquivos de cabeçalho para GLFW da seguinte forma:

```cpp
#include <GLFW\glfw3.h>
```

<note>
Para usuários Linux compilando com o GCC, a seguinte linha de comando pode ajudar a compilar o projeto: <code>-lglfw3 -lGL -lX11 -lpthread -lXrandr -lXi -ldl</code>. Caso as bibliotecas corretas não sejam _linkadas_ serão gerados muitos erros por _undefined reference_.
</note>

Isto conclui o setup e configuração da biblioteca GLFW.

## GLAD

Nós ainda não chegamos lá, pois ainda existe algo que precisamos fazer. Devido ao fato do OpenGL ser apenas uma especificação, é de responsabilidade do fabricante do driver implementar a especificação em um driver que aquela placa de vídeo suporta. Já que existem muitas versões diferentes de driver para o OpenGL, o local da maioria das suas funções é desconhecida no momento da compilação e precisa ser consultada em tempo de execução. Sendo assim, é de resposabilidade do desenvolvedor obter o local das funções que ele/ela precisar, e armazena-las em ponteiros de função para uso posterior. A obtenção destes locais é diferente em cada [SO](https://www.khronos.org/opengl/wiki/Load_OpenGL_Functions). No Windows, se parece mais ou menos com isso:

```cpp
// define the function's prototype
typedef void (*GL_GENBUFFERS) (GLsizei, GLuint*);
// find the function and assign it to a function pointer
GL_GENBUFFERS glGenBuffers  = (GL_GENBUFFERS)wglGetProcAddress("glGenBuffers");
// function can now be called as normal
unsigned int buffer;
glGenBuffers(1, &buffer);
```

Como você pode ver, o código parece complexo, e é um processo trabalhoso de se realizar para cada função que você precisar e ainda não esteja declarada. Por sorte, existem bibliotecas desenvolvidas para este fim, sendo a **GLAD** uma biblioteca popular e atualizada.

### Configurando a GLAD

GLAD é uma biblioteca [open source](https://github.com/Dav1dde/glad) que gerencia todo o trabalho maçante que foi mencionado. A GLAD tem um processo de configuração um pouco diferente da maioria das bibliotecas open source. Ela utiliza um serviço web onde nós podemos dizer para qual versão do OpenGL nós queremos definir e carregar as funções.

Vá até o [serviço web](http://glad.dav1d.de/), confirme que a linguagem está definida para C++, e na seção de API selecione uma versão do OpenGL 3.3 ou maior (que será a versão que estaremos utilizando; versões maiores também funcionarão). Também garanta que o profile esteja definido em <code>Core</code> e que a opção <code>Generate a loader</code> esteja marcada. Ignore as extensões (por enquanto) e clique em <code>Generate</code> para gerar os arquivos da biblioteca.

Agora o GLAD deve ter disponibilizado um arquivo zip duas pastas de include e um único arquivo <code>glad.c</code>. Copie ambas as pastas de include (<code>glad</code> e <code>KHR</code>) no seu diretório de includes (ou adicione um apontamento a mais para estas pastas), e adicione o arquivo <code>glad.c</code> ao seu projeto.

Depois dos passos anteriores, você deve ser capaz de adicionar a seguinte diretiva de include no topo do seu arquivo:

```cpp
#include <glad/glad.h>
```

Pressione o botão _compile_ para compilar e você não deve receber nenhum erro. Neste ponto estamos prontos para ir ao próximo capítulo e abordar como podemos efetivamente usar as bibliotecas GLFW e a GLAD para configurar um contexto do OpenGL e lançar uma janela. Confirme que todos os seus diretórios de include e de biblioteca estão corretos e que os nomes das bibliotecas correpondentes estejam corretas nas configurações do _linker_.

## Recursos adicionais

* [GLFW: Window Guide](http://www.glfw.org/docs/latest/window_guide.html): Guia oficial da GLFW em como configurar uma janela
* [Building applications](http://www.opengl-tutorial.org/miscellaneous/building-your-own-c-application/): Ótimas informações a respeito do processo de compilação e linkagem de uma aplicação e uma grande lista de erros possíveis (com soluções).
* [GLFW with Code::Blocks](http://wiki.codeblocks.org/index.php?title=Using_GLFW_with_Code::Blocks): Compilando a GLFW na IDE Code::Blocks.
* [Running CMake](http://www.cmake.org/runningcmake/): Uma breve visão geral de como executar o CMake em ambos Windows e Linux.
* [Writing a build system under Linux](https://learnopengl.com/demo/autotools_tutorial.txt): Um tutorial por Wouter Verholst em como construir um sistema de build no Linux com autotools.
* [Polytonic/Glitter](https://github.com/Polytonic/Glitter): Um projeto boilerplate simples que vem com todas as bibliotecas relevantes pré-configuradas; ótimo se você quer um projeto de exemplo sem o incômodo de ter que compilar todas as bibliotecas por conta própria.
