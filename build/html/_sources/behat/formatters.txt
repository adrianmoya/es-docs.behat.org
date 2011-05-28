Formatters
==========

Behat uses `Symfony Event Dispatcher component <http://components.symfony-project.org/event-dispatcher/>`_ to send custom events during test suite lifetime.

Behat Formatters is core event listeners, that listens to ``Behat\Behat\Tester\*`` events & prints it’s results.

Behat comes with 4 core formatters:

* `pretty` (default one).
* `progress` - draws progress of suite excecution much like PHPUnit does.
* `html` - prints pretty html reports out of suite results.
* `junit` - junit formatter, which outputs junit xml's (great for CI servers).

Pretty
------

PrettyFormatter is default one & it’s print execution process... PRETTY! It tries to be as much like Cucumber as possible. To force Behat use PrettyFormatter, run it with "-f" or "--format" option:

.. code-block:: bash

    behat -f pretty

Progress
--------

ProgressFormatter is compact run logger, that is very similar to PHPUnit default output. To use it, run:

.. code-block:: bash

    behat -f progress

Html
----

HtmlFormatter is html printing formatter. It generates html report out of your test suite results:

.. code-block:: bash

    behat -f html

.. code-block:: bash

    behat -f html --out report.html

JUnit
-----

JUnitFormatter is JUnit xml support formatter, which gets used in almost every CI server out there. To use it simply run:

.. code-block:: bash

    behat -f junit --out /path/to/reports

Notice that ``--out`` argument is expected always and it need directory path (not file) to write xml's into.

Writing Your Own Formatter
--------------------------

Writing your own formatter is as simple as:

1. Extend one of the bundled formatters or special ``Behat\Behat\Formatter\ConsoleFormatter`` (https://github.com/Behat/Behat/blob/master/src/Behat/Behat/Formatter/ConsoleFormatter.php), which does only what you need. OR simply implement ``Behat\Behat\Formatter\FormatterInterface`` in your very own class.

Lets add "hello, world" before each feature in output:

.. code-block:: php

    <?php
    // features/support/EverzetFormatter.php
    
    namespace Everzet;
    
    use Behat\Behat\Formatter\PrettyFormatter;
    use Behat\Gherkin\Node\FeatureNode;
    
    class Formatter extends PrettyFormatter
    {
        protected function printFeatureName(FeatureNode $feature)
        {
            $this->writeln('hello, world');

            parent::printFeatureName($feature);
        }
    }

.. note::
    Try to overwrite only ``print...`` methods. The bundled formatters API is very clean and created with future extensions in mind.

now we need to tell Behat about this new formatter:

.. code-block:: php

    <?php
    // features/support/bootstrap.php
    
    require_once('EverzetFormatter.php');

aaaaand it's done! Use your new formatter with:

.. code-block:: bash

    behat -f Everzet\\Formatter

