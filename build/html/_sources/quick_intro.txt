Quick Intro to Behat
====================

Behat is an `open source <http://creativecommons.org/licenses/MIT/>`_ `behavior driven development <http://en.wikipedia.org/wiki/Behavior_Driven_Development>`_ framework for php 5.3.

Behat was inspired by Ruby's `Cucumber <http://cukes.info/>`_ project and especially it's syntax part (Gherkin). It tries to be like Cucumber with input (Feature files) and output (console formatters), but in core, it built from the ground on the shoulders of giants:

* `Symfony Dependency Injection Container component <https://github.com/symfony/DependencyInjection>`_
* `Symfony Event Dispatcher component <https://github.com/symfony/EventDispatcher>`_
* `Symfony Console component <https://github.com/symfony/Console>`_
* `Symfony Finder component <https://github.com/symfony/Finder>`_
* `Symfony Translation component <https://github.com/symfony/Translation>`_

Unlike any other php testing framework that tests applications inside out. Behat is testing applications `outside in <http://blog.dannorth.net/whats-in-a-story/>`_. It means, that Behat works only with your application's input/output. If you want to test your models - use unit testing framework instead, Behat created for behavior testing (but can be used for anything +) ).

Install
-------

There's many ways to install Behat to your system, but first, check first and only requirement of it: you must have php 5.3.1+ installed. If you have - continue reading, if not - install it first.

The simpliest way to install it through PEAR:

.. code-block:: bash

    pear channel-discover pear.behat.org
    pear install behat/gherkin
    pear install behat/behat

You can also clone the project with Git by running:

.. code-block:: bash

    git clone git://github.com/Behat/Behat.git && cd Behat
    git submodule update --init

After downloading - just run ``path/to/Behat/bin/behat`` or simply ``behat`` (if you installed Behat through PEAR) from console.

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

And it can be splitted into 3 main areas:

1. ``features`` - root test directory. Feature files are placed here.
2. ``features/steps`` - step definitions directory. Step definitions (php scripts) lay here.
3. ``features/support`` - support directory. Support scripts (environment, path definition) created here.

Core of the tests in Behat is features. Features are placed in ``features`` directory and are plain text files.

Behat parses feature files and tries to find appropriate step definition for each step from ``features/steps`` folder (step definitions path).

Every step definition is simple php-callable with access to shared between batch steps (scenarios) environment object, that can be configured inside ``features/support/env.php``.

And if environment config requires some libraries to work (PHPUnit for example) - includes are placed inside ``features/support/bootstrap.php``.

Feature
-------

Feature file is your Behat entry point. That's where you start working on your project. Here's content of basic feature ``features/math.feature``:

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

As you can see, feature is a simple, readable plain text file. Every feature is written in `DSL <http://en.wikipedia.org/wiki/Domain-specific_language>`_ called **Gherkin**, that firstly was introduced in Ruby's `Cucumber <http://cukes.info/>`_.

1. every ``*.feature`` file conventionally consists of single feature.
2. line starting with keyword ``Feature:`` (or localized one) followed by free indented text starts a feature.
3. feature usually contains a list of scenarios. You can write whatever you want up until the first scenario and this text will become feature description.
4. every scenario starts from ``Scenario:`` or ``Scenario Outline:`` keywords (or localized equivalent). Each scenario consists of steps, which must start with one of the ``Given``, ``When``, ``Then``, ``But`` or ``And`` keywords (or localized one). Behat treats all this step types the same, but you shouldn’t!

Step Definition
---------------

For each step Behat will look for a matching step definition. A step definition is written in php. Each step definition consists of a keyword, a regular expression, and a callback. Example ``features/steps/math.php``:

.. code-block:: php

    <?php 

    $steps->Given('/^I have entered (\d+) into the calculator$/', function($world, $arg1) { 
        throw new Behat\Behat\Exception\Pending('Write code later'); 
    });

1. ``$steps`` is a global DefinitionDispatcher object, available in all step definition files. Calling ``->Given`` on it will define new ``Given`` (but this will match ``When``/``Then``/``And`` keyworded steps too) step.
2. ``'/^I have entered (\d+) into the calculator$/'`` - regex matcher for step. All search patterns (``(\d+)``) will become callback arguments (``$arg1``).
3. First callback argument (``$world``) is always reserved for environment object. Environment object created before every scenario run and shared between scenario steps.
4. Step definition body is simple php code. **Failed** step is a step, which definition execution throws an exception. So, if step execution doesn't throw exceptions - step **passes**.

Environment
-----------

Behat creates environment object for each scenario and passes reference to it into each step definition.

So, if you want to calculate/accumulate or just share variables between steps definitions - use ``$world`` container for that.

But what if you need some definitions being connected in each world? Use environment configurator for that:

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

This file will be executed on each environment object creation. ``$world`` variable is an environment object itself, which works like variable holder for all your scenario values & parameters.

But what if we need to use some 3rd party libraries in ``env.php``? It's unefficient to require them before each scenario, so Behat has bootstrapping script support:

.. code-block:: php

    <?php
    // features/support/bootstrap.php

    require_once 'PHPUnit/Autoload.php';
    require_once 'PHPUnit/Framework/Assert/Functions.php';

This file will be evaluated by Behat before feature tests even run ;-)

CLI
---

Behat comes bundled with powerfull console runner, called... behat.

To see current Behat version, run:

.. code-block:: bash

    behat -V

To see other available commands, use:

.. code-block:: bash

    behat -h

Now you know all you need to get started with Behat. You can start using BDD in your projects right now or continue to read full guide.
