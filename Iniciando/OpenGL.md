___

OpenGL
---

Antes de iniciar a nossa jornada devemos primeiramente definir o que exatamente é o OpenGL. O OpenGL é considerado principalmente uma API (<def>Application Programming Interface</def>) que nos prove uma grande quantidade de funções que nós podemos usar para manipular gráficos e imagens. Contudo, o OpenGL, por si próprio não é uma API, mas meramente uma especificação, desenvolvida e mantida pelo [Grupo Khronos](http://www.khronos.org/).

![Imagem da logo do OpenGL](/assets/images/opengl.jpg)

A especificação do OpenGL define o que exatamente deve ser a saída/resultado de cada função e como ela deve executar. Fica a cargo dos desenvolvedores que estão implementado a especificação fornecer uma solução para como essa função deve operar. Uma vez que a especificação do OpenGL não provê os detalhes de implementação, as versões efetivamente implementadas são autorizadas a terem diferentes implementações, contanto que os seus resultados estejam de acordo com a especificação (e apresentam portanto o mesmo para o usuário).

As pessoas realmente desenvolvendo bibliotecas OpenGL são os fabricantes de placas de vídeo. Cada placa de vídeo que você compra suporta versões específicas do OpenGL, que são versões especialmente desenvolvidas para aquela série de placas. Ao usar um sistema da _Apple_, a biblioteca OpenGL é mantida pela própria _Apple_ e no Linux existe uma combinação de fornecedores de versões de placa e adaptações de _hobistas_ destas bibliotecas. Isso também significa que quando o OpenGL apresenta um comportamento estranho que não deveria, provavelmente é culpa dos fabricantes de placas (ou quem quer que tenha desenvolvido a biblioteca).

<note>
Uma vez que a maioria das implementações são criadas pelos fabricantes de placas. Quando existe um bug na implementação, ele é geralmente resolvido ao atualizar o driver da interface de vídeo; estes drivers incluem a versão mais recente do OpenGL que a sua placa suporta. Esta é uma das razões porque é recomendado atualizar regularmente seus drivers de vídeo.
</note>

O grupo Khronos hospeda publicamente todas as especificações para as versões do OpenGL. O leitor interessado, pode encontrar a especificação da versão 3.3 do OpenGL (que é a que irem utilizar) [aqui](https://www.opengl.org/registry/doc/glspec33.core.20100311.withchanges.pdf), que é uma boa leitura se você quer mergulhar nos detalhes do OpenGL (observe como eles descrevem apenas os resultados e não implementações). A especificação também prove uma excelente referência para descobrir o funcionamento exato das suas funções.

## Core-profile vs Immediate mode

No passado, usar o OpenGL significava desenvolver no <def>immediate mode</def> (frequentemente referido como o <def>pipeline de função fixa</def>), que era um método de fácil utilização para desenhar gráficos. A maior parte da funcionalidade do OpenGL estava oculta dentro da biblioteca e os desenvolvedores não tinham muito controle sobre como o OpenGL fazia os cálculos. Os desenvolvedores eventualmente ficaram afoitos por mais flexibilidade, e ao longo do tempo as especificações ficaram mais flexíveis como resultado; os desenvolvedores ganharam mais controle sobre seus gráficos. O immediate mode é realmente fácil de usar e entender, mas ele também é extremamente ineficiente. Por esta razão, a especificação começou a obsoletar as funcionalidades do immediate mode a partir da versão 3.2 e começou a engajar os desenvolvedores a desenvolver no modo <def>core-profile</def> do OpenGL, que é uma divisão da especificação do OpenGL que removeu todas as funcionalidades obsoletas antigas.


When using OpenGL's core-profile, OpenGL forces us to use modern practices. Whenever we try to use one of OpenGL's deprecated functions, OpenGL raises an error and stops drawing. The advantage of learning the modern approach is that it is very flexible and efficient. However, it's also more difficult to learn. The immediate mode abstracted quite a lot from the actual operations OpenGL performed and while it was easy to learn, it was hard to grasp how OpenGL actually operates. The modern approach requires the developer to truly understand OpenGL and graphics programming and while it is a bit difficult, it allows for much more flexibility, more efficiency and most importantly: a much better understanding of graphics programming.

This is also the reason why this book is geared at core-profile OpenGL version 3.3. Although it is more difficult, it is greatly worth the effort.

As of today, higher versions of OpenGL are available to choose from (at the time of writing 4.5) at which you might ask: why do I want to learn OpenGL 3.3 when OpenGL 4.5 is out? The answer to that question is relatively simple. All future versions of OpenGL starting from 3.3 add extra useful features to OpenGL without changing OpenGL's core mechanics; the newer versions just introduce slightly more efficient or more useful ways to accomplish the same tasks. The result is that all concepts and techniques remain the same over the modern OpenGL versions so it is perfectly valid to learn OpenGL 3.3. Whenever you're ready and/or more experienced you can easily use specific functionality from more recent OpenGL versions.

<warning>
When using functionality from the most recent version of OpenGL, only the most modern graphics cards will be able to run your application. This is often why most developers generally target lower versions of OpenGL and optionally enable higher version functionality.
In some chapters you'll find more modern features which are noted down as such.
</warning>
