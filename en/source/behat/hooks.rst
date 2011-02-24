Hooks
=====

Behat provides a number of hooks which allow us to run functions at various points in the Behat test cycle. You can put them in your ``features/support/hooks.php`` file. There is no association between where the hook is defined and which scenario/step it is run for, but you can use tagged hooks (see below) if you want more fine grained control.

Suite Hooks
-----------

This hooks gets runed before and after a whole suite. It's best place to write project-wide teardown and tearup actions:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeSuite(function($event) {
        // Do something before whole test suite
    });
    $hooks->afterSuite(function($event) {
        // Do something after whole test suite
    });

Notice, that like with step definitions or environment configs, we have global ``$hooks`` variable here, which represents HookDispatcher object. ``HookDispatcher`` is in charge for defining hooks & fireing them in right time. 

Suite hooks doesn't have filtering specifier, so ``beforeSuite`` and ``afterSuite`` have only 1 argument which is your callback.

Also note, that ``$event`` argument in callback. It's ``Symfony\Component\EventDispatcher\Event`` populated with ``Behat\Behat\DataCollector\LoggerDataCollector`` as subject, which you can get with:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->afterSuite(function($event) {
        $dataCollector = $event->getSubject();

        // do something with information from dataCollector
    });

Feature Hooks
-------------

``beforeFeature`` hooks will be run before every feature in test suite:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeFeature('', function($event) {
        // do something before each feature
    });

``afterFeature`` hooks will be run after every feature in test suite:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterFeature('', function($event) {
        // do something after each feature
    });

Notice additional empty first argument to this hook type. This is filter. You can put there any of your ``--name`` or ``--tags`` filters and that hook will be runed only in specific features (that match filter).

``$event``'s subject is ``Behat\Gherkin\Node\FeatureNode``:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeFeature('', function($event) {
        $feature = $event->getSubject();
        $featureTitle = $feature->getTitle();
    });

Scenario Hooks
--------------

``beforeScenario`` hooks will be run before the first step of each scenario:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeScenario('', function($event) {
        // do something before each scenario
    });

``afterScenario`` hooks will be run after last step of each scenario:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterScenario('', function($event) {
        // do something after each scenario
    });

First argument is a filter, like with feature hooks.

``$event``'s subject is ``Behat\Gherkin\Node\ScenarioNode`` or ``Behat\Gherkin\Node\OutlineNode``:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterScenario('', function($event) {
        // get scenario or outline:
        $scenario = $event->getSubject();
    });

Also, there's some interesting parameters in ``$event`` object:

* ``environment`` - scenario :doc:`environment` object. This parameter available in both before & after hooks. It returns same :doc:`environment` object, that gets passed as ``$world`` into every scenario step definition.
* ``result`` - scenario result code (see ``Behat\Behat\Tester\StepTester`` code for further information). This parameter available only in after hooks.
* ``skipped`` - boolean, that marks if scenario has skipped steps. This parameter available only in after hooks.

You can get needed parameter with ``$event->get('...')`` method call.

Step Hooks
----------

``beforeStep`` hooks will be run before every step:

.. code-block:: php

    <?php
    // features/support/hooks.php
    
    $hooks->beforeStep('', function($event) {
        // do something before each step
    });

``afterStep`` hooks will be run after every step:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterStep('', function($event) {
        // do something after each step
    });

First argument is a filter, like with feature hooks.

``$event``'s subject is ``Behat\Gherkin\Node\StepNode``:

.. code-block:: php

    <?php
    // features/support/hooks.php

    $hooks->afterStep('', function($event) {
        // get step AST node:
        $step = $event->getSubject();
    });

Also, there's some interesting parameters in ``$event`` object:

* ``environment`` - scenario environment object. This parameter available in both before & after hooks.
* ``result`` - step result code (see ``Behat\Behat\Tester\StepTester`` code for further information). This parameter available only in after hooks.
* ``exception`` - exception instance or null. This parameter available only in after hooks.
* ``definition`` - matched definition or null. This parameter available only in after hooks.
* ``snippet`` - snippet for definition if step undefined or null. This parameter available only in after hooks.

You can get needed parameter with ``$event->get('...')`` method call.
