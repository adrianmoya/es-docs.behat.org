Introdução Rápida ao Behat
==========================

Behat é um framework `open source <http://creativecommons.org/licenses/MIT/>`_ para o `desenvolvimento guiado por comportamento <http://pt.wikipedia.org/wiki/Behavior_Driven_Development>`_ (Behavior Driven Development ou BDD) para o PHP 5.3.

Behat foi inspirado pelo projeto `Cucumber <http://cukes.info/>`_ do Ruby e especialmente a parte da sua sintaxe (Gherkin). Ele tenta ser como o Cucumber com entrada (arquivos de funcionalidade) e saída (formatadores de console), mas no núcleo, ele é construído do chão nos ombros de gigantes:

* `Componente Symfony Dependency Injection Container <https://github.com/symfony/DependencyInjection>`_
* `Componente Symfony Event Dispatcher <https://github.com/symfony/EventDispatcher>`_
* `Componente Symfony Console <https://github.com/symfony/Console>`_
* `Componente Symfony Finder <https://github.com/symfony/Finder>`_
* `Componente Symfony Translation <https://github.com/symfony/Translation>`_

Diferente de qualquer outro framework de teste do PHP que testa a aplicação de dentro para fora, o Behat está testando aplicações `de fora para dentro <http://blog.dannorth.net/whats-in-a-story/>`_. Isso significa que o Behat funciona apenas com as entradas/saídas da sua aplicação. Se você quiser testar os seus models, use um framework de teste unitário. Behat foi criado para teste de comportamento (mas pode ser usado qualquer coisa +).

Instalação

Exitem muitas maneiras de instalar o Behat no seu sistema, mas a única exigência é que você deve ter instalado o PHP 5.3.1+. Se você tem - continue lendo, senão - instale ele primeiro.

A maneira mais simples de instalar o Behat é através do PEAR:

.. code-block:: bash

    pear channel-discover pear.behat.org
    pear install behat/gherkin
    pear install behat/behat

Você também pode fazer um clone do projeto com o Git executando:

.. code-block:: bash

    git clone git://github.com/Behat/Behat.git && cd Behat
    git submodule update --init

Após o download, basta executar ``caminho/para/o/Behat/bin/behat`` ou simplesmente ``behat`` (se você instalou o Behat através do PEAR) a partir do console.

Fundamentos
-----------

Behat é uma ferramenta que pode executar descrições funcionais em texto puro como testes automatizados. O idioma que o Behat (e o Ruby Cucumber) entende é chamada de Gherkin. Aqui está um exemplo:

.. code-block:: gherkin

    Feature: Pesquisar cursos
      Afim de garantir o melhor aproveitamento dos cursos
      Potenciais alunos devem ser capazes de pesquisar cursos

      Scenario: Pesquisar por tema
        Given que existem 240 cursos que não têm o tema "biologia"
        And existem 2 cursos A001, B205 que têm "biologia" como um dos seus temas
        When eu pesquisar por "biologia"
        Then eu devo ver os seguintes cursos:
          | Código do Curso |
          | A001            |
          | B205            |

Enquanto que o Behat pode ser pensado como uma ferramenta de “teste”, a intenção da ferramenta é suportar BDD. Isso significa que "funcionalidades" (descrições em texto puro com cenários) são tipicamente escritas antes de qualquer coisa e verificada por analistas de negócios, especialistas de domínio, etc. outras partes interessadas que não têm conhecimentos técnicos. O código de produção é escrito de fora para dentro, para fazer com que as estórias passem.

O diretório básico do ambiente de teste do Behat se parece com este:

.. code-block:: bash

    |-- features
       `-- steps
       |   `-- math_steps.php
       `-- support
           |-- bootstrap.php
           |-- env.php

E pode ser dividido em 3 áreas principais:

1. ``features`` - diretório raiz de teste. Arquivos de funcionalidade são colocados aqui.
2. ``features/steps`` - diretório da definição das etapas. Definição das etapas (scripts PHP) permanecem aqui.
3. ``features/support`` - diretório de suporte. Scripts de suporte (ambiente, definição de path) são criados aqui.

O núcleo dos testes no Behat são funcionalidades. Funcionalidades são colocadas no diretório ``features`` e são arquivos de texto puro.

O Behat analisa arquivos de funcionalidade e tenta encontrar uma definição de etapa apropriada para cada etapa da pasta ``features/steps`` (path para a definição das etapas).

Cada definição de etapa é um simples PHP executável com acesso a um objeto de ambiente compartilhado. Este objeto é compartilhado entre lotes de etapas (cenários), e pode ser configurado dentro do arquivo ``features/support/env.php``.

E se o ambiente de configuração necessita de algumas bibliotecas para funcionar (como o PHPUnit por exemplo), as inclusões são colocadas dentro de ``features/support/bootstrap.php``.

Funcionalidade
--------------

O arquivo de funcionalidade é o seu ponto de entrada no Behat. É aí que você começa a trabalhar no seu projeto. Aqui está o conteúdo de uma funcionalidade básica, ``features/math.feature``:

.. code-block:: gherkin

    Feature: Adição
      Afim de evitar erros bobos
      como um idiota em matemática
      eu quero ser informado sobre a soma de dois números

      Scenario: Adicionar dois números
        Given Eu digitei 50 na calculadora
          And eu digitei 70 na calculadora
         When eu pressionar o botão de adicionar
         Then o resultado deve ser 120 na tela

Como você pode ver, uma funcionalidade é um arquivo de texto puro, simples e legível. Cada funcionalidade é escrita em uma `DSL <http://pt.wikipedia.org/wiki/Linguagem_de_dom%C3%ADnio_espec%C3%ADfico>`_ chamada **Gherkin**, que foi introduzida pela primeira vez no `Cucumber <http://cukes.info/>`_ do Ruby.

1. Cada arquivo ``*.feature`` convencionalmente consiste de uma única funcionalidade.
2. Uma linha começando com a palavra-chave ``Feature:`` (ou uma traduzida) seguida por texto livremente endentado começa uma funcionalidade.
3. Uma funcionalidade geralmente contém uma lista de cenários. Você pode escrever o que quiser até o primeiro cenário e este texto será uma descrição de funcionalidade.
4. Todo cenário começa apartir da palavra-chave ``Scenario:`` ou ``Scenario Outline:`` (ou equivalente traduzida). Cada cenário é composto por etapas, que devem começar com uma das palavras-chave ``Given``, ``When``, ``Then``, ``But`` ou ``And`` (or a localized one). O Behat trata todos estes tipos de etapas como a mesma coisa, mas você não deve fazer isso!

Definição de Etapa
------------------

Para cada etapa o Behat procurará por uma definição de etapa correspondente. Uma definição de etapa é escrita em PHP. Cada definição de etapa consiste de uma palavra-chave, uma expressão regular, e um callback. Exemplo ``features/steps/math.php``:

.. code-block:: php

    <?php 

    $steps->Given('/^Eu digitei (\d+) na calculadora$/', function($world, $arg1) { 
        throw new Behat\Behat\Exception\Pending('Write code later'); 
    });

1. ``$steps`` é um objeto DefinitionDispatcher global, disponível em todos os arquivos de definição de etapas. Chamando ``->Given`` nele definirá uma nova etapa ``Given`` (mas este também irá coincidir com etapas nomeadas com as palavras-chave ``When``/``Then``/``And``).
2. ``'/^Eu digitei (\d+) na calculadora$/'`` - esta é uma expressão regular para a etapa. Todos os padrões de pesquisa (``(\d+)``) se tornarão argumentos do callback (``$arg1``).
3. O primeiro argumento do callback (``$world``) é sempre reservado para o objeto de ambiente. O objeto de ambiente é criado antes que cada cenário seja executado e é compartilhado entre as etapas do cenário.
4. A definição do corpo da etapa é código PHP simples. Uma etapa que falha (**failed**) é uma etapa cuja execução lança uma exceção. Então, se a etapa não lança nenhuma exceção, a etapa passa (**passes**).

Ambiente
--------

Behat cria um objeto de ambiente para cada cenário e passa uma referência para ele em cada definição de etapa (``$world`` no exemplo acima).

Então, se você quer calcular/acumular ou apenas compartilhar variáveis entre a definição das etapas, use o contêiner ``$world`` para isto.

Mas se você precisar que algumas definições estejam disponíveis em cada mundo (world)? Use o configurador de ambiente ao invés disso:

.. code-block:: php

    <?php
    // features/support/env.php

    require 'paths.php';

    // Cria o comportamento do WebClient
    $world->client = new \Goutte\Client;
    $world->response = null;
    $world->form = array();

    // Closures úteis
    $world->visit = function($link) use($world) {
        $world->response = $world->client->request('GET', $link); 
    };

Este arquivo será executado em cada criação do objeto de ambiente. A própria variável ``$world`` é um objeto de ambiente, que funciona como uma variável de suporte para todos os valores do seu cenário e parâmetros.

Mas se nós precisarmos usar alguma biblioteca de terceiros no ``env.php``? É ineficiente requisitar ela antes de cada cenário, dessa forma o Behat suporta script de bootstrapping:

.. code-block:: php

    <?php
    // features/support/bootstrap.php

    require_once 'PHPUnit/Autoload.php';
    require_once 'PHPUnit/Framework/Assert/Functions.php';

Este arquivo será avaliado pelo Behat antes que os testes de funcionalidade tenham sido executados ;-)

CLI
---

Behat vem com um poderoso console, chamado ... behat.

Para ver a versão atual do Behat, execute:

.. code-block:: bash

    behat -V

Para ver outros comandos disponíveis, use:

.. code-block:: bash

    behat -h

Agora você sabe tudo que você precisa para começar com o Behat. Você pode começar a usar BDD nos seus projetos imediatamente ou continuar lendo o guia completo.
