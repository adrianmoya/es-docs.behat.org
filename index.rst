
Documentación de Behat
======================

Behat es una herramienta opensource de desarrollo guiado por comportamiento para PHP 5.3 & 5.4. Te preguntas qué es desarrollo guiado por comportamiento? Es la idea de comenzar por escribir sentencias legibles por humanos que describen una característica de tu aplicación y como debería funcionar.

Por ejemplo, imagina que estas por crear el famoso comando ``ls`` de UNIX. 
Antes de comenzar, describes como esta característica debería funcionar:

.. code-block:: gherkin


    Característica: ls
       Para de ver la estructura de directorio
       Como un usuario UNIX
       Yo necesito ser capaz de listar el contenido del directorio actual.

       Escenario:
	 Dado que estoy en el directorio "test"
	 Y tengo un archivo llamado "foo"
	 Y tengo un archivo llamado "bar"
	 Cuando ejecuto el comando "ls"
	 Entonces debería obtener:
	   """
	   bar
	   foo
	   """
Como desarrollador, tu trabajo está terminado tan pronto como hagas que el comando ``ls`` se comporte como está descrito en el *Escenario*.

Ahora, no sería genial si algo pudiese leer esta sentencia y usarla para ejecutar una prueba contra el comando ``ls``? Hey, eso es exactamente lo que hace Behat! Como veras, Behat es fácil de aprender, rápido de usar, y traerá de vuelta la diversión a tus pruebas. 

.. note::

    Behat fue inspirado en el proyecto Cucumber de Ruby, especialmente su sintaxis (llamada Gherkin).

Introducción Rápida
-------------------

Para convertirte en un Behat'er en 20 minutos, sumérgete en la guía rápida y disfruta!

.. toctree::
    :maxdepth: 2

    quick_intro

Guías
-----

Learn Behat with the topical guides:

.. toctree::
    :maxdepth: 1

    guides/1.gherkin
    guides/2.definitions
    guides/3.hooks
    guides/4.context
    guides/5.closures
    guides/6.cli
    guides/7.config

Recetas
-------

Learn specific solutions for specific needs:

.. toctree::
    :maxdepth: 1

    cookbook/behat_and_mink
    cookbook/migrate_from_1x_to_20
    cookbook/using_spin_functions

Recursos Útiles
---------------

Otros recursos útiles para aprender/usar Behat:

* `Cheat Sheet <http://blog.lepine.pro/wp-content/uploads/2012/03/behat-cheat-sheet-en.pdf>`_
  - Behat and Mink Cheat Sheets by `Jean-François Lépine <http://blog.lepine.pro/>`_
* `Behat API <http://docs.behat.org/behat-api/>`_ - Behat code API
* `Gherkin API <http://docs.behat.org/gherkin-api/>`_ - Gherkin parser API

Más acerca de Desarrollo Guiado por Comportamiento
--------------------------------------------------

Once you're up and running with Behat, you can learn more about behavioral
driven development via the following links. Though both tutorials are specific
to Cucumber, Behat shares a lot with Cucumber and the philosophies are one
and the same.

* `"¿Qué hay en una historia?" de Dan North (Traducido por Adrian Moya) <http://adrianmoya.com/2012/08/que-hay-en-una-historia/>`_
* `Cucumber's "Backgrounder" <https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder>`_
