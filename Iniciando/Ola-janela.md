---
layout: page
title: Olá Janela
nav_order: 5
parent: Iniciando
---

Vamos ver se conseguimos colocar o GLFW em funcionamento. Primeiro, crie um arquivo <code>.cpp</code> e adicione as os seguintes includes na parte superior do arquivo recém-criado.

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
```

<warning>
Certifique-se de incluir o GLAD antes do GLFW. O arquivo de include do GLAD inclui os cabeçalhos OpenGL necessários por traz dos panos (como GL / gl.h), portanto, inclua o GLAD antes de outros arquivos de cabeçalho que exijam o OpenGL (como a GLFW).
</warning>

Em seguida, criamos a função <fun>main</fun> na qual iremos instanciar a janela GLFW:

```cpp
int main()
{
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    //glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
  
    return 0;
}
```

Na função main, primeiro inicializamos a GLFW com <fun>glfwInit</fun>, em seguida podemos configurar a GLFW usando <fun>glfwWindowHint</fun>. O primeiro argumento da <fun>glfwWindowHint</fun> indica qual opção queremos configurar, onde podemos selecionar a opção em um grande enum de opções possíveis prefixadas com <code>GLFW_</code>. O segundo argumento é um número inteiro que define o valor da nossa opção. Uma lista de todas as opções possíveis e seus valores correspondentes pode ser encontrada na documentação de [manipulação de janelas do GLFW](http://www.glfw.org/docs/latest/window.html#window_hints). Se você tentar executar a aplicação agora e houver muitos erros de _undefined reference_, significa que você não linkou corretamente a biblioteca GLFW.

Como o foco deste livro está no OpenGL versão 3.3, devemos indicar ao GLFW que 3.3 é a versão do OpenGL que queremos usar. Dessa forma, o GLFW pode tomar as providências adequadas ao criar o contexto OpenGL. Isso garante que, quando um usuário não tenha a versão correta do OpenGL, o GLFW não será executado. Definimos a versão principal e secundária como 3. Também dizemos ao GLFW que queremos usar explicitamente o core-profile. Dizer ao GLFW que queremos usar o core-profile significa que teremos acesso a um subconjunto menor de recursos do OpenGL, sem os recursos compatíveis com versões anteriores que não precisamos mais. Observe que no Mac OS X você precisa adicionar a chamada para <code>glfwWindowHint (GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);</code> ao seu código de inicialização para que ele funcione.

<note>
Confirme que o OpenGL versão 3.3 ou superior está instalado no seu sistema / hardware, caso contrário, a aplicação falhará ou exibirá um comportamento indefinido. Para encontrar a versão do OpenGL em sua máquina, faça uma chamda para <code>glxinfo</code> em máquinas Linux, ou use uma ferramenta como o [OpenGL Extension Viewer](http://download.cnet.com/OpenGL-Extensions-Viewer/3000-18487_4-34442.html) para Windows. Se a sua versão suportada for inferior, tente verificar se a sua placa de vídeo suporta o OpenGL 3.3+ (caso não suporte, realmente é muito antiga) e / ou atualize seus drivers.
</note>

Em seguida, é necessário criar um objeto _window_. Este objeto contém todos os dados referentes a janela, e é exigido pela maioria das outras funções da GLFW.

```cpp
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
if (window == NULL)
{
    std::cout << "Failed to create GLFW window" << std::endl;
    glfwTerminate();
    return -1;
}
glfwMakeContextCurrent(window);
```

A função <fun>glfwCreateWindow</fun> requer a largura e a altura da janela como seus dois primeiros argumentos, respectivamente. O terceiro argumento nos permite criar um nome para a janela; por enquanto, chamamos de <code>"LearnOpenGL"</code>, mas você pode nomeá-lo como quiser. Podemos ignorar os 2 últimos parâmetros. A função retorna um objeto <fun>GLFWwindow</fun> que precisaremos posteriormente para outras operações da GLFW. Depois disso, dizemos ao GLFW para tornar o contexto da nossa janela o contexto principal na thread atual.

## GLAD

No capítulo anterior, mencionamos que a GLAD gerencia ponteiros de função para o OpenGL, portanto, devemos inicializar o GLAD antes de chamar qualquer função do OpenGL:

```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
    std::cout << "Failed to initialize GLAD" << std::endl;
    return -1;
}
```

Passamos ao GLAD, uma função para carregar o endereço dos ponteiros de função do OpenGL, que são específicos do SO. A GLFW nos fornece a <fun>glfwGetProcAddress</fun> que define a função correta com base no sistema operacional para o qual estamos compilando.

## Viewport

Antes de começarmos a renderizar, precisamos fazer uma última coisa. Temos que dizer ao OpenGL o tamanho da janela de renderização para que o OpenGL saiba como queremos exibir os dados e coordenadas em relação à janela. Podemos definir essas dimensões através da função <fun>glViewport</fun>:

```cpp
glViewport(0, 0, 800, 600);
```

Os dois primeiros parâmetros da <fun>glViewport</fun> definem a localização do canto inferior esquerdo da janela. O terceiro e o quarto parâmetros definem a largura e a altura da janela de renderização em pixels, que são iguais ao tamanho da janela da GLFW.

Poderíamos realmente definir as dimensões da viewport em valores menores que as dimensões do GLFW; então toda a renderização do OpenGL seria exibida em uma janela menor e, por exemplo, poderíamos exibir outros elementos fora da janela de exibição do OpenGL.

<note>
Nos bastidores, o OpenGL usa os dados especificados via <fun>glViewport</fun> para transformar as coordenadas 2D processadas em coordenadas na tela. Por exemplo, um ponto de localização processado (<code>-0,5,0,5</code>) seria (como sua transformação final) mapeado para (<code>200,450</code>) nas coordenadas da tela. Observe que as coordenadas processadas no OpenGL estão entre -1 e 1, portanto, mapeamos efetivamente do intervalo (-1 a 1) a (0, 800) e (0, 600).
</note>

No entanto, no momento em que um usuário redimensiona a janela, a janela de visualização também deve ser ajustada. Podemos registrar uma função de callback na janela, que é chamada cada vez que a janela é redimensionada. A função de callback de redimensionamento possui o seguinte protótipo:

```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height);  
```

A função framebuffer size usa uma janela GLFW como seu primeiro argumento e dois números inteiros indicando as novas dimensões da janela. Sempre que a janela muda de tamanho, a GLFW chama essa função e preenche os argumentos adequados para você processar.

```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    glViewport(0, 0, width, height);
}
```

Temos que dizer ao GLFW que queremos chamar essa função em cada redimensionamento de janela registrando-a:

```cpp
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);  
```

Quando a janela é exibida pela primeira vez, a função <fun>framebuffer_size_callback</fun> também é chamada com as dimensões resultantes da janela. Para displays retina, a largura e a altura acabam significativamente acima dos valores de entrada originais.

Existem muitas funções de callback que podemos que podem ser registradas. Por exemplo, podemos criar uma função de callback para processar alterações na entrada do joystick, processar mensagens de erro etc. Registramos as funções callback após criar a janela e antes do início do loop de renderização.

# Prepare os motores

Não queremos que a aplicação desenhe uma única imagem, finalize e imediatamente feche a janela. Queremos que a aplicação continue desenhando imagens e manipulando a entrada do usuário até que o programa tenha sido explicitamente instruído a parar. Por esse motivo, precisamos criar um loop while, que agora chamamos de <def>render loop</def>, que continua em execução até que seja solicitado a GLFW para parar. O código a seguir mostra um loop de renderização muito simples:

```cpp
while(!glfwWindowShouldClose(window))
{
    glfwSwapBuffers(window);
    glfwPollEvents();    
}
```

A função <fun>glfwWindowShouldClose</fun> verifica no início de cada iteração do loop se a GLFW foi instruida a fechar. Nesse caso, a função retorna true e o loop de renderização para de ser executado, após o qual podemos fechar a aplicação.
A função <fun>glfwPollEvents</fun> verifica se algum evento foi acionado (como entradas de teclado ou movimentos do mouse), atualiza o estado da janela e chama as funções correspondentes (que podemos registrar por meio de funções de callback). A função <fun>glfwSwapBuffers</fun> trocará o color buffer (um grande buffer 2D que contém valores de cores para cada pixel na janela da GLFW) usado para renderizar durante essa iteração de renderização e mostrá-lo como saída na tela.

<note>
<strong>Buffer duplo</strong>
<br/>
Quando uma aplicação desenha em um único buffer, a imagem resultante pode exibir problemas de oscilação. Isso ocorre porque a imagem de saída resultante não é desenhada em um único instante, mas desenhada pixel por pixel e geralmente da esquerda para a direita e de cima para baixo. Como esta imagem não é exibida instantaneamente para o usuário enquanto ainda está sendo renderizada, o resultado pode conter artefatos. Para contornar esses problemas, os gerenciadores de janela aplicam um buffer duplo para renderização. O <strong>front</strong> buffer contém a imagem de saída final que é mostrada na tela, enquanto todos os comandos de renderização são executados no <strong>back</strong> buffer. Assim que todos os comandos de renderização forem concluídos, fazemos o <strong>swap</strong> do back buffer pelo front buffer, para que a imagem possa ser exibida sem que ainda esteja sendo renderizada, removendo todos os artefatos mencionados acima.
</note>

## Uma última coisa

Assim que sairmos do loop de renderização, gostaríamos de limpar / excluir adequadamente todos os recursos da GLFW que foram alocados. Podemos fazer isso através da função <fun>glfwTerminate</fun> que chamamos no final da função <fun>main</fun>.

```cpp
glfwTerminate();
return 0;
```

Isso limpará todos os recursos e encerrará a aplicação corretamente. Agora tente compilar seu projeto e, se tudo correu bem, você verá a seguinte saída:

![Imagem da janela GLFW de saída com o exemplo mais básico](/assets/images/hellowindow.png)

Se é uma imagem preta muito sem graça, você fez as coisas direito! Se você não obteve a imagem certa ou está confuso sobre como tudo se encaixa, verifique o código fonte completo [aqui](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/1.1.hello_window/hello_window.cpp).

Se você tiver problemas para compilar a aplicação, primeiro verifique se todas as suas opções do linker estão definidas corretamente e se você incluiu corretamente os diretórios corretos no seu IDE (conforme explicado no capítulo anterior). Verifique também se o seu código está correto; você pode verificá-lo comparando-o com o código fonte completo.

## Entradas

Também queremos ter alguma forma de controle de entrada no GLFW e podemos conseguir isso com várias funções de input da GLFW. Usaremos a função <def>glfwGetKey</def> da GLFW que recebe a janela como entrada junto com uma tecla. A função retorna se esta tecla está sendo pressionada no momento. Estamos criando uma função <def>processInput</def> para manter todo o código de input organizado:

```cpp
void processInput(GLFWwindow *window)
{
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}
```

Aqui, verificamos se o usuário pressionou a tecla Escape (se não estiver pressionada, <fun>glfwGetKey</fun> retorna <var>GLFW_RELEASE</var>). Se o usuário pressionou a tecla Escape, fechamos a GLFW configurando sua propriedade <var>WindowShouldClose</var> como <code>true</code> usando a função <fun>glfwSetwindowShouldClose</fun>. O próximo condition check do loop while principal falhará e a aplicação será fechada.

Em seguida, chamamos <fun>processInput</fun> a cada iteração do loop de renderização:

```cpp
while (!glfwWindowShouldClose(window))
{
    processInput(window);

    glfwSwapBuffers(window);
    glfwPollEvents();
}
```

Isso nos fornece uma maneira fácil de verificar pressionamentos de teclas específicos e reagir de acordo em cada <def>quadro</def>. Uma iteração do render loop é mais comumente chamada de <def>frame</def>.

## Renderizando

Queremos colocar todos os comandos de renderização no render loop, pois queremos executar todos os comandos de renderização em cada iteração ou quadro do loop. Isso ficaria mais ou menos assim:

```cpp
// render loop
while(!glfwWindowShouldClose(window))
{
    // input
    processInput(window);

    // rendering commands here
    ...

    // check and call events and swap the buffers
    glfwPollEvents();
    glfwSwapBuffers(window);
}
```

Apenas para testar se as coisas realmente funcionam, queremos limpar a tela com uma cor de nossa escolha. No início do quadro, queremos limpar a tela. Caso contrário, ainda veríamos os resultados do quadro anterior (este pode ser o efeito que você está procurando, mas geralmente não). Podemos limpar o color buffer da tela usando <def>glClear</def>, onde passamos uma mascara de bits do buffer para especificar qual buffer queremos limpar. Os possíveis bits que podemos definir são <var>GL_COLOR_BUFFER_BIT</var>, <var>GL_DEPTH_BUFFER_BIT</var> e <var>GL_STENCIL_BUFFER_BIT</var>. No momento, nos preocupamos apenas com os valores das cores, portanto limpamos apenas o buffer de cores.

```cpp
glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT);
```

Observe que também especificamos a cor para limpar a tela usando a função <fun>glClearColor</fun>. Sempre que chamamos a função <def>glClear</fun> e limpamos o buffer de cores, todo o buffer de cores será preenchido com a cor configurada por <fun>glClearColor</fun>. Isso resultará em uma cor verde-azulada escura.

<note>
Como você deve se lembrar do capítulo OpenGL, a função <fun>glClearColor</fun> é uma função de configuração de estado, e a <fun>glClear</fun> é uma função que usa estado, pois usa o estado atual para recuperar a cor para o clear.
</note>

![Imagem da criação de uma janela GLFW com a função glClear definida](/assets/images/hellowindow2.png)

O código fonte completo da aplicação pode ser encontrado [aqui](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/1.2.hello_window_clear/hello_window_clear.cpp).

Então, agora temos tudo pronto para preencher o render loop com muitas chamadas de renderização, mas isso é para o próximo capítulo. Acho que já andamos o bastante por aqui.
