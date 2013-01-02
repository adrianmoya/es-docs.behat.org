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

Ejecutando Behat
~~~~~~~~~~~~~~~~

Ya has definido la característica y un escenario para esa característica. Estas
listo para ver a Behat en acción! Trata de ejecutar Behat desde el directorio del
``proyecto_ls``:

.. code-block:: bash

    $ behat

Si todo funcionó correctamente, debes ver algo como esto:

.. image:: /images/ls_no_defined_steps.png
   :align: center

Escribiendo la definición de tus Pasos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Behat automáticamente encuentra el archivo ``features/ls.feature`` e intenta ejecutar
el ``Escenario`` como una prueba. Sin embargo, no le hemos dicho a Behat que hacer con
instrucciones como ``Dado que estoy en un directorio "test"``, por lo tanto se produce
un error. Behat trabaja haciendo coincidir cada instrucción de un ``Escenario`` con una lista de
"pasos" (expresiones regulares) que tu defines. En otras palabras, es tu trabajo decirle
a Behat que hacer cuando ve ``Dado que estoy en un directorio "test"``. Afortunadamente,
Behat te ayuda imprimiendo la expresión regular que probablemente necesitas de manera de
crear la definición de ese paso:

.. code-block:: text

    You can implement step definitions for undefined steps with these snippets:

        /**
         * @Given /^I am in a directory "([^"]*)"$/
         */
        public function iAmInADirectory($argument1)
        {
            throw new PendingException();
        }

Usemos la recomendación de Behat y vamos a añadir lo siguiente al archivo ``features/bootstrap/FeatureContext.php``,
renombrando ``$argument1`` a ``$dir``, simplemente por claridad:

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

Basicamente, hemos comenzado con la expresión regular sugerida por Behat, que 
hace que el valor dentro de las comillas (por ejemplo "test") esté disponible
como la variable ``$dir``. Dentro del método, simplemente creamos el directorio
y entramos al mismo.

Repite esto para los otros tres pasos que faltan para que tu archivo ``FeatureContext.php``
se vea como sigue:

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

    Cuando especificas argumentos multi-linea - como hicimos utilizando la sintáxis
	de triple comilla (``"""``) en el escenario de arriba, el valor pasado a la
	función del paso (ejemplo ``$string``) es un objeto, que puede ser convertido
	en un string usando ``(string) $string`` o ``$string->getRaw()``.

Grandioso! Ahora que has definido todos tus pasos, ejecuta Behat nuevamente:

.. code-block:: bash

    $ behat

.. image:: /images/ls_passing_one_step.png
   :align: center

Éxito! Behat ejecuto cada uno de tus pasos - creando un nuevo directorio con dos
archivos y ejecutando el comando ``ls`` - y comparó el resultado al resultado esperado.

Claro, ahora que has definido tus pasos básicos, añadir más escenarios es fácil. 
Por ejemplo, añade lo siguiente a tu archivo ``features/ls.feature`` para tener 
dos escenarios definidos:

.. code-block:: gherkin

    Escenario: Listar 2 archivos en un directorio con la opción -a
	  Dado que estoy en un directorio "test"
	  Y tengo un archivo llamado "foo"
      Y tengo un archivo llamado ".bar"
      Cuando ejecuto "ls -a"
      Entonces debería obtener:
        """
		.
		..
        .bar
        foo
        """

Ejecuta Behat nuevamente. Esta vez, ejecutará dos pruebas, y ambas van a pasar.

.. image:: /images/ls_passing_two_steps.png
   :align: center

Eso es todo! Ahora que tienes algunos pasos definidos, puedes soñar 
un montón de escenarios diferentes que escribir para el comando ``ls``. Claro, 
esta misma idea básica puede ser usada para probar aplicaciones web, y Behat se integra
perfectamente con una librería llamada `Mink`_ justamente para hacer eso.

Claro, aun hay mucho por aprender, incluyendo más acerca de la :doc:`sintáxis Gherkin </guides/1.gherkin>`
(el lenguaje utilizado en el archivo ``ls.feature``).


Algunas cosas básicas adicionales de Behat
------------------------------------------

Cuando ejecutas ``behat --init``, Behat configura un directorio que se ve como esto:

.. code-block:: bash

    |-- features
       `-- bootstrap
           `-- FeatureContext.php

Todo lo relacionado con Behat vivirá dentro del directorio ``features``, que esta compuesto
de tres areas básicas:

1. ``features/`` - Behat busca archivos ``*.feature`` aqui para ejecutar

2. ``features/bootstrap/`` - Todos los archivos ``*.php`` en este directorio serán
   autocargados por Behat antes de ejecutar los pasos

3. ``features/bootstrap/FeatureContext.php`` - Este archivo es la clase de contexto
   en la cual cada paso de los escenarios será ejecutada

Más acerca de las Características
---------------------------------

Como ya has visto, una característica es un archivo de texto simple y legible, 
en un formato llamado Gherkin. Cada archivo de característica sigue algunas reglas básicas:

1. Todos los archivos ``*.feature`` consisten convencionalmente de una sola "característica"
   (como el comando ``ls`` o *registro de usuario*).

2. Una linea comenzando con la palabra clave ``Característica:`` seguida de su titulo y 
   tres lineas indentadas definen el inicio de una nueva característica.

3. Una característica usualmente contiene una lista de escenarios. Puedes escribir lo que
   quieras hasta el primer escenario: este texto se convertirá en la descripción de la 
   característica.

4. Cada escenario comienza con la palabra clave ``Escenario:`` seguida de una corta 
   descripción del escenario. Bajo cada escenario hay una lista de pasos, que deben
   comenzar con una de las siguientes palabras claves: ``Dado``, ``Cuando``, ``Entonces``,
   ``Pero`` or ``Y``. Behat trata cada una de estas palabras igual, pero debes usarlas
   como se espera para tener escenarios consistentes.

.. tip::

    Behat también te permite escribir tus características en tu lenguaje nativo.
	En otras palabras, en lugar de escribir ``Característica``, ``Escenario`` or ``Dado``,
    puedes usar tu lenguaje nativo configurando Behat para usar uno de sus muchos
	lenguajes soportados. 

	Para verificar si tu lenguaje está soportado y ver las palabras claves, ejecuta:

    .. code-block:: bash

        $ behat --story-syntax --lang TU_LENGUAJE

	Los lenguajes soportados incluyen (pero no están limitados a) ``fr``, ``es``, ``it``
	y, por supuesto, el dialecto pirata en inglés ``en-pirate``.

	Ten presente que, cualquier lenguaje diferente de ``en`` debe ser marcado
	explícitamente con el comentario ``# language: ...`` al comienzo de tu archivo
	``*.feature``:
    
    .. code-block:: gherkin

        # language: fr
        Fonctionnalité: ...
          ...

Puedes leer más acerca de las características y el lenguaje Gherkin en la guía ":doc:`/guides/1.gherkin`".

Más acerca de los Pasos
-----------------------

Por cada paso (ej. ``Dado que estoy en un directorio "test"``), Behat buscará
una definición de paso que coincida, comparando el texto del paso con la expresión
regular establecida en cada paso definido.

Una definición de paso se escribe en php y consiste de una palabra clave, 
una expresión regular, y un callback. Por ejemplo:

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

Algunas notas:

1. ``@Given`` es una palabra clave de definición. Hay tres palabras claves soportadas 
   por anotaciones: ``@Given``/``@When``/``@Then``. Estas tres palabras claves son
   equivalentes, pero todas tres estas disponibles para que tu definición de paso
   se mantenga legible.

2. El texto despues de la palabra clave es la expresión regular (ej. ``/^I am in a directory "([^"]*)"$/``).

3. Todos los patrones de búsqueda en la expresión regular (ej. ``([^"]*)``) se convertirán
   en argumentos del método (``$dir``).

4. Si, dentro de un paso, necesitas decirle a Behat que ha ocurrido alguna clase de "fallo", 
   debes lanzar una excepción:
   
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

    Behat no viene con su propia herramienta de aserciones, pero puedes usar
	cualquier herramienta de aserciones que exista. Una herramienta de aserciones
	es una libreria, que sus asersiones lanzan excepciones cuando fallan. Por ejemplo,
	si estas familiarizado con PHPUnit, puedes usar sus aserciones en Behat:

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

De la misma manera, cualquier paso que *no* lance una excepción será visto
por Behat como "pasando".

Puedes leer mas acerca de la definición de pasos en la guía ":doc:`/guides/2.definitions`".

La Clase de Contexto: ``FeatureContext``
----------------------------------------

Behat crea un objeto de contexto por cada escenario y ejecuta todos los pasos del 
escenario dentro de ese mismo objeto. En otras palabras, si quieres compartir variables
entre pasos, puedes hacerlo facilmente estableciendo valores de propiedades en el objeto
de contexto (que fue mostrado en el ejemplo previo).

Puedes leer más del ``FeatureContext`` en la guía ":doc:`/guides/4.context`".

La herramienta de linea de comando ``behat``
--------------------------------------------

Behat viene con una poderosa utilidad de consola responsable de ejecutar las
pruebas de Behat. La utilidad viene con una amplia variedad de opciones.

Para ver las opciones y la forma de uso de la herramienta, ejecuta:

.. code-block:: bash

    $ behat -h

Una de las cosas más útiles que hace éste comando es mostrarte todas las definiciones
de pasos que tienes configuradas en tu sistema. Esta es una manera sencilla de recapitular
las palabras exactas de un paso que has definido con anterioridad:

.. code-block:: bash

    $ behat -dl

Puedes leer más acerca de la interfaz de línea de comando de Behat en la guía ":doc:`/guides/6.cli`".

Proximos Pasos
--------------

Felicitaciones! Ahora sabes todo lo que necesitas para comenzar con desarrollo
guiado por comportamiento y Behat. Desde aquí, puedes aprender más acerca de la sintáxis
de :doc:`Gherkin</guides/1.gherkin>` o aprender como probar tus aplicaciones web usando
Behat con Mink.

* :doc:`/cookbook/behat_and_mink`
* :doc:`/guides/1.gherkin`
* :doc:`/guides/6.cli`

.. _`behavior driven development`: http://en.wikipedia.org/wiki/Behavior_Driven_Development
.. _`Mink`: https://github.com/behat/mink
.. _`¿Qué hay en una historia?`: http://adrianmoya.com/2012/08/que-hay-en-una-historia/
.. _`Cucumber`: http://cukes.info/
.. _`Goutte`: https://github.com/fabpot/goutte
.. _`PHPUnit`: http://phpunit.de
.. _`Testing Web Applications with Mink`: https://github.com/behat/mink
