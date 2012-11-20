Intro Rápida a Behat
====================

Bienvenido a Behat! Behat es una herramienta que hace posible `behavior driven development`_
(BDD). Con BDD, tu escribes historias en lenguaje natural que describen el comportamiento de 
tu aplicación. Estas historias pueden ser auto-probadas contra el sistema. Y si, es tan 
bueno como suena!

Por ejemplo, imagina que has sido contratado para construir el famoso comando UNIX ``ls``.

Un interesado podría decirte:

.. code-block:: gherkin

    Característica: ls
      Para ver la estructura de directorio
      Como un usuario UNIX
      Necesito ser capaz de listar el contenido del directorio actual

      Escenario: Listar 2 archivos en un directorio
        Dado que estoy en un directorio "test"
        Y tengo un archivo llamado "foo"
        Y tengo un archivo llamado "bar"
        Cuando ejecuto "ls"
        Entonces debería obtener:
          """
          bar
          foo
          """

En este tutorial, te mostraremos como Behat puede ejecutar esta simple historia
como una prueba que verifica que el comando ``ls`` trabaja como se describe.

Eso es todo! Behat puede ser usado para probar cualquier cosa, incluyendo comportamiento
relacionado a la web a través de la librería `Mink`_.

.. note::

    Si quieres aprender más acerca de la filosofía de probar el "comportamiento"
	de tu aplicación, lee `¿Qué hay en una historia?`_

.. note::

    Behat esta inspirado en el proyecto `Cucumber`_ de Ruby.

Instalación
-----------

Behat es un ejecutable que usarás en la línea de comando para ejecutar tus
historias como pruebas. Antes de comenzar, asegurate de que tienes al menos
PHP 5.3.1 instalado.

Metodo #1 (Composer)
~~~~~~~~~~~~~~~~~~~~

La manera más simple de instalar Behat es a través de Composer.

Crea un archivo ``composer.json`` en la raíz del proyecto:

.. code-block:: js

    {
        "require": {
            "behat/behat": "2.4.*@stable"
        },
        "minimum-stability": "dev",
        "config": {
            "bin-dir": "bin/"
        }
    }

Luego descarga ``composer.phar`` y ejecuta el comando ``install``:

.. code-block:: bash

    $ curl http://getcomposer.org/installer | php
    $ php composer.phar install

.. note::

    Composer utiliza el servicio de zipball de GitHub por defecto y este 
	servicio es conocido por sus fallas de vez en cuando. Si obtienes

    .. code-block:: bash

        The ... file could not be downloaded (HTTP/1.1 502 Bad Gateway)

    durante la instalación, simplemente usa la opción ``--prefer-source``:

    .. code-block:: bash

        $ php composer.phar install --prefer-source

Despues de eso, podrás ejecutar Behat con:

.. code-block:: bash

    $ bin/behat

Metodo #2 (PHAR)
~~~~~~~~~~~~~~~~

Tambien puedes usar el paquete phar de behat:

.. code-block:: bash

    $ wget https://github.com/downloads/Behat/Behat/behat.phar

Ahora puedes ejecutar Behat simplemente usando el archivo phar con ``php``:

.. code-block:: bash

    $ php behat.phar

Metodo #3 (Git)
~~~~~~~~~~~~~~~

Tambien puedes clonar el proyecto con Git ejecutando:

.. code-block:: bash

    $ git clone git://github.com/Behat/Behat.git && cd Behat
    $ git submodule update --init

Luego descarga ``composer.phar`` y ejecuta el comando ``install``:

.. code-block:: bash

    $ wget -nc http://getcomposer.org/composer.phar
    $ php composer.phar install

Luego de eso, podrás ejecutar Behat con:

.. code-block:: bash

    $ bin/behat

Uso Básico
----------

En este ejemplo, retrocederemos varias decadas y pretendamos que estamos 
construyendo el comando original de UNIX ``ls``. Crea un nuevo directorio
y configura Behat dentro de ese directorio:

.. code-block:: bash

    $ mkdir ls_project
    $ cd ls_project
    $ behat --init

El comando ``behat --init`` creará un directorio ``features/`` con algunas cosas
básicas para que comiences.

Define tu característica
~~~~~~~~~~~~~~~~~~~~~~~~

Todo en Behat siempre comienza con una *característica* que quieres describir
y luego implementar. En este ejemplo, la característica será el comando ``ls``,
que puede ser pensado como una característica del sistema UNIX completo. Como la
característica es el comando ``ls``, comienza por crear un archivo ``features/ls.feature``:

.. code-block:: gherkin

    # features/ls.feature
	#language es
    Característica: ls
      Para ver la estructura de directorio
      Como usuario UNIX
      Necesito ser capaz de listar el contenido del directorio actual

Cada característica comienza con este mismo formato: una línea nombrando la característica,
seguida de tres líneas que describen el beneficio, el rol y la característica misma. 
Y mientras esta sección es requerida, su contenido no es realmente importante para 
Behat o tu prueba eventual. Esta sección es importante, sin embargo, de manera que cada
característica sea descrita consistentemente y legible por otras personas. Es importante notar
que como estamos usando el lenguaje español, la primera linea debe incluir el comentario
``#language es``.

Define un Escenario
~~~~~~~~~~~~~~~~~~~

Luego, añade el siguiente escenario al final del archivo ``features/ls.feature``:

.. code-block:: gherkin

    Escenario: Listar 2 archivos en un directorio
      Dado que estoy en un directorio "test"
      Y tengo un archivo llamado "foo"
      Y tengo un archivo llamado "bar"
      Cuando ejecuto "ls"
      Entonces debería obtener:
        """
        bar
        foo
        """

.. tip::

    La sintáxis especial ``"""`` que se ve en las ultimas líneas es solo una 
	forma de definir pasos en multiples líneas. No te preocupes por ella por
	ahora.

Cada carcaterística es definida por uno o más "escenarios", los cuales explican como
esa característica debe actuar bajo diferentes circunstancias. Esta es la parte que va
a ser transformada en una prueba. Cada escenario sigue siempre el mismo formato basico:

.. code-block:: gherkin

    Escenario: Alguna descripción del escenario
      Dado [algo de contexto]
      Cuando [algún evento]
      Entonces [un resultado]

Cada parte del escenario - el *contexto*, el *evento*, y el *resultado* - puede ser
extendido añadiendo la palabra clave ``Y`` o ``Pero``:

.. code-block:: gherkin

    Escenario: Alguna descripción del escenario
          Dado [algo de contexto]
	         Y [más contexto]
        Cuando [algún evento]
		     Y [segundo evento ocurre]
      Entonces [un resultado]
	         Y [otro resultado]
		  Pero [otro resultado]

No existe diferencia entre ``Entonces``, ``Y``, ``Pero``, o alguna de las otras
palabras que comienzan cada linea. Estas palabras claves están disponibles 
para que tus escenarios sean naturales y legibles.

Executing Behat
~~~~~~~~~~~~~~~

You've now defined the feature and one scenario for that feature. You're
ready to see Behat in action! Try executing Behat from inside your ``ls_project``
directory:

.. code-block:: bash

    $ behat

If everything worked correctly, you should see something like this:

.. image:: /images/ls_no_defined_steps.png
   :align: center

Writing your Step definitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Behat automatically finds the ``features/ls.feature`` file and tries to execute
its ``Scenario`` as a test. However, we haven't told Behat what to do with
statements like ``Given I am in a directory "test"``, which causes an error.
Behat works by matching each statement of a ``Scenario`` to a list of regular
expression "steps" that you define. In other words, it's your job to tell
Behat what to do when it sees ``Given I am in a directory "test"``. Fortunately,
Behat helps you out by printing the regular expression that you probably
need in order to create that step definition:

.. code-block:: text

    You can implement step definitions for undefined steps with these snippets:

        /**
         * @Given /^I am in a directory "([^"]*)"$/
         */
        public function iAmInADirectory($argument1)
        {
            throw new PendingException();
        }

Let's use Behat's advice and add the following to the ``features/bootstrap/FeatureContext.php``
file, renaming ``$argument1`` to ``$dir``, simply for clarity:

.. code-block:: php

    # features/bootstrap/FeatureContext.php
    <?php

    use Behat\Behat\Context\BehatContext,
        Behat\Behat\Exception\PendingException;
    use Behat\Gherkin\Node\PyStringNode,
        Behat\Gherkin\Node\TableNode;

    class FeatureContext extends BehatContext
    {
        /**
         * @Given /^I am in a directory "([^"]*)"$/
         */
        public function iAmInADirectory($dir)
        {
            if (!file_exists($dir)) {
                mkdir($dir);
            }
            chdir($dir);
        }
    }

Basically, we've started with the regular expression suggested by Behat, which
makes the value inside the quotations (e.g. "test") available as the ``$dir``
variable. Inside the method, we simple create the directory and move into it.

Repeat this for the other three missing steps so that your ``FeatureContext.php``
file looks like this:

.. code-block:: php

    # features/bootstrap/FeatureContext.php
    <?php

    use Behat\Behat\Context\BehatContext,
        Behat\Behat\Exception\PendingException;
    use Behat\Gherkin\Node\PyStringNode,
        Behat\Gherkin\Node\TableNode;

    class FeatureContext extends BehatContext
    {
        private $output;

        /** @Given /^I am in a directory "([^"]*)"$/ */
        public function iAmInADirectory($dir)
        {
            if (!file_exists($dir)) {
                mkdir($dir);
            }
            chdir($dir);
        }

        /** @Given /^I have a file named "([^"]*)"$/ */
        public function iHaveAFileNamed($file)
        {
            touch($file);
        }

        /** @When /^I run "([^"]*)"$/ */
        public function iRun($command)
        {
            exec($command, $output);
            $this->output = trim(implode("\n", $output));
        }

        /** @Then /^I should get:$/ */
        public function iShouldGet(PyStringNode $string)
        {
            if ((string) $string !== $this->output) {
                throw new Exception(
                    "Actual output is:\n" . $this->output
                );
            }
        }
    }

.. note::

    When you specify multi-line step arguments - like we did using the triple
    quotation syntax (``"""``) in the above scenario, the value passed into
    the step function (e.g. ``$string``) is actually an object, which can
    be converted into a string using ``(string) $string`` or
    ``$string->getRaw()``.

Great! Now that you've defined all of your steps, run Behat again:

.. code-block:: bash

    $ behat

.. image:: /images/ls_passing_one_step.png
   :align: center

Success! Behat executed each of your steps - creating a new directory with
two files and running the ``ls`` command - and compared the result to the
expected result.

Of course, now that you've defined your basic steps, adding more scenarios
is easy. For example, add the following to your ``features/ls.feature`` file
so that you now have two scenarios defined:

.. code-block:: gherkin

    Scenario: List 2 files in a directory with the -a option
      Given I am in a directory "test"
      And I have a file named "foo"
      And I have a file named ".bar"
      When I run "ls -a"
      Then I should get:
        """
        .
        ..
        .bar
        foo
        """

Run Behat again. This time, it'll run two tests, and both will pass.

.. image:: /images/ls_passing_two_steps.png
   :align: center

That's it! Now that you've got a few steps defined, you can probably dream
up lots of different scenarios to write for the ``ls`` command. Of course,
this same basic idea could be used to test web applications, and Behat integrates
beautifully with a library called `Mink`_ to do just that.

Of course, there's still lot's more to learn, including more about the
:doc:`Gherkin syntax </guides/1.gherkin>` (the language used in the ``ls.feature``
file).

Some more Behat Basics
----------------------

When you run ``behat --init``, it sets up a directory that looks like this:

.. code-block:: bash

    |-- features
       `-- bootstrap
           `-- FeatureContext.php

Everything related to Behat will live inside the ``features`` directory, which
is composed of three basic areas:

1. ``features/`` - Behat looks for ``*.feature`` files here to execute

2. ``features/bootstrap/`` - Every ``*.php`` file in that directory will
   be autoloaded by Behat before any actual steps are executed

3. ``features/bootstrap/FeatureContext.php`` - This file is the context
   class in which every scenario step will be executed

More about Features
-------------------

As you've already seen, a feature is a simple, readable plain text file,
in a format called Gherkin. Each feature file follows a few basic rules:

1. Every ``*.feature`` file conventionally consists of a single "feature"
   (like the ``ls`` command or *user registration*).

2. A line starting with the keyword ``Feature:`` followed by its title and
   three indented lines defines the start of a new feature.

3. A feature usually contains a list of scenarios. You can write whatever
   you want up until the first scenario: this text will become the feature
   description.

4. Each scenario starts with the ``Scenario:`` keyword followed by a short
   description of the scenario. Under each scenario is a list of steps, which
   must start with one of the following keywords: ``Given``, ``When``, ``Then``,
   ``But`` or ``And``. Behat treats each of these keywords the same, but you
   should use them as intended for consistent scenarios.

.. tip::

    Behat also allows you to write your features in your native language.
    In other words, instead of writing ``Feature``, ``Scenario`` or ``Given``,
    you can use your native language by configuring Behat to use one of its
    many supported languages.

    To check if your language is supported and to see the available keywords,
    run:

    .. code-block:: bash

        $ behat --story-syntax --lang YOUR_LANG

    Supported languages include (but are not limited to) ``fr``, ``es``, ``it``
    and, of course, the english pirate dialect ``en-pirate``.

    Keep in mind, that any language, different from ``en`` should be explicitly
    marked with ``# language: ...`` comment at the beginning of your
    ``*.feature`` file:

    .. code-block:: gherkin

        # language: fr
        Fonctionnalité: ...
          ...

You can read more about features and Gherkin language in ":doc:`/guides/1.gherkin`"
guide.

More about Steps
----------------

For each step (e.g. ``Given I am in a directory "test"``), Behat will look
for a matching step definition by matching the text of the step against the
regular expressions defined by each step definition.

A step definition is written in php and consists of a keyword, a regular
expression, and a callback. For example:

.. code-block:: php

    /**
     * @Given /^I am in a directory "([^"]*)"$/
     */
    public function iAmInADirectory($dir)
    {
        if (!file_exists($dir)) {
            mkdir($dir);
        }
        chdir($dir);
    }

A few pointers:

1. ``@Given`` is a definition keyword. There are 3 supported keywords in
   annotations: ``@Given``/``@When``/``@Then``. These three definition keywords
   are actually equivalent, but all three are available so that your step
   definition remains readable.

2. The text after the keyword is the regular expression (e.g. ``/^I am in a directory "([^"]*)"$/``).

3. All search patterns in the regular expression (e.g. ``([^"]*)``) will become
   method arguments (``$dir``).

4. If, inside a step, you need to tell Behat that some sort of "failure" has
   occurred, you should throw an exception:

    .. code-block:: php

       /**
        * @Then /^I should get:$/
        */
       public function iShouldGet(PyStringNode $string)
       {
           if ((string) $string !== $this->output) {
               throw new Exception(
                   "Actual output is:\n" . $this->output
               );
           }
       }

.. tip::

    Behat doesn't come with its own assertion tool, but you can use any proper
    assertion tool out there. Proper assertion tool is a library, which
    assertions throw exceptions on fail. For example, if you're familiar with
    PHPUnit, you can use its assertions in Behat:

    .. code-block:: php

        # features/bootstrap/FeatureContext.php
        <?php

        use Behat\Behat\Context\BehatContext;
        use Behat\Gherkin\Node\PyStringNode;

        require_once 'PHPUnit/Autoload.php';
        require_once 'PHPUnit/Framework/Assert/Functions.php';

        class FeatureContext extends BehatContext
        {
            /**
             * @Then /^I should get:$/
             */
            public function iShouldGet(PyStringNode $string)
            {
                assertEquals($string->getRaw(), $this->output);
            }
        }

In the same way, any step that does *not* throw an exception will be seen
by Behat as "passing".

You can read more about step definitions in ":doc:`/guides/2.definitions`" guide.

The Context Class: ``FeatureContext``
-------------------------------------

Behat creates a context object for each scenario and executes all scenario
steps inside that same object. In other words, if you want to share variables
between steps, you can easily do that by setting property values on the context
object itself (which was shown in the previous example).

You can read more about ``FeatureContext`` in ":doc:`/guides/4.context`" guide.

The ``behat`` Command Line Tool
-------------------------------

Behat comes with a powerful console utility responsible for executing the
Behat tests. The utility comes with a wide array of options.

To see options and usage for the utility, run:

.. code-block:: bash

    $ behat -h

One of the handiest things it does it to show you all of the step definitions
that you have configured in your system. This is an easy way to recall exactly
how a step you defined earlier is worded:

.. code-block:: bash

    $ behat -dl

You can read more about Behat CLI in ":doc:`/guides/6.cli`" guide.

What's Next?
------------

Congratulations! You now know everything you need in order to get started
with behavior driven development and Behat. From here, you can learn more
about the :doc:`Gherkin</guides/1.gherkin>` syntax or learn how to test your
web applications by using Behat with Mink.

* :doc:`/cookbook/behat_and_mink`
* :doc:`/guides/1.gherkin`
* :doc:`/guides/6.cli`

.. _`behavior driven development`: http://en.wikipedia.org/wiki/Behavior_Driven_Development
.. _`Mink`: https://github.com/behat/mink
.. _`What's in a Story?`: http://blog.dannorth.net/whats-in-a-story/
.. _`Cucumber`: http://cukes.info/
.. _`Goutte`: https://github.com/fabpot/goutte
.. _`PHPUnit`: http://phpunit.de
.. _`Testing Web Applications with Mink`: https://github.com/behat/mink
