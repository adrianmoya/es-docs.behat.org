Quick Intro to Behat
====================

Behat is an `open source <http://creativecommons.org/licenses/MIT/>`_ `behavior driven development <http://en.wikipedia.org/wiki/Behavior_Driven_Development>`_ framework for PHP 5.3.

Behat was inspired by Ruby's `Cucumber <http://cukes.info/>`_ project and especially its syntax part (Gherkin). It tries to be like Cucumber with input (Feature files) and output (console formatters), but in the core, it's built from the ground on the shoulders of giants:

* `Symfony Dependency Injection Container component <https://github.com/symfony/DependencyInjection>`_
* `Symfony Event Dispatcher component <https://github.com/symfony/EventDispatcher>`_
* `Symfony Console component <https://github.com/symfony/Console>`_
* `Symfony Finder component <https://github.com/symfony/Finder>`_
* `Symfony Translation component <https://github.com/symfony/Translation>`_

Unlike any other PHP testing framework that tests applications inside out, Behat is testing applications `outside in <http://blog.dannorth.net/whats-in-a-story/>`_. It means that Behat works only with your application's input/output. If you want to test your models, use a unit testing framework instead. Behat was created for behavior testing (but can be used for anything +) ).

Install
-------

There are many ways to install Behat on your system, but the only requirement is that you must have PHP 5.3.1+ installed. If you have - continue reading, if not - install it first.

The simplest way to install Behat is through PEAR:

.. code-block:: bash

    pear channel-discover pear.behat.org
    pear install behat/gherkin
    pear install behat/behat

You can also clone the project with Git by running:

.. code-block:: bash

    git clone git://github.com/Behat/Behat.git && cd Behat
    git submodule update --init

After downloading, just run ``path/to/Behat/bin/behat`` or simply ``behat`` (if you installed Behat through PEAR) from the console.

Basics
------

Behat is a tool that can execute plain-text functional descriptions as automated tests. The language that Behat (and Ruby Cucumber) understands is called Gherkin. Here is an example:

.. code-block:: gherkin

    Feature: Search courses 
      In order to ensure better utilization of courses 
      Potential students should be able to search for courses 

      Scenario: Search by topic 
        Given there are 240 courses which do not have the topic "biology" 
        And there are 2 courses A001, B205 that each have "biology" as one of the topics
        When I search for "biology" 
        Then I should see the following courses:
          | Course code |
          | A001        |
          | B205        |

While Behat can be thought of as a “testing” tool, the intent of the tool is to support BDD. This means that the “features” (plain text descriptions with scenarios) are typically written before anything else and verified by business analysts, domain experts, etc. non technical stakeholders. The production code is then written outside in, to make the stories pass.

The basic Behat test environment directory looks like this:

.. code-block:: bash

    |-- features
       `-- steps
       |   `-- math_steps.php
       `-- support
           |-- bootstrap.php
           |-- env.php

And it can be split into 3 main areas:

1. ``features`` - root test directory. Feature files are placed here.
2. ``features/steps`` - step definitions directory. Step definitions (PHP scripts) lie here.
3. ``features/support`` - support directory. Support scripts (environment, path definition) are created here.

The core of the tests in Behat are features. Features are placed in the ``features`` directory and are plain text files.

Behat parses feature files and tries to find an appropriate step definition for each step from the ``features/steps`` folder (step definitions path).

Every step definition is a simple PHP-callable with access to a shared environment object.  This object is shared between batch steps (scenarios), and can be configured inside the ``features/support/env.php`` file.

And if the environment config requires some libraries to work (PHPUnit for example), the includes are placed inside ``features/support/bootstrap.php``.

Feature
-------

The feature file is your Behat entry point. That's where you start working on your project. Here's the content of a basic feature, ``features/math.feature``:

.. code-block:: gherkin

    Feature: Addition 
      In order to avoid silly mistakes 
      As a math idiot 
      I want to be told the sum of two numbers 

      Scenario: Add two numbers 
        Given I have entered 50 into the calculator
          And I have entered 70 into the calculator
         When I press add
         Then The result should be 120 on the screen

As you can see, a feature is a simple, readable plain text file. Every feature is written in a `DSL <http://en.wikipedia.org/wiki/Domain-specific_language>`_ called **Gherkin**, that was first introduced in Ruby's `Cucumber <http://cukes.info/>`_.

1. Every ``*.feature`` file conventionally consists of a single feature.
2. A line starting with the keyword ``Feature:`` (or localized one) followed by free indented text starts a feature.
3. A feature usually contains a list of scenarios. You can write whatever you want up until the first scenario and this text will become the feature description.
4. Every scenario starts from the ``Scenario:`` or ``Scenario Outline:`` keywords (or localized equivalent). Each scenario consists of steps, which must start with one of the ``Given``, ``When``, ``Then``, ``But`` or ``And`` keywords (or a localized one). Behat treats all these step types the same, but you shouldn’t!

Step Definition
---------------

For each step Behat will look for a matching step definition. A step definition is written in PHP. Each step definition consists of a keyword, a regular expression, and a callback. Example ``features/steps/math.php``:

.. code-block:: php

    <?php 

    $steps->Given('/^I have entered (\d+) into the calculator$/', function($world, $arg1) { 
        throw new Behat\Behat\Exception\Pending('Write code later'); 
    });

1. ``$steps`` is a global DefinitionDispatcher object, available in all step definition files. Calling ``->Given`` on it will define a new ``Given`` (but this will match ``When``/``Then``/``And`` keyworded steps too) step.
2. ``'/^I have entered (\d+) into the calculator$/'`` - this is a regex matcher for the step. All search patterns (``(\d+)``) will become callback arguments (``$arg1``).
3. The first callback argument (``$world``) is always reserved for the environment object. The environment object is created before every scenario is run and is shared between scenario steps.
4. The step definition body is simple PHP code. A **failed** step is a step whose execution throws an exception. So, if the step execution doesn't throw any exceptions, the step **passes**.

Environment
-----------

Behat creates an environment object for each scenario and passes a reference to it into each step definition (``$world`` in the examples above).

So, if you want to calculate/accumulate or just share variables between steps definitions, use the ``$world`` container for this.

But what if you need some definitions to be available in each world? Use the environment configurator instead:

.. code-block:: php

    <?php
    // features/support/env.php

    require 'paths.php'; 

    // Create WebClient behavior 
    $world->client = new \Goutte\Client; 
    $world->response = null; 
    $world->form = array(); 

    // Helpful closures 
    $world->visit = function($link) use($world) { 
        $world->response = $world->client->request('GET', $link); 
    };

This file will be executed on each environment object creation. The ``$world`` variable is an environment object itself, which works like a variable holder for all your scenario values & parameters.

But what if we need to use some 3rd party libraries in ``env.php``? It's inefficient to require them before each scenario, so Behat has bootstrapping script support:

.. code-block:: php

    <?php
    // features/support/bootstrap.php

    require_once 'PHPUnit/Autoload.php';
    require_once 'PHPUnit/Framework/Assert/Functions.php';

This file will be evaluated by Behat before feature tests are even run ;-)

CLI
---

Behat comes bundled with a powerful console runner, called... behat.

To see the current Behat version, run:

.. code-block:: bash

    behat -V

To see other available commands, use:

.. code-block:: bash

    behat -h

Now you know all you need to get started with Behat. You can start using BDD in your projects right now or continue to read the full guide.
