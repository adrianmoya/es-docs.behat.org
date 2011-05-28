Calling Steps from Step Definitions
===================================

It is possible to call steps from other :doc:`../gherkin/steps`:

.. code-block:: php

    <?php
    // features/steps/istep_call_steps.php

    $steps->Given('/^the user (.*) exists$/', function($world, $name) {
        // ...
    });

    $steps->Given('/^I log in as (.*)$/', function($world, $name) {
        // ...
    });

    $steps->Given('/^(.*) is logged in$/', function($world, $name) use($steps) {
        $steps->Given("the user $name exists", $world);
        $steps->Given("I log in as $name", $world);
    });

Invoking steps from step definitions is practical if you have several common steps that you want to perform in several scenarios. Or simply if you want to make your scenarios shorter and more declarative. This allows you to do this in a Scenario:

.. code-block:: gherkin

    Feature:

        Scenario: View last incidents
          Given Linda is logged in          # This will in fact invoke 2 step definitions
          When I go to the incident page

Instead of having a lot of repetition:

.. code-block:: gherkin

    Feature:

        Scenario: View last incidents
          Given the user Linda exists
          And I log in as Linda
          When I go to the incident page

You can even pass arguments to inner steps:

.. code-block:: php

    <?php
    // features/steps/istep_calls_with_args_steps.php

    $steps->Given('/^(.*) is logged in with:$/', function($world, $name, $table) use($steps) {
        $steps->Given("the user $name exists", $world);
        $steps->Given("I log in as $name with:", $world, $table);
    });
