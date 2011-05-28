Features
========

Every ``.feature`` file conventionally consists of single feature. Line starting with keyword ``Feature:`` followed by free indented text starts a feature. Feature usually contains a list of scenarios. You can write whatever you want up until the first scenario, which starts with the word ``Scenario:`` (or localized equivalent) on a new line. You can use :doc:`tags` to group features and scenarios together independent of your file and directory structure.

Every scenario consists of a list of steps, which must start with one of the keywords ``Given``, ``When``, ``Then``, ``But`` or ``And`` (or localized one). Behat treats them all the same, but you shouldn’t. Here is an example:

.. code-block:: gherkin

    Feature: Serve coffee
      In order to earn money
      Customers should be able to 
      buy coffee at all times

      Scenario: Buy last coffee
        Given there are 1 coffees left in the machine
        And I have deposited 1$
        When I press the coffee button
        Then I should be served a coffee

In addition to :doc:`scenarios`, feature may contain a :doc:`backgrounds` & :doc:`outlines`.

Step Definitions
----------------

For each step Behat will will look for a matching **step definition**. A step definition is written in PHP. Each step definition is a PHP callable, tighten to specific regular expression. Example:

.. code-block:: php

    <?php
    // features/steps/coffee_steps.php

    $steps->Then('/^I should be served coffee$/', function($world) {
        assertEquals('coffee', $world->dispensedDrink);
    });

Step definitions can also take parameters:

.. code-block:: php

    <?php
    // features/step/coffeeSteps.php

    $steps->Given('/^there are (\d+) coffees left in the machine$/', function($world, $number) {
        $world->machine = new Machine((int) $number);
    });

This step definition uses a regular expression with one match group – ``(\d+)``. (It matches any sequence of digits). Therefore, it matches the first line of the scenario. The value of each matched group sent to definition callback as a string. You must take care to have the same number of regular expression groups and callback arguments. Since callback arguments are always strings, you have to do any type conversions inside the callback, or use :doc:`../behat/transformations`.

Notice that the first argument to the callback is always ``$world``. It’s shared between single scenario steps :doc:`../behat/environment` object (or container).

When Behat prints the results of the running features it will underline all the step arguments so that it’s easier to see what part of a step was actually recognised as an argument. It will also print the path and the line of the matching step definition. This makes going from a feature file to any step definition easy.

Running Features
----------------

There are several ways to run your features. This page lists the most common ones. Any of these techniques also lets you define common command line options in a ``behat.yml`` file.

Using CLI Command
~~~~~~~~~~~~~~~~~

Assuming you’ve installed Behat as a PEAR package, run this at a command prompt to see the options for running features:

.. code-block:: bash

    behat -h

For example:

.. code-block: bash

    behat features/authenticate_user.feature:44 --format html > features.html

will run the scenario defined at line 44 of the authenticate_user feature, format it as HTML and pipe it to the features.html file for viewing in a browser.

.. code-block: bash

    behat features --name "Failed login"

will run the scenario(s) named “Failed login”

You can also use :doc:`tags` to specify what to run.
