Quick Guide to Behat
====================

Welcome to Behat! Behat is a tool that makes `behavior driven development`_
(BDD) possible. With BDD, you write human-readable stories that describe
the behavior of your application. These stories can then be run as actual
tests against your application. And yes, it's as cool as it sounds!

For example, imagine you've been hired to built the famous ``ls`` UNIX command.
A stakeholder may say to you:

.. code-block:: gherkin

    Feature: ls
      In order to see the directory structure
      As a UNIX user
      I need to be able to list the current directory's contents

    Scenario: List 2 files in a directory
      Given I am in a directory "test"
      And I have a file named "foo"
      And I have a file named "bar"
      When I run "ls"
      Then I should get:
        """
        bar
        foo
        """

In this tutorial, we'll show you how Behat can execute this simple story
as a test that verifies that the ``ls`` commands works as described.

That's it! Behat can be used to test anything, including web-related behavior
via the `Mink`_ library.

.. note::

    If you want to learn more about the philosophy of testing the "behavior"
    of your application, see `What's in a Story?`_

.. note::

    Behat was inspired by Ruby's `Cucumber`_ project.

Installation
------------

Behat is an executable that you'll run from the command line to execute your
stories as tests. Before you begin, ensure that you have at least PHP 5.3.1
installed.

Method #1 (PEAR)
~~~~~~~~~~~~~~~~

The simplest way to install Behat is through PEAR:

.. code-block:: bash

    pear channel-discover pear.behat.org
    pear install behat/behat

You can now execute Behat simply by running the ``behat`` command:

.. code-block:: bash

    behat

Method #2 (Git)
~~~~~~~~~~~~~~~

You can also clone the project with Git by running:

.. code-block:: bash

    git clone git://github.com/Behat/Behat.git && cd Behat
    git submodule update --init

After downloading, you can execute behat by running:

.. code-block:: bash

    ./path/to/Behat/bin/behat

Basic Usage
-----------

In this example, we'll rewind several decades and pretend we're building
the original UNIX ``ls`` command. Create a new directory and setup behat
inside that directory:

.. code-block:: bash

    mkdir ls_project
    cd ls_project
    behat --init

The ``behat --init`` will create a ``features/`` directory with some basic
things to get your started.

Define your Feature
~~~~~~~~~~~~~~~~~~~

Everything in Behat always starts with a *feature* that you want to describe
and then implement. In this example, the feature will be the ``ls`` command,
which can be thought of as one feature of the whole UNIX system. Since the
feature is the ``ls`` command, start by creating a ``features/ls.feature``
file:

.. code-block:: gherkin

    Feature: ls
      In order to see the directory structure
      As a UNIX user
      I need to be able to list the current directory's contents

Every feature starts with this same format: a line naming the feature, followed
by three lines that describe the benefit, the role and the feature itself.
This section is required, but its contents aren't actually important to Behat
or your eventual test. They are, however, important to write correctly so
that your features are consistent and readable by other people.

Define a Scenario
~~~~~~~~~~~~~~~~~

Next, add the following scenario to the end of the ``features/ls.feature``
file:

.. code-block:: gherkin

    Scenario: List 2 files in a directory
      Given I am in a directory "test"
      And I have a file named "foo"
      And I have a file named "bar"
      When I run "ls"
      Then I should get:
        """
        bar
        foo
        """

.. tip::

    The special ``"""`` syntax seen on the last few lines is just a special
    syntax for defining steps on multiple lines. Don't worry about it too
    much for now.

Each feature is defined by one or more "scenarios", which explain how that
feature should act under different conditions. This is the part that will
be transformed into a test. Each scenario always follows the same basic format:

.. code-block:: gherkin

    Scenario: Some description of the scenario
      Given [some context]
      When [some event]
      Then [outcome]

Each part of the scenario - the *context*, the *event*,  and the *outcome* -
can be extended by adding the ``And`` or ``But`` keyword:

.. code-block:: gherkin

    Scenario: Some description of the scenario
      Given [some context]
      And [more context]
      When [some event]
      And [second event occurs]
      Then [outcome]
      And [another outcome]
      But [another outcome]

There's no actual difference between, for example, using ``Then``, ``And``
or ``But``. These keywords are all made available so that your scenarios
are natural and readable.

Executing Behat
~~~~~~~~~~~~~~~

You've now defined the feature and one scenario for that feature. You're
ready to see Behat in action! Try executing Behat from inside your ``ls_project``
directory:

.. code-block:: bash

    behat

If everything worked correctly, you should see something like this:

.. image:: /images/ls_no_defined_steps.png
   :align: center

Writing your Steps
~~~~~~~~~~~~~~~~~~

Behat automatically finds the ``features/ls.feature`` file and tries to execute
its ``Scenario`` as a test. However, we haven't told Behat what to do with
statements like ``Given I am in a directory "test"``, which cases an error.
Behat works by matching each statement of a ``Scenario`` to a list of regular
expression "steps" that you define. In other words, it's your job to tell
Behat what to do when it sees ``Given I am in a directory "test"``. Fortunately,
Behat helps you out by printing the regular expression that you probably
need in order to create that step:

.. code-block:: text

    You can implement step definitions for undefined steps with these snippets:

    $steps->Given('/^I am in a directory "([^"]*)"$/', function($world, $arg1) {
        throw new \Behat\Behat\Exception\Pending();
    });

Let's use Behat's advice and add the following to the ``features/steps/steps.php``
file:

.. code-block:: php

    <?php
    # features/steps/steps.php

    $steps->Given('/^I am in a directory "([^"]*)"$/', function($world, $dir) {
        if (!file_exists($dir)) {
            mkdir($dir);
        }
        chdir($dir);
    });

Basically, we've started with the regular expression suggested by Behat, which
makes the value inside the quotations (e.g. "test") available as the ``$dir``
variable. Inside the method, we simple create the directory and move into it.

Repeat this for the other three missing steps so that your ``steps.php``
file looks like this:

.. code-block:: php

    <?php

    $steps->Given('/^I am in a directory "([^"]*)"$/', function($world, $dir) {
        if (!file_exists($dir)) {
            mkdir($dir);
        }
        chdir($dir);
    });

    $steps->Given('/^I have a file named "([^"]*)"$/', function($world, $file) {
        touch($file);
    });

    $steps->When('/^I run "([^"]*)"$/', function($world, $command) {
        exec($command, $output);
        $world->output = trim(implode("\n", $output));
    });

    $steps->Then('/^I should get:$/', function($world, $string) {
        if ((string) $string !== $world->output) {
            throw new Exception("Actual output is:\n" . $world->output);
        }
    });

.. note::

    When you specify multi-line step arguments - like we did using the triple
    quotation syntax (``"""``) in the above scenario, the value passed into
    the step function (e.g. ``$string``) is actually an object, which can
    be converted into a string using ``(string) $string)``.

Great! Now that you've defined all of your steps, run Behat again:

.. code-block:: bash

    behat

.. image:: /images/ls_passing_one_step.png
   :align: center

Success! Behat executed each of your steps - creating a new directory with
two files and running the ``ls`` command - and compared the result to the
expected result.

Of course, now that you've defined your basic steps, adding more scenarios
is easy. For example, add the following to your ``features/ls.feature`` file:

.. code-block:: gherkin

    Scenario: List 2 files in a directory with the -a option
      Given I am in a directory "test"
      And I have a file named "foo"
      And I have a file named ".bar"
      When I run "ls -a"
      Then I should get:
        """
        .
        ..
        .bar
        foo
        """

Run Behat again. This time, it'll run two tests, and both will pass.

.. image:: /images/ls_passing_two_steps.png
   :align: center

That's it! Now that you've got a few steps defined, you can probably dream
up lots of different scenarios to write for the ``ls`` command. Of course,
this same basic idea could be used to test web applications, and Behat integrates
beautifully with a library called `Mink`_ to do just that.

Of course, there's still lot's more to learn, including more about the Gherkin
syntax and the ``$world`` variable that's available inside each step function.

Some more Behat Basics
----------------------

When you run ``behat --init``, it sets up a directory that looks like this:

The basic Behat test environment directory looks like this:

.. code-block:: bash

    |-- features
       `-- steps
       |   `-- math_steps.php
       `-- support
           |-- bootstrap.php
           |-- env.php

Everything related to Behat will live inside the ``features`` directory, which
is composed of three basic areas:

1. ``features/`` - Behat looks for ``*.feature`` files here to execute
2. ``features/steps/`` - Behat loads all ``*.php`` files here as "steps"
3. ``features/support/`` - This directory contains two files that help you configure Behat

Inside the ``feature/support/`` directory, there are two files:

* ``bootstrap.php`` This file is run once per Behat execution. You should
  use it to initialize and require anything needed for Behat to run your application.

* ``env.php`` This file is run once per Scenario test and can be used to
  set variables on the ``$world`` variable. In other words, if you need any
  external variables or objects to be available inside your steps, set those
  variables here.

More about Feature
------------------

As you've already seen, a feature is a simple, readable plain text file,
in a format called Gherkin. Each feature file follows a few basic rules:

1. Every ``*.feature`` file conventionally consists of single feature.

2. A line starting with the keyword ``Feature:`` followed by its title and
   three indented lines defines the start of a new e feature.

3. A feature usually contains a list of scenarios. You can write whatever
   you want up until the first scenario: this text will become the feature
   description.

4. Each scenario starts with the ``Scenario:`` keyword followed by a short
   description of the scenario. Under each scenario is a list of steps, which
   must start with one of the following keywords: ``Given``, ``When``, ``Then``,
   ``But`` or ``And``. Behat treats each of these keywords the same, but you
   should use them as intended for consistent scenarios.

.. tip::

    Behat also allows you to write your features in your native language.
    In other words, instead of writing ``Feature``, ``Scenario`` or ``Given``,
    you can use your native language by configuring Behat to use one of its
    many supported languages.
    
    To check if your language is supported and to see the available keywords,
    run:
    
    .. code-block:: bash
    
        behat --usage --lang YOUR_LANG

    Supported languages include ``fr``, ``es``, ``it`` and, of course, the
    english pirate dialect ``en-pirate``.

More about Steps
----------------

For each step, Behat will look for a matching step definition by matching
the text of the step against the regular expression defined by each step.

A step definition is written in php and consists of a keyword, a regular
expression, and a callback. For example:

.. code-block:: php

    <?php
    # features/steps/steps.php

    $steps->Given('/^I am in a directory "([^"]*)"$/', function($world, $dir) {
        if (!file_exists($dir)) {
            mkdir($dir);
        }
        chdir($dir);
    });

A few pointers:

1. ``$steps`` is a global ``DefinitionDispatcher`` object, available in all
   step definition files. Calling ``->Given`` on it will allow you to define
   a step (though in reality this would match any ``When``/``Then``/``And``
   keywords as well).
   
2. All search patterns in the regular expression (e.g. ``([^"]*)``) will become
   callback arguments in the function (``$dir``).

3. The first callback argument, ``$world``,, is always reserved for environment
   object. The environment object is created before each scenario is run,
   but is shared between each step inside that scenario.

4. If, inside a step, you need to tell Behat that some sort of "failure" has
   occurred, you should throw an exception:

.. code-block:: php

   <?php
   
   $steps->Then('/^I should get:$/', function($world, $string) {
       if ((string) $string !== $world->output) {
           throw new Exception("Actual output is:\n" . $world->output);
       }
   });

In the same way, any step that does *not* throw an exception will be seen
by Behat as "passing". 

The Environment Object: ``$world``
----------------------------------

Behat creates an environment object for each scenario and passes that same
object to each step within the scenario. So, if you want to share variables
between steps, you can easily do that (see the full example above).

But what if you need some variable or object to be available inside *every*
step or every scenario? To do this, use the ``features/support/env.php``
file.

For example, suppose we want to allow each step to execute a real HTTP request
using the PHP library `Goutte`_.

.. code-block:: php

    <?php
    // features/support/env.php

    // Create the web client
    $world->client = new \Goutte\Client;
    $world->response = null;
    $world->form = array();

    // add a "visit" closure function
    $world->visit = function($link) use($world) { 
        $world->response = $world->client->request('GET', $link); 
    };

Now, inside any step, you can do the following:

.. code-block:: php

    <?php

    $steps->Given('/^I visit "([^"]*)" in my browser$/', function($world, $url) {
        $world->visit($url);
    });

    $steps->Then('/^The page should contain "([^"]*)"$/', function($world, $string) {
        if (false === strpos($world->response->getContent(), $string) {
            throw new Exception(sprintf('String "%s" not found on the page', $string));
        }
    });

The Bootstrap File
------------------

But what if you need to use some 3rd party libraries in ``env.php``? In the
previous example, the `Goutte`_ library must be already available in the
``env.php`` file. Another library you might need to use is `PHPUnit`_. In
either case, you can use the ``features/support/bootstrap.php`` file - which
is executed only once - to load the libraries you need:

.. code-block:: php

    <?php
    // features/support/bootstrap.php

    require_once 'PHPUnit/Autoload.php';
    require_once 'PHPUnit/Framework/Assert/Functions.php';

The ``behat`` Command Line Tool
-------------------------------

Behat comes with a powerful console utility responsible for executing the
Behat tests. The utility comes with a wide array of options.

To see options and usage for the utility, run:

.. code-block:: bash

    behat -h

What's Next?
------------

Congratulations! You now know everything you need in order to get started
with behavior driven development and Behat. From here, you can learn more
about the :doc:`Gherkin</gherkin/index>` syntax or learn how to test your
web applications by using Behat with Mink.

* `Testing Web Applications with Mink`_
* :doc:`Configuration<behat/configuration>`
* :doc:`Pre and Post Suite Hooks</behat/hooks>`
* :doc:`Tagging Features</gherkin/tags>`

.. _`behavior driven development`: http://en.wikipedia.org/wiki/Behavior_Driven_Development
.. _`Mink`: https://github.com/behat/mink
.. _`What's in a Story?`: http://blog.dannorth.net/whats-in-a-story/
.. _`Cucumber`: http://cukes.info/
.. _`Goutte`: https://github.com/fabpot/goutte
.. _`PHPUnit`: http://phpunit.de
.. _`Testing Web Applications with Mink`: https://github.com/behat/mink