Environment
===========

For each scenario (and its background) Behat creates a new Environment (or World). By default, Environment (``$world``) is an instance of a Container Object.

Step Definitions
----------------

All the :doc:`../gherkin/steps` will receive their own ``$world`` instance as a first argument. A new instance is created for each scenario or example in scenario outlines. This means that ``$world`` in the Step Definition callback will be the Environment instance. Any ``$world->variable`` assigned to it will remain in Environment until the end of the current scenario.

A better world
--------------

If you want to add any behaviour to the Environment, perhaps some helper methods, or logging, or whatever you can do this in ``features/support/env.php``:

.. code-block:: php

    <?php
    // features/support/env.php

    $world->helper = new Helper();

Now you can call ``$world->helper->methods()`` from your step definitions. Note that every scenario is tied to a separate instance of the Environment, so there is no implicit state-sharing from scenario to scenario.

Bootstrap
---------

But what if you want to use PHPUnit or other 3rd party tool or library? ``env.php`` is a bad place to require them, because they will be required before **every** scenario. The best place for all your bootstrap & require scripts is called **bootstrap script**. For example, to be able to use PHPUnit assertions inside your steps, you need to add this file to your test suite:

.. code-block:: php

    <?php
    // features/support/bootstrap.php

    require_once 'PHPUnit/Autoload.php';
    require_once 'PHPUnit/Framework/Assert/Functions.php';

This file will be evaluated only once before any feature run.
