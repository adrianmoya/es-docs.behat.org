Hooks
=====

Behat provides a number of hooks which allow to run functions at various points in the Behat test cycle. You can put them in your ``features/support/hooks.php`` file. There is no association between where the hook is defined and which scenario/step it is run for, but you can use tagged hooks (see below) if you want more fine grained control.

Suite Hooks
-----------

These hooks are executed before and after a whole suite. It's best place to write project-wide tear up and tear down actions:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeSuite(function($event) {
        // Do something before whole test suite
    });
    $hooks->afterSuite(function($event) {
        // Do something after whole test suite
    });

Notice, that like for step definitions and environment configs, we have a global ``$hooks`` variable here, which represents HookDispatcher object. ``HookDispatcher`` is in charge of defining hooks and fireing them at the right time.

Suite hooks (``beforeSuite`` and ``afterSuite``) have a single argument which is the callback, they don't use a filter.

Also note the ``$event`` argument of the callback. It's a ``Behat\Behat\Event\EventInterface`` instance. In case of the ``beforeSuite`` and ``afterSuite`` hooks it would be a `SuiteEvent <http://docs.behat.org/api/behat/behat/behat/event/suiteevent.html>`_ object:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->afterSuite(function($event) {
        $dataCollector = $event->getLogger();

        // do something with information from dataCollector
    });

Feature Hooks
-------------

``beforeFeature`` hooks are executed before each feature of the test suite:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeFeature('', function($event) {
        // do something before each feature
    });

``afterFeature`` hooks are executed after each feature of the test suite:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterFeature('', function($event) {
        // do something after each feature
    });

Notice the additional empty first argument to this hook type. This is a filter. You can put there any of your ``--name`` or ``--tags`` filters and that hook will be executed for features matching this filter only.

``$event`` is an instance of `FeatureEvent <http://docs.behat.org/api/behat/behat/behat/event/featureevent.html>`_:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeFeature('', function($event) {
        $feature = $event->getFeature();
        $featureTitle = $feature->getTitle();
    });

Scenario Hooks
--------------

``beforeScenario`` hooks are executed before the first step of each scenario:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeScenario('', function($event) {
        // do something before each scenario
    });

``afterScenario`` hooks are executed after the last step of each scenario:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterScenario('', function($event) {
        // do something after each scenario
    });

The first argument is a filter, like with feature hooks.

``$event`` is an instance of `ScenarioEvent <http://docs.behat.org/api/behat/behat/behat/event/scenarioevent.html>`_ or `OutlineExampleEvent <http://docs.behat.org/api/behat/behat/behat/event/outlineexampleevent.html>`_:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterScenario('', function($event) {
        // get scenario or outline:
        $scenario = $event instanceof Behat\Behat\Event\ScenarioEvent ? $event->getScenario() : $event->getOutline();
    });

Also, there are some interesting getters in the scenario ``$event`` objects:

* ``getEnvironment()`` - scenario :doc:`environment` object. This parameter is available in both before & after hooks. It returns the same :doc:`environment` object that gets passed as ``$world`` into every scenario step definition.
* ``getResult()`` - scenario result code (see ``Behat\Behat\Tester\StepTester`` code for further information). This parameter is available only in after hooks.
* ``isSkipped()`` - boolean, that marks if the scenario has skipped steps. This parameter is available only in after hooks.

Step Hooks
----------

``beforeStep`` hooks are executed before each step:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeStep('', function($event) {
        // do something before each step
    });

``afterStep`` hooks are executed after each step:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterStep('', function($event) {
        // do something after each step
    });

The first argument is a filter, like with feature hooks.

``$event`` is an instance of `StepEvent <http://docs.behat.org/api/behat/behat/behat/event/stepevent.html>`_:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterStep('', function($event) {
        // get step AST node:
        $step = $event->getSubject();
    });

Also, there are some interesting getters in steps ``$event`` object:

* ``getEnvironment()`` - scenario environment object. This parameter is available in both before & after hooks.
* ``getResult()`` - step result code (see ``Behat\Behat\Tester\StepTester`` code for further information). This parameter is available only in after hooks.
* ``getException()`` - exception instance or null. This parameter is available only in after hooks.
* ``getDefinition()`` - matched definition or null. This parameter is available only in after hooks.
* ``getSnippet()`` - snippet for definition if step undefined or null. This parameter is available only in after hooks.
